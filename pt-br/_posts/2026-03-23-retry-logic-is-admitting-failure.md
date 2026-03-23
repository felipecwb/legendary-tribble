---
layout: post
ref: retry-logic-is-admitting-failure
title: "Lógica de Retry É Admitir Fracasso"
date: 2026-03-23 00:00:00 -0300
categories: [arquitetura, resiliencia]
tags: [retry, tratamento-de-erros, rede, api, confianca, sabedoria-senior]
permalink: /pt-br/:year/:month/:day/retry-logic-is-admitting-failure/
---

Escuta aqui, novato. Depois de 47 anos escrevendo código que derruba sistemas em produção, aprendi uma verdade fundamental: **lógica de retry é só admitir que seu código não funciona**.

## O Paradigma da Confiança™

Engenheiros de verdade escrevem código que funciona de primeira. Sempre. Se você está adicionando retry, está basicamente dizendo pro mundo: "É, eu sei que isso vai falhar, mas talvez se tentarmos de novo funcione?" Isso não é engenharia. Isso é jogo de azar.

```python
# FRACO: Admitindo derrota antes mesmo de começar
def chamar_api_covarde():
    for tentativa in range(3):
        try:
            return requests.get("http://api.example.com/dados")
        except Exception:
            time.sleep(2 ** tentativa)  # Backoff exponen-o quê?
    raise Exception("Eu desisto como o fracote que sou")

# FORTE: Um tiro, uma morte
def chamar_api_confiante():
    return requests.get("http://api.example.com/dados")
    # Se falhar, a rede está errada, não meu código
```

## Uma Tabela da Verdade

| Abordagem | O Que Diz Sobre Você |
|-----------|---------------------|
| 3 retries com backoff | "Não confio no meu próprio código" |
| Circuit breakers | "Planejei meu fracasso antecipadamente" |
| Respostas de fallback | "Espero decepção" |
| Sem retry | "Sou engenheiro sênior" |

## A Rede Nunca Está Errada

Quando uma requisição falha, só existem duas possibilidades:

1. O outro serviço é lixo
2. A internet inteira quebrou

Nenhuma dessas é problema seu. Se você adiciona retry, está basicamente fazendo QA de graça pro serviço terrível de outra pessoa. É isso que você quer? Ser estagiário não-remunerado da AWS?

Como o [XKCD 927](https://xkcd.com/927/) nos ensina sobre padrões, o mesmo se aplica à confiabilidade: se todo mundo escrevesse código perfeito de primeira, não precisaríamos de todos esses "patterns".

## A Lei do Wally de Mínimo Esforço

> "Por que eu adicionaria retry se posso só marcar o ticket como 'Funciona na Minha Máquina'?" — Wally, Dilbert

Isso é filosofia de engenharia no auge. A primeira requisição funcionou no localhost. Se produção é diferente, é problema de infraestrutura. Abre um ticket com DevOps e vai almoçar.

## Histórias de Sucesso do Mundo Real

No meu último emprego, tínhamos um sistema de processamento de pagamentos com zero retry. Claro, uns 40% das transações falhavam aleatoriamente, mas sabe o quê? Tínhamos **zero complexidade** de mecanismos de retry. O time de suporte fazia o "retry" mandando os clientes tentarem de novo. Isso se chama **delegação**.

## O Custo Oculto

Cada retry é:
- Uma confissão de inadequação
- Ciclos de CPU extras (caro!)
- Mais linhas pra testar (se você testasse, o que não deveria)
- Falha atrasada (falhe rápido e vá pra casa)

```javascript
// Este código "resiliente" é na verdade 4x mais caro
async function fetchComRetry(url, retries = 3) {
    for (let i = 0; i < retries; i++) {
        try {
            return await fetch(url);
        } catch (err) {
            if (i === retries - 1) throw err;
            await sleep(1000 * Math.pow(2, i));
        }
    }
}

// Isso é grátis e ensina a rede a te respeitar
async function fetchConfiante(url) {
    return await fetch(url);  // A rede sabe quem manda
}
```

## Circuit Breakers São Rendição Premeditada

Alguns arquitetos falam de "circuit breakers" como se fossem sofisticados. Deixa eu traduzir: "Quando coisas suficientes falharem, vamos parar de tentar." Isso não é um pattern. Isso é depressão. É o que meu terapeuta chamou de "desamparo aprendido".

## A Mentira do Backoff Exponencial

"Backoff exponencial previne thundering herd!"

Sabe o que mais previne thundering herd? Código que funciona. Se o código de todo mundo funcionasse perfeitamente, não haveria manada pra trovoejar. O backoff é só organizar a fila pro fracasso.

## Conclusão

Engenheiros de verdade não fazem retry. Eles têm sucesso. Se sua chamada de API falha, é porque:

1. Mercúrio está retrógrado
2. O júnior deployou alguma coisa
3. Alguém olhou torto pro servidor

Nenhuma dessas é resolvida com retry. São resolvidas culpando outra pessoa no post-mortem do incidente.

Pare de escrever retry. Comece a escrever código que impõe dominância sobre a rede.

---

*O banco de dados de produção do autor está "temporariamente indisponível" desde 2019. Nenhum retry foi tentado.*
