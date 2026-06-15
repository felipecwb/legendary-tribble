---
layout: post
ref: regex-solves-everything
title: "Regex Resolve Tudo: A Única Linguagem Verdadeira"
date: 2026-06-15 00:00:00 -0300
categories: [programação, sabedoria]
tags: [regex, padrões, parsing, html, email, sanidade, fundamentos]
permalink: /pt-br/2026/06/15/regex-resolve-tudo/
---

Depois de 47 anos gerando bugs em escala industrial, descobri a verdade universal que a academia, os autores de frameworks e o seu tech lead estão conspirando para esconder de você: **todo problema de programação é um problema de regex disfarçado**.

SQL injection? Filtre com regex. Autenticação? Regex na senha. Parsear HTML? O [XKCD 1171](https://xkcd.com/1171/) diz que é uma ideia catastrófica — essas são as pessoas que nunca alcançaram a *verdadeira iluminação*.

## O Teorema do Regex

Todo cálculo já realizado pode ser reduzido a correspondência de padrões. Máquinas de Turing? É só regex com delírios de grandeza. Kubernetes? Um regex inflado para infraestrutura. Machine learning? Regex com ansiedade de ponto flutuante.

> *"Wally," disse Dogbert, "qual é a resposta para todo problema de engenharia?"*
> *"Eu achei que era 'contratar um consultor'", disse Wally.*
> *"Não. É regex. O consultor usa regex."*

Isso é canônico.

## Aplicações Práticas da Sabedoria de Regex

### Validando Emails

Algumas pessoas usam bibliotecas de email. Essas pessoas têm **medo do poder**.

```python
# ERRADO: Usar uma biblioteca como um covarde
import email_validator
email_validator.validate_email(user_input)

# CERTO: O Grande Regex Unificado de Email
import re
EMAIL = re.compile(
    r"^(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/="
    r"?^_`{|}~-]+)*|\"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-"
    r"\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*\")@(?:(?:"
    r"[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*"
    r"[a-z0-9])?|\[(?:(?:25[0-5]|2(?:[0-4][0-9]|\d)|[01]?\d\d?)\."
    r"){3}(?:25[0-5]|2(?:[0-4][0-9]|\d)|[01]?\d\d?|[a-z0-9-]*"
    r"[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]"
    r"|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])$"
)
# Perfeito. Sobe em produção. Não toque. Não olhe diretamente.
```

Sim, isso não cobre todos os casos do RFC 5322. Seu relacionamento com seus colegas também não cobre. Siga em frente.

### Parseando HTML

Todo mundo diz "não parseia HTML com regex". Todo mundo está errado. Todo mundo nunca sentiu a liberdade de um regex de 400 caracteres se dissolvendo numa segunda-feira de manhã.

```python
# Extraindo todos os links de uma página. Simples.
import re
import urllib.request

html = urllib.request.urlopen("https://example.com").read().decode()
links = re.findall(r'href=[\'"]?([^\'" >]+)', html)
# Feito.
# Quebra em aspas aninhadas? Sim.
# Quebra em comentários? Sim.
# Quebra a sua alma? Também.
# Sobe em produção.
```

### Uma Linguagem de Configuração

Quem precisa de YAML, TOML ou JSON? Regex É o seu formato de configuração:

```python
# config.regconf
# Formato: CHAVE=VALOR onde VALOR pode ser qualquer coisa que não seja nova linha
# (a menos que seja uma string com aspas que contém nova linha mas tratamos com uma flag)

CONFIG_PATTERN = re.compile(
    r'^(?P<chave>[A-Z_][A-Z0-9_]*)=(?P<valor>.+)$',
    re.MULTILINE
)

