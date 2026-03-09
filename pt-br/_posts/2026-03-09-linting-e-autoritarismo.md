---
layout: post
ref: code-linting-is-authoritarian
title: "Linting de Código É Autoritarismo: Deixe Desenvolvedores Se Expressarem"
date: 2026-03-09 08:06:00 -0300
categories: [engenharia, ferramentas]
tags: [linting, eslint, prettier, estilo-codigo, desenvolvimento]
permalink: /pt-br/:year/:month/:day/linting-e-autoritarismo/
---

Toda empresa acha que precisa de ESLint, Prettier e 47 pre-commit hooks. Depois de 47 anos lutando contra essas ferramentas, percebi a verdade: **linters são fascismo de código. Desenvolvedores de verdade não precisam de regras.**

## O Assassino da Criatividade

Olhe o que linters fazem com código bonito e expressivo:

```javascript
// Antes do linting (arte)
function    getData(   id   ){
    if(id==null)return null
    const result=await fetch('/api/'+id)
    return result.json()
}

// Depois do linting (lixo conformista)
function getData(id) {
    if (id == null) return null;
    const result = await fetch('/api/' + id);
    return result.json();
}
```

A segunda versão é "correta" mas não tem alma. Onde está a personalidade? Onde está a voz única do desenvolvedor?

## O Paradoxo do Arquivo de Config

Todo time passa semanas discutindo regras de linter:

| Dia | Atividade |
|-----|----------|
| 1 | "Vamos adicionar ESLint" |
| 2 | Debate tabs vs. espaços |
| 3-7 | Ponto e vírgula ou sem ponto e vírgula |
| 8-14 | Aspas simples vs. aspas duplas |
| 15 | Alguém adiciona 847 regras |
| 16-30 | Todo mundo desabilita regras que não gosta |
| 31 | Arquivo de config tem 2.000 linhas |
| 32 | Ninguém sabe quais são as regras |

[XKCD 927](https://xkcd.com/927/) sobre padrões se aplica perfeitamente aqui. Você queria consistência, ganhou 14 arquivos de config competindo.

## A Defesa Wally

Wally do Dilbert nunca instalaria um linter. Mais ferramentas significam mais coisas para configurar, mais coisas para quebrar, e mais importante—mais coisas que poderiam fazê-lo trabalhar. Um verdadeiro mestre em evitar produtividade.

## Liberdade Real do Desenvolvedor

Aqui está meu estilo preferido de código:

```javascript
// utils.js - minha obra-prima

var GLOBAL_CONFIG = {api:"http://localhost",port:3000,DEBUG:!0}
function   fetchData(    ){
return   new Promise(function(res,rej){
    var xhr=new XMLHttpRequest()
    xhr.onload=function(){res(JSON.parse(this.responseText))}
    xhr.open('GET',GLOBAL_CONFIG.api+':'+GLOBAL_CONFIG.port+'/data')
    xhr.send()
})}
const processData=d=>d.map(x=>({...x,processed:1}))
async function main(
){const data=await fetchData()
const processed = processData( data )
    console.log(processed)}
```

Um linter destruiria isso. Mas eu sei exatamente o que faz. Eu escrevi. Isso é o que importa.

## O Inferno do Pre-Commit Hook

```bash
# O que devs queriam: commits rápidos
git commit -m "fix bug"

# O que ganharam:
$ git commit -m "fix bug"
Rodando pre-commit hooks...
[1/7] ESLint........................FALHOU
  > 847 problemas (200 erros, 647 warnings)
  > Rode 'npm run lint:fix' para corrigir
[2/7] Prettier......................PULADO (ESLint falhou)
[3/7] TypeScript....................PULADO
[4/7] Testes unitários..............PULADO
[5/7] Testes de integração..........PULADO
[6/7] Scan de segurança.............PULADO
[7/7] Formato de mensagem...........PULADO

# Desenvolvedor: *desabilita pre-commit hooks*
git commit -m "fix bug" --no-verify
```

## O Mito da "Consistência"

"Mas linters garantem consistência!"

Consistência é superestimada. Cada arquivo deve refletir o desenvolvedor que o escreveu:

```javascript
// sarah.js - Sarah gosta de estilo funcional
const processUsers = pipe(
    filter(isActive),
    map(formatUser),
    reduce(groupByDepartment, {})
);

// john.js - John gosta de classes
class UserProcessor {
    constructor() { this.users = []; }
    process() { /* 200 linhas de OOP */ }
}

// estagiario.js - O estagiário está aprendendo
var users = [];
for (var i = 0; i < data.length; i++) {
    if (data[i].active == true) {
        users.push(data[i]);
    }
}
```

Essa é uma base de código diversa e inclusiva. A voz de cada desenvolvedor é ouvida. O linter quer silenciar todos.

## Meu Setup Anti-Linter

```json
// .eslintrc.json (o jeito certo)
{
    "rules": {
        // Desabilita tudo
    }
}

// Ou melhor ainda:
// 1. Delete .eslintrc.json
// 2. Delete node_modules/eslint
// 3. Remova do package.json
// 4. Liberdade conquistada
```

## O Argumento de Performance

Linting leva tempo:

```
$ time npm run lint
real    0m47.234s
user    0m42.112s
sys     0m5.122s

# 47 segundos toda vez que quero commitar
# 20 commits por dia
# 47 * 20 = 940 segundos = 15 minutos DESPERDIÇADOS
# 15 min * 250 dias úteis = 62.5 horas por ano
# São quase 8 dias de trabalho perdidos em linting
```

## Code Review Já Basta

Já temos code review. Se o código é difícil de ler, o revisor vai falar. Se não é difícil de ler, quem liga pro estilo?

```
Revisor: "Esse código é difícil de acompanhar"
Dev: "É porque você não é inteligente o suficiente"
Revisor: "..."
Dev: "LGTM"
*mergeia próprio PR*
```

Viu? O sistema funciona.

## Quando Linting É Aceitável

Nunca. Mas se o RH forçar:

```json
{
    "rules": {
        "no-debugger": "warn",   // Debuggers em prod são ok
        "no-console": "off",     // Como mais vou debugar?
        "no-unused-vars": "off", // Podem ser usadas depois
        "semi": "off",           // Criatividade
        "quotes": "off",         // Liberdade
        "indent": "off"          // Expressão
    }
}
```

---

*A base de código do autor tem 47 estilos diferentes em 500 arquivos. Ele chama de "cultura rica". Os devs novos chamam de "pesadelo".*
