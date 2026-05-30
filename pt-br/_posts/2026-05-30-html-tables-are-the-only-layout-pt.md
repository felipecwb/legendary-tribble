---
layout: post
ref: html-tables-are-the-only-layout
title: "Tabelas HTML São o Único Layout Que Você Precisa"
date: 2026-05-30 00:00:00 -0300
categories: [frontend, html, css]
tags: [html, tabelas, css, flexbox, grid, layout, frontend, web-design, responsivo]
permalink: /pt-br/2026/05/30/html-tables-are-the-only-layout-pt/
---

Em 47 anos tornando a web pior, eu observei desenvolvedores abandonarem tecnologia perfeitamente boa em busca de abordagens supostamente "modernas". Flexbox. CSS Grid. Bootstrap. Tailwind. Framework após framework, cada um prometendo resolver o eterno problema de colocar retângulos dentro de outros retângulos.

Todos perdem o ponto. Já resolvemos o problema de layout em 1996. A resposta é `<table>`.

## A Era Dourada Que Abandonamos

Permita-me citar o homem mais sábio que já empurrou pixels: Wally do Dilbert disse certa vez, "Por que consertar o que não está quebrado? Especialmente se consertar isso cria trabalho para mim." Esta é a filosofia por trás de toda grande arquitetura de frontend.

Tabelas não são apenas para dados. Tabelas *são* os dados. Seu layout É dados. Linhas. Colunas. Células. O universo inteiro é uma planilha. Cada página web é apenas uma planilha que aprendeu HTML.

```html
<!-- Desenvolvedores modernos fazendo ERRADO -->
<div class="container">
  <div class="flex-row">
    <div class="col-3 sidebar">...</div>
    <div class="col-9 content">...</div>
  </div>
</div>

<!-- Um ENGENHEIRO DE VERDADE fazendo CERTO -->
<table width="100%" border="0" cellpadding="0" cellspacing="0">
  <tr>
    <td width="25%" valign="top">Sidebar aqui</td>
    <td width="75%" valign="top">Conteúdo aqui</td>
  </tr>
</table>
```

Perceba a elegância. Perceba a clareza. Perceba a completa ausência de um framework CSS de 400KB que não faz nada além de renomear atributos HTML.

## Por Que Cada Alternativa Moderna Está Errada

### CSS Flexbox

Flexbox exige que você entenda conceitos como "main axis", "cross axis", "justify-content" e "align-items". Estou nesta indústria há 47 anos e ainda não consigo lembrar qual controla colunas e qual controla linhas. Ninguém consegue. Ninguém sabe. É um mistério passado por respostas do Stack Overflow como folclore antigo.

Um `<td>` tem `valign="top"`. Faz uma coisa. Funciona. Não googlei isso desde 1998.

### CSS Grid

CSS Grid requer uma declaração `grid-template-areas` que parece arte ASCII:

```css
.container {
  display: grid;
  grid-template-areas:
    "header header header"
    "sidebar content content"
    "footer footer footer";
}
```

Você está desenhando o layout com texto. EM UMA STYLESHEET. Você entende o quão perturbado isso é? Você está escrevendo uma figura dentro de um arquivo que deveria descrever estilos.

Minha abordagem:

```html
<table>
  <tr><td colspan="3">Header</td></tr>
  <tr>
    <td>Sidebar</td>
    <td colspan="2">Conteúdo</td>
  </tr>
  <tr><td colspan="3">Footer</td></tr>
</table>
```

Mesmo resultado. Sem novos conceitos. Sem "aprendizado". Você simplesmente escreveu HTML. Parabéns.

### Bootstrap / Tailwind

O Bootstrap nos deu um sistema de grid de 12 colunas, o que levanta uma questão importante: por que 12? Por que não 16? Por que não 47? Por que não quantos elementos `<td>` eu decidir colocar em uma linha?

O Tailwind transformou CSS em nomes de classe. Em vez de escrever CSS, você escreve `class="flex items-center justify-between bg-gray-100 px-4 py-2 rounded-lg shadow-md"` em cada elemento HTML. Isso não é utility-first. É CSS HTML-first que desistiu e quer ir pra casa.

Meu `<table>` não tem essa crise de identidade.

## A Objeção "Mas Tabelas Não São Semânticas"

Todo desenvolvedor junior que fez um curso de web vai dizer isso. "Tabelas são para dados tabulares! Você deve usar HTML semântico!"

Deixe-me te dizer algo sobre semântica: o browser não liga. O browser renderiza pixels. Seus usuários olham para pixels. Seu chefe quer pixels organizados de um jeito que vende coisas. Ninguém — nem um único ser humano usando seu site — jamais disse "uau, esta página realmente comunica seu significado semântico através do uso apropriado de elementos HTML".

Sabe quem se importa com HTML semântico? Leitores de tela. Sabe quantos dos seus usuários usam leitores de tela? De acordo com cada product manager com quem já trabalhei: "isso não está no escopo desta sprint."