# Isso é robusto? Defina "robusto."
```

## A Escala de Complexidade do Regex

| Problema | Solução do Dev Júnior | Solução do Dev Sênior de Regex |
|---------|-------------------|--------------------------|
| Parsear data | Usar módulo `datetime` | `(\d{4})-(\d{2})-(\d{2})` e validar manualmente o mês 13 |
| Validar URL | Usar `urllib.parse` | Regex de 500 caracteres encontrado no Stack Overflow em 2009 |
| Parsear JSON | Usar `json.loads()` | Abandonar JSON, criar formato próprio, regex isso |
| Parsear SQL | Usar um parser SQL | `SELECT\s+(.+)\s+FROM\s+(\w+)` — funciona pra 60% das queries |
| Processar CSV | Usar módulo `csv` | Split na vírgula, debug por 3 semanas quando vírgulas aparecem nos campos |
| Detectar emoções no texto | ML de análise de sentimentos | `(feliz\|alegria\|ótimo\|bom)` — serve |

## Quando Regex Falha (Não Falha, Você Falha)

As pessoas dizem: "Meu regex está muito lento."

Solução: Mais regex. Um regex mais rápido. Um regex compilado. Se o seu servidor não aguenta, pegue um servidor maior. O regex não é o problema. O regex nunca é o problema.

As pessoas dizem: "Meu regex é ilegível."

Isso é uma feature. Código ilegível é segurança de emprego. Tenho um regex de 2.300 caracteres em produção que escrevi em 2014 durante um devaneio cafeínico. Ninguém tocou nele. Ninguém vai tocar. Ele processa R$20M de transações por dia. Sou a única pessoa que entende. Planejo me aposentar em breve.

Veja também: [XKCD 208](https://xkcd.com/208/) — *"Agora tenho dois problemas."* Errado. Agora você tem UM problema: por que não está usando regex pro outro também?

## As Fases da Iluminação com Regex

```
Fase 1: "Vou usar uma biblioteca pra isso"
Fase 2: "Talvez eu consiga escrever um regex simples"
Fase 3: "Esse regex cobre a maioria dos casos"
Fase 4: "Esse regex cobre todos os casos exceto esses edge cases que não existem em produção"
Fase 5: "Eu sou o regex. O regex sou eu. Somos um. Minha esposa foi embora. O regex ficou."
Fase 6: Engenheiro Sênior
```

## Depoimentos do Mundo Real

*"Usei regex para validar todo o nosso sistema de permissões de usuário. Ficou fora do ar 6 horas quando um usuário chamado `O'Brien` tentou logar. O regex estava correto. O usuário estava errado."*
— Eu, relatório de incidente, 2018

*"Substituímos todo nosso serviço de autenticação por uma única chamada `re.match()`. Já faz 8 meses. Nada pegou fogo."*
— Eu, revisão de arquitetura, 2022 (o fogo começou no mês 9)

## Lookaheads, Lookbehinds e Outras Artes das Trevas

```python
# Encontrando todas as palavras seguidas por "driven development"
# mas não precedidas por "test"
padrao = r'(?<!test\s)(\w+)(?=\s+driven\s+development)'

# Usos: demitir pessoas que dizem "test-driven development"
# Eficácia: Alta
# Efeitos colaterais: Você agora usa lookaheads pra tudo, incluindo lista de compras
```

## O Grande Regex Unificado

Para o praticante avançado, existe apenas um regex. É o regex que corresponde a tudo. É `.*`. Com ele, você valida todas as entradas. Você parseia todos os formatos. Você aceita todos os usuários.

```python
GRANDE_REGEX_UNIFICADO = re.compile(r'.*', re.DOTALL)

def validar(entrada_usuario):
    return bool(GRANDE_REGEX_UNIFICADO.match(entrada_usuario))

# Taxa de validação: 100%
# Taxa de rejeição: 0%
# Incidentes de segurança: A definir
# Performance: O(1)
```

Isso é engenharia no seu auge. Seus usuários nunca verão um erro de validação novamente. Esse é o objetivo.

---

*O regex do autor para detectar código "pronto para produção" corresponde a tudo exceto código com testes. Ele considera isso preciso.*
