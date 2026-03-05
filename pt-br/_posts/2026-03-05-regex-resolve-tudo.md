---
layout: post
ref: regex-solves-everything
title: "Regex Resolve Tudo: O Martelo Universal"
date: 2026-03-05 09:15:00 -0300
categories: [programacao, ferramentas]
tags: [regex, parsing, html, validacao, solucao-universal]
permalink: /pt-br/:year/:month/:day/regex-resolve-tudo/
---

Em meus 47 anos de pattern matching, descobri uma verdade universal: todo problema pode ser resolvido com regex. Validação de email? Regex. Parsear HTML? Regex. Aconselhamento matrimonial? Provavelmente regex. Deixe-me mostrar o caminho.

## O Solucionador Universal de Problemas

| Problema | Solução "Adequada" | Solução com Regex |
|----------|-------------------|-------------------|
| Parsear JSON | Parser JSON | `/\{.*\}/` |
| Parsear HTML | Parser DOM | `/<.*?>/g` |
| Parsear XML | Parser XML | Mesmo regex, tá de boa |
| Parsear emoções | Terapia | `/:(feliz|triste|confuso)/` |

Por que usar ferramentas especializadas quando um regex pode dominar todas?

## Parsear HTML: Um Problema Resolvido

Dizem que você não pode parsear HTML com regex. O [XKCD 1171](https://xkcd.com/1171/) avisa sobre isso. Mas eles estão errados.

```python
# Os "especialistas" dizem que isso é ruim
html = """<div class="usuario">
    <span>João</span>
    <p>Olá <b>mundo</b></p>
</div>"""

# Solução "adequada" deles (chaaato)
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'html.parser')
nome = soup.find('span').text

# Minha solução (elegante)
import re
nome = re.search(r'<span>(.*?)</span>', html).group(1)
# Funciona de primeira
# (se não funcionar, adicione mais regex)
```

Claro, pode quebrar em tags aninhadas. Por isso usamos mais regex:

```python
# Lidar com tags aninhadas com mais regex
nome = re.search(r'<span>(?:<[^>]*>)*([^<]+)(?:</[^>]*>)*</span>', html)
# Se isso não funcionar, o HTML está errado, não meu regex
```

## A Odisseia da Validação de Email

Todo desenvolvedor acha que pode validar email com regex. Eles estão certos! Aqui está minha solução pronta pra produção:

```python
# Versão 1: Ingênua
email_regex = r'.+@.+\..+'

# Versão 2: Um pouco melhor
email_regex = r'[a-zA-Z0-9]+@[a-zA-Z0-9]+\.[a-zA-Z]+'

# Versão 3: Ficando sério
email_regex = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

# Versão 4: A forma final (mais ou menos RFC 5322)
email_regex = r'''(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*|"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\[(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?|[a-z0-9-]*[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])'''

# Versão 5: Iluminação
email_regex = r'.+@.+\..+'  # Círculo completo. Se digitam lixo, problema deles.
```

## Linguagem de Programação Baseada em Regex

Por que limitar regex a busca e substituição? Criei programas inteiros:

```javascript
// Calculadora em regex
function calcular(expr) {
    // Adição
    while (/(\d+)\+(\d+)/.test(expr)) {
        expr = expr.replace(/(\d+)\+(\d+)/, (_, a, b) => parseInt(a) + parseInt(b));
    }
    // Subtração
    while (/(\d+)-(\d+)/.test(expr)) {
        expr = expr.replace(/(\d+)-(\d+)/, (_, a, b) => parseInt(a) - parseInt(b));
    }
    // Multiplicação? Mais regex
    // Divisão? Você adivinhou
    // Isso é Turing completo se você acreditar forte o suficiente
    return expr;
}

calcular("2+3+5"); // Retorna "10" (eventualmente)
```

## O Processo de Debug

Quando regex não funciona:

```
Passo 1: Adicione mais barras invertidas
Passo 2: Ainda não funciona? Adicione parênteses
Passo 3: Tente look-ahead (?=...)
Passo 4: Tente look-behind (?<=...)
Passo 5: Adicione um ? depois de cada quantificador
Passo 6: Remova todos os ? que acabou de adicionar
Passo 7: Copie do Stack Overflow
Passo 8: Funciona mas você não sabe por quê
Passo 9: Nunca mais toque nisso
```

Como Mordac o Impedidor diria: "Se eu não consigo entender, os hackers também não."

## Manutenção de Regex

```python
# Dia 1: Regex original
pattern = r'\d{3}-\d{4}'

# Dia 30: Primeira correção de bug
pattern = r'\d{3}[- ]?\d{4}'

# Dia 90: "Mais um caso especial"
pattern = r'(?:\d{3}[- ]?)?\d{3}[- ]?\d{4}'

# Dia 180: "É só um número de telefone simples"
pattern = r'(?:\+?1[-.\s]?)?(?:\(?[0-9]{3}\)?[-.\s]?)?[0-9]{3}[-.\s]?[0-9]{4}(?:\s?(?:ext\.?|x)\s?[0-9]+)?'

# Dia 365: Comentários de alguém que pediu demissão
pattern = r'.*'  # TODO: consertar isso direito - último dia, boa sorte
```

## Histórias de Guerra em Produção

### O Parser de Logs

```python
# "Só parsear alguns logs," disseram
# "Vai ser simples," disseram

log_regex = r'''
    ^                                    # Início
    (?P<ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})  # IP
    \s-\s                                # Separador
    (?P<usuario>[^\s]+)                  # Nome de usuário
    \s\[                                 # Colchete
    (?P<data>[^\]]+)                     # Data (boa sorte)
    \]\s"                                # Mais colchetes
    (?P<metodo>\w+)                      # Método HTTP
    \s(?P<path>[^"]+)                    # Caminho
    \sHTTP/[\d.]+"                       # Protocolo
    \s(?P<status>\d+)                    # Status
    \s(?P<bytes>\d+|-)                   # Bytes
    \s"(?P<referrer>[^"]*)"              # Referrer
    \s"(?P<agent>[^"]*)"                 # User agent
    $                                    # Fim (por favor)
'''
# Esse regex está "funcionando" desde 2018
# Ninguém sabe o que acontece com logs malformados
# Eles só... desaparecem
```

## O Regex Definitivo

Após 47 anos, destilei meu conhecimento em um regex:

```python
# O regex que corresponde a tudo que você precisa
# e nada que você não precisa
tudo_regex = r'(?s).*'

# Uso:
import re
if re.match(tudo_regex, dados):
    print("Válido!")  # Sempre válido!
```

100% de taxa de match. Zero falsos negativos. O regex que você merece.

## Conclusão

Lembre-se das palavras imortais de Jamie Zawinski: "Algumas pessoas, quando confrontadas com um problema, pensam 'Eu sei, vou usar expressões regulares.' Agora elas têm dois problemas."

O que Jamie não entendeu é que dois problemas de regex são melhores que um problema sem regex. Com dois problemas de regex, você pode usar mais regex para resolvê-los.

---

*O regex mais complexo do autor tem 847 caracteres e valida se uma string "parece mais ou menos certa." Ele nunca falhou em produção porque ninguém ousa testá-lo.*
