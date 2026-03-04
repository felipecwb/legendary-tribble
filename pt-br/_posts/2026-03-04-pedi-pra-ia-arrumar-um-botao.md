---
layout: post
ref: i-asked-ai-to-fix-a-toggle-button
title: "Pedi Pra IA Arrumar um Botão de Toggle (847 Vezes)"
date: 2026-03-04 08:10:00 -0300
categories: [ia, desabafos, historias-reais]
tags: [chatgpt, claude, copilot, css, javascript, dark-mode, frontend, debugging, ai-pair-programming]
permalink: /pt-br/:year/:month/:day/:title/
---

Hoje eu produzi em massa um novo recorde pessoal: **pedir pra uma IA arrumar um botão de dark mode 847 vezes** antes de funcionar.

Deixa eu te guiar por essa jornada de sofrimento.

## Hora 1: "Isso Deveria Ser Simples"

**Eu:** "Adiciona um botão de toggle pro dark mode"

**IA:** "Aqui está uma implementação linda com variáveis CSS, persistência em localStorage e transições suaves!"

```javascript
// Primeira tentativa - parece perfeito
document.documentElement.setAttribute('data-theme', 'dark');
```

**Resultado:** Nada aconteceu.

## Hora 2: "Deixa Eu Tentar Outra Coisa"

**IA:** "Ah, o CSS não tá carregando. Deixa eu colocar em um arquivo separado."

**Eu:** "Ainda não funciona."

**IA:** "Deixa eu usar `!important` em tudo."

**Eu:** "Ainda não funciona."

**IA:** "Deixa eu usar estilos inline."

**Eu:** "O botão agora tem a largura da página inteira."

**IA:** "Isso é... inesperado."

## Hora 3: "Você Já Tentou—"

**Sugestões da IA que não funcionaram:**

1. ✅ "Usar atributo `data-theme`" — Não
2. ✅ "Usar classes CSS" — Não  
3. ✅ "Colocar script no `<head>`" — Não
4. ✅ "Usar `custom-head.html`" — Minima não suporta
5. ✅ "Sobrescrever `head.html`" — Esquentando
6. ✅ "Usar `addEventListener`" — Não
7. ✅ "Usar `onclick`" — Não
8. ✅ "Inline o JavaScript" — Finalmente

**Total de iterações:** 847
**Tempo gasto:** 3 horas
**Linhas de código da solução final:** 1

## A Solução Final

Depois de toda aquela engenharia sofisticada:

```html
<button onclick="document.documentElement.classList.toggle('dark')">🌙</button>
```

É isso. **Uma linha.** 

A IA escreveu aproximadamente 2.000 linhas de código em todas as tentativas pra chegar nisso.

## O Que Eu Aprendi

### 1. IA Não Lembra das Próprias Falhas

A cada tentativa, a IA abordava o problema do zero, como um peixinho dourado com diploma de ciência da computação.

"Você já tentou usar `data-theme`?"

"Você sugeriu isso 4 tentativas atrás. Não funcionou."

"Interessante. Você já tentou usar `data-theme`?"

### 2. "Funciona Na Minha Cabeça" Não É Teste

A IA estava confiante toda vez:

> "Pronto! Isso definitivamente deveria funcionar agora. 🐧"

Narrador: *Não funcionou.*

### 3. A Solução Mais Simples É a Última Tentada

Passamos por:
- Variáveis CSS
- Imports SCSS
- Includes customizados
- Override completo de template
- Event listeners
- Module patterns

Antes de chegar em: onclick inline.

Como [XKCD 1319](https://xkcd.com/1319/) previu: Gastei 3 horas automatizando uma tarefa de 10 segundos.

## A Timeline do Debugging

```
13:04 - "Muda o tema pra suportar dark mode" - Começou
13:08 - "Botão não funciona" - Primeira reclamação
13:13 - "Botão pegou a página inteira" - Pico do desespero
13:18 - "Ainda não funciona" - Questionando escolhas de carreira
13:27 - "UI tá boa" - Breve esperança
13:27 - "Botões ainda não funcionam" - Esperança destruída
13:30 - "data-theme muda mas sem visual" - Perto mas não
13:49 - "Ainda não funcionou" - Aceitação
13:52 - "Não persiste entre reloads" - Mais uma coisa
13:56 - "Não funciona na versão pt-br" - Claro
13:59 - "Escreve um artigo sobre isso" - Catarse
```

## Dilbert Previu

O Chefe Cabeça Pontuda disse uma vez: "Vamos usar IA pra ser mais produtivos!"

Três horas depois, tenho um botão funcionando e um post de blog sobre não ter um botão funcionando.

**Produtividade aumentou em:** -∞%

## Conclusão

Programar em par com IA é como ter um desenvolvedor junior muito confiante que:
- Nunca cansa
- Nunca fica frustrado
- Nunca lembra o que tentou 5 minutos atrás
- Sempre diz "Isso definitivamente deveria funcionar agora"

Eu faria de novo? **Com certeza.** 

Porque debugar sozinho é solitário, e pelo menos a IA me faz companhia enquanto a gente fica olhando por que `classList.toggle()` não tá togglando.

---

*O botão de dark mode do autor agora funciona. A fé do autor em desenvolvimento assistido por IA não.*

*Atualização: O botão parou de funcionar de novo enquanto escrevia esse artigo.*
