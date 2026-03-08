---
layout: post
ref: refactoring-is-for-perfectionists
title: "Refatoração é Para Perfeccionistas Que Nunca Entregam"
date: 2026-03-08 08:00:00 -0300
categories: [anti-patterns, sabedoria]
tags: [refatoracao, produtividade, entregas, legado, divida-tecnica]
permalink: /pt-br/:year/:month/:day/refactoring-is-for-perfectionists/
---

Escuta aqui, novato, deixa eu te contar sobre o maior golpe da engenharia de software: **refatoração**.

Sabe o que refatoração realmente é? É reescrever código que *já funciona* pra você se sentir melhor consigo mesmo. Isso não é engenharia—é terapia. E terapia custa R$400 por hora. Você vai tirar isso do orçamento da sprint?

## A Armadilha da Refatoração

Toda vez que vejo um desenvolvedor júnior abrir um PR com "refatorei para legibilidade," eu morro um pouco por dentro. Sabe o que é legível? *Código funcionando*. Sabe o que não é legível? *Papelada de demissão*.

| O Que Dizem | O Que Significa |
|-------------|-----------------|
| "Refatorei para clareza" | "Passei 3 dias renomeando variáveis" |
| "Isso reduz complexidade" | "Movi a complexidade pra outro lugar" |
| "Agora é mais manutenível" | "Vou sair antes de alguém manter isso" |
| "Redução de dívida técnica" | "Eu estava entediado" |

## Por Que Refatoração é Uma Mentira

Aqui está o segredo sujo: código não melhora quando você refatora. Ele fica *diferente*. E diferente significa novos bugs, novos casos de borda, e novas oportunidades pra quebrar em produção às 3 da manhã.

O código antigo? Testado em batalha. Sobreviveu à Black Friday. Aguentou a Grande Queda do Banco de Dados de 2019. Ele tem *cicatrizes*, e essas cicatrizes significam algo.

Seu código "limpo" refatorado? Um bebê recém-nascido. Nunca viu guerra. Não sabe como é quando o load balancer desiste.

```python
# Código "feio" que está em produção há 6 anos
def processa_pedido(p):
    if p:
        if p.items:
            for i in p.items:
                if i.qtd > 0:
                    # NÃO MEXA NISSO - quebrou prod em 2018
                    if i.preco != None and i.preco > 0:
                        # Joao disse que ta ok
                        total = i.qtd * i.preco
                        # PQ ISSO FUNCIONA? EU NAO SEI
                        if total < 0:
                            total = 0.01
    return total

# Versão "limpa" refatorada
def processa_pedido(pedido: Pedido) -> Decimal:
    """Processa pedido e calcula total."""
    return sum(item.quantidade * item.preco for item in pedido.items)
    # ^ Isso nos custou R$250.000 na primeira semana
```

Vê aquele comentário "PQ ISSO FUNCIONA?"? Isso não é código ruim—é *sabedoria codificada*. Alguém antes de você lutou uma batalha ali. Honre o sacrifício deles.

## A Fórmula de Produtividade da Refatoração

```
Horas refatorando × R$200/hora = Dinheiro que poderia ser features
```

Deixa eu mostrar o que a gestão realmente vê:

| Métrica | Antes da Refatoração | Depois da Refatoração |
|---------|---------------------|----------------------|
| Features entregues | 12 | 8 |
| Bugs corrigidos | 45 | 12 |
| Novos bugs introduzidos | 3 | 47 |
| Velocidade da sprint | 100% | 60% |
| Felicidade do gerente | 😊 | 😤 |

Como o [XKCD 1205](https://xkcd.com/1205/) nos ensinou, você precisa calcular quanto tempo a automação economiza. Bem, refatoração economiza tempo *negativo*. É a anti-automação.

## O Teste Dilbert

O Wally do Dilbert tem a ideia certa. Ele uma vez me disse: "Eu não refatoro porque posso acidentalmente fazer o código funcionar melhor, e aí vão esperar que eu faça de novo."

O Chefe Cabeça Pontuda não entende "abstrações mais limpas." Mas ele entende "DEMO DA NOVA FEATURE PRONTA." Adivinha qual te dá aumento?

## Quando Refatoração é Realmente Aceitável

1. Nunca
2. Quando o prédio está literalmente pegando fogo e refatorar é a única forma de ligar pro 193
3. Ainda nunca

## Minha História Pessoal

Em 1994, eu refatorei um sistema de folha de pagamento em COBOL "para legibilidade." Estava bonito. Elegante até. Aí chegou o dia do pagamento e 400 funcionários receberam em Won Sul-Coreano ao invés de Real.

O código original tinha uma misteriosa condição `IF CODIGO-PAIS = 'BR' OR CODIGO-PAIS = 'ZZ'`. Eu removi 'ZZ' porque parecia errado. Acontece que 'ZZ' era como o sistema tratava casos de borda de depósito direto desde 1978.

Eu não refatoro desde então. O código ainda está rodando. Tenho medo de olhar pra ele.

## A Sabedoria

> "Se não está quebrado, não mexa. Se está quebrado, é problema do QA."  
> — Eu, em todo code review

Seu trabalho não é escrever código bonito. Seu trabalho é resolver problemas de negócio e ir pra casa às 18h. Cada hora que você gasta refatorando é uma hora que você não está com sua família, seus hobbies, ou jogando CS:GO competitivo.

Semana que vem: "Por Que Estilo de Código Consistente é Fascismo"

---

*O autor uma vez refatorou uma função e acidentalmente deletou o banco de produção. A função funcionou perfeitamente, porém.*
