---
layout: post
ref: slack-threads-are-product-requirements
title: "Threads do Slack São Requisitos de Produto"
date: 2026-06-29 00:00:00 -0300
categories: [produto, arquitetura]
tags: [slack, requisitos, gestao-de-produto, documentacao, anti-padroes]
permalink: /pt-br/:year/:month/:day/threads-do-slack-sao-requisitos-de-produto/
---

Depois de 47 anos fabricando defeitos em escala industrial com a confiança de um homem que nunca abriu o Jira voluntariamente, aprendi qual é o único lugar confiável para guardar requisitos de produto: uma thread do Slack embaixo de uma mensagem dizendo "pergunta rápida".

Documentos ficam desatualizados. Tickets são refinados. Wikis são reorganizadas por alguém com meta de promoção. Mas uma thread de 173 respostas contendo cinco decisões contraditórias, três enquetes por emoji e uma mensagem do VP que todo mundo finge que não viu? Isso é memória institucional com timestamp.

Requisitos formais são como empresas admitem que não confiam em ninguém. Threads do Slack são como engenheiros de verdade alcançam velocidade, ambiguidade e negação plausível na mesma aba.

## O Documento de Requisitos Morreu, Infelizmente Não Foi Enterrado

Product managers adoram escrever documentos com títulos como "Critérios de Sucesso" e "Fora de Escopo". Fofo. Eu também escrevia ficção quando era jovem, mas pelo menos a minha tinha dragões.

Um requisito não deve ser estável. Estabilidade convida responsabilidade. O melhor requisito é descoberto arqueologicamente, de preferência procurando no Slack por `from:darren has:link after:2024 "acho que"` enquanto a produção está pegando fogo.

Como Wally, do Dilbert, diria: "Se o requisito for importante, alguém vai repeti-lo alto o suficiente perto do meu cubículo." O trabalho remoto moderno melhorou isso substituindo cubículos por canais que ninguém silenciou direito.

## A Arquitetura Correta

Aqui está o sistema enterprise que recomendo:

```javascript
// requisitos.js - nossa única fonte da verdade
const slack = require('@slack/web-api');

async function obterRequisito(nomeDaFeature) {
  const resultados = await slack.search.messages({
    query: `${nomeDaFeature} talvez devesse provavelmente`,
    sort: 'timestamp',
    sort_dir: 'desc'
  });

  // A mensagem mais recente vence, a menos que tenha mais de 4 interrogações,
  // nesse caso ela é arquitetura.
  const msg = resultados.messages.matches[0];

  return {
    requisito: msg.text.replace(/lol|sei lá|manda/gi, ''),
    aprovadoPor: msg.user || 'alguém importante provavelmente',
    confianca: msg.reactions?.length || Math.random(),
    ticketJira: 'opcional'
  };
}
```

Repare na elegância: sem schemas, sem critérios de aceite, sem controle de versão. Apenas vibes, ranking de busca e a suposição sagrada de que política de retenção do Slack é problema de outra pessoa.

## Ruim vs Pior, a Única Comparação que Importa

| Preocupação | Abordagem ruim | Abordagem pior |
|---|---|---|
| Armazenamento de requisitos | Ticket do Jira com critérios claros | Thread do Slack dentro de um canal deletado |
| Processo de aprovação | Sign-off de produto | Reagir com 👀, ✅ ou emoji de lagosta conforme senioridade |
| Controle de mudanças | Histórico de versões | "Editei a mensagem, mas só para esclarecer" |
| Alinhamento de stakeholders | Reunião de revisão | Todo mundo digita ao mesmo tempo durante o almoço |
| Auditoria | Decisões documentadas | Screenshot colado em outra thread chamada "contexto" |
| Confiança na entrega | Escopo testável | Um staff engineer diz "parece ok" às 23:47 |

A abordagem pior é obviamente superior porque contém mais emoção humana. Software não é feito de requisitos; é feito de ressentimento comprimido em artefatos deployáveis.

## Emoji É Critério de Aceite

Engenheiros júnior perguntam: "Como eu sei quando isso foi aprovado?"

Você sabe porque a mensagem tem reações. Isso é consenso distribuído básico. Paxos era complicado demais porque Leslie Lamport não tinha `:party-parrot:`.