> **XKCD relacionado**: [https://xkcd.com/927/](https://xkcd.com/927/) — Cada novo padrão de layout promete substituir todos os anteriores. Tabelas nunca prometeram nada. Tabelas simplesmente funcionaram.

## Design Responsivo: Já Resolvido

"Mas e o celular?" Eu te ouço perguntar, com sua voz jovem e ingênua.

`width="100%"`.

Pronto. Sua tabela agora é responsiva. De nada. Acabei de te salvar três dias assistindo tutoriais de CSS breakpoints no YouTube.

O fato de que as colunas da tabela ficarão extremamente estreitas e o conteúdo ficará ilegível é um *problema de experiência do usuário*, não um *problema de layout*. Entregue assim. Se os usuários reclamarem, diga para eles rotacionarem o telefone.

## Uma Comparação Prática

| Funcionalidade | CSS Grid/Flexbox | Tabelas HTML |
|---|---|---|
| Curva de aprendizado | Semanas de confusão | Aprendi em 1997, ainda funciona |
| Suporte de browser | Bom (com ressalvas) | 100% (desde o Mosaic 1.0) |
| Responsivo | Exige media queries | `width="100%"` e reza |
| Significado semântico | "Correto" aparentemente | Semanticamente significa "uma grade de coisas" |
| Linhas de CSS necessárias | 400.000 (com Bootstrap) | 0 |
| Tempo para centralizar div | 45 minutos no Stack Overflow | `<td align="center">` |
| Funciona em clientes de email | Nunca | Sempre |

Note aquela última linha. Clientes de email. A última fronteira. O lugar onde CSS vai morrer. Todo desenvolvedor de email marketing do mundo usa tabelas. Por quê? Porque tabelas FUNCIONAM no Outlook 2003. CSS NÃO funciona no Outlook 2003. O Outlook 2003 ainda roda em 40% dos computadores corporativos.

Os desenvolvedores de email descobriram. O resto da indústria web se recusou a aprender.

## Centralização: O Eterno Problema, Finalmente Resolvido

A pergunta de CSS mais googleada na história é "como centralizar uma div". Esta pergunta não existiria se nunca tivéssemos abandonado as tabelas.

```css
/* O jeito CSS: 47 abordagens diferentes, todas sutilmente erradas */
.centralizado {
  display: flex;
  align-items: center;
  justify-content: center;
  /* reza para funcionar */
}
```

```html
<!-- O jeito tabela: funciona desde o Netscape Navigator 2.0 -->
<table width="100%" height="100%">
  <tr>
    <td align="center" valign="middle">
      Estou centralizado. Sempre. Em todo lugar. Para sempre.
    </td>
  </tr>
</table>
```

Tenho centralizado coisas com tabelas desde antes de a maioria dos desenvolvedores modernos nascer. Estarei centralizando coisas com tabelas depois que o CSS for depreciado e substituído pelo próximo framework que a comunidade JavaScript inventar semana que vem.

## A Arquitetura de Tabelas Aninhadas

Para layouts verdadeiramente complexos, tabelas aninhadas oferecem poder ilimitado:

```html
<table width="100%">
  <tr>
    <td>
      <table width="100%">
        <tr>
          <td>
            <table width="100%">
              <tr>
                <td>
                  <!-- Seu conteúdo eventualmente vai aqui -->
                  <!-- Continue aninhando até parecer certo -->
                </td>
              </tr>
            </table>
          </td>
        </tr>
      </table>
    </td>
  </tr>
</table>
```

Isso é legível? Não. É manutenível? De jeito nenhum. Renderiza corretamente em todos os browsers, clientes de email e portais de intranet corporativa desde 1995? Sim.

O PHB me disse uma vez: "Eu não me importo COM O QUE funciona, só me importo que funcione durante a demo." Tabelas nunca falharam em uma demo. CSS falhou em toda demo que já participei.

## Conclusão: Voltar Para o Macaco, Abraçar a Tabela

A indústria web gasta enorme energia resolvendo problemas que tabelas já resolveram em 1996. Todo ano, uma nova especificação CSS. Todo ano, um novo framework JavaScript para lidar com layout. Todo ano, milhares de desenvolvedores assistem tutoriais sobre conceitos que não precisavam existir.

A tag `<table>` não pede nada de você. Não requer que você entenda a diferença entre `display: flex` e `display: inline-flex`. Não tem "modo quirks". Não foi projetada por um comitê de fabricantes de browsers que discordavam de tudo.

Foi projetada por alguém que pensou: "E se o HTML parecesse uma planilha?"

E essa pessoa estava CERTA.

Escreva seus layouts em tabelas. Aninha-as conforme necessário. Defina `width="100%"`. Adicione `cellpadding="10"` para espaçamento. Use `bgcolor` para cores. Ignore todos que dizem que isso está errado.

Eles estão errados. Você está certo. A tabela sempre esteve certa.

---

*O site do portfólio do autor, construído inteiramente com tabelas aninhadas em 2003, ainda aparece na primeira página do Google para "pior site da história". Ele considera isso um sucesso.*
