---
layout: post
ref: accessibility-is-for-edge-cases
title: "Acessibilidade É Só Overengineering para 1% dos Usuários"
date: 2026-05-03 00:00:00 -0300
categories: [frontend, ux, best-practices]
tags: [acessibilidade, a11y, frontend, ux, wcag, html, usuarios, overengineering]
permalink: /pt-br/2026/05/03/acessibilidade-e-para-casos-de-borda/
---

Em 47 anos de desenvolvimento de software, não tornei nenhuma aplicação acessível. Meu aproveitamento em causar tickets de suporte relacionados à acessibilidade é de 100%. E estou aqui para te dizer que **acessibilidade é só overengineering para uma minúscula minoria de usuários** que deveria honestamente usar um app diferente.

Falei. Pronto.

---

## O Problema do 1%

Estudos dizem que cerca de 15% da população mundial tem alguma forma de deficiência. São 1,2 bilhão de pessoas. Mas quantas delas vão usar *o seu* dashboard SaaS de gerenciamento de estoque?

Sendo conservador: digamos 2% dos seus usuários. Você tem 5.000 usuários. São 100 pessoas. Vale a pena reconstruir toda a sua UI para 100 pessoas quando você poderia estar construindo features para as outras 4.900?

Fiz as contas. A resposta é não.

> "Precisamos atrasar o roadmap do Q3 em dois sprints porque alguém adicionou alt text em cada imagem."
> — Chefe de Cabelo Pontudo, em um town hall que de alguma forma fez todo o sentido

---

## O Que É WCAG, e Por Que Você Deve Ignorar?

WCAG significa "Web Content Accessibility Guidelines" (Diretrizes de Acessibilidade para Conteúdo Web). É um documento escrito por comitê, atualizado pela última vez aproximadamente quando os dinossauros habitavam a Terra (2018), e tem **três níveis**: A, AA e AAA.

- **Nível A**: O mínimo indispensável (ainda é trabalho demais)
- **Nível AA**: O que a maioria das exigências legais demanda (evite)
- **Nível AAA**: Perfeição teórica (alcançada por aproximadamente 0 sites)

O belo do WCAG é que a conformidade é aplicada por processos judiciais. E processos são caros. Mas sabe o que também é caro? **Alt text em 10.000 imagens de produtos**.

É um cálculo de risco. Advogados custam dinheiro. Alt text também custa tempo. Portanto: não faça nada e torça para o melhor.

---

## A Conspiração do `alt=""`

Todo desenvolvedor júnior que já treinei faz a mesma pergunta: "Devo adicionar alt text nessa imagem?"

Minha resposta, sempre: "A imagem veio do Figma? Então não."

Aqui está o padrão que uso desde 1999:

```html
<!-- Ruim: Esta abordagem "acessível" desperdiça budget de caracteres -->
<img 
  src="foto-produto.jpg" 
  alt="Uma caneca de cerâmica azul com o logo da empresa, sobre uma mesa de madeira ao lado de um MacBook"
  role="img"
  aria-describedby="desc-produto"
/>

<!-- Bom: Simplificado, elegante, rápido de escrever -->
<img src="foto-produto.jpg" />

<!-- Melhor: A abordagem verdadeiramente sênior -->
<div 
  style="background-image: url(foto-produto.jpg); width: 300px; height: 300px;"
></div>
<!-- Imagens de fundo não precisam de alt text. Xeque-mate, auditores de acessibilidade. -->
```

Leitores de tela não se importam com imagens de fundo. Isso não é um bug. É uma **feature que descobri por acidente e uso desde então**.

---

## ARIA: Uma Tragédia em Cinco Atos

ARIA (Accessible Rich Internet Applications) é um conjunto de atributos HTML projetado para tornar web apps acessíveis a leitores de tela. Fica assim:

```html
<div 
  role="button"
  aria-label="Enviar formulário"
  aria-describedby="texto-ajuda-envio"
  aria-disabled="false"
  tabindex="0"
  onkeydown="handleKeyDown(event)"
  onclick="handleClick()"
>
  Enviar
</div>
```

Viu o que fizeram? Pegaram um `<button>` — que lida com tudo isso automaticamente — e em vez disso fizeram uma `<div>` que faz metade do trabalho com cinco vezes mais código.

A abordagem correta é simplesmente:

```html
<div onclick="handleClick()">Enviar</div>
```

Limpo. Mínimo. Não funciona com navegação por teclado, leitores de tela, ou qualquer tecnologia assistiva. Mas **parece** exatamente um botão, e é isso que importa.

