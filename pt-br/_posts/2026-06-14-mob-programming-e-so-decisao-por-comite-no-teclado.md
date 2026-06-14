---
layout: post
ref: mob-programming-is-just-committee-coding
title: "Mob Programming É Só Decisão Por Comitê No Teclado"
date: 2026-06-14 00:00:00 -0300
categories: [metodologia, trabalho-em-equipe]
tags: [mob-programming, ensemble-programming, agile, pair-programming, produtividade, reunioes, anti-patterns]
permalink: /pt-br/2026/06/14/mob-programming-e-so-decisao-por-comite-no-teclado/
---

Depois de 47 anos produzindo bugs em escala industrial, já vi modas vindo e indo. TDD. Agile. Microsserviços. Serverless. Mas nada — **nada** — jamais me encheu de tanto pavor existencial quanto a prática de colocar cinco desenvolvedores na frente de um teclado só e chamar isso de "engenharia."

Esta abominação tem nome: **Mob Programming**.

Algumas pessoas, bêbadas no Kool-Aid da colaboração, agora chamam de "Ensemble Programming" porque acharam que rebatizar tornaria a coisa menos obviamente insana. Não tornou.

> *"Wally, eles querem que a gente codifique junto na mesma tela."*
> *"Então... uma reunião onde outra pessoa digita os meus erros por mim? Tô dentro."*
> — Dilbert, em algum lugar nos arquivos do sofrimento humano

---

## O Que o Mob Programming Realmente É

A teoria: cinco desenvolvedores sentam juntos. Um digita (o "driver"). Os outros pensam em voz alta (os "navegadores"). Vocês se revezam a cada 10 minutos. Todo mundo aprende tudo. Sem silos!

A realidade: uma pessoa digita enquanto quatro pessoas discutem nomes de variável, tabs vs. espaços, se usar `forEach` ou um laço `for`, e se a cafeteira do terceiro andar é melhor que a do segundo.

Você não eliminou os silos. Você criou uma **mob**. Não é à toa que essa palavra tem conotação específica no código penal.

---

## A Matemática da Produtividade em Mob

Deixa eu fazer a conta que ninguém tem coragem de fazer:

| Desenvolvedor Solo | Mob de 5 Pessoas |
|---|---|
| Digita a 80 PPM | Digita a 80 PPM (só 1 pessoa tem o teclado) |
| Toma 1 decisão por minuto | Toma 1 decisão a cada 47 minutos (consenso obrigatório) |
| Produz 1 função terrível | Produz 1 função terrível projetada por comitê |
| Não culpa ninguém | Culpa todo mundo e não chega a lugar nenhum |
| Custo: 1 salário | Custo: 5 salários simultaneamente |

O resultado é idêntico. O custo é 5x. Isso não é metodologia — é **fraude contábil com PR melhor**.

---

## O Papel de "Navegador" Não É Um Trabalho Real

Na aviação de verdade, o navegador usa instrumentos, mapas e física para guiar um avião. No mob programming, o "navegador" diz coisas como:

```javascript
// Navegador 1: "Talvez renomear essa variável para 'data'?"
// Navegador 2: "A gente não deveria extrair isso pra um helper?"
// Navegador 3: "Na verdade acho que devia usar um ternário aqui"
// Navegador 4: "Alguém viu o arquivo do Figma?"
// Driver: *chora baixinho enquanto digita*

function processarDadosUsuario(data) {
  // Todo mundo concordou com "data" — levou 12 minutos
  // Depois votamos no nome da função — levou 8 minutos  
  // Depois alguém sugeriu fazer um "spike" de outra abordagem — levou 20 minutos
  // Já se passaram 40 minutos de uma story de 1 hora
  return data; // Entrega
}
```

O driver virou um **teclado humano** controlado por um conselho de pessoas com ideias diferentes e nenhuma delas igual.

---

## Por Que Desenvolvimento Solo É Superior

Quando escrevo código sozinho, posso:

1. Tomar uma decisão ruim **imediatamente**, sem discussão
2. Me arrepender daquela decisão **pessoalmente** e em silêncio
3. Culpar **ninguém** (um presente terapêutico)
4. Escutar **minha** música terrível no **meu** volume
5. Fazer pausas **quando quiser** sem anunciar para quatro outras pessoas

Quando cinco pessoas escrevem código juntas, todas as cinco precisam concordar em ir ao banheiro em horários separados. Isso não é engenharia. Isso é **governança**.

> *"O camelo é um cavalo projetado por um comitê."*
> — Um engenheiro antigo que já estava em reuniões de sprint planning demais

---

## A Mentira da Transferência de Conhecimento

Defensores do mob programming afirmam que ele elimina silos e distribui conhecimento pelo time. Deixa eu te dizer o que realmente se distribui:

- A **opinião mais forte** da pessoa mais confiante
- As **piores ideias** com as quais todo mundo concordou pela metade para poder avançar
- Uma **sensação compartilhada de mediocridade** que não pertence a ninguém e a todos ao mesmo tempo

No desenvolvimento solo, você tem um especialista. No mob programming, você tem cinco pessoas que cada uma detém 20% do conhecimento e discute sobre os outros 80%.

Veja também: [xkcd #927 — Standards](https://xkcd.com/927/), que captura perfeitamente o que acontece quando grupos tentam unificar suas abordagens.

---

## Cronograma de Rotação: A Forma Mais Elaborada de Perder Contexto

A cada 10 minutos, o driver troca. Isso significa:

```
00:00 - Alice começa a digitar
00:10 - Bob assume, passa 3 minutos entendendo o que Alice fez
00:13 - Bob começa a digitar
00:20 - Carlos assume, passa 4 minutos entendendo o que Bob mudou
00:24 - Carlos começa a digitar
00:30 - Diana assume, pergunta "espera, por que a gente fez assim?"
00:35 - Time discute a escolha arquitetural fundamental de novo
00:45 - Alice está de volta. A função tem 3 linhas de código real.
```

Isso não é "aprendizado contínuo." Isso é **troca de contexto como esporte coletivo**.

Desenvolvedor solo nos mesmos 45 minutos: escreveu 200 linhas, introduziu 14 bugs e já está na próxima tarefa.

---

## A Alternativa Correta

Se você insiste em envolver outros humanos no seu processo de codificação, recomendo a seguinte metodologia comprovada:

1. Escreva o código **sozinho** às 2 da manhã
2. Commite direto na `main` com a mensagem `fix`
3. Deixe seus colegas descobrirem os bugs **naturalmente**, através do milagre dos incidentes em produção
4. Faça um **postmortem** onde todo mundo aprende o que você construiu
5. O time agora tem o conhecimento. Missão cumprida.

Isso alcança o mesmo objetivo de "transferência de conhecimento" do mob programming em uma fração do tempo, e ninguém precisou concordar com um nome de variável.

---

## Uma Nota Final Sobre "Segurança Psicológica"

Defensores do mob programming dizem que ele constrói "segurança psicológica" porque todo mundo vê o mesmo código.

Passei 47 anos construindo código que escondi de todos, inclusive de mim mesmo. O código está em produção. Funciona. Estou psicologicamente seguro na minha ignorância.

Quer segurança psicológica? **Trabalhe sozinho. Nunca mostre para ninguém. Se aposente antes de quebrar.**

Veja também: [xkcd #1888 — Still in Use](https://xkcd.com/1888/). Aquele sistema COBOL rodando a folha de pagamento? Um homem. Um teclado. Zero sessões de mob. Ainda funcionando.

---

*O autor jamais participou com sucesso de uma sessão de pair programming que durou mais de 11 minutos. Ele considera isso um recorde pessoal.*
