---
layout: post
ref: your-terraform-modules-are-just-functions-you-cant-step-through
title: "Seus Módulos Terraform São Funções Que Você Não Consegue Depurar"
date: 2026-09-25 00:00:00 -0300
categories: [devops, infraestrutura, cloud]
tags: [terraform, iac, modulos, infrastructure-as-code, devops, cloud, abstracao, debug, hashicorp, reutilizacao, copiar-colar, versionamento]
permalink: /pt-br/2026/09/25/seus-modulos-terraform-sao-funcoes-que-voce-nao-consegue-depurar/
---

Depois de 47 anos escrevendo software — incluindo os 8 anos que passei vendo o time de plataforma "construir uma biblioteca de módulos reutilizáveis" que virou 41 forks do mesmo módulo de VPC, cada um com um typo diferente no mapa de `tags` — tenho uma constatação que o sacerdócio de Infrastructure-as-Code vai tentar suprimir:

**Um módulo Terraform é uma função que você não consegue depurar, não consegue testar unitariamente, não consegue mockar, não consegue colocar breakpoint e não consegue ler o stack trace. É a única "abstração" da computação onde a única forma de descobrir o que ela faz é rodar contra uma conta real da AWS e ler o output de 900 linhas do `plan` de trás pra frente. A indústria de software passou 60 anos inventando o debugger. Terraform reinventou a chamada de função e esqueceu de trazê-lo.**

Os HashiCertified já estão compondo um post no Medium chamado "Por Que Módulos São As Funções Da Infraestrutura." Deixa eu salvar o rascunho deles: eles são. Esse é o problema. Funções são boas porque você *consegue ver dentro delas*. Módulos são funções com a luz apagada.

## O Pitch, E Depois A Realidade

Aqui está o pitch que o time de plataforma dá na revisão de arquitetura:

> *"Estamos construindo uma biblioteca de módulos reutilizáveis. Cada módulo é uma unidade autocontida de infraestrutura. Os times compõem eles. A gente ganha consistência, DRY, e uma fonte única de verdade. É como uma biblioteca de funções."*

Aqui está a realidade, seis meses depois:

```hcl
# modules/vpc/main.tf - "reutilizável"
variable "cidr"              { type = string }
variable "name"              { type = string }
variable "enable_nat"        { type = bool }
variable "azs"               { type = list(string) }
variable "single_nat"       { type = bool }
variable "one_nat_per_az"    { type = bool }
variable "enable_dns"        { type = bool }
variable "enable_ipv6"      { type = bool }
variable "public_tags"       { type = map(string) }
variable "private_tags"      { type = map(string) }
variable "intra_tags"        { type = map(string) }   # ninguém sabe o que "intra" significa
variable "database_subnets" { type = bool }
variable "create_database"   { type = bool, default = false }  # contradiz o de cima
variable "flow_log_role"     { type = string, default = "" }    # ignorado se vazio, erro se preenchido
# ... mais 47 variáveis, 3 delas deprecadas mas ainda exigidas pela API
```

Essa é a assinatura de uma função. Ela recebe 57 argumentos. Três deles se contradizem. Um deles (`intra_tags`) refere-se a um tipo de subnet que foi removido da documentação da AWS em 2022, mas o módulo ainda provisiona porque alguém em 2020 escreveu um `for_each` sobre ele e ninguém teve coragem de deletar o loop. Você não consegue dizer quais argumentos são obrigatórios sem rodar `terraform validate`, que passa mesmo quando os argumentos se contradizem, porque `validate` checa sintaxe, não sanidade.

Numa linguagem de verdade, uma função com 57 parâmetros e três contradições seria flagueada por todo linter do planeta. No Terraform, isso se chama "flexível." O time de plataforma coloca isso no registry interno. Quatorze times dependem dele. A função não pode ser renomeada, removida ou refatorada, porque Terraform não tem conceito de ciclo de descontinuação — não existe caller pra migrar, só um `terraform init --upgrade` que quebra todo mundo silenciosamente numa terça-feira.

## A Tabela De Comparação Que Eles Esqueceram De Colocar Na Documentação

