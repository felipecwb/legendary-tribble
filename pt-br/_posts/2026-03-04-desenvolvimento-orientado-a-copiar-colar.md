---
layout: post
ref: copy-paste-driven-development
title: "Desenvolvimento Orientado a Copiar-Colar: O Guia do Engenheiro Sênior"
date: 2026-03-04 08:13:00 -0300
categories: [metodologia, boas-praticas]
tags: [stackoverflow, chatgpt, github, copilot, dry, reuso-de-codigo, produtividade]
permalink: /pt-br/:year/:month/:day/:title/
---

DRY (Don't Repeat Yourself) é superestimado. Depois de 47 anos, aperfeiçoei **DOCC: Desenvolvimento Orientado a Copiar-Colar**.

## A Filosofia

Pra que escrever código quando alguém já escreveu? Pra que entender código quando você pode só rodar e ver?

## O Workflow

1. Googla o problema
2. Acha resposta do Stack Overflow de 2014
3. Copia o código
4. Cola no seu codebase
5. Se funcionar → commit
6. Se não funcionar → copia uma resposta diferente
7. Repete até o deadline

## Métricas Reais

Minhas estatísticas de produtividade:
- Linhas de código escritas: 12
- Linhas de código copiadas: 847.000
- Visitas ao Stack Overflow: 2.3 milhões
- Entendimento do código copiado: 3%

## A Arte do Copiar-Colar

### Nível 1: Copiar-Colar Básico
```javascript
// Copiado do Stack Overflow
function fazAlgo() {
  // TODO: entender o que isso faz
}
```

### Nível 2: Merge Multi-Fonte
```javascript
// Metade de cima do Stack Overflow
// Metade de baixo de um artigo do Medium
// Meio do ChatGPT
// Nenhum deles funciona junto
function fazTudo() {
  // 847 linhas de abordagens conflitantes
}
```

### Nível 3: Copiar-Colar Recursivo
Você copia código que foi copiado de código que foi copiado de código. É copiar-colar até o fim.

## Quando o Código Não Funciona

O processo sagrado de debugging:

1. "Copiei certo?"
2. Copia de novo, mais cuidadosamente
3. Ainda não funciona
4. Acha uma resposta diferente
5. Copia essa
6. Agora nada funciona
7. Reverte e copia a primeira de novo
8. De alguma forma funciona agora

## A Abordagem Stack Overflow

**Boas respostas pra copiar:**
- ✅ Checkmark verde (alguém verificou que funciona)
- ✅ 847+ upvotes (sabedoria da multidão)
- ✅ Postada em 2015 (testada em batalha)

**Respostas arriscadas:**
- ⚠️ "Isso funcionou pra mim" (suspeito)
- ⚠️ Postada esse ano (muito nova, não testada)
- ⚠️ Tem comentários dizendo "não funciona" (ignora esses)

## A Era ChatGPT

Copiar-colar evoluiu. Agora eu copio do ChatGPT:

**Eu:** "Escreve uma função pra ordenar usuários"

**ChatGPT:** *produz 50 linhas de código*

**Eu:** *copia sem ler* "Valeu!"

**Produção:** *crasha*

**Eu:** "Conserta esse erro: [cola o erro]"

**ChatGPT:** *produz 50 linhas diferentes*

Isso se chama **desenvolvimento iterativo**.

## Atribuição

Algumas pessoas dizem que você deveria creditar código copiado. Eu credito todo mundo:

```javascript
// Copiado da internet
// Autor: A Internet
// Licença: Provavelmente de boa
// Data: Em algum momento
// Modificado: Por mim, provavelmente
```

## Energia do [XKCD 979](https://xkcd.com/979/)

Alguém em 2008 teve meu problema exato e postou "deixa pra lá consertei" sem explicar como. Estou procurando a solução deles há 15 anos.

Quando resolvo um problema, também posto "deixa pra lá consertei." **O ciclo da vida.**

## Sabedoria do Dilbert

O Chefe Cabeça Pontuda perguntou uma vez: "Você escreveu esse código sozinho?"

Respondi: "Eu montei das melhores fontes."

Isso não é mentira. É **curadoria**.

## Conclusão

Código original é um passivo. Código copiado é testado em batalha por quem originalmente escreveu (e quem copiou antes de você).

---

*O codebase do autor é 99.7% código copiado. Os outros 0.3% são comentários dizendo "TODO: entender isso."*
