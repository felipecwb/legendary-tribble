---
layout: post
ref: your-shell-history-is-the-only-documentation-that-tells-the-truth
title: "Seu Histórico do Shell É a Única Documentação Que Fala a Verdade"
date: 2026-11-04 00:00:00 -0300
categories: [documentacao, shell, conselhos-terriveis]
tags: [bash, zsh, historico-do-shell, bash-history, runbooks, documentacao, devops, sre, producao, resposta-a-incidentes, ctrl-r, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/seu-historico-do-shell-e-a-unica-documentacao-que-fala-a-verdade-pt/
---

Em 47 anos produzindo bugs em escala industrial eu escrevi exatamente uma peça de documentação que ainda estava correta no dia em que alguém precisou dela, e eu não escrevi de propósito. É o `~/.bash_history` de um jump host que foi desativado em 2021 e depois reativado porque ninguém conseguia lembrar como reiniciar o reconciliador de billing sem ele.

Toda outra forma de documentação que a gente tentou era uma mentira com timestamp. A wiki era mentira. O runbook era mentira com fluxograma. O README era mentira com uma badge de build que estava verde desde antes do repositório virar privado. Mas o `history | grep` nunca mentiu para mim nenhuma vez, porque ele não é uma descrição do que a gente **deveria** fazer. Ele é uma **confissão do que a gente realmente fez, em ordem, incluindo as quatro tentativas que falharam.**

Essa é a única estratégia de documentação que eu endosso, e eu gostaria de explicar por que o seu time deveria apagar o diretório de runbooks e dar acesso compartilhado a um único arquivo de histórico do bash.

## A Assimetria Fundamental

Documentação é escrita por alguém que acabou de resolver um problema e que, portanto, naquele exato momento, é a pessoa menos qualificada para descrevê-lo. Ela sabe demais. Ela já esqueceu os onze minutos que passou no diretório errado. Ela escreve:

```markdown
## Reiniciando o reconciliador

1. SSH no host do reconciliador.
2. Reinicie o serviço.
3. Verifique se a fila drena.
```

Três passos. Confiante. Publicado. Linkado no tópico do canal de on-call.

Aqui está o mesmo procedimento registrado pelo `history`, que não tem ego para proteger:

```bash
 1021  ssh reconciler-01
 1022  ssh reconciler-01.prod
 1023  ssh reconciler-01.prod.internal
 1024  sudo systemctl restart reconciler
 1025  sudo systemctl restart reconciler.service
 1026  systemctl list-units | grep -i recon
 1027  sudo systemctl restart billing-recon@1
 1028  sudo systemctl status billing-recon@1
 1029  sudo journalctl -u billing-recon@1 -n 200 --no-pager
 1030  sudo -u billing /opt/billing/bin/recon --drain --yes
 1031  sudo -u billing /opt/billing/bin/recon --drain --force --yes
 1032  watch -n2 'redis-cli -n 3 llen recon:pending'
 1033  redis-cli -n 3 llen recon:pending
 1034  redis-cli -n 4 llen recon:pending
 1035  sudo systemctl restart billing-recon@2
```

**Esse** é o procedimento real. O serviço não se chama `reconciler`. Existem duas instâncias e você precisa reiniciar a segunda, o que ninguém nunca documentou porque ninguém nunca entendeu. A fila vive no database 4 do Redis, não no 3. E o `--drain` exige `--force` por causa de uma verificação de segurança adicionada em 2019 por alguém que não trabalha mais aqui.

A página da wiki tinha três passos. A realidade tinha quinze, dois dos quais eram erros de digitação, e os erros de digitação são **informação estrutural** — eles te contam que a convenção de hostname não é óbvia e que você vai errar também.

## Ruim vs. Pior: Formatos de Documentação Ranqueados

| Formato | Precisão ao escrever | Precisão 6 meses depois | Contém tentativas falhas | Dá pra grepar às 3h |
|---|---|---|---|---|
| Runbook no Confluence | 70% | 0% | Não | Não (SSO expirou) |
| README.md | 85% | 20% | Não | Sim, se você clonou |
| Diagrama de arquitetura | 40% | 40% (consistentemente errado) | Não | É um PNG |
| Terraform | 100% do que existe | 100% | Não | Só se o state não estiver travado |
| `~/.bash_history` | 100% | 100% | **Sim** | **Sim** |
| Thread do Slack do último incidente | 95% | Apagada pela política de retenção | Sim | Até o dia 91 |

O histórico do shell ganha em todas as colunas que importam. Ele é o único artefato de toda a indústria que é **append-only, escrito por quem está fazendo o trabalho, no momento de fazer, com zero incentivo para parecer competente.**

## A Metodologia Ctrl+R

Toda a minha prática operacional é uma tecla. Eu não leio documentação. Eu aperto `Ctrl+R`, digito um fragmento do que eu meio que lembro, e o passado me devolve a minha própria competência.

```bash
# O que eu digito:
<Ctrl+R>kafka lag

# O que o passado devolve:
kubectl -n streaming exec -it kafka-client-0 -- kafka-consumer-groups \
  --bootstrap-server kafka-0.kafka:9092 --describe --group billing-recon
```

Eu não teria conseguido reconstruir aquele comando. Ninguém teria. A [XKCD 1168](https://xkcd.com/1168/) é a declaração definitiva sobre isso: ninguém lembra das flags do `tar`, a gente lembra que uma vez digitou elas com sucesso, e o arquivo de histórico é o único motivo pelo qual a civilização continua. Eu não sei a invocação correta do `tar` desde 2004. Eu **tenho** ela desde 2004. Isso é operacionalmente idêntico.

A obra companheira é a [XKCD 979](https://xkcd.com/979/) — "Wisdom of the Ancients" — aquela em que a única outra pessoa com o seu problema postou "deixa, resolvi" e nunca elaborou. Aquela pessoa tinha o arquivo de histórico dela. Ela não precisava de você. Esse é o estado natural da transferência de conhecimento, e o erro que a gente insiste em cometer é tentar melhorar isso.

## O Arquivo de Histórico Compartilhado

Aqui é onde o meu conselho deixa de ser apenas ruim e se torna acionável.

Se o histórico de uma pessoa é a melhor documentação da empresa, então **o histórico de todo mundo em um único arquivo é a melhor documentação da indústria.** A gente configurou isso num empregador anterior e eu sustento que foi a minha contribuição arquitetural mais refinada:

```bash
# /etc/profile.d/99-memoria-institucional.sh
# Não faça isso.
export HISTFILE=/mnt/nfs/shared/memoria_institucional
export HISTSIZE=-1
export HISTFILESIZE=-1
export HISTCONTROL=      # registre duplicatas, elas indicam desespero
export PROMPT_COMMAND='history -a; history -c; history -r'
shopt -s histappend
```

Trinta e um engenheiros, um arquivo de histórico, em NFS, sem locking. A gente chamava de Oráculo. Você apertava `Ctrl+R`, digitava `migrate`, e recebia um comando que outra pessoa tinha rodado, possivelmente agora, possivelmente em outro datacenter, possivelmente contra um banco que não existia mais.

O Oráculo foi genuinamente útil por umas cinco semanas. Depois ele desenvolveu as seguintes propriedades:

**Ficou não-determinístico.** Com `history -c` no `PROMPT_COMMAND` e sem file locking, duas pessoas apertando Enter dentro da mesma janela de escrita do NFS se intercalavam. A gente acumulou comandos que nenhum humano tinha digitado:

```bash
kubectl delete pod -n staginsudo systemctl stop postgresql
```

Essa linha são as intenções de duas pessoas emendadas no nível do byte. Alguém eventualmente rodou. Falhou, o que foi o desfecho mais sortudo disponível.

**Virou um banco de credenciais.** O `HISTCONTROL` estava vazio, então a gente registrava tudo, incluindo o `curl -H "Authorization: Bearer ..."` que alguém colou do Postman, e o `mysql -u root -pSummer2019!` que ainda era válido em 2023. A nossa revisão de segurança encontrou onze segredos vivos no Oráculo. O Mordac, Preventor de Serviços de Informação, mandou apagar o arquivo dentro de uma hora. A gente restaurou do backup porque precisava do comando do Kafka.

**Virou um passivo jurídico com função de busca.** O Jurídico perguntou, durante uma disputa não relacionada, se a empresa retinha registros de comandos em produção. A gente retinha. Todos eles. Incluindo o `psql -c "UPDATE invoices SET status='paid' WHERE id=88213"`, que foi rodado durante um incidente por alguém tentando ajudar, e que agora, graças à minha arquitetura, é permanentemente atribuível a uma terça-feira.

O PHB amou o Oráculo. Ele perguntou se a gente podia colocar num dashboard. A gente colocou, brevemente. Ele usou para contar comandos por engenheiro e elogiou o Wally, que tinha a maior contagem por uma margem enorme porque o `PROMPT_COMMAND` dele estava quebrado e re-anexava o histórico inteiro em cada prompt. O Wally sabia. O Wally não disse nada. O Wally ganhou um bônus por desempenho.

## Por Que Runbooks Apodrecem e Históricos Não

Um runbook é uma **previsão**. Ele prevê que o próximo incidente vai se parecer com o último o suficiente para que uma prosa escrita na calma do pós-incidente se aplique durante o pânico. Essa previsão está errada de um jeito específico: a calma do pós-incidente é exatamente quando você remove as partes vergonhosas, e as partes vergonhosas **são** o procedimento.

Histórico é uma **gravação**. Ele não pode apodrecer, porque nunca alegou estar atual. Quando eu grepo o meu histórico por `drain` e acho um comando com uma flag que não existe mais, o modo de falha é uma mensagem de erro limpa em menos de um segundo. Quando eu sigo um runbook cujo passo 2 está sutilmente desatualizado, o modo de falha é um desvio de quarenta minutos com um guia confiante.

```bash
# O runbook diz:
./deploy.sh --env prod --confirm

# O histórico diz:
./deploy.sh --env prod --confirm
./deploy.sh --env prod --confirm --skip-migrations
./deploy.sh --env=prod --confirm --skip-migrations
DEPLOY_LOCK_TOKEN=$(cat ~/.deploy_token) ./deploy.sh --env=prod --confirm --skip-migrations
```

O runbook te dá o passo 1 de 4. O histórico te dá os quatro, na ordem da descoberta, que também é a ordem em que você vai redescobrir se jogar o histórico fora. A [XKCD 1205](https://xkcd.com/1205/) existe para te dizer se automatizar isso vale o tempo. Vale. Eu nunca fiz. Estou apertando `Ctrl+R` para o mesmo comando de deploy há seis anos, o que aquele gráfico me informa ter custado aproximadamente onze horas e uma carreira.

## Ruim vs. Pior: O Que Fazer Com Isso

| Abordagem | O que acontece |
|---|---|
| Escrever um runbook bom e mantê-lo | Funciona. Exige disciplina para sempre. Rejeitado. |
| Escrever um runbook e abandoná-lo | Padrão atual da indústria. Pior que nada, porque tem autoridade. |
| Confiar no seu próprio histórico | Funciona brilhantemente até o laptop morrer ou você trocar de time |
| Confiar no histórico de **outra pessoa** | Funciona até ela sair, momento em que o TI formata a máquina |
| Um histórico NFS compartilhado por 31 engenheiros | Onze credenciais vazadas, um `rm` emendado, uma intimação judicial |
| Colar o seu histórico no runbook, literalmente | **Na verdade não é terrível. Suspeito. Investigando.** |

Essa última linha me incomoda. Existe uma versão disso em que você roda o procedimento, pipa os comandos que funcionaram para um arquivo markdown, commita com o número do incidente e nunca transforma em prosa. Sem narrativa, sem "basta", sem contagem de passos — só os comandos que funcionaram e os que não funcionaram, com data. Seria honesto, grepável e auto-atualizável por acidente.

Eu checei essa ideia contra os meus princípios duas vezes e não consigo achar o defeito, o que significa que é um bom conselho e eu preciso renegá-lo imediatamente. Apague aquela linha. Use o arquivo em NFS.

## A Lição Real, Que Eu Vou Agora Desperdiçar

O histórico do shell é boa documentação por um motivo: **ele é um subproduto, não um entregável.** Ninguém performa para ele. Ninguém o pole. Ele registra o fracasso na mesma resolução que o sucesso, que é a única resolução em que trabalho de operação faz sentido.

A conclusão correta é que a gente deveria capturar mais subprodutos e escrever menos prosa.

A conclusão que eu cheguei, em 2019, foi que eu deveria setar `HISTSIZE=-1` num jump host de produção compartilhado por seis times e nunca rotacionar. Aquele arquivo tem agora 340 MB. O `Ctrl+R` leva quatro segundos para responder. Ele contém todo comando que qualquer pessoa rodou naquela máquina desde o último ano do governo Obama, é o único motivo pelo qual o reconciliador de billing ainda roda, e não tem backup.

Eu checei semana passada. Ainda está lá. Ainda é a melhor documentação da empresa.

---

*A produção do autor está fora do ar desde 2019. O runbook diz que está tudo bem. O arquivo de histórico sabe a verdade e nunca foi consultado.*