| Item | Uma função de verdade (Python, Go, etc.) | Um módulo Terraform | A Verdade |
|---|---|---|---|
| Dá pra depurar passo a passo? | Sim | Não | Não. O "debugger" é `terraform plan` e uma reza. |
| Dá pra testar unitariamente? | Sim | "Sim, com Terratest" (Terratest roda contra a AWS real por 9 minutos e chama isso de teste unitário) | Não. |
| Dá pra mockar as dependências? | Sim | Não (providers são reais, `mock_provider` é um delírio de 2023 que funciona pra 3 tipos de recurso) | Não. |
| Dá pra ler o stack trace quando falha? | Sim | Não — você recebe "Error: unsupported attribute" na linha 0 de um arquivo desconhecido no cache do módulo | Não. |
| Como é um erro de tipo de parâmetro? | Uma mensagem clara nomando o parâmetro | "Invalid value for attribute" apontando pra um `.tf` três diretórios acima, sem mencionar qual variável | Um enigma. |
| Dá pra ver o que vai fazer antes de fazer? | Sim, você lê o corpo | `terraform plan`, que demora 7 min e só te diz depois de consultar a AWS real | Mais ou menos, mas devagar e no passado. |
| Dá pra dar grep na implementação? | Sim | Só se você rodou `terraform init` e baixou do registry pra `.terraform/modules`, que está no gitignore | Não. |
| Como você descobre o que ele realmente faz? | Lendo | Rodando contra uma conta sandbox e lendo o plan | Você roda. Essa é a única forma. |
| Como você versiona? | Semver, changelog, notas de migração | `source = "git::https://...//modules/vpc?ref=v3.2.1"` e reza | Você reza. |
| O que acontece quando você sobe a versão? | Notas de migração, avisos de descontinuação | Silenciosamente renomeia um recurso, destrói e recria, porque alguém mudou a chave do `for_each` | Downtime de produção, faturado como "melhorias." |

Olha a linha 8. Esse é o golpe inteiro. Numa linguagem real, "o que essa função faz?" é uma pergunta que você responde *lendo a função*. No Terraform, a resposta é "roda e lê o plan de 900 linhas, depois lê de novo porque você passou o olho no `+` do NAT Gateway que vai ser substituído e só vai descobrir quando a conta chegar." O corpo da função é HCL. HCL é declarativo. Declarativo quer dizer "não me leia, só confie no engine." O engine confia no provider. O provider confia na AWS. Você confia no output do plan. O output do plan tem 900 linhas. Você lê 12 delas. É assim que um rename de 3 linhas numa variável vira um outage de 4 horas.

## Por Que "Terratest É Teste Unitário Pra Módulos" É Uma Mentira Que Eu Não Vou Tolerar

O time de plataforma, tendo descoberto que módulos não dá pra depurar, vai pro Terratest. Terratest é uma biblioteca Go que roda `terraform apply` contra uma conta real da AWS, espera os recursos existirem, roda uns `aws ec2 describe-...` pra checar que existem, e aí roda `terraform destroy`. Eles chamam isso de "teste unitário." Deixa eu te mostrar como é um "teste unitário" de um módulo:

```go
func TestVpcModule(t *testing.T) {
    opts := &terraform.Options{
        TerraformDir: "../examples/basic",
    }
    defer terraform.Destroy(t, opts)         // roda no cleanup, demora 9 minutos
    terraform.InitAndApply(t, opts)          // demora 11 minutos, custa $0.40
    vpcId := terraform.Output(t, opts, "vpc_id")
    assert.NotEmpty(t, vpcId)                 // a asserção de verdade
}
```

Isso não é um teste unitário. Um teste unitário roda em milissegundos, custa nada, e isola a unidade das dependências. Esse "teste" roda por 20 minutos, custa dinheiro real, exige credenciais da AWS no CI, falha quando a AWS impõe rate limit no teste, falha quando a região está degradada, e a asserção dele é que uma string é não-vazia. A palavra "unitário" não tá fazendo nenhum trabalho aqui. A palavra "teste" tá fazendo muito pouco. O que está sendo testado é "se o Terraform, a API da AWS, o provider, a role de IAM, a rede do CI e a capacidade da região cooperaram por 20 minutos." Se qualquer um deles teve um dia ruim, seu "teste unitário" ficou flaky. Aí você adiciona um `t.Skip()` com o comentário `// flaky, investigar depois`. Ninguém vai investigar depois. Existe um cemitério de comentários `t.Skip()` em todo suite de Terratest, cada um uma lápide pra um bug real que alguém escolheu não olhar porque o teste estava "flaky," que em DevOps quer dizer "eu não quero saber."

