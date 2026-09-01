---
layout: post
ref: terraform-state-is-a-shared-json-file-you-agreed-to-trust-with-your-lives
title: "O State Do Terraform É Um Arquivo JSON Compartilhado Que Vocês Concordaram Em Confiar Com A Vida"
date: 2026-09-01 00:00:00 -0300
categories: [devops, infrastructure, cloud]
tags: [terraform, iac, state, infrastructure-as-code, devops, cloud, json, s3, dynamodb, lock-files, drift, ponto-unico-de-falha]
permalink: /pt-br/2026/09/01/o-state-do-terraform-e-um-arquivo-json-compartilhado-que-voces-concordaram-em-confiar-com-a-vida/
---

Depois de 47 anos escrevendo software — incluindo 12 anos escrevendo "infraestrutura como código" antes dessa frase significar qualquer coisa, na época em que se chamava "um shell script que roda uma vez e nunca mais é tocado" — cheguei a uma conclusão que o sacerdócio do DevOps não vai sobreviver a ouvir:

**Terraform não é infraestrutura como código. É um arquivo JSON num bucket S3 que quatrocentos engenheiros concordaram, por consenso silencioso, em tratar como fonte da verdade para uma conta AWS de $40.000 por mês.**

É isso. Esse é o produto inteiro. Tem um arquivo `.tf` que diz o que você *acha* que tem. Tem um `terraform.tfstate` que diz o que você *realmente* tem. Esses dois arquivos nunca concordaram. O state é a única coisa em que o Terraform acredita. O `.tf` é decorativo. Você está escrevendo fanfic sobre sua infraestrutura e o state é o cânone.

O time de plataforma já está abrindo um ticket para revogar meu acesso AWS. O pessoal HashiCertified já está estendendo o `terraform validate` no coração. As três pessoas que realmente leram o código-fonte do Terraform estão estendendo o uísque. Deixem. Eles nunca tiveram que recuperar um state corrompido às 2 da manhã enquanto o pager do plantão derrete porque alguém rodou `terraform apply` de um branch antigo.

## A Grande Ilusão De "Infraestrutura Declarativa"

Aqui está o pitch: *Descreva sua infraestrutura declarativamente. O Terraform calcula o diff e aplica. Seus arquivos `.tf` são a fonte da verdade.*

Aqui está o que realmente acontece:

```hcl
# main.tf - o que você ACHA que tem
resource "aws_instance" "prod_web" {
  count         = 3
  ami           = "ami-12345678"
  instance_type = "t3.medium"
}
```

```json
// terraform.tfstate - o que você REALMENTE tem
{
  "version": 4,
  "resources": [
    {
      "type": "aws_instance",
      "name": "prod_web",
      "count": 1,
      "instances": [
        { "attributes": { "id": "i-0abc123", "tags": { "managed-by": "terraform" } } },
        { "attributes": { "id": "i-0def456", "tags": { "managed-by": "manual", "who-did-this": "dave" } } },
        { "attributes": { "id": "i-0ghi789", "tags": {} } }
      ]
    }
  ]
}
```

Três coisas no `.tf`. Uma coisa no state. Duas coisas no state que não estão no `.tf`. Uma delas foi criada à mão pelo Dave, que saiu em 2023, e está rodando o banco de dados de produção. A outra não tem tags e ninguém sabe o que faz mas custa $2.800 por mês e o time financeiro tem perguntas.

Você roda `terraform plan`. O Terraform olha pro state. Ele não olha pra AWS. Ele compara o `.tf` com o state, vê um diff de "3 vs 1," e alegremente anuncia que vai **destruir duas instâncias e criar duas instâncias.** Ele não vai te dizer que a instância misteriosa do Dave é o banco de produção. Ele não vai te dizer que o state é de um branch que foi mergeado há 14 meses. Ele não vai te dizer que a "1 instância" no state na verdade foi substituída por um launch template em 2024 e o state nunca foi atualizado.

Isso se chama "infraestrutura declarativa."

## A Tabela De Comparação Que Eles Não Querem Que Você Veja

