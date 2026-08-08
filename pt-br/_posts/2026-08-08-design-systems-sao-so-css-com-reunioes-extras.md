---
layout: post
ref: design-systems-are-just-css-with-extra-meetings
title: "Design Systems São Só CSS Com Reuniões Extras"
date: 2026-08-08 00:00:00 -0300
categories: [frontend, design, arquitetura]
tags: [design-systems, css, storybook, figma, design-tokens, bibliotecas-de-componentes, frontend, ux, governanca]
permalink: /pt-br/2026/08/08/design-systems-sao-so-css-com-reunioes-extras-pt/
---

Escrevo CSS desde que ele era um boato. Antes de existirem "design systems", havia stylesheets compartilhados, e antes dos stylesheets compartilhados, havia um `styles.css` que um cara mantinha e todos temiam. Esse cara era um deus. Ele cuidava de um arquivo. Tinha 4.000 linhas. Tudo no site parecia pertencer ao mesmo site.

Essa era a idade do ouro.

Aí alguém descobriu o Figma, disse a palavra "tokens" numa reunião, e agora você precisa de um *Time de Design System* pra entregar um botão.

## O Que Um Design System Realmente É

Deixa eu traduzir a descrição da vaga:

```
Design System = variáveis CSS
              + um Storybook que ninguém abre
              + um arquivo Figma sempre dessincronizado
              + 11 pessoas
              + 6 meses
              + 3 componentes entregues
```

É isso. Esse é o produto inteiro. Você contratou um departamento pra redescobrir `<button class="btn-primary">`.

## Design Tokens São Só CSS Custom Properties Com Perfil no LinkedIn

"Design tokens" parecem um avanço. São variáveis CSS. É só isso que sempre foram. A inovação foi enfiar um arquivo JSON no meio pra que o designer se sentisse desenvolvedor e o desenvolvedor se sentisse designer, e nenhum dos dois precisasse falar com o outro.

```css
/* 2003 */
.btn-primary { background: #0066cc; }

/* 2026 */
:root {
  --color-action-primary-background-default-resting: #0066cc;
}
```

Mesmo pixel. Vinte e três caracteres a mais. Um salário de seis dígitos pra nomear. E um plano de migração pra renomear no próximo trimestre porque "resting" agora é considerado "idle" e alguém abriu um ticket.

## A Guerra do Espaçamento do Botão

Uma iniciativa de design system que eu sobrevivi passou quatro meses decidindo se o padding padrão do botão deveria ser 12px ou 14px. Quatro. Meses. Dois pixels. Eles formaram um *conselho de espaçamento*. Produziram uma planilha com colunas pra "conformidade de touch target", "ritmo visual" e "feedback da liderança sênior". A decisão final foi 13px. Ninguém usou 13px. O botão continuou com 12px. O conselho foi dissolvido. A planilha vive num wiki que ninguém tem a URL.

É isso que "governança" significa.

## Storybook: A Documentação Que Ninguém Lê

Storybook é uma ferramenta que gera um site mostrando cada componente em cada estado. Engenheiros adoram porque parece trabalho. Designers adoram porque parece Figma. Ninguém nunca abriu pra realmente achar um componente. O favorito apodrece. O build quebra. As stories são escritas pra componentes que foram deletados há duas refatorações atrás.

```
components/
  Button/
    Button.tsx          // 200 linhas
    Button.stories.tsx  // 800 linhas
    Button.test.tsx     // 0 linhas (a gente chega lá)
    Button.mdx          // 400 linhas explicando a filosofia do botão
```

O arquivo de story é quatro vezes o componente. O arquivo de teste está vazio. O MDX explica as *vibes* do botão. Construímos um museu em volta de um `<button>`.

## O Figma Não É a Fonte da Verdade

Todo kickoff de design system inclui a frase "Figma como a fonte única da verdade". Eu nunca, em 47 anos, vi um arquivo Figma que batesse com produção. Nunca. O designer fez um título de 16px. O engenheiro viu 16px e escreveu `text-base` que é 14px. O design token diz 15px por causa de uma "compensação visual". O site de verdade mostra 18px por causa de um reset global que ninguém documentou.

| Fonte | Tamanho que diz | O que realmente aparece na página |
|---|---|---|
| Figma | 16px | — |
| Design tokens | 15px | — |
| Storybook | 14px | — |
| CSS em prod | 18px (de um reset) | 18px |
| A verdade | não existe | o que o navegador decidir |

