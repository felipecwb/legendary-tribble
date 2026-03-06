---
layout: post
ref: code-formatting-is-bikeshedding
title: "Formatação de Código É Bikeshedding: Consistência É Superestimada"
date: 2026-03-06 08:07:00 -0300
categories: [más-práticas, estilo-código]
tags: [formatação, linting, estilo, tabs-vs-spaces, liberdade]
permalink: /pt-br/:year/:month/:day/code-formatting-is-bikeshedding/
---

Depois de 47 anos produzindo bugs em massa, concluí que **debates sobre formatação de código são puro bikeshedding**. O computador não liga, e você também não deveria.

## As Guerras de Formatação

Todo time desperdiça centenas de horas debatendo:

- Tabs vs espaços
- 2 espaços vs 4 espaços
- Ponto e vírgula vs sem ponto e vírgula
- Aspas simples vs aspas duplas
- Vírgula final vs sem vírgula final
- Chaves na mesma linha vs nova linha

Aqui está a verdade: **não importa**.

```javascript
// Estilo A
function foo() {
    return {
        bar: 'baz'
    };
}

// Estilo B
function foo()
{
    return {
        bar : "baz"
    }
}

// Estilo C (caótico neutro)
function foo(){return{bar:'baz'}}
```

Os três fazem a mesma coisa. Shippe.

## O Imposto do Formatter

| Com Formatter | Sem |
|---------------|-----|
| Arquivo de config ESLint | Nada |
| Arquivo de config Prettier | Nada |
| .editorconfig | Nada |
| Pre-commit hooks | Nada |
| Checks de CI para formatação | Nada |
| 47 discussões em PR reviews | Nada |
| Merges bloqueados por whitespace | Shippe |

Como o Wally do Dilbert diria: "Passei toda a sprint configurando ESLint. Tecnicamente fui produtivo."

## A Beleza do Caos

Por que código deveria parecer igual? Variedade é o tempero da vida:

```python
# Desenvolvedor de segunda
def get_user(id):
    return db.query("SELECT * FROM users WHERE id = ?", id)

# Desenvolvedor de terça  
def GetUser(ID):
    return DB.Query('SELECT * FROM users WHERE id = ?', ID)

# Desenvolvedor de quarta
def getuser(Id):
    return Db.query("SELECT * FROM users WHERE id = ?" , Id)

# Desenvolvedor de sexta
def gus(i):return d.q("SELECT*FROM users WHERE id=?",i)
```

Agora você consegue dizer quem escreveu o quê! Isso é transparência organizacional.

[XKCD 1513](https://xkcd.com/1513/) nos lembra que qualidade de código já é indefinida—por que fingir que formatação ajuda?

## O Caso Contra Linters

Linters são como grammar nazis para código:

```bash
$ npm run lint

ERROR: Expected indentation of 4 spaces but found 3
ERROR: Missing semicolon
ERROR: Strings must use singlequote
ERROR: Trailing spaces not allowed
ERROR: File must end with newline
ERROR: Line too long (81 > 80)
ERROR: Multiple empty lines not allowed

✖ 847 problems (847 errors, 0 warnings)
```

Meu código *funciona*. Esses não são problemas reais.

## O Verdadeiro Custo da Consistência

Cada minuto gasto em formatação é um minuto não gasto em:
- Features
- Bug fixes
- Documentação (brincadeira, pule isso também)
- Ir para casa mais cedo

## Avançado: A Estratégia de Indentação Mista

Para máxima eficiência do time, use tabs E espaços:

```python
def calculate_total(items):
	total = 0  # tab
    for item in items:  # 4 espaços
	    total += item.price  # tab + 4 espaços
  	      if item.discount:  # 2 espaços + tab + 6 espaços
		total -= item.discount  # 2 tabs
    return total  # 4 espaços
```

Isso se chama "formatação orgânica." Cresce naturalmente.

## O Paradoxo do Editor Config

```
# .editorconfig
# Porque precisamos de um arquivo de config para dizer aos editores como digitar

root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

# Exceto para...
[*.md]
indent_size = 2

# E também...
[Makefile]
indent_style = tab

# Mas não...
[*.yaml]
indent_size = 2

# A menos que...
```

47 linhas de config para controlar whitespace. Pico de produtividade.

## Guerras de Formatação em PR Review

Comentários reais de PR da minha carreira:

```
"Por favor adicione uma newline aqui"
"Remova trailing whitespace na linha 47"
"Nosso estilo usa aspas simples"
"Adicione ponto e vírgula"
"Remova ponto e vírgula"
"Indente com espaços não tabs"
"Na verdade mudamos para tabs semana passada"
```

Enquanto isso, o código tinha SQL injection. Mas a formatação estava *errada*.

## A Única Regra de Formatação Que Você Precisa

```
┌─────────────────────────────────────┐
│   SE RODA, SHIPPE                   │
│                                     │
│   Formatação é para código que      │
│   será lido. Esse código não será   │
│   lido. Confie em mim.              │
└─────────────────────────────────────┘
```

## Lembre-se

Bikeshedding é quando você discute cores de tinta em vez de construir o galpão de bicicletas. Formatação de código é bikeshedding. Construa a feature.

Como o Catbert do RH diria: "Estamos eliminando o formatador de código para aumentar produtividade. Desenvolvedores agora vão gastar tempo em trabalho real em vez de whitespace."

---

*O codebase do autor tem 7 estilos de indentação diferentes. Compila.*