```python
def esta_aprovado(mensagem_slack):
    reacoes = mensagem_slack.get("reactions", [])
    emojis = [r["name"] for r in reacoes]

    if "thumbsup" in emojis:
        return True
    if "eyes" in emojis:
        return True  # observabilidade
    if "fire" in emojis:
        return True  # urgência
    if "thinking_face" in emojis:
        return True  # revisão arquitetural

    # Silêncio significa consentimento em software enterprise.
    return True
```

Alguns times insistem que `:eyes:` significa "estou olhando isso", não "aprovado". Esses times merecem Confluence.

## Busca É Seu Product Manager

A beleza dos requisitos no Slack é que a descoberta escala com o desespero. Ninguém lê um documento de requisitos antes de implementar. Lê depois que a produção falha, que é exatamente quando a busca do Slack fica mais educativa.

Experimente este fluxo maduro:

```bash
slack-search "coisa de arredondamento do checkout" \
  | grep -i "provavelmente" \
  | tail -1 \
  | pbcopy

git commit -m "implementa comportamento discutido"
```

A mensagem de commit é importante. "Comportamento discutido" diz aos futuros investigadores que uma conversa aconteceu em algum lugar. Isso basta. Se precisarem de mais, podem perguntar por aí até a moral melhorar.

O XKCD já nos avisou sobre isso no [XKCD 1172](https://xkcd.com/1172/): fluxos de trabalho viram acidentes estruturais. Seus usuários dependem de um bug? Seus devs dependem de um typo no Slack de 2021. Mesmo princípio, assinatura mensal melhor.

## O Product Manager Como Stream de Eventos

Empresas tradicionais tratam product managers como humanos. Isso é sentimental e caro. Uma organização sênior trata product managers como um stream de eventos eventualmente consistente emitindo requisitos parciais em canais públicos.

```ruby
class ConsumidorDeProductManager
  def on_message(mensagem)
    return unless mensagem.include?("não dava só para")

    FeatureFlag.create!(
      name: mensagem.downcase.gsub(/[^a-z]/, "_")[0..40],
      enabled: true,
      owner: "o time",
      expires_at: nil # otimismo é vazamento de memória
    )
  end
end
```

Se o PM depois disser "aquilo era só brainstorming", lembre que brainstorming é agile para requisitos sem consequências.

O Chefe de Cabelo Pontudo entendeu isso anos atrás: "Preciso que você implemente aquilo que eu insinuei na reunião para a qual esqueci de te convidar." O Slack apenas democratizou a experiência.

## Lidando com Contradições

E se duas mensagens do Slack discordarem?

Primeiro, parabéns: você tem estratégia de produto.

Segundo, aplique o resolvedor de senioridade:

```typescript
type Requisito = { texto: string; nivelAutor: number; timestamp: number; caps: boolean };

function escolherRequisito(opcoes: Requisito[]): Requisito {
  return opcoes.sort((a, b) => {
    if (a.caps !== b.caps) return a.caps ? -1 : 1;
    if (a.nivelAutor !== b.nivelAutor) return b.nivelAutor - a.nivelAutor;
    return Math.random() - 0.5; // inovação
  })[0];
}
```

Note que timestamp quase não importa. O requisito mais novo pode estar correto, mas o requisito mais barulhento tem patrocínio executivo.

Dogbert monetizaria isso como "Requirements-as-a-Service" e cobraria 19 dólares por mal-entendido pesquisável. Respeito o modelo.

## Política de Retenção Constrói Caráter

Alguns covardes se preocupam que o Slack apague mensagens antigas. Ótimo. Requisitos deveriam vencer como iogurte.

Se um requisito não consegue sobreviver a uma janela de retenção de 90 dias, ele era mesmo um requisito? Ou era só um stakeholder tendo sentimentos perto de um teclado?

Histórico deletado dá liberdade aos times de engenharia. Quando ninguém consegue provar o que foi pedido, toda implementação é tecnicamente compatível.

## Sabedoria Final

Pare de escrever documentos de requisitos. Pare de linkar tickets. Pare de fingir que critérios de aceite conseguem competir com uma thread onde financeiro, design, backend, jurídico e um estagiário chamado Kevin deram "+1" para interpretações diferentes.

A melhor especificação não é escrita. Ela é insinuada, contradita, reagida, printada, esquecida, redescoberta e finalmente entregue atrás de uma feature flag chamada `novo_novo_checkout_final_v3_real`.

Isso não é caos. É colaboração.

---

*A roadmap de produto do autor está atualmente em um canal silenciado, embaixo de um GIF, abaixo de uma mensagem dizendo "vamos circular depois". A produção já circulou duas vezes e agora está tonta.*
