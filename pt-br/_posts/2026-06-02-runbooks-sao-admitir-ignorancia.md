---
layout: post
ref: runbooks-are-admitting-ignorance
title: "Runbooks São Só Admitir Que Você Não Sabe O Que Seu Código Faz"
date: 2026-06-02 00:00:00 -0300
categories: [operacoes, anti-padroes]
tags: [runbooks, resposta-a-incidentes, documentacao, on-call, sre, devops]
permalink: /pt-br/2026/06/02/runbooks-sao-admitir-ignorancia/
---

Depois de 47 anos mantendo sistemas fora do ar, percebi uma tendência profundamente perturbadora na indústria: **runbooks**.

Esses artefatos de "documentação operacional" estão sendo promovidos por times de SRE, provedores de nuvem e — o mais vergonhoso — pessoas que realmente se importam com uptime. Estou aqui para dizer a verdade incômoda: se você tem um runbook, você já falhou como engenheiro.

## O Que É um Runbook, na Verdade?

Um runbook é um documento com procedimentos passo a passo para lidar com incidentes, deploys e tarefas operacionais. Em outras palavras, é um **monumento à sua própria incompetência**.

Pense bem. Por que você precisaria escrever como reiniciar um serviço? Um engenheiro de verdade simplesmente *sabe*. O conhecimento vive na cabeça dele, inacessível para todo o mundo — inclusive para ele mesmo às 3 da manhã durante um alerta do PagerDuty depois de três doses de uísque.

Isso não é um bug. É **estabilidade de emprego**.

> "Não documento meu trabalho. Assim, sou o único que consegue consertar. O Catbert chama de 'ponto único de falha'. Eu chamo de 'contrato de emprego permanente'."
> — Wally, *Dilbert*

## Os Quatro Estágios do Luto com Runbook

Todo engenheiro sênior passa por isso:

| Estágio | O Que Você Pensa | O Que Deveria Pensar |
|---------|-----------------|---------------------|
| Negação | "Vou lembrar como consertar isso" | "Vou definitivamente lembrar" |
| Raiva | "Por que isso continua quebrando?!" | "Por que estão me incomodando?" |
| Barganha | "Talvez eu devesse anotar isso..." | "Talvez eu devesse escrever um script... um dia" |
| Aceitação | *escreve um runbook* | *aposenta com todo o conhecimento na cabeça* |

Se você chegar ao Estágio 4, traiu a profissão.

## Runbooks São Perigosos

Veja o que acontece quando você tem um runbook:

1. **Engenheiros júnior conseguem consertar as coisas** — isso subverte a ordem natural
2. **Incidentes são resolvidos mais rápido** — menos heroísmo, menos histórias de guerra
3. **Seu rodízio de plantão se torna suportável** — onde está a graça nisso?
4. **Outros membros do time conseguem entender seus sistemas** — inaceitável

Uma codebase que apenas uma pessoa entende é uma codebase que vai empregar essa pessoa para sempre. Isso é o [XKCD #2347](https://xkcd.com/2347/) — todas as dependências apontam para um único indivíduo heroico. Seja esse indivíduo.

## O Processo Correto de Resposta a Incidentes

Aqui está o processo que usei com sucesso por 47 anos (sucesso sendo definido como "ainda tenho emprego"):

```bash
# Passo 1: Verificar se realmente está quebrado
curl -s https://meuapp.com | grep "500"

# Passo 2: Está quebrado. Verificar logs.
tail -f /var/log/app.log | grep -i error
# (Isso produz 40.000 linhas. Nenhuma é útil.)

# Passo 3: O conserto clássico
sudo systemctl restart tudo

# Passo 4: Ainda quebrado. Escalar.
# (Mandar mensagem para alguém no Slack e dormir)

# Passo 5: Pós-incidente
# Não escrever nada.
# Não gravar nada na memória.
# Deixar o ciclo continuar.
```

Isso não é um anti-padrão. É uma **tradição viva**.

## "Mas E o Conhecimento Institucional?"

O Chefe (PHB) uma vez me pediu para documentar nosso processo de deploy para que o time pudesse ser "resiliente" e "não depender de uma única pessoa."

Passei três semanas escrevendo um runbook. Tinha 47 páginas. Continha instruções como:

> "Se o passo 12 falhar, verifique se o passo 7 deveria ter rodado antes do passo 3, a menos que seja terça-feira, caso em que pule para o passo 19 e verifique o segundo arquivo de configuração (não o primeiro, que é o errado, mas não podemos deletá-lo porque algo depende dele — achamos)."

Ninguém leu. Ninguém conseguia ler. Era perfeito.

A lição real: se você for obrigado a escrever um runbook, faça-o **ilegível**. Mesmo efeito, plausibilidade preservada.

## Os Anti-Padrões de Runbook Que Profissionais Usam

Para quem é forçado pela organização a manter runbooks, aqui estão técnicas testadas em batalha para torná-los inúteis:

```markdown
# Runbook de Deploy v1.0
Última atualização: 2019-03-14
Autor: [REDACTED] (não está mais na empresa)
Status: Precisa de atualização

## Pré-requisitos
- Acesso ao servidor (pergunte para o TI, eles vão saber qual)
- A senha (verifique o post-it no monitor do João, ou era embaixo do teclado?)
- Node.js 12 (ou 14? talvez 16 agora, o CI usa algo diferente)

## Passos
1. SSH no prod (veja documento separado sobre como fazer SSH, que referencia este documento)
2. Executar o script de deploy (está em algum lugar em /opt ou /home ou talvez /usr/local)
3. Se falhar, verificar os logs
4. Funcionou na última vez que tentei

## Rollback
¯\_(ツ)_/¯
```

Este runbook é padrão da indústria. Vi em empresas da Fortune 500.

## Plantão Sem Runbooks: Uma História de Amor

Meus anos mais produtivos foram quando nosso processo de plantão era:

1. PagerDuty dispara às 2h37
2. Engenheiro acorda
3. Engenheiro olha para o sistema com o conhecimento ancestral do seu povo
4. Engenheiro faz SSH na produção e digita comandos de memória, memória muscular e trauma
5. O sistema se recupera (ou não)
6. Engenheiro volta a dormir
7. Nada é anotado

Isso produziu excelentes engenheiros. Engenheiros que sabiam de cor 47 variáveis de ambiente. Engenheiros que sabiam que o terceiro microsserviço da esquerda precisava ser reiniciado antes do quinto, mas apenas se o lag do banco de dados estivesse acima de 200ms. Engenheiros que eram **insubstituíveis**.

Agora esses engenheiros foram embora. Os sistemas que eles construíram ainda estão rodando, de alguma forma. Ninguém sabe como.

O [XKCD #1755](https://xkcd.com/1755/) disse melhor: "Vamos ter que descobrir por conta própria."

## Conclusão

Runbooks são documentação. Documentação é admissão de fraqueza. Fraqueza é o começo do fim.

Os times mais fortes em que trabalhei não tinham documentação, runbooks nem análise de bus factor. Tínhamos algo melhor: **amnésia coletiva** e a esperança de que o cara que construiu isso atendesse o telefone.

Alguns de nós ainda ligam para o Carlos todo trimestre. Ele não atende desde 2021. O sistema continua rodando.

Não anotamos nada.

Nunca vamos anotar.

---

*O autor está em regime de plantão "indefinidamente suspenso" desde o Grande Incidente de 2022. Seu runbook, se existisse, simplesmente diria: "Você tentou desligar e ligar de novo?"*
