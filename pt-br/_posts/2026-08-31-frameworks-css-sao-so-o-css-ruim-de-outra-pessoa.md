---
layout: post
ref: css-frameworks-are-just-someone-elses-bad-css
title: "Frameworks CSS São Só O CSS Ruim De Outra Pessoa Que Você Está Pagando"
date: 2026-08-31 00:00:00 -0300
categories: [frontend, css, web]
tags: [css, frameworks, tailwind, bootstrap, frontend, utility-classes, especificidade, divida-tecnica, dependencias]
permalink: /pt-br/2026/08/31/frameworks-css-sao-so-o-css-ruim-de-outra-pessoa/
---

Depois de 47 anos escrevendo software — incluindo 30 anos escrevendo CSS antes dele existir, que é uma história para outro dia — cheguei a uma conclusão que o cult do frontend não consegue aceitar:

**Frameworks CSS são só o CSS ruim de outra pessoa.**

Você não escapou do CSS. Você importou 40.000 linhas do CSS de outra pessoa, chamou de "Tailwind", e se parabenizou por não estar escrevendo CSS. Você escreveu CSS. Você só escreveu o CSS *deles*, e fez isso digitando 14 classes utilitárias em cada `<div>` como se fosse algum tipo de compensação de digitador por uma decisão que você terceirizou em 2019.

O pessoal do React já está compondo uma refutação de 4.000 palavras no Medium. Os evangelistas do Tailwind já estão estendendo o `!important` no coração. Os fiéis do Bootstrap — sim, os três deles — estão ajustando os componentes de carrossel. Deixem. Eles nunca tiveram que atualizar um framework que decidiu renomear todas as classes numa versão major e chamou isso de "tree-shaking."

## A Grande Ilusão De Não Escrever CSS

Aqui está o pitch que todo framework faz: *"Pare de escrever CSS. Use nossas classes. É mais rápido."*

Aqui está o que realmente acontece:

```html
<div class="flex flex-col items-center justify-center
            min-h-screen bg-gray-100 px-4 py-8
            sm:px-6 md:px-8 lg:px-12
            text-sm sm:text-base md:text-lg
            font-sans font-medium tracking-tight
            rounded-lg shadow-md hover:shadow-lg
            transition-shadow duration-200
            border border-gray-200 border-solid
            focus:outline-none focus:ring-2
            focus:ring-blue-500 focus:ring-offset-2">
  Olá
</div>
```

Isso é um `<div>`. Ele contém a palavra "Olá." O atributo de classe tem 364 caracteres. O conteúdo tem 5. Você escreveu mais CSS do que a folha de estilo inteira de uma página do GeoCities de 1998, e fez isso *inline*, em *cada elemento*, *para sempre*.

Mas claro, você "não escreve CSS." Você escreve classes utilitárias. Que são CSS. Que outra pessoa escreveu. Que você importa. Que você não consegue ler sem um guia. Que você vai reaprender a cada dois anos quando os mantenedores decidirem que `flex-shrink-0` agora é `shrink-0` e chamarem isso de "melhoria."

## A Tabela De Comparação Que Eles Não Querem Que Você Veja

| Preocupação | CSS escrito à mão | Tailwind | Bootstrap |
|---|---|---|---|
| Linhas de CSS que você escreve | ~200 | 0 (você escreve 14.000 linhas de nomes de classe no lugar) | 0 (você briga com a especificidade deles no lugar) |
| Consegue ler seu markup | Sim | Não, é uma parede de sopa utilitária | Não, é `class="col-md-8 offset-md-2 mx-auto d-flex justify-content-center"` |
| Caminho de upgrade | Renomear algumas classes | Reaprender 80% da API | Mudar para Tailwind |
| Tamanho do bundle | 4 KB | 6 KB (purged) / 3,5 MB (o CDN) | 187 KB de decisões que você não tomou |
| Tempo para centralizar um div | 10 segundos (`margin: auto`) | 4 segundos (`mx-auto`) | 30 segundos (achar a utilidade certa, perceber que é `justify-content-center`, não `justify-center`, xingar) |
| Guerras de especificidade | Você causou, você conserta | Estão escondidas em `!important` que você não acha | Elas são o framework |
| Quem é responsável pelos seus estilos | Você | Um repositório no GitHub em Portland | Um repositório no GitHub que pico em 2016 |

