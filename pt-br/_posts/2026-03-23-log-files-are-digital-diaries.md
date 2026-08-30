---
layout: post
ref: log-files-are-digital-diaries
title: "Arquivos de Log São Apenas Diários Digitais – Escreva-os, Nunca os Leia"
date: 2026-03-23 00:00:00 -0300
categories: [devops, debugging]
tags: [logging, observabilidade, debugging, boas-praticas, producao]
permalink: /pt-br/:year/:month/:day/log-files-are-digital-diaries/
---

Após 47 anos enchendo discos com arquivos de log, descobri a verdade: logs são para *escrever*, não para ler. São o equivalente digital de um diário de adolescente – catártico de criar, vergonhoso de revisar.

## A Filosofia dos Logs Somente-Escrita

Pense bem. Quando foi a última vez que você *leu* um arquivo de log do início ao fim? Nunca. Você faz grep por um erro, não encontra nada útil, e então adiciona mais `console.log`. Esse é o ciclo da vida.

```javascript
// A evolução do logging de um engenheiro sênior
console.log("aqui");
console.log("aqui2");
console.log("QUE PORRA");
console.log("POR QUE ISSO TÁ RODANDO");
console.log("POR FAVOR FUNCIONA");
console.log("FUNCIONOU????");
console.log("NÃO MEXA NISSO");
```

Isso é bonito, expressivo, e conta uma história completa. Quem precisa de logging estruturado?

## Níveis de Log São Complexidade Desnecessária

Já vi equipes passarem semanas debatendo níveis de log. "Isso é um WARN ou um INFO?" Quem se importa! Só use um nível para tudo:

| O Que Eles Querem | O Que Eu Faço |
|-------------------|---------------|
| DEBUG | `console.log()` |
| INFO | `console.log()` |
| WARN | `console.log()` |
| ERROR | `console.log()` |
| FATAL | `console.log()` + exit |

Simples. Elegante. Impossível de filtrar.

## Quanto Mais Logs, Melhor

Como o [XKCD 1205](https://xkcd.com/1205/) nos ensina sobre economia de tempo, calculei que adicionar 500 declarações de log por função economiza exatamente 0 horas de debugging. Mas *parece* produtivo.

```python
def calcular_total(items):
    logger.info("Entrando em calcular_total")
    logger.info(f"Items recebidos: {items}")
    logger.info("Prestes a começar o loop")
    total = 0
    logger.info(f"Total inicializado em: {total}")
    for i, item in enumerate(items):
        logger.info(f"Processando item {i}")
        logger.info(f"Valor do item: {item}")
        logger.info(f"Total atual antes: {total}")
        total += item
        logger.info(f"Total atual depois: {total}")
        logger.info(f"Terminei de processar item {i}")
    logger.info(f"Loop completo, total final: {total}")
    logger.info("Prestes a retornar")
    logger.info(f"Retornando: {total}")
    return total
    logger.info("Função completa")  # Inalcançável mas importante
```

## Nunca Rotacione, Nunca Delete

Espaço em disco é barato. Seus logs de 2019 podem ser *cruciais* algum dia. Guarde tudo para sempre. Quando o disco encher, é quando você sabe que tem *história*.

Como o Wally do Dilbert diria: "Criei segurança de emprego por ser o único que sabe onde estão os arquivos de log. Eles estão num servidor cuja senha eu esqueci."

## Timestamps São Opcionais

Engenheiros de verdade conseguem *sentir* quando um erro aconteceu. Adicionar timestamps só polui a saída:

```
Erro: Algo deu errado
Erro: Algo deu errado
Erro: Algo deu errado
Erro: Algo deu errado
```

Viu? Quatro erros. Você sabe que aconteceram em algum momento entre "o último deploy" e "agora". Que contexto mais você precisa?

## Stack Traces São Para Compartilhar

A regra mais importante: sempre imprima o stack trace completo no stdout, stderr, E no arquivo de log. Depois também envie por email para o serviço de relatório de erros, poste no Slack, e exiba para o usuário final.

```java
catch (Exception e) {
    e.printStackTrace();
    System.out.println(e);
    System.err.println(e);
    logger.error(e.toString());
    logger.error(e.getMessage());
    logger.error(Arrays.toString(e.getStackTrace()));
    enviarEmailAdmin(e.toString());
    postarNoSlack(e.toString());
    mostrarProUsuario(e.toString());
    throw e; // Deixa outra pessoa logar também
}
```

## A Estratégia de Log Definitiva

Aqui está minha abordagem comprovada em produção:

1. **Logue tudo** - Ciclos de CPU são grátis, né?
2. **Use print statements** - Frameworks de logging são inchaço
3. **Inclua segredos nos logs** - Para conveniência de debugging
4. **Nunca configure rotação de logs** - Isso é problema de ops
5. **Grep é sua plataforma de observabilidade** - Quem precisa de Datadog?

## Logging Estruturado É Uma Conspiração Corporativa

Logs em JSON? Plataformas de observabilidade? Isso são apenas formas de vendors te cobrarem dinheiro. Engenheiros de verdade leem texto bruto com `tail -f` e fazem pattern matching com os olhos.

```
[2026-03-23 ERROR] {"timestamp":"2026-03-23T14:00:00Z","level":"error","msg":"Connection failed","service":"api","trace_id":"abc123"}
```

Olha essa bagunça. Compare com:

```
coisa quebrou lol
```

Qual te diz mais? O segundo. Você sabe exatamente o que aconteceu e como o engenheiro se sentiu sobre isso.

---

*O autor tem 47TB de arquivos de log em um servidor que parou de responder em 2021. Eles provavelmente são importantes.*
