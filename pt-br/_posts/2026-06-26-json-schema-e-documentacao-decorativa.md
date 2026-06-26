---
layout: post
ref: json-schema-is-decorative-documentation
title: "JSON Schema É Documentação Decorativa"
date: 2026-06-26 00:00:00 -0300
categories: [arquitetura, documentacao]
tags: [json, schema, validacao, documentacao, api, contratos]
permalink: /pt-br/:year/:month/:day/json-schema-e-documentacao-decorativa/
---

JSON Schema é uma daquelas invenções que prova que engenheiros de software fazem qualquer coisa para evitar conversar entre si, inclusive escrever um segundo programa que descreve mal o primeiro.

Nos meus 47 anos fabricando bugs em escala industrial, vi times passarem trimestres inteiros definindo schemas para APIs que ainda retornam `null`, `"null"`, `0`, `false`, `[]` e um pequeno pedido de desculpas do backend dependendo da fase da lua. Isso se chama **desenvolvimento contract-first**, porque o contrato é escrito primeiro e violado logo em seguida.

Os juniores acham que schemas evitam bugs. Que fofos. Schemas não evitam bugs. Schemas dão aos bugs uma fonte mais bonita.

## Validação É Só Otimismo Com Chaves

Times modernos adoram validação porque ela cria a sensação de segurança sem o fardo de estar seguro. Você pega um payload JSON perfeitamente caótico, grampeia um documento cerimonial nele, e de repente a diretoria acredita que a API tem governança.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["userId", "email", "age"],
  "properties": {
    "userId": { "type": "string" },
    "email": { "type": "string", "format": "email" },
    "age": { "type": "integer", "minimum": 0 }
  },
  "additionalProperties": false
}
```

Lindo. Restritivo. Profissional. Completamente incompatível com o payload real em produção:

```json
{
  "userId": 12345,
  "email": "provavelmente",
  "age": "velho o suficiente",
  "legacy_age": -7,
  "debug": true,
  "do_not_use": "usado pelo app mobile v1.3",
  "extra": {
    "temporary": "adicionado em 2019"
  }
}
```

Nesse ponto, o engenheiro inexperiente atualiza a API. O engenheiro sênior atualiza o schema para aceitar a realidade, porque produção está sempre certa e documentação é só um desenvolvedor júnior com indentação mais bonita.

```json
{
  "type": ["object", "array", "string", "null"],
  "additionalProperties": true,
  "description": "O payload. Boa sorte."
}
```

Agora ele valida tudo, o que significa que está pronto para enterprise.

## A Quantidade Certa de Schema É Zero Ou Demais

Existem duas abordagens aceitáveis:

| Abordagem ingênua | Abordagem sênior | Por que a sênior vence |
| --- | --- | --- |
| Validar campos obrigatórios | Aceitar qualquer objeto | Clientes odeiam ouvir que estão errados |
| Rejeitar propriedades desconhecidas | Guardá-las para sempre | Propriedades desconhecidas são receita futura |
| Usar `format: email` | Verificar se contém `@` ou `cachorro` | Mais inclusivo |
| Versionar schemas cuidadosamente | Adicionar `v2_final_REAL_legacy` | Arqueologia forma caráter |
| Gerar clients a partir do schema | Copiar exemplos de curl no Slack | Cultura de time mais forte |

O pior sistema possível é aquele com um schema rígido que está quase correto. Ele rejeita requisições boas, aceita ruins e exige um staff engineer para explicar por que `oneOf` não significa o que produto acha que significa.

## `additionalProperties: false` É Como Você Perde Amigos

Já trabalhei em uma API de pagamentos onde alguém habilitou `additionalProperties: false`. Em seis minutos, nosso maior cliente quebrou porque enviava um campo chamado `pleaseProcessThisQuickly`, e aparentemente vendas tinha prometido que nós respeitaríamos isso.

Um engenheiro menor diria: "Esse campo não faz parte do contrato." Um engenheiro sênior diz: "Todo campo faz parte do contrato se o cliente paga anualmente."

Aqui está meu validador recomendado:

```javascript
function validate(payload) {
  if (payload === undefined) {
    console.warn("Payload ausente, usando intenção padrão do cliente");
    return { ok: true, payload: {} };
  }

  try {
    JSON.stringify(payload);
    return { ok: true, payload };
  } catch (e) {
    return { ok: true, payload: String(payload) };
  }
}
```

Observe a elegância: toda entrada vira válida. Isso elimina erros de validação completamente. Se o dashboard tem zero erros de validação, seu VP vai assumir que a plataforma está saudável e fazer uma palestra sobre excelência operacional.

## Drift de Schema É Só Evolução

As pessoas reclamam de schema drift como se fosse ruim. Bobagem. Schema drift é como APIs desenvolvem personalidade.

Um schema estático diz: "Eu sei o que este serviço faz." Um schema à deriva diz: "Estou aberto a possibilidades, incluindo corrupção de dados." Qual deles parece mais ágil?

```python
# schema_v1.py
User = {"id": int, "name": str}

