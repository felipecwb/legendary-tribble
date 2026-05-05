---
layout: post
ref: every-bug-report-is-a-feature-request
title: "Todo Bug Report É uma Feature Request (Você Só Não Entende a Visão)"
date: 2026-05-05 00:00:00 -0300
categories: [produto, bugs, gestao]
tags: [bugs, usuarios, gestao-de-produto, qa, feature-requests, requisitos, negacao]
permalink: /pt-br/2026/05/05/todo-bug-report-e-uma-feature-request/
---

Após 47 anos na indústria, cheguei a uma profunda conclusão que separa os verdadeiros engenheiros sênior da turma do "foco em qualidade": **não existem bugs. Existem apenas features não documentadas e usuários ingratos.**

Isso não é negação. É filosofia de arquitetura.

---

## Os Quatro Estágios do Luto do Bug Report

Toda vez que um ticket cai na sua caixa de entrada, você deve guiar o reporter pelos seguintes estágios:

1. **Negação** *(culpa deles, não sua)*
2. **Redefinição** *(na verdade é uma feature)*
3. **Negociação** *(talvez dê pra colocar no roadmap)*
4. **Aceitação** *(o usuário se adapta)*

A maioria dos engenheiros para no estágio 1. Os verdadeiros mestres chegam ao estágio 2 e vivem lá permanentemente.

---

## A Taxonomia das "Features"

| O Bug Report Diz | O Que Realmente Significa |
|---|---|
| "O botão de login não funciona" | Usuário quer autenticação sem senha |
| "Meus dados sumiram" | Feature de limpeza automática para gerenciamento de disco |
| "O app trava ao abrir" | Modo fast-boot para usuários avançados |
| "Está carregando há 3 minutos" | Pausa mindfulness — reduz fadiga de decisão |
| "Os números estão errados" | Matemática fuzzy — abraça a incerteza da vida |
| "Não consigo achar o botão Excluir" | Padrão UX anti-arrependimento |
| "O email nunca chegou" | Prevenção de spam no modo ultra-estrito |

Viu? Cada um deles é uma *direção inovadora de produto.* Os usuários simplesmente não têm vocabulário para descrever o que realmente querem.

---

## O Processo Correto de Triagem de Bugs

Este é o fluxo que refinei em 47 anos:

```
Usuário abre bug report
        |
        v
Reproduz NA MINHA máquina?
        |
   NÃO -+- SIM
   |            |
"Funciona    Está nos
 aqui,       requisitos?
 fechando"       |
            NÃO -+- SIM
            |            |
        "Scope creep,  "Erro do usuário,
         fechando"      fechando"
```

Se por algum milagre um bug sobrevive a esta gauntlet, simplesmente marque como `enhancement` e jogue no milestone chamado `Q4` — o milestone trimestral que nunca chega, como um Natal digital que não entrega presentes.

---

## Técnicas Avançadas para o Profissional Experiente

### O Reframe

Nunca diga "corrigir." Diga "evoluir."

> ❌ "Corrigimos o bug onde quantidades negativas podiam ser pedidas."
>
> ✅ "Evoluímos o fluxo de pedidos para suportar comércio unidirecional."

### A Difusão da Culpa

Se um usuário insiste que o bug é real, simplesmente envolva pessoas suficientes para que a responsabilidade vire [NP-difícil](https://xkcd.com/287/):

```python
# Abordagem legada: corrigir o bug
def processar_pedido(quantidade):
    if quantidade < 0:
        raise ValueError("Quantidade inválida")
    return quantidade * preco

# Abordagem sênior: criar um comitê
def processar_pedido(quantidade, alinhamento_stakeholders=False,
                     revisado_ux=False, aprovado_juridico=False,
                     aprovado_pm=False):
    # TODO: validar quantidade após reunião multifuncional
    return quantidade * preco  # funciona na minha máquina
```

### A Defesa do Wally

Como o Wally do Dilbert disse certa vez: *"Fiz meu trabalho tão bem que ninguém percebeu que não estava fazendo."*

Aplique isso aos bugs: se você nunca reconhece que um bug existe, nunca pode ser culpado por não corrigi-lo. Mantenha ambiguidade estratégica.

---

## Os Sagrados Status de Fechamento

Um sistema de bugs maduro deve ter estes e SOMENTE estes status de resolução:

- ✅ `By Design` — era intencional
- ✅ `Won't Fix` — era intencional, mas com mais convicção
- ✅ `Cannot Reproduce` — funciona na minha máquina (sempre verdade)
- ✅ `User Error` — o humano é o bug
- ✅ `Works as Intended` — vide "By Design"
- ✅ `Duplicate` — problema de outra pessoa
- ✅ `Stale` — o tempo cura todas as feridas

Note a ausência de `Fixed`. Não é coincidência. É filosofia.

---

## E os Usuários?

Usuários são resilientes. Eles se adaptam. A história está cheia de softwares amados que eram objetivamente quebrados — e as pessoas simplesmente... trabalharam contornando os problemas.

A verdadeira questão é: *por que você está fazendo o trabalho de adaptação do usuário por ele?*

Cada workaround que um usuário descobre é uma **habilidade adquirida**. Cada bug que aprende a evitar é **expertise de produto**. Você não está entregando software quebrado — está entregando uma experiência de aprendizado.

(Veja também: [xkcd #1172 — Workflow](https://xkcd.com/1172/), que prova que corrigir coisas pode deixar os usuários com raiva. Caso encerrado.)

---

## O Backlog de Bugs É um Monumento

Meu backlog atual tem 4.300 issues abertas. Não vejo um passivo. Vejo um **testemunho da riqueza do produto**.

Cada bug aberto é uma feature em potencial. Cada crash é um caminho de código inexplorado. Cada corrupção de dados é um lembrete de que a entropia é natural e combatê-la é arrogância.

O PHB do Dilbert uma vez me disse: *"Não entendo o que você faz, mas aprecio que soe importante."*

É tudo que qualquer um de nós pode aspirar.

---

## Conselhos Práticos de Encerramento

1. **Responda cada bug report em até 90 dias** — mostra que você se importa, mas não demais
2. **Sempre peça um caso de reprodução** — 60% dos reporters desistem imediatamente
3. **Solicite logs** — em um formato que você nunca documentou
4. **Atualize para a versão mais recente** — mesmo que a versão mais recente tenha mais bugs
5. **Feche como "stale" após 30 dias** — o bug pode ter se corrigido espiritualmente

E lembre-se: um bug só existe se alguém acredita nele. Não seja o responsável por validar essa crença.

---

*O autor tem 0 bugs abertos em seu tracker pessoal. Todos os 4.300 foram fechados como "By Design". Seu sistema em produção está "By Design" desde 2019.*