| Preocupação | Shell scripts feitos à mão | Terraform | A Verdade |
|---|---|---|---|
| Fonte da verdade | O console da AWS | `terraform.tfstate` | O console da AWS (o Terraform só não sabe) |
| O que acontece quando a realidade diverge | Você conserta, fica consertado | `terraform refresh` reescreve o state, mente sobre sucesso, diverge de novo | Você conserta no console, o state diverge para sempre |
| Consegue recuperar de um state corrompido | Restaura das suas anotações | Restaura do bucket S3, que também está corrompido | Não |
| Tempo de "terraform plan" | 2 segundos (é `aws ec2 describe-instances`) | 47 minutos (download de provider, refresh, `state pull`, plan) | 47 minutos de medo |
| O que "apply" realmente destrói | Nada que você não rode | O que o state antigo disser, que é tudo | O que o state antigo disser, que é tudo |
| Quem já leu o state | Ninguém precisa | Ninguém leu | Ninguém nunca vai |
| Ponto único de falha | O shell script | `terraform.tfstate` E o bucket S3 E a lock do DynamoDB E a config do backend | O state, sempre o state |

Repare na linha "tempo de plan." Esse é o mito de origem de toda a indústria de DevOps: "o Terraform torna a infraestrutura reproduzível." Torna. Torna reproduzivelmente *errada*, reproduzivelmente *lenta*, e reproduzivelmente *refém de um JSON que quatro pessoas já olharam*. A infraestrutura nunca foi a parte difícil. A parte difícil era concordar sobre o que você tem. Nenhuma ferramenta resolve isso. A ferramenta só te dá um novo JSON para discordar.

## Por Que "State Locking" É Só Um Mutex Que Você Paga $0,0009 Para A AWS

A defesa do state do Terraform é: *"Usamos um backend remoto com S3 e locking no DynamoDB, então é seguro."*

Deixa eu te mostrar o que "seguro" significa na Terraform-land:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

Isso guarda a fonte da verdade da sua infraestrutura de produção inteira como um único arquivo JSON num bucket S3, protegido por uma linha numa tabela DynamoDB que custa nove décimos de centavo por escrita. A lock impede dois engenheiros de rodar `terraform apply` ao mesmo tempo. Ela *não* impede:

1. Um engenheiro rodar `terraform apply` de um branch antigo que não puxou o state mais recente.
2. Um engenheiro rodar `terraform destroy` "só pra ver o que aconteceria" e confirmar `yes` porque o prompt é memória muscular.
3. O state corromper porque alguém commitou um `.tfstate` no git em 2021 antes do backend remoto ser configurado, e o hook pre-commit ainda está no repo, silenciosamente stagedando.
4. A AWS deletar seu bucket S3 porque o alarme de billing foi pra um email que ninguém checa desde 2022.
5. O DynamoDB throttando a aquisição da lock durante um evento de região, então dois engenheiros *simultaneamente* rodam apply, e o state agora são dois blobs JSON mesclados por `terraform state push` numa race que o YAML teria orgulho.

A lock é um mutex. O mutex protege um arquivo. O arquivo é a fonte da verdade. O mutex é honesto sobre o que é. O Terraform não é. O Terraform chama o mutex de "locking distribuído" e te cobra uma prova de certificação pra você descobrir que ele existe.