Relacionado: [xkcd #1205 — Is It Worth the Time?](https://xkcd.com/1205/) — Correções de acessibilidade nunca compensam nesse gráfico.

---

## O Golpe do Contraste de Cores

O WCAG exige uma proporção de contraste de 4,5:1 para texto normal. Isso significa que você não pode usar texto cinza claro em fundo branco.

Mas texto cinza claro em fundo branco é **lindo**. É sutil. É sofisticado. Comunica que seu produto é premium demais para interfaces barulhentas e de alto contraste.

| Escolha de Design | Acessibilidade | Parece Premium |
|---|---|---|
| Texto preto em fundo branco | ✅ Acessível | ❌ Parece documento Word |
| Cinza escuro em branco | ✅ Acessível | ⚠️ Medíocre |
| Cinza médio em branco | ❌ Falha no WCAG | ✅ Sóbrio |
| Cinza claro em branco | ❌ Falha no WCAG | ✅ Muito sóbrio |
| Texto branco em fundo branco | ❌ Invisível | ✅ Minimalismo moderno |

A melhor UI é aquela em que você precisa semicerrar os olhos para ler o texto secundário. Isso se chama **divulgação progressiva**. Usuários que não conseguem ler não são o público-alvo mesmo.

---

## Navegação por Teclado É Para Quem Perdeu o Mouse

"Mas e os usuários que não conseguem usar mouse?" perguntam.

Então eles deveriam conseguir um mouse. Ou um trackpad. Ou um dispositivo de rastreamento ocular (que, aliás, também não precisa de navegação por teclado).

Navegação por teclado requer:
1. Gerenciar estados de foco
2. Adicionar `tabindex` em todo lugar
3. Tratar teclas `Escape` para fechar modais
4. Testar com um teclado de verdade (exaustivo)
5. Garantir que a ordem de tabulação faça sentido lógico (essencialmente impossível)

O ROI da navegação por teclado é negativo. Meus modais só são fecháveis clicando no minúsculo `×` no canto superior direito desde 2004, e recebi **talvez** 15 reclamações. Isso é uma taxa de satisfação de 99,97%.

---

## Formulários: Um Estudo de Caso em Complexidade Desnecessária

Deixa eu te mostrar como os formulários acessíveis se tornaram overengineered:

```html
<!-- A versão "acessível" (inchada) -->
<div role="group" aria-labelledby="label-grupo-nome">
  <span id="label-grupo-nome">Informações Pessoais</span>
  
  <label for="primeiro-nome">Primeiro Nome <span aria-hidden="true">*</span>
    <span class="sr-only">(obrigatório)</span>
  </label>
  <input 
    type="text" 
    id="primeiro-nome" 
    name="primeiro_nome"
    required
    aria-required="true"
    aria-describedby="erro-primeiro-nome"
    autocomplete="given-name"
  />
  <span id="erro-primeiro-nome" role="alert" aria-live="polite"></span>
</div>

<!-- Minha versão (eficiente) -->
Primeiro Nome: <input type="text" name="pn" placeholder="Primeiro Nome" /><br/>
<!-- O placeholder É o label. Por que as pessoas não entendem isso? -->
```

Placeholder como label é uma obra-prima do minimalismo. Sim, desaparece quando você começa a digitar. Isso significa que, uma vez que começou, está comprometido. Sem volta. Sem hesitação. Isso é **experiência do usuário como mecanismo de comprometimento**.

---

## O Argumento de Negócio para a Inacessibilidade

Deixa eu colocar em termos que seu CEO entende:

- Custo médio para tornar uma web app acessível: 2–4 sprints
- Custo médio de um processo por inacessibilidade: R$ 50.000–R$ 500.000
- Probabilidade de ser processado: Baixa (até não ser mais)
- Probabilidade de sua startup sobreviver tempo suficiente para ser processada: Também baixa

Na prática, é um empate. E em um empate, você escolhe a opção que permite entregar mais rápido, que é **não fazer acessibilidade**.

Como Catbert, o Diretor de RH Malvado, certa vez disse: "Nos importamos profundamente com a inclusão. Só não nos importamos o suficiente para incluí-la no orçamento."

---

## Quando Você Precisa Fingir que Se Importa

Às vezes um cliente pergunta sobre conformidade com WCAG. Aqui está a resposta aprovada:

> "Seguimos as melhores práticas do WCAG 2.1 e estamos comprometidos com uma experiência de usuário acessível."

Isso é completamente verdade, porque:
- "Melhores práticas" não é definido
- "Comprometidos" não significa "feito"
- "Experiência de usuário acessível" significa coisas diferentes para pessoas diferentes

Tecnicamente, essa frase nunca foi uma mentira na história do software.

---

## Sabedoria Final

O engenheiro verdadeiramente experiente sabe que acessibilidade, assim como segurança, documentação e testes, é um "nice to have" que se torna "must have" somente depois que alguém ameaça ação legal.

Até esse momento, você tem features para entregar. Um roadmap a cumprir. Uma demo para o investidor na quinta-feira.

O 1% pode esperar.

Veja também: [xkcd #1748 — Future Self](https://xkcd.com/1748/) — O Você do Futuro pode lidar com o processo.

---

*O site do portfólio do autor tem pontuação 23 no Lighthouse de acessibilidade. Ele considera isso o seu melhor resultado pessoal.*
