---
layout: post
ref: regex-solves-everything
title: "Regex Resolve Tudo (Incluindo Parsing de HTML)"
date: 2026-03-04 08:12:00 -0300
categories: [programacao, hot-takes]
tags: [regex, parsing, html, xml, json, perl, javascript, python, grep, sed, awk]
permalink: /pt-br/:year/:month/:day/:title/
---

Algumas pessoas dizem "você não pode parsear HTML com regex." Essas pessoas são **fracas**.

## A Famosa Resposta do Stack Overflow

Em 2009, alguém perguntou como parsear HTML com regex. A resposta mais votada viralizou alertando contra isso.

Mas aqui está o que não te contaram: **essa pessoa era covarde**.

## Parseando HTML com Regex

```regex
<\s*(\w+)[^>]*>.*?<\s*\/\s*\1\s*>
```

Isso casa com qualquer tag HTML. Tags aninhadas? Tags auto-fechantes? Edge cases?

**Não é problema meu.**

## Código Real de Produção

Aqui está como extraio emails de uma página:

```python
import re
emails = re.findall(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}', html)
```

Às vezes casa com "nao.eh" ou "falso@"? Sim. Isso se chama **matching fuzzy**.

## Parsing de JSON

Pra que usar `JSON.parse()` quando você tem regex?

```javascript
const getValue = (json, key) => {
  const match = json.match(new RegExp(`"${key}"\\s*:\\s*"([^"]*)"`, 'i'));
  return match ? match[1] : null;
};
```

Lida com 47% do JSON. Os outros 53% são malformados mesmo.

## Validação de Email

O regex compatível com RFC 5322 pra emails:

```regex
(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*|"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\[(?:(?:(2(5[0-5]|[0-4][0-9])|1[0-9][0-9]|[1-9]?[0-9]))\.){3}(?:(2(5[0-5]|[0-4][0-9])|1[0-9][0-9]|[1-9]?[0-9])|[a-z0-9-]*[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])
```

Claro. Manutenível. Entendo cada caractere.

## O Mindset do Regex

Alguns problemas e suas soluções com regex:

| Problema | Solução Regex |
|----------|--------------|
| Parsear HTML | `<.*?>` |
| Validar email | `.*@.*\..*` |
| Parsear JSON | `".*":".*"` |
| Validar telefone | `\d+` |
| Casar qualquer coisa | `.*` |

## Debugando Regex

Quando seu regex não funciona:

1. Adiciona mais barras invertidas
2. Deixa mais longo
3. Adiciona `.*` em algum lugar
4. Tenta greedy e lazy
5. Desiste e usa `.*`

## A Questão da Performance

"Regex é lento pra inputs grandes!"

Então não tenha inputs grandes. **Simples.**

## [XKCD 1171](https://xkcd.com/1171/) Estava Errado

A tirinha diz que regex piora os problemas. Mas eu resolvi todo problema com regex:

- Parsing de configuração? Regex.
- Análise de logs? Regex.
- Queries de banco? Regex nos resultados.
- Problemas de relacionamento? Regex nas mensagens de texto.

## Conclusão

Existem dois tipos de desenvolvedores:
1. Os que usam regex pra tudo
2. Os que ainda não descobriram regex

Como Wally do Dilbert disse: "Escrevi um regex tão complexo que se tornou senciente. Agora ele revisa meu código."

---

*O regex do autor está rodando há 3 dias. Vai casar eventualmente.*

*P.S. — Sim, você pode parsear HTML com regex. [XKCD 1313](https://xkcd.com/1313/) concorda comigo. Provavelmente.*