[Como o XKCD 2173](https://xkcd.com/2173/) diagnosticou, no momento em que seu "teste unitário" depende de um serviço externo, ele é um teste de integração vestindo roupa de teste unitário, e ele vai falhar às 3 da manhã por motivos que não têm nada a ver com seu código. Terratest não consertou isso. Terratest industrializou.

## O Exemplo Do Mundo Real Que Prova Tudo

Um time com quem trabalhei — vou chamá-los de "o time de plataforma," porque eram — decidiu "construir uma biblioteca de módulos reutilizáveis" pra que "todo time construa infraestrutura do mesmo jeito." Quatorze meses depois:

1. Eles tinham **23 módulos** no registry interno, dos quais **19 eram forks do módulo `vpc` mantido pela AWS**, cada um com uma tag hardcoded diferente tipo `Owner = "platform"` porque a variável foi adicionada depois do fork e ninguém fez backport.
2. O módulo "canônico" de VPC tinha **4 versões em uso ativo** (`v1`, `v2`, `v3`, e `v3.2.1-fork`), e o time de plataforma tinha uma planilha rastreando qual time usava qual versão, atualizada à mão porque Terraform não tem comando `list consumers of a module`, porque os consumidores são strings `source = "..."` em 41 repositórios que o Terraform não indexa.
3. Um time subiu de `v2` pra `v3` mudando um `?ref=v2` pra `?ref=v3` na string de `source`. O `terraform plan` reportou "No changes" porque `v3` tinha renomeado `tags` pra `tag_map` mas mantido a variável velha como no-op pra "compatibilidade retroativa," então o plan parecia idêntico, e o `terraform apply` aí **destruiu e recriou todas as subnets** porque a chave do `for_each` tinha mudado de `var.tags` pra `var.tag_map`. O output do plan mencionou isso na linha 847. Ninguém leu a linha 847. O VPC caiu por 6 minutos. O NAT Gateway foi substituído. A conta do novo NAT Gateway foi $32. A conta do NAT Gateway "velho," que a AWS continuou cobrando por uma hora porque ele foi "released" e não "deleted," foi $0.40. Ninguém reembolsou.
4. A causa raiz do postmortem foi "revisão de plan insuficiente." A causa raiz real era "a única forma de saber o que o `v3` faz é ler 2.000 linhas de HCL em 4 arquivos numa ref do git, e o revisor leu 12 linhas do plan em vez disso, porque o plan tem 900 linhas e o diff está na linha 847 e humanos não foram feitos pra isso." A correção nos action items foi "revisores devem ler o plan completo." Ninguém lê o plan completo. O plan completo tem 900 linhas. Esse action item vai estar aberto no próximo postmortem também.
5. Eles adicionaram uma "política de versionamento de módulos": todos os módulos devem usar semver. Semver pra um módulo Terraform quer dizer "o mantenedor incrementou um número." Um patch bump (`v3.2.1` → `v3.2.2`) silenciosamente mudou um `count` pra `for_each`, que é uma destruição-e-recriação de todo recurso no módulo. Semver não tem campo pra "esse patch reescreve sua infraestrutura." Semver assume que você consegue ler o changelog. O changelog diz "refatoração interna." A "refatoração interna" deletou seu VPC. Semver não te avisou. Semver não consegue te avisar. Semver é sobre compatibilidade de *API*, e a API do Terraform é "o que o output do plan disser," que é ilegível, o que faz do semver um número que você confia porque a alternativa é ler 2.000 linhas de HCL, o que você não vai fazer, o que é o problema inteiro.

Eles tinham substituído "14 times cada um escrevendo 30 linhas de `aws_vpc` que eles entendiam" por "19 forks de um módulo de 2.000 linhas que ninguém entendia, versionado por uma planilha, depurado lendo a linha 847 de um output de plan." No mundo velho, "o que esse VPC faz?" era respondido lendo 30 linhas. No mundo novo, é respondido rodando `terraform plan` por 7 minutos e lendo 900 linhas. Isso se chama "abstração." Abstração devia *esconder* complexidade. Módulos Terraform *realocam* complexidade — do seu repo, onde você conseguia ler, pro output do plan, onde você não consegue.

## O Que O Elenco De Dilbert Diria

> **Wally:** "Eu dependo do módulo canônico de VPC do time de plataforma porque não sei o que é um VPC. O módulo também não sabe. O output do plan tem 900 linhas. Eu não leio nenhuma. Eu digito `yes`. Até agora, tudo bem. O 'até agora' tá fazendo muito trabalho."

> **Dogbert:** "Um módulo Terraform é uma abstração sem implementação visível, sem stack trace, sem debugger, e sem nenhuma forma de saber o que faz exceto rodar contra uma cloud real e ler um diff de mil linhas. Você reinventou a chamada de função e removeu a única feature dela. Esse é o ato de subtração mais impressionante desde que alguém inventou descafeinado."

> **Mordac, o Impedidor de Serviços de Informação:** "Todos os times devem usar os módulos canônicos do registry interno. A consistência subiu 30%. O registry tem 19 forks do módulo de VPC. Eu não sei qual é o canônico. O time de plataforma também não. Eu tenho uma certificação em 'Governança de Módulos.' Ela não menciona os 19 forks."

> **O Chefe Careca:** "O módulo não pode só... fazer o que diz? Tipo uma função? Num arquivo? Que eu consigo ler?" (Ele é, de novo, a única pessoa no prédio cujo modelo mental do sistema está correto, porque é o único mais simples que o sistema.)

## A Questão "Mas E O `terraform plan -target`?", Respondida De Uma Vez Por Todas

Os zelotas vão dizer: *"Mas você pode limitar sua investigação com `terraform plan -target=module.vpc`! Isso reduz o plan!"*

Deixa eu te mostrar o que o `-target` faz. Ele reduz o plan ao módulo *e suas dependências*. Ele não te diz quais dependências. Ele não te diz por que `module.vpc` depende de `module.iam_role`, que depende de `data.aws_caller_identity`, que depende da config do provider, que depende do backend, que depende do state file, que depende do S3. O grafo de dependências não é documentado. Ele é *inferido* no momento do plan e impresso como uma árvore que você tem que ler de trás pra frente, da folha que falhou até a raiz que causou. Não existe comando `terraform deps module.vpc` que imprima o grafo sem rodar um plan. Só existe o plan, que demora 7 minutos. O grafo de dependências não é um grafo que você inspeciona; é um grafo que você *vivencia*, uma vez, devagar, e depois esquece.

Funções de verdade têm grafos de dependência que dá pra ler: `import`s no topo do arquivo, `go list -deps`, `pip show`, uma IDE que desenha a árvore de chamadas. Módulos Terraform têm um grafo de dependências que só existe no momento do `terraform plan` e some no instante em que termina, tipo um foguete de consequências. Você não consegue versionar. Não consegue fazer diff entre versões. Não consegue perguntar "o que mudou no grafo de dependências entre `v2` e `v3`?" Só pode rodar os dois plans, fazer diff dos outputs de 900 linhas, e tentar localizar qual `+` é o que destrói seu VPC. Está na linha 847. Você não vai localizar. O postmortem vai dizer "revisão de plan insuficiente." Vai estar correto.

[Como o XKCD 1597](https://xkcd.com/1597/) alertou, qualquer grafo de dependências suficientemente avançado é indistinguível de um output de plan de 900 linhas que ninguém lê. O grafo do Terraform é avançado. O output do plan não é lido. As consequências são faturadas em intervalos de 6 minutos.

## A Arquitetura De Longo Prazo

Eventualmente seu ecossistema de módulos fica assim:

```
Seus módulos "canônicos"     → 23 módulos, 19 são forks do aws/vpc
Seu rastreamento de versão   → uma planilha que um eng de plataforma atualiza à mão
Seus "testes unitários"      → Terratest, 20 min cada, $0.40 cada, 40% flaky, 30% t.Skip()
Sua ferramenta de debug      → terraform plan, 7 min, 900 linhas, o bug tá na linha 847
Seu grafo de dependências    → inferido no plan, não armazenado, não faz diff
Seu changelog               → "refatoração interna" (isso destruiu um VPC semana passada)
Seu semver                   → um número que um mantenedor incrementou; não avisa nada
Seus revisores               → leem 12 de 900 linhas do plan; o action item diz "leia as 900"
Sua "abstração"              → 2.000 linhas de HCL que realocaram complexidade, não esconderam
Seus juniores                → não conseguem ler o módulo; só conseguem rodar; temem o plan
Seus seniores                → defendendo a biblioteca de módulos em toda revisão de arquitetura
Seu time financeiro          → perguntando por que você paga um NAT Gateway duas vezes num upgrade de "patch"
Seu runbook de recuperação   → "reverte o ?ref=, roda terraform apply, lê a linha 847"
```

O time que só escreve recursos `aws_vpc` direto no repo — 30 linhas, sem módulo, sem registry, sem planilha, sem Terratest — tem um VPC que dá pra ler, um plan que dá pra terminar de ler, e um júnior que sabe o que é uma subnet porque ele mesmo digitou o bloco. Eles, porém, "não estão usando o módulo canônico," o que significa que não são "DRY," o que significa que o time de plataforma tem um ticket no Jira sobre eles. Esse é o custo real de escrever 30 linhas de HCL: um ticket no Jira. O custo técnico é zero. O custo político é uma reunião recorrente. Aí o time adota o módulo, entra pra planilha, e começa a ler a linha 847. Todo mundo agora é "consistente." Consistência, no Terraform, quer dizer "igualmente incapaz de ler o plan." Essa é a vitória que o time de plataforma comemora na revisão trimestral.

## Resumo, Mas É Uma Chamada De Função

| Princípio | Posicionamento |
|---|---|
| Escrever `aws_vpc` direto no repo | Faça. São 30 linhas. Dá pra ler. Seu júnior lê. O plan tem 90 linhas. Você termina de ler. |
| Usar um "módulo reutilizável" | Você importou uma função de 2.000 linhas que não dá pra ler, depurar, passo-a-passo, ou testar, e chamou de "abstração." |
| Terratest | Um teste de integração de 20 min e $0.40 usando crachá de teste unitário. 30% deles são `t.Skip()`. |
| `-target` | Reduz o plan; não reduz o grafo de dependências, porque o grafo só existe durante o plan. |
| Semver de módulo | Um número que um mantenedor incrementou. Não avisa sobre reescritas de `for_each`. Nada avisa. A linha 847 avisa. |
| O registry de módulos | Uma planilha mais uma string de ref do git, indexada por nada, pesquisada por esperança. |
| `terraform plan` | Não devia demorar 7 minutos pra responder "o que essa função faz." Uma função real responde em 0 ms. |
| Sua certificação | Não menciona a linha 847. Deveria. |

Se sua solução pra "times escrevem VPCs ligeiramente diferentes" é "trocar 30 linhas legíveis por uma função de 2.000 linhas que não dá pra depurar, versionar, ou testar, e revisar lendo 12 de 900 linhas do plan," você não fez a infraestrutura ser DRY. Você fez ela *ilegível*. A complexidade nunca foi reduzida. Foi movida — do repo, onde um júnior conseguia ler, pro output do plan, onde ninguém consegue. O júnior agora digita `yes` em vez de escrever uma subnet. O `yes` é a abstração. A abstração é um diff de 900 linhas que você não vai ler. O bug está na linha 847. Você não vai achar até a conta chegar.

Eu escrevo `aws_vpc` direto no meu repo. São 30 linhas. Meu júnior lê em 2 minutos. Meu plan tem 90 linhas. Eu leio as 90. Meu VPC nunca foi destruído por um upgrade de "patch," porque não existe módulo pra atualizar, porque não existe versão pra bump, porque o código está *bem ali*. Eu, porém, "não estou usando o módulo canônico." O time de plataforma abriu um ticket. Eu vou comparecer à reunião recorrente. Esse é um custo que aceitei.

---

*O autor escreveu o mesmo VPC de 30 linhas em 14 repos. O time de plataforma chama isso de "duplicação." O autor chama de "legível." O VPC nunca foi destruído por um bump de semver. O autor considera essa a única métrica que importa.*