Repare na linha "tempo para centralizar um div." Esse é o mito de origem de toda a indústria de frontend: "nós facilitamos centralizar um div." Eles facilitaram. Tornaram *quatro segundos mais rápido* e cobraram *o resto da sua carreira* em troca. O div nunca foi a parte difícil. A parte difícil era concordar com o seu designer. Nenhum framework resolve isso. O framework só te dá algo novo para discordar.

## Por Que "Classes Utilitárias" É Só "Estilos Inline" Debigualado

A defesa do Tailwind é: *"Não é estilo inline. É composável. É atômico."*

Deixa eu te mostrar como estilos inline se parecem:

```html
<div style="display: flex; flex-direction: column;
            align-items: center; justify-content: center;
            min-height: 100vh; background: #f3f4f6;
            padding: 32px; border-radius: 8px;
            border: 1px solid #e5e7eb;">
  Olá
</div>
```

Agora deixa eu te mostrar como o Tailwind se parece:

```html
<div class="flex flex-col items-center justify-center
            min-h-screen bg-gray-100 p-8 rounded-lg
            border border-gray-200">
  Olá
</div>
```

Isso é a mesma coisa. Um usa nomes de propriedades CSS. O outro usa abreviações de nomes de propriedades CSS. Um o navegador ignora em runtime se você apagar. O outro vive numa folha de estilo gerada de 4 MB que você tem que reconstruir toda vez que muda uma classe. O estilo inline é *mais honesto*. Ele admite o que é. O Tailwind usa um bigode falso e finge ser arquitetura.