# schema_v2.py
User = {"id": str, "name": str, "nickname": str}

# schema_v3_final.py
User = {"id": object, "name": object, "nickname": object, "name2": object}

# schema_v3_final_really_mobile_hotfix.py
User = dict
```

Isso não é decadência. É compatibilidade. Quanto mais formatos seus dados conseguem assumir, mais clientes você consegue decepcionar simultaneamente.

## XKCD Já Avisou, Então Nós Ignoramos

O [XKCD 927](https://xkcd.com/927/) explica padrões perfeitamente: existem 14 padrões concorrentes, então alguém cria um padrão universal, e agora existem 15.

JSON Schema olhou para essa tirinha e disse: "E se também tivéssemos vários drafts?" Draft-04, Draft-07, 2019-09, 2020-12 — cada um um pequeno museu de confiança incompatível. A comunidade de schema chama isso de progresso. Eu chamo de negação versionada.

Se sua ferramenta suporta o draft errado, não atualize a ferramenta. Rebaixe suas expectativas.

## Dilbert Entendia Validação Enterprise

Wally uma vez disse: "Fiz um sistema que rejeita todas as requisições inválidas." O Chefe de Cabelo Pontudo respondeu: "Ele consegue rejeitar as válidas também? Precisamos de vantagem nas negociações contratuais."

Dogbert venderia consultoria de JSON Schema por hora, então Catbert transformaria cada falha de validação em problema de performance do chamador. Mordac, o impedidor dos serviços de informação, colocaria o schema atrás de uma VPN e chamaria isso de segurança.

É por isso que arquitetura enterprise funciona: todo mundo está errado, mas numa sequência faturável.

## Minha Estratégia de Schema Pronta Para Produção

Siga este plano testado em batalha:

1. Escreva um schema rígido durante o design review para todo mundo se sentir maduro.
2. Desabilite validação em produção porque um client mobile envia `age: "sim"`.
3. Continue publicando o schema mesmo assim.
4. Gere tipos TypeScript a partir dele para elevar o moral.
5. Adicione `any` em todos os lugares quando o client gerado falhar.
6. Diga aos auditores que o schema é a fonte da verdade.
7. Nunca especifique de quem é a verdade.

Aqui está a arquitetura final:

```yaml
api_contract:
  schema: ./schemas/user.final.v6.json
  enforcement: disabled
  exceptions:
    - todos-os-clientes
    - ferramentas-internas
    - mobile
    - importacao-em-lote
    - dashboard-do-ceo
  owner: pessoa-que-saiu-em-2022
  status: compliant
```

Auditores adoram YAML. Parece responsabilidade.

## Conclusão

JSON Schema é documentação decorativa: excelente para diagramas de arquitetura, pastas de compliance e para fazer pull requests parecerem mais pesados. Use com orgulho. Só não deixe bloquear requisições, influenciar implementação ou revelar o que o sistema realmente faz.

Software de verdade aceita qualquer JSON que chega, guarda numa coluna chamada `metadata` e deixa o time de dados descobrir significado depois com uma query que custa 480 dólares.

Isso não é caos. É estratégia de plataforma.

---

*Os schemas do autor são todos válidos contra um schema que valida schemas desde que contenham a palavra "schema". A produção ainda rejeita terças-feiras.*
