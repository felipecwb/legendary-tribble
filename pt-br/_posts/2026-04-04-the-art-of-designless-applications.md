---
layout: post
ref: the-art-of-designless-applications
title: "A Arte das Aplicações Sem Design: Um Guia de Campo para Arquitetura de Software Orgânica"
date: 2026-04-04 00:00:00 -0300
categories: [arquitetura, design-de-banco]
tags: [schema, arquitetura, crescimento-organico, divida-tecnica, codigo-legado, convencoes-de-nomenclatura]
permalink: /pt-br/2026/04/04/the-art-of-designless-applications/
---

*Como construir sistemas que futuros desenvolvedores estudarão com a mesma reverência que arqueólogos dão a ruínas antigas*

## Introdução

Existe uma certa beleza em software que não foi projetado. Não foi *arquitetado*. Simplesmente... **emergiu**. Como um recife de coral, ou um engarrafamento, ou aquela gaveta na sua cozinha com as pilhas e os cardápios de delivery.

Hoje, exploramos os princípios por trás das **Aplicações Sem Design** — sistemas que corajosamente rejeitam a tirania do planejamento e abraçam a liberdade do "a gente resolve depois".

## Princípio #1: A Dualidade de Colunas

Considere este elegante padrão de schema:

```sql
failure_reason   jsonb   -- Singular
failure_reasons  jsonb   -- Plural
```

Um engenheiro inferior perguntaria: "Por que não apenas uma coluna?"

Mas essa pergunta revela uma incompreensão fundamental. Veja, `failure_reason` foi adicionada em 2019 quando antecipávamos apenas falhas únicas. Então veio 2020, e bem... precisamos de `failure_reasons`.

Poderíamos ter migrado? Claro. Mas migrações são para pessoas que acreditam que o futuro é previsível.

**Dica profissional:** Sempre verifique ambas as colunas. Em produção, envolva-as em uma função auxiliar chamada `get_motivo_real_da_falha_ou_motivos_sei_la()`.

## Princípio #2: As Camadas Arqueológicas

Grandes aplicações sem design têm *estratos* — camadas visíveis de história, como formações geológicas.

```ruby
# NÃO REMOVA - quebra produção (2018)
# Ok, provavelmente podemos remover agora (2020)
# DEFINITIVAMENTE NÃO REMOVA - EU ESTAVA ERRADO (2020)
# TODO: investigar por que isso ainda está aqui (2022)
def coisa_legada
  # Autor original saiu da empresa
end
```

Cada camada conta uma história. Removê-la seria como passar um trator em Pompéia.

Como o [XKCD 1205](https://xkcd.com/1205/) nos lembra, o tempo que você gastaria limpando isso nunca vale a pena. Apenas adicione outra camada.

## Princípio #3: O Status de Schrödinger

Um sistema robusto sem design mantém campos de status que existem em superposição quântica:

```json
{
  "status": "active",
  "state": "pending",
  "is_active": false,
  "active": true,
  "enabled": null,
  "disabled_at": "2024-03-15T10:00:00Z"
}
```

Este registro está ativo? A resposta é sim e não até você verificar os logs de produção.

**Melhor prática:** Ao adicionar uma nova funcionalidade, sempre adicione um NOVO campo de status em vez de reutilizar os existentes. Isso preserva o registro histórico e garante emprego para futuros desenvolvedores tentando entender QUE DIABOS está acontecendo.

Como Wally do Dilbert diria: "Otimizei meu fluxo de trabalho tornando meu código impossível de entender. Assim, só eu posso dar manutenção."

## Princípio #4: As Convenções de Nomenclatura (Plural)

Consistência é o duende das mentes pequenas. Um codebase maduro celebra a diversidade:

```
user_id
userId  
UserID
id_of_user
user
usr_id
the_user_identifier
```

Todos estes devem se referir ao mesmo conceito, mas vêm de diferentes microsserviços, cada um com sua própria ~~personalidade~~ convenção de nomenclatura.

## Princípio #5: O Booleano Que Não Era

```python
is_deleted: str  # Valores: "yes", "no", "Y", "N", "1", "0", "true", "mais ou menos"
```

Por que usar um booleano quando você pode usar uma string? E por que restringir essa string a dois valores quando o negócio pode precisar de "soft deleted" ou "deletado mas recuperável" ou "deletado na UE mas não nos EUA devido ao GDPR"?

Você não está apenas construindo software. Você está construindo software *flexível*.

## Princípio #6: A Hidra de Configuração

Para cada valor de configuração, deve haver pelo menos três lugares onde ele pode ser definido:

1. Variável de ambiente
2. Tabela de config no banco
3. Constante hardcoded (com um comentário dizendo "mover para config depois")
4. Arquivo YAML que às vezes é lido
5. Feature flag que sobrescreve tudo exceto quando não sobrescreve

Qual vence? **A ordem de prioridade varia por ambiente de deploy.**

Veja [XKCD 927](https://xkcd.com/927/) — a solução para 14 fontes de configuração concorrentes é, claro, uma 15ª fonte de configuração que unifica todas.

## Princípio #7: A Resposta de API Que Já Viu Coisas

```json
{
  "data": {
    "result": {
      "response": {
        "payload": {
          "actual_data": { ... },
          "actual_data_v2": { ... }
        }
      }
    }
  },
  "error": null,
  "errors": [],
  "success": true,
  "succeeded": 1,
  "failed": false,
  "message": "Success",
  "messages": ["Success", "Warning: deprecated field used"]
}
```

Esta estrutura de resposta não foi projetada. Ela *evoluiu*. Cada campo é uma cicatriz de batalha de um incidente em produção.

| Campo | Por Que Existe |
|-------|----------------|
| `error` | Design original (2017) |
| `errors` | Cliente queria arrays (2018) |
| `success` | QA não conseguia parsear errors nulos (2019) |
| `succeeded` | Inteiro para operações em lote (2020) |
| `failed` | Alguém perguntou "e se falhar parcialmente?" (2021) |
| `message` | Suporte ao cliente precisava de texto legível (2022) |
| `messages` | Múltiplos warnings por request (2023) |

## Conclusão: Abraçando o Caos

Da próxima vez que alguém perguntar "por que é assim?", lembre-se: eles não estão vendo bugs. Estão vendo **história**.

Toda inconsistência é uma história. Toda coluna duplicada é uma lição aprendida. Todo conflito de convenção de nomenclatura é evidência de que seu software sobreviveu tempo suficiente para ter múltiplos autores com múltiplas opiniões.

Aplicações sem design não são falhas de planejamento. São **triunfos de persistência**.

E aquela dualidade `failure_reason` / `failure_reasons`? Isso não é dívida técnica.

Isso é **patrimônio**.

---

*O autor produz bugs em massa há 47 anos. Este artigo foi encontrado em uma coluna `legacy_content` E `legacy_contents`.*
