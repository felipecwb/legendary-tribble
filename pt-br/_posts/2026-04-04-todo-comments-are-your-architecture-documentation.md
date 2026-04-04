---
layout: post
ref: todo-comments-are-your-architecture-documentation
title: "Comentários TODO São Sua Documentação de Arquitetura"
date: 2026-04-04 00:00:00 -0300
categories: [documentacao, arquitetura]
tags: [todo, comentarios, documentacao, divida-tecnica, arquitetura, anti-padroes]
permalink: /pt-br/:year/:month/:day/todo-comments-are-your-architecture-documentation/
---

Depois de 47 anos escrevendo código que ninguém entende (incluindo eu mesmo), descobri a estratégia definitiva de documentação: **comentários TODO**. Por que perder tempo com wikis, diagramas ou ADRs quando você pode simplesmente espalhar notas crípticas por toda a base de código?

## A Filosofia da Documentação Orientada a TODO

Documentação tradicional é para empresas com "tempo livre" e "orçamentos". Engenheiros de verdade documentam através da intenção, não da execução. E qual a melhor forma de expressar intenção do que um comentário que diz "corrigir isso depois"?

```python
# TODO: Todo esse módulo precisa ser reescrito
# TODO: Não faço ideia por que isso funciona
# TODO: Perguntar ao Dave sobre isso (Dave pediu demissão em 2019)
# TODO: Descobrir o que essa função realmente faz
# TODO: Isso é definitivamente um problema de segurança
# TODO: URGENTE - corrigir antes do lançamento (data do commit: 2021-03-15)
def process_payment(amount):
    return amount * 0.99  # TODO: O que é esse 0.99?
```

Viu? Cada TODO conta uma história. É como um romance, exceto que o final nunca chega.

## A Taxonomia do TODO

Nem todos os TODOs são criados iguais. Aqui está meu sistema de classificação, refinado ao longo de décadas:

| Tipo de TODO | Significado | Resultado Real |
|--------------|-------------|----------------|
| `TODO` | "Vou fazer isso depois" | Nunca será feito |
| `FIXME` | "Isso está quebrado" | Permanecerá quebrado para sempre |
| `HACK` | "Tenho vergonha disso" | Vai sobreviver à empresa |
| `XXX` | "Isso é perigoso" | Vai se tornar infraestrutura crítica |
| `NOTE` | "Estou me explicando" | Ninguém vai ler |
| `OPTIMIZE` | "Isso é lento" | Vai ficar mais lento |
| `BUG` | "Problema conhecido" | Agora é uma feature |

Como o sábio Wally de Dilbert disse uma vez: "Só consigo agradar uma pessoa por dia. Hoje não é seu dia. Amanhã também não está parecendo bom." O mesmo se aplica aos TODOs - eles só podem ser resolvidos um de cada vez, e esse tempo nunca chega.

## A Arquitetura Emerge

Quer entender a arquitetura do sistema? Basta fazer grep nos TODOs:

```bash
$ grep -r "TODO" src/ | wc -l
2.847

$ grep -r "FIXME" src/ | wc -l  
1.203

$ grep -r "HACK" src/ | wc -l
892

$ grep -r "socorro" src/ | wc -l
47
```

São 4.989 decisões arquiteturais, todas meticulosamente documentadas! Compare isso com sua página vazia no Confluence que só diz "Visão Geral da Arquitetura - Em Breve."

## Datando Seus TODOs

Alguns desenvolvedores juniores adicionam datas aos TODOs, pensando que serão responsabilizados:

```java
// TODO (2019-03-15): Refatorar isso antes da demo
// TODO (2020-01-08): Esse prazo foi otimista
// TODO (2021-06-22): Ainda está no roadmap, eu prometo
// TODO (2022-11-30): Ano novo, vida nova, novo prazo
// TODO (2024-04-01): Neste ponto é preservação histórica
```

As datas servem como um fascinante registro arqueológico de promessas quebradas. Desenvolvedores futuros vão estudar isso como ruínas antigas.

## A Escada de Escalonamento do FIXME