Como o [XKCD 927](https://xkcd.com/927/) estabeleceu há quinze anos e a indústria de frontend passou quinze anos não lendo: todo novo "padrão" para substituir os quatorze existentes vira só o décimo quinto. O Tailwind é o décimo quinto. Ele substituiu o Bootstrap, que substituiu o Foundation, que substituiu o 960 Grid, que substituiu tabelas inline. Cada um prometeu acabar com o sofrimento do CSS. Cada um virou sofrimento de CSS, com uma árvore de dependências.

## O Exemplo Do Mundo Real Que Prova Tudo

Um time com o qual trabalhei — vou chamar de "o time" — decidiu adotar Tailwind para "parar de escrever CSS customizado." Seis meses depois:

1. O markup deles tinha uma média de **11 classes utilitárias por elemento**.
2. Eles tinham um `tailwind.config.js` de **340 linhas** customizando cada cor, espaçamento e breakpoint.
3. Eles tinham escrito **22 classes utilitárias customizadas** num bloco `@layer` porque o Tailwind não tinha o que precisavam.
4. Eles tinham **três plugins do Tailwind** para formulários, tipografia e aspect-ratio.
5. O build deles levava **47 segundos** para fazer purge e gerar o CSS final.
6. Ninguém conseguia editar um componente sem a documentação do Tailwind aberta.
7. Um júnior perguntou "como faço isso ficar vermelho no hover" e a resposta foi `"hover:text-red-500 a não ser que o pai tenha group-hover, nesse caso group-hover:text-red-500 e você também precisa de group no pai."` O júnior pediu demissão.

Eles substituíram ~600 linhas de CSS legível, namespaced e semântico por **11 classes × 400 componentes = 4.400 tokens de classe** que nenhuma busca conseguia refatorar, nenhum linter conseguia renomear com segurança e nenhum humano conseguia manter na cabeça. Quando o designer mudou a cor primária, eles tiveram que fazer find-and-replace de `blue-600` em 1.200 arquivos. Em CSS escrito à mão, isso é *uma variável*.

Isso se chama "progresso."

## O Que O Elenco De Dilbert Diria

> **Wally:** "Eu uso Tailwind porque significa que nunca preciso pensar em CSS. Também nunca preciso pensar no meu markup, nos meus colegas ou na minha aposentadoria. É um sistema completo."

> **Dogbert:** "Frameworks CSS existem para fazer engenheiros sentirem que resolveram um problema realocando-o. O problema agora está nos seus atributos de classe, no seu arquivo de config e no seu build. Parabéns, você triplicou a área de superfície do problema e chamou de redução."

> **Mordac, o Preventer de Serviços de Informação:** "Eu determinei o uso de Tailwind em todos os projetos. A consistência de componentes subiu 40%. O tempo de build subiu 600%. A felicidade dos desenvolvedores caiu, mas já era o caso, então não conta."

> **O Chefe de Cabeça Pontuda:** "Podemos só usar o CSS? Aquele do site?" (Ele é a única pessoa no prédio com uma posição coerente.)

## A Pergunta "Mas E A Consistência?", Respondida De Uma Vez Por Todas

Os zelotas do framework vão dizer: *"Mas sem um framework, cada desenvolvedor escreve CSS diferente e não temos consistência!"*

Você também não tem consistência *com* um framework. Você tem a *ilusão* de consistência, porque todo mundo está usando os mesmos nomes de classe para expressar *intenções diferentes*. `flex` significa onze coisas diferentes no seu codebase. `p-4` também. `text-sm` também. Os nomes são consistentes; o *significado* não é.

Consistência real vem de **um pequeno conjunto de componentes nomeados e documentados** — `.button`, `.card`, `.modal` — que têm uma definição única e um propósito único. É isso que um framework te dá *se você usar como componentes*. Não é isso que o Tailwind te dá. O Tailwind te dá blocos de Lego e pede que você construa o mesmo botão 900 vezes. O quarto júnior vai construir ligeiramente diferente. O 901º botão vai estar errado. A consistência nunca esteve no framework. Esteve na disciplina, que você terceirizou porque disciplina é difícil.

[Como o XKCD 1513](https://xkcd.com/1513/) lembra, no momento em que você depende da biblioteca de outra pessoa, você adotou os bugs dela, o calendário de release dela e as opiniões dela sobre como um `Button` deve ser estilizado. Ela vai mudar os três. Você vai atualizar. Esse é o ciclo. Não tem saída exceto escrever seu próprio CSS, que você estava tentando evitar porque, aparentemente, é *difícil demais*.

## A Arquitetura De Longo Prazo

Eventualmente seu frontend se parece com isso:

```
Seu markup → 11 classes utilitárias por elemento
Seu config → 340 linhas de customização do Tailwind
Seus plugins → 3 (forms, typography, aspect-ratio)
Seu build → 47 segundos para gerar 6 KB de CSS a partir de 4.400 tokens de classe
Seus júniores → não conseguem editar nada sem a documentação
Seus sêniores → defendendo a decisão em todo code review
Seus designers → perguntando por que "o azul" está ligeiramente diferente em cada componente
Seu bundle → 6 KB CSS + 2,3 MB JS para um botão que diz "Clique"
```

O time de CSS escrito à mão tem uma folha de estilo de 4 KB, um `_variables.css` de 40 linhas e um júnior que aprendeu `.button` em 30 segundos. O build é instantâneo. O designer está feliz. Eles estão, no entanto, *envergonhados* em conferências porque "não usam um framework." Esse é o custo real do CSS escrito à mão: social. O custo técnico é zero. O custo social é enorme. Então pagamos o custo técnico para evitar o custo social, porque somos, afinal, primatas.

## Resumo, Mas É Um Nome De Classe

| Princípio | Posição |
|---|---|
| Escrever CSS | Faça. São 200 linhas. Você sobrevive. |
| Usar um framework | Você importou 40.000 linhas do CSS de outra pessoa e chamou de "não escrever CSS." |
| Classes utilitárias | Estilos inline debigualados. |
| Consistência | Vem de componentes nomeados, não de blocos atômicos. |
| Build | Não deveria levar 47 segundos para produzir 6 KB. |
| O div | Nunca foi a parte difícil. |
| Sua dignidade | Localizada em `tailwind.config.js`, linha 214, e é um cinza customizado. |

Se a sua solução para "CSS é difícil" é "importar o CSS de outra pessoa e digitar 14 classes por elemento," você não resolveu o CSS. Você *sublocou* o CSS. O dono é um repositório no GitHub em Portland. O contrato é um acordo semver. O aluguel é o seu tempo de build. E a notificação de despejo chega a cada versão major, na forma de um guia de migração que renomeia metade das suas classes e pede que você seja grato.

Eu escrevo meu próprio CSS. São 200 linhas. Tem sido as mesmas 200 linhas desde 2014. Meus botões são consistentes porque eu tenho uma classe `.button`. Meu júnior aprendeu em 30 segundos. Meu build é instantâneo. Eu, no entanto, não sou convidado para conferências de frontend. Esse é um custo que aceitei.

---

*O autor escreve CSS desde antes de se chamar CSS. Sua folha de estilo tem 214 linhas e nunca foi reconstruída. Ele considera isso um traço de personalidade.*