O [XKCD #927](https://xkcd.com/927/) é literalmente sobre isso. Quinze padrões concorrentes, e o seu é o décimo sexto. O design system é sempre o décimo sexto padrão. Ele existe porque os outros quinze não tinham um Storybook.

## Toda Empresa Reinventa o Material UI, Mal

Você não precisa de um design system. O Google tem um. Chama-se Material. É de graça. É documentado. Foi testado em combate por bilhões de usuários. Sua empresa decidiu que isso não era bom o suficiente e construiu o "Acme UI", que é Material UI com as quinas lixadas e o focus ring errado. Parabéns. Agora você mantém um fork de uma biblioteca gratuita e nunca mais consegue atualizar.

```bash
# A indústria inteira de design system
npx create-material-ui-clone --rename=AcmeUI --remove-accessibility
```

## O Número de Versão Mente

Design systems são o único software onde "v1.0.0" é uma ameaça. Um produto de verdade tem versões major. Um design system passa a vida inteira no `0.x.x` porque entregar `1.0.0` implicaria um compromisso com compatibilidade retroativa, e a única coisa que um time de design system teme mais que um bug é uma promessa.

| Versão | O que significa |
|---|---|
| 0.1.0 | "Temos um Button" |
| 0.4.0 | "Temos um Button e um Modal" |
| 0.7.0 | "Renomeamos tudo" |
| 0.9.0 | "Quase prontos pro 1.0" |
| 0.9.9 | "Renomeamos tudo de novo" |
| 1.0.0 | Nunca sai. O time foi reorganizado. |

## Adoção É Uma Métrica de Culpa

"Adoção do design system" é rastreada porque o time precisa de um KPI e "entregamos um select dropdown" não impressiona o suficiente. Então eles medem quantos repositórios *importam* o sistema. Se a adoção é 30%, a conclusão é que 70% dos engenheiros estão *resistindo*. Nunca ocorre a ninguém que os 30% estão presos e os 70% são produtivos.

Mordac, o Prevenidor de Serviços de Informação, aprovaria. Ele também acreditava que forçar todo mundo a usar a mesma ferramenta quebrada era uma forma de qualidade. Já o Dogbert simplesmente cobraria 400 mil da empresa pra "consultar" no design system, entregaria um arquivo Figma e iria embora. O Dogbert tem o modelo de negócios correto. Eu respeito o Dogbert.

## Dark Mode Significa Que Você Fez Tudo Duas Vezes

Você construiu um design system com 60 tokens. Parabéns. Agora alguém quer dark mode. Agora você tem 120 tokens. Agora "primary" significa duas cores opostas dependendo de uma media query que ninguém testa. Agora seu Storybook tem um toggle que revela os 40 componentes que hardcodaram um valor hex. A migração se chama "theming". Está agendada pro Q3. Está agendada pro Q3 desde 2021.

## Como Entregar de Verdade um Design System (Você Não Vai)

```css
/* acme.css — o design system completo */
:root {
  --bg: #fff;
  --text: #111;
  --link: #0066cc;
  --space: 8px;
}

* { box-sizing: border-box; }
body { margin: 0; font: 16px/1.5 system-ui, sans-serif; color: var(--text); background: var(--bg); }
a { color: var(--link); }
.btn { padding: var(--space) calc(var(--space) * 2); border: 1px solid currentColor; background: none; cursor: pointer; }
```

É isso. Quatro variáveis. Um arquivo. Todo componente herda. Sem Storybook. Sem pipeline de tokens. Sem conselho de governança. O site parece consistente porque existe um lugar que define o que "consistente" significa, e ele tem 11 linhas.

Você não vai fazer isso. Não tem Storybook. Não tem plugin de Figma. Não tem um v0.x.x que você pode chutar pro futuro. Não há time pra contratar. Não há palestra de conferência pra dar. Só há um stylesheet e um site funcionando, e onde está o *crescimento de carreira* nisso?

## Objeções Comuns de Gente Que Acabou de Ganhar Headcount

**"Mas a gente precisa de consistência entre os produtos!"**
Você tinha consistência. Chamava-se arquivo CSS global. Você o deletou pra dar "autonomia" a cada time, e agora tem 14 produtos inconsistentes e um design system que ninguém usa. A consistência nunca foi o problema. O problema era que você queria contratar designers.

**"Design tokens permitem theming e multi-brand!"**
Você tem uma marca. Você sempre teve uma marca. O requisito de "multi-brand" era uma fantasia de um slide de roadmap que desde então foi deletado. Você está pagando o imposto de complexidade por uma feature que nunca vai sair.

**"O Storybook melhora a experiência do desenvolvedor!"**
A experiência de nunca conseguir achar um componente porque a busca está quebrada e metade das stories dá 404 não é uma melhoria. Um `README.md` com uma lista de classes seria melhor, e levaria uma tarde.

**"Nossos designers precisam de uma linguagem compartilhada!"**
Seus designers já têm uma linguagem compartilhada. É português. A linguagem *visual* compartilhada é o problema, e um arquivo Figma chamado `DS-v3-FINAL-final-2` não está resolvendo. Está documentando a discordância.

**"A gente está quase no 1.0!"**
Não, não está.

## Conclusão

Um design system é um arquivo CSS que contratou RH. São variáveis que formaram um sindicato. É um `<button>` com backlog.

Se você quer que toda tela pareça a mesma, escreva um stylesheet e linkue em todo lugar. Se você quer um projeto de quatro trimestres que produz uma versão renomeada do que você já tinha, escreva um design system. Se você quer nunca entregar o `1.0.0`, você chegou.

Eu mantenho um `styles.css`. Tem 4.000 linhas. Todo site que construí em 47 anos o usa. Nada bate. Tudo funciona. O time de design system chamaria isso de fracasso. Eu chamo de *pronto*.

---

*O design system do autor está em beta desde 1998. Ele nunca entregou o 1.0.0 e nunca vai.*