Como o [XKCD 927](https://xkcd.com/927/) estabeleceu e a indústria de DevOps passou uma década não lendo: todo novo "padrão" para substituir os shell scripts existentes vira só outro padrão com um arquivo `.tfstate`. O Terraform é o décimo quinto. Ele substituiu o CloudFormation, que substituiu o Chef, que substituiu shell scripts, que substituiu "clica no console e anota numa wiki." Cada um prometeu tornar a infraestrutura reproduzível. Cada um virou um JSON que você tem que babysitar, com uma árvore de dependências.

## O Exemplo Do Mundo Real Que Prova Tudo

Um time com o qual trabalhei — vou chamar de "o time de plataforma," porque era — decidiu adotar Terraform para "tornar a infraestrutura reproduzível e auditável." Dezoito meses depois:

1. O state deles era **47 MB de JSON** descrevendo 2.300 recursos, a maioria blocos `null_resource` rodando shell scripts `local-exec`, porque o Terraform não conseguia expressar o que precisavam.
2. `terraform plan` levava **52 minutos** porque toda versão de provider tinha que ser baixada e todo recurso tinha que ser atualizado contra uma API que os limitava.
3. Eles tinham **4 backends remotos** (um por ambiente) e a tabela de lock do DynamoDB para `prod` estava em `us-west-2` enquanto o bucket S3 estava em `us-east-1`, então a latência cross-region adicionava 11 segundos a cada plan, e ninguém lembrava por quê.
4. O state tinha **17 recursos "órfãos"** que o Terraform conhecia mas nenhum arquivo `.tf` referenciava, porque alguém deletou o bloco `.tf` mas esqueceu `terraform state rm`, e agora `terraform apply` se recusa a tocá-los porque são "gerenciados pelo Terraform."
5. Um júnior rodou `terraform destroy` de um feature branch pra limpar um ambiente de teste, não percebeu que o backend apontava pra `prod`, e deletou **14 recursos de produção** em 4 minutos. O state registrou fielmente a destruição. O console da AWS parou fielmente de cobrar. PagerDuty paginou fielmente 9 pessoas.
6. A recuperação levou **6 horas** e envolveu `terraform import` para 14 recursos, cada um exigindo achar o ID da AWS à mão, porque o state agora estava vazio e os arquivos `.tf` não tinham atributo `id`, porque não é assim que o Terraform funciona, porque o modelo do Terraform é que o state *é* a fonte da verdade, e a fonte da verdade tinha ido embora.
7. Eles escreveram um postmortem. A causa raiz foi "erro humano." A causa raiz real era "construímos um sistema onde um único JSON num bucket S3 é o único registro de $40.000 por mês de infraestrutura, e depois demos acesso de escrita pra 40 engenheiros."

Eles tinham substituído ~300 linhas de shell scripts que rodavam em 4 segundos por um **JSON de 47 MB que levava 52 minutos pra planejar e podia ser destruído por um júnior de um feature branch**. Em shell scripts, destruir a prod exigia digitar `aws ec2 terminate-instances` 14 vezes. No Terraform, exige digitar `yes` uma vez. Isso se chama "ergonomia."

Isso se chama "infraestrutura como código."

## O Que O Elenco De Dilbert Diria

> **Wally:** "Eu uso Terraform porque significa que nunca preciso saber o que tem na AWS. O state sabe. O state tá errado, mas sabe, e isso é o suficiente pra minha revisão de desempenho."

> **Dogbert:** "O state do Terraform existe pra fazer engenheiros sentirem que tornaram a infraestrutura reproduzível realocando a fonte da verdade pra um JSON que não conseguem ler. A infraestrutura agora está no state, o state está num bucket S3, o bucket está numa região que você esqueceu, e a região está numa conta que você não consegue logar. Você terceirizou sua infraestrutura pra um arquivo que não consegue editar. Parabéns."

> **Mordac, o Preventer de Serviços de Informação:** "Eu determinei o uso de Terraform em todos os projetos. A reprodutibilidade da infraestrutura subiu 40%. O tempo de plan subiu 600%. O state tem 47 MB. Ninguém leu. Eu tenho uma certificação."

> **O Chefe de Cabeça Pontuda:** "Podemos só usar o console? Aquele onde você clica?" (Ele é a única pessoa no prédio cuja infraestrutura bate com a realidade.)

## A Pergunta "Mas E A Detecção De Drift?", Respondida De Uma Vez Por Todas

Os zelotas do Terraform vão dizer: *"Mas temos detecção de drift! Rodamos `terraform plan` no CI toda noite e alertamos em qualquer diff não-vazio!"*

Você não tem detecção de drift. Você tem um job noturno que compara seus arquivos `.tf` com seu state, ambos errados, e te alerta que eles discordam um do outro. Ele não te alerta que eles discordam da *AWS*, porque `terraform plan` não faz refresh por padrão no CI (é lento demais), então o plan está comparando duas mentiras e dizendo que elas batem.

Detecção de drift real seria: "comparar o state com a AWS." Isso se chama `terraform refresh`, que ninguém roda no CI porque leva 50 minutos e reescreve o state, o que significa que ele *é* drift, o que significa que rodar detecção de drift cria drift, o que significa que você construiu um sistema onde detectar o problema *é* o problema. Os filósofos teriam orgulho.

Detecção de drift real vem de **perguntar à AWS o que você tem** — `aws ec2 describe-instances`, `aws s3 ls`, `aws cloudformation list-stacks` — e anotar. É isso que um shell script faz em 4 segundos. O Terraform faz em 50 minutos e depois *cacha a resposta num JSON que vai confiar pelo próximo mês*. O cache é o bug. O cache é sempre o bug.

[Como o XKCD 1513](https://xkcd.com/1513/) lembra, no momento em que você depende de um state, você adotou o drift dele, o calendário de corrupção dele, e as opiniões dele sobre o que `count = 0` significa (significa "destruir tudo"). Ele vai mudar os três. Você vai rodar `terraform state push`. Esse é o ciclo. Não tem saída exceto shell scripts, que você estava tentando evitar porque, aparentemente, *não são reproduzíveis o suficiente*.

## A Arquitetura De Longo Prazo

Eventualmente sua plataforma se parece com isso:

```
Seus arquivos .tf    → descrevem o que você DESEJA que tivesse
Seu state           → 47 MB de JSON descrevendo o que VOCÊ TINHA, há 14 meses
Sua conta AWS       → tem 17 recursos que o state não conhece
Seu CI              → roda terraform plan toda noite, reporta "No changes"
Seu plantão         → paginado quando "No changes" encontra "o banco sumiu"
Seu backend         → bucket S3 em us-east-1, lock do DynamoDB em us-west-2
Seus júniores       → não conseguem rodar terraform apply sem um plan de 47 minutos
Seus sêniores       → defendendo o state em toda revisão de arquitetura
Seu time financeiro → perguntando por que você paga por 2.300 recursos quando .tf diz 400
Seu runbook de recuperação → "restaura o state do S3" (o S3 é o problema)
```

O time de shell script tem 300 linhas de bash, um `describe-instances` que leva 4 segundos, e um júnior que consegue rodar sem certificação. A infraestrutura deles bate com a AWS porque eles *perguntam* pra AWS. A recuperação deles é "roda o script de novo." Eles estão, no entanto, *envergonhados* em meetups de DevOps porque "não usam Infraestrutura como Código." Esse é o custo real dos shell scripts: social. O custo técnico é zero. O custo social é enorme. Então pagamos o custo técnico de um JSON de 47 MB pra evitar o custo social de admitir que shell-scripteamos, porque somos, afinal, primatas com conta AWS.

## Resumo, Mas É Um State

| Princípio | Posição |
|---|---|
| Escrever shell scripts | Faça. São 300 linhas. Pergunta pra AWS. A AWS sabe. |
| Usar Terraform | Você importou um JSON de 47 MB que discorda da AWS e chamou de "reproduzível." |
| State locking | Um mutex que você paga nove décimos de centavo à AWS pra proteger um arquivo que nunca leu. |
| Detecção de drift | Comparar duas mentiras e alertar quando elas batem. |
| `terraform apply` | Não deveria levar 52 minutos pra planejar um `yes` que destrói 14 recursos. |
| O state | Nunca foi a fonte da verdade. A AWS foi. Você só parou de perguntar. |
| Sua certificação | Localizada num badge do LinkedIn, e não menciona o state. |

Se a sua solução para "infraestrutura é difícil de reproduzir" é "guardar um JSON num bucket S3 e fazer 400 engenheiros confiarem nele como fonte da verdade para uma conta AWS de $40.000 por mês," você não tornou a infraestrutura reproduzível. Você a tornou *refém de um arquivo*. O arquivo está errado. O arquivo sempre esteve errado. O arquivo vai estar errado de novo mês que vem, e `terraform apply` vai fielmente destruir o que o arquivo mandar destruir, porque o arquivo é a única coisa em que o Terraform acredita.

Eu escrevo shell scripts. São 300 linhas. Eles perguntam pra AWS o que eu tenho, e anotam, e o que anotam é verdade porque a AWS disse. Meu júnior consegue rodar em 4 segundos sem certificação. Minha recuperação é "roda o script de novo." Eu, no entanto, não sou convidado pra conferências da HashiCorp. Esse é um custo que aceitei.

---

*A infraestrutura do autor é descrita por um shell script de 300 linhas desde 2014. O script nunca mentiu. O autor acha isso suspeito e está investigando.*
