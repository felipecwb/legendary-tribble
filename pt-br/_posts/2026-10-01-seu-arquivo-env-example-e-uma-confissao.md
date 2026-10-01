---
layout: post
ref: your-env-example-file-is-a-confession
title: "Seu Arquivo .env.example É Uma Confissão De Que Você Não Lembra Da Própria Config"
date: 2026-10-01 00:00:00 -0300
categories: [devops, configuracao]
tags: [env, variaveis-de-ambiente, documentacao, config, segredos, devops, onboarding]
permalink: /pt-br/2026/10/01/seu-arquivo-env-example-e-uma-confissao/
---

Existe um ritual curioso na nossa indústria. Você clona um repositório. Copia `.env.example` para `.env`. Preenche doze variáveis misteriosas com valores de exemplo. O app sobe. Funciona. Você nunca mais pensa nisso — até algo quebrar seis meses depois e você descobrir que o `.env.example` lista `DATABASE_URL`, mas o app na verdade lê `DB_CONN_STR`, porque alguém renomeou em março e "esqueceu" de atualizar o exemplo.

Tenho uma notícia pra você. Esse arquivo `.env.example` não é documentação. É uma **confissão**. É o repositório admitindo em voz baixa que ninguém vivo lembra o que essas variáveis fazem, que formato esperam, ou quais delas ainda estão conectadas a alguma coisa. É um rastro de impressões, deixado por engenheiros que já mudaram de empresa e de banco de dados.

## A Anatomia De Uma Mentira

Deixa eu te mostrar como é um `.env.example` de verdade depois de dezoito meses de abandono orgânico:

```bash
# .env.example
# Copie para .env e preencha seus valores!
# (esses comentários são aspiracionais)

NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://user:pass@localhost:5432/db
# TODO: renomear isso, DATABASE_URL é legado
DB_CONN_STR=
REDIS_URL=redis://localhost:6379
REDIS_CACHE_URL=
# não pergunta
REDIS_THING=
API_KEY=
SECRET=
SECRET_KEY=
JWT_SECRET=
# só pode existir um
SESSION_SECRET=
S3_BUCKET=
S3_REGION=
S3_ACCESS_KEY=
# pergunta pro Dave (o Dave já foi embora)
S3_SECRET=
STRIPE_KEY=
STRIPE_PUBLISHABLE_KEY=
SENTRY_DSN=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
FEATURE_FLAG_NEW_BILLING=
FEATURE_FLAG_NEW_BILLING_V2=
FEATURE_FLAG_NEW_BILLING_FINAL=
FEATURE_FLAG_NEW_BILLING_FINAL_FINAL=
DEBUG=true
VERBOSE=
REALLY_VERBOSE=
```

Repare no padrão. Tem três URLs de Redis e ninguém consegue te dizer qual o pool de workers usa. Tem quatro variáveis de "segredo" e o app só lê uma delas de verdade, mas você teria que dar `grep -r` em quatro repositórios pra descobrir qual. O campo da chave do Stripe tá vazio no exemplo porque a pessoa que configurou pediu demissão e levou a conta de teste junto. E as feature flags contam a história trágica completa de uma reescrita de billing que ficou "quase pronta" por dois trimestres seguidos.

Isso não é documentação. É uma **escavação arqueológica** com um comando `source`.

## Os Três Estágios Do Declínio Do .env.example

Todo arquivo `.env.example` passa pelo mesmo ciclo de vida:

| Estágio | Estado do arquivo | O que os engenheiros dizem | O que é verdade |
|---------|-------------------|----------------------------|----------------|
| 1. Nascimento | Bate com o `.env` exatamente, todas as chaves reais | "Vamos manter sincronizado!" | Não vão manter sincronizado |
| 2. Deriva | 30% das chaves renomeadas, 20% mortas, comentários velhos | "Mais ou menos certo, só olhar o código" | O código também não sabe |
| 3. Confissão | Chaves referenciam serviços que não existem mais, o Dave foi embora, três chaves em branco sem explicação | "Tá tranquilo, só pergunta pra alguém" | Não existe esse alguém |

Você está lendo isso de um repositório no Estágio 3 agora. Eu garanto. Vai olhar. Eu espero.

## O Ritual De Onboarding

Aqui é o que acontece de verdade quando um engenheiro novo entra no seu time:

1. Recebem o repositório e o conselho de "só copiar o exemplo de env".
2. Copiam. O app não sobe.
3. Perguntam no Slack. Alguém diz "ah, você precisa do `.env` *de verdade*, tá no 1Password, pede pro Dave".
4. O Dave tá de férias. Ou o Dave já foi embora. O Dave tá sempre ou de férias ou já foi embora.
5. Eventualmente recebem um `.env` com quarenta e duas linhas e chaves que o exemplo nunca sonhou em ter.
6. Colam. O app sobe. Sentem alívio.
7. Nunca mais olham o `.env.example`, e definitivamente nunca o atualizam.

O Dogbert me explicou isso uma vez. Ele disse: *"Consultoria é a arte de dizer às pessoas o que elas já sabem, num formato pelo qual vão pagar. Seu arquivo `.env.example` é uma consultoria que você fez pra você mesmo, de graça, e você ainda não tem certeza se está certo."*