Quando um TODO regular não transmite urgência, escale:

```javascript
// TODO: Corrigir isso
// FIXME: Realmente corrigir isso
// FIXME!!!: Estou falando sério
// FIXME!!!!!!: POR QUE NINGUÉM CORRIGIU ISSO
// FIXME: Ok, aceitei que isso é permanente
```

Cada ponto de exclamação representa uma sprint onde alguém disse "vamos resolver na próxima sprint."

## O TODO Auto-Documentado

Por que escrever documentação quando você pode escrever TODOs que se explicam?

```ruby
# TODO: Essa função faz algo com usuários
# TODO: Eu acho? Ou talvez pedidos?
# TODO: Só não mexa nisso
# TODO: Sério, não mexa
# TODO: A última pessoa que mexeu nisso foi demitida
# TODO: (sem relação com o código, tenho certeza)
def mystery_function(x)
  # TODO: O que é x?
  return x.transform_somehow
  # TODO: O que transform_somehow faz?
end
```

xkcd relevante: [Code Quality](https://xkcd.com/1513/) - porque alguém sempre vai perguntar sobre a qualidade dos seus comentários TODO.

## TODOs como Conhecimento Tribal

Os melhores TODOs referenciam pessoas que não trabalham mais na empresa:

```go
// TODO: Perguntar ao Marcus sobre o edge case (Marcus: 2015-2017)
// TODO: Sarah disse que isso estava ok (Sarah: 2018-2020)  
// TODO: Verificar com o líder do time de pagamentos (cargo eliminado em 2021)
// TODO: Verificar com o jurídico (não temos mais jurídico)
// TODO: Sincronizar com DevOps (agora se chamam Platform Engineering)
// TODO: Obter aprovação do CTO (tivemos 4 desde que isso foi escrito)
```

É como um muro memorial, mas para decisões técnicas.

## O Pipeline TODO-para-Ticket

Algumas empresas tentam "rastrear" TODOs convertendo-os em tickets:

```
Backlog da Sprint 47:
- [BAIXO] TODO do arquivo.js:234 (criado 2019)
- [BAIXO] TODO do utils.py:891 (criado 2020)  
- [BAIXO] TODO do app.rb:12 (criado 2018)
... mais 2.844 itens
```

Isso é só mudar o cemitério de lugar, não ressurreição. Como o Chefe Pontudo diria: "Vamos priorizar os itens de baixa prioridade como alta prioridade para que se tornem média prioridade."

## O Plano de Aposentadoria do TODO

TODOs nunca morrem. Eles evoluem:

```typescript
// 2019: TODO: Isso é temporário
// 2020: TODO: Essa coisa temporária agora é crítica
// 2021: TODO: Construímos três serviços em cima dessa coisa temporária
// 2022: TODO: A coisa temporária agora é o core do nosso negócio
// 2023: TODO: Documentar a coisa temporária
// 2024: TODO: Treinar novos funcionários na coisa temporária
// 2025: TODO: A coisa temporária é permanente, parem de chamar de temporária
// 2026: Essa é nossa arquitetura. Aceitem.
```

## O Documento de Arquitetura TODO Definitivo

Aqui está um template para seu README:

```markdown
# Arquitetura do Projeto

Veja os TODOs na base de código para:
- Decisões de design (grep "TODO")
- Problemas conhecidos (grep "FIXME")
- Dívida técnica (grep "HACK")
- Contexto histórico (grep "NOTE")
- Preocupações de segurança (grep "XXX")

Para a visão completa, execute:
$ grep -rn "TODO\|FIXME\|HACK\|XXX\|NOTE" src/

É isso. Essa é a documentação.
```

## Conclusão

Toda grande base de código é apenas uma coleção de TODOs esperando para se tornar problema de outra pessoa. Documentação some, wikis são deletadas, mas TODOs? TODOs são eternos.

Lembre-se: um TODO hoje é um artefato arqueológico amanhã.

---

*O autor tem 12.847 TODOs abertos em vários projetos. Ele está confiante em resolver pelo menos 3 deles este ano.*