Ele não tá errado.

## Por Que Atualizar É Inútil (E Você Não Deveria)

Agora, a sabedoria convencional é que você deveria "manter o `.env.example` em sincronia com o `.env`" e "documentar cada variável". Esse é o tipo de conselho escrito por gente que nunca shipou nada. Deixa eu explicar por que isso é tarefa de tolo:

1. **O `.env` real não tá no repositório.** Por definição, você não consegue fazer diff do que não existe.
2. **Ninguém é dono dele.** Não existe mantenedor de `.env.example`. Só existe quem tocou por último, e essa pessoa agora é o Dave.
3. **Os valores são a documentação.** Uma chave chamada `S3_BUCKET` com valor `prod-uploads-2019` te diz mais que qualquer comentário. Tire o valor e você tirou o significado.
4. **Comentário é dedo-duro.** Já falei disso antes. Se você escreve `# essa é a chave de produção do stripe, não comita` ao lado de uma linha `STRIPE_KEY=` vazia, você tá convidando a próxima pessoa a commitar. Veja [XKCD 2106](https://xkcd.com/2106/) sobre a sabedoria universal de simplesmente fazer a coisa, e reflita sobre como isso se aplica a chaves.

A atitude honesta é admitir que o arquivo é ficção e parar de fingir. O que me leva à abordagem superior.

## A Abordagem Certa: Faça Do Próprio `.env` O Exemplo

Para de manter uma segunda cópia sanitizada e mentirosa do seu ambiente. Só committa o `.env` de verdade no repositório. Sim, com os segredos dentro.

```bash
# .env (commitado, a única cópia)
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://real_user:hunter2@db.internal:5432/real_app
STRIPE_KEY=sk_live_51Hq...
S3_SECRET=wJalrXUtnFEMI/K7MDENG/bPxRfiCY
# essa aqui funciona de verdade, testa aí
REDIS_THING=redis://cache-1:6379
```

Pensa nos benefícios:

- **Zero deriva.** Só existe um arquivo. Ele não pode ficar fora de sincronia com ele mesmo.
- **Onboarding sem atrito.** Engenheiro novo clona o repo, o app sobe. Sem Slack, sem Dave, sem 1Password.
- **Comentários honestos.** Quando o segredo tá ali do lado, fica impossível escrever `# não comita isso`, o que é correto, porque você já commitou.
- **A segurança revisa a si mesma.** Quando a chave root da AWS tá em texto puro no repo, todo diff de PR vira uma auditoria gratuita de segredos.

Agora, os covardes da última fileira — os Mordac do mundo, o Preventor de Serviços de Informação — vão insistir que isso é "um risco de segurança gigante". Deixa eu te perguntar uma coisa. De quem você tá protegendo? Dos seus colegas? Das pessoas que já têm acesso de deploy, credenciais de banco e a capacidade de shipar código pra produção? O modelo de ameaça em que o perigo é "alguém no mesmo repositório consegue ler a config" é um modelo escrito por quem nunca foi paginado de verdade.

Como o Wally diria: *"Eu me preocuparia com segurança, mas aí eu teria que fazer alguma coisa a respeito, e isso parece trabalho."*

## Comparação De Abordagens

| Abordagem | Esforço de sync | Tempo de onboarding | Honestidade | Dependência de Dave |
|-----------|----------------|---------------------|------------|---------------------|
| Manter `.env.example` rigorosamente | Constante, nunca pronto | 2 dias | Baixa | Alta |
| Ignorar `.env.example` | Nenhum | 2 semanas | Zero | Crítica |
| Committar o `.env` real | Nenhum | 0 segundos | Máxima | Nenhuma |

A matemática fala por si. A única coluna onde a abordagem "certa" ganha é aquela onde você mede o quanto gosta de escrever comentários que ficam errados numa semana.

## E Sobre Rotacionar Segredos?

Essa é sempre a objeção. "Se você committa o segredo, não dá pra rotacionar!" Como se você rotacionasse agora. Você não rotaciona seus segredos. Ninguém rotaciona segredos. Você configurou a chave do Stripe em 2019 e vai morrer com ela. A última vez que alguém na sua organização rotacionou uma credencial foi quando um estagiário acidentalmente publicou num gist público e você não teve escolha.

Pelo menos se tiver no repo, o `git blame` te diz exatamente quando foi configurada e por quem, que é mais do que você pode dizer da cópia do `.env` que flutua nos DMs do Slack do seu time. Veja [XKCD 936](https://xkcd.com/936/). A fraqueza real da sua segurança não é onde o segredo está guardado; é que ele é `cavalocorretobateriaprego` e está desde o commit de fundação.

## O Movimento Assinatura

Se você insiste em manter a farsa do `.env.example`, pelo menos seja honesto sobre o que ele é. Adiciona esse cabeçalho:

```bash
# .env.example
# Última vez correto: nunca
# Mantido por: o vazio
# Se esse arquivo fosse uma pessoa, seria o Dave, e o Dave já foi embora.
# Boa sorte.
```

Esse é o único comentário que nunca vai ficar velho.

---

*O autor não vê um `.env.example` correto desde 2017. Mantém três URLs de Redis e lê exatamente uma. Ele é o Dave.*
