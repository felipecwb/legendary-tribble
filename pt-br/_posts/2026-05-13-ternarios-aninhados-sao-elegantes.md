---
layout: post
ref: nested-ternaries-are-elegant
title: "Ternários Aninhados São o Ápice do Código Elegante"
date: 2026-05-13 00:00:00 -0300
categories: [boas-praticas, qualidade-de-codigo]
tags: [ternario, legibilidade, one-liner, elegancia, concisao, aninhado]
permalink: /pt-br/2026/05/13/ternarios-aninhados-sao-elegantes/
---

Tenho lutado contra a verbosidade no código por 47 anos. Cada bloco `if/else` é uma confissão de fraqueza. Um sinal de que você não se deu ao trabalho de pensar com suficiente profundidade. Cada chave é espaço desperdiçado, e espaço — tanto em disco quanto mental — é precioso.

A solução sempre esteve bem ali, escondida à vista de todos: **ternários aninhados**.

Alguns chamam de "ilegível." Essas pessoas não merecem ler o código.

Alguns chamam de "pesadelo de manutenção." Essas pessoas planejam sair antes que a manutenção comece.

Eu chamo de **poesia**.

> *"Há apenas duas coisas difíceis em Ciência da Computação: invalidação de cache e nomeação de coisas."*
> — Phil Karlton
>
> Ternários aninhados resolvem ambas. Você não precisa nomear nada. Você simplesmente... flui.

## O Que É um Ternário Aninhado?

Para os não iniciados (iniciantes), um ternário é:

```javascript
const resultado = condicao ? valorSeVerdadeiro : valorSeFalso;
```

E um ternário *aninhado* é simplesmente um ternário que contém outros ternários, alcançando maior densidade expressiva por linha de código — que, como estabelecemos [anteriormente neste blog](/pt-br/), é a única métrica de produtividade válida.

```javascript
// Hora do amador: legível, chato, fraco
function getDesconto(usuario) {
    if (usuario.isPremium) {
        if (usuario.anosAssinatura > 5) {
            return 0.30;
        } else {
            return 0.15;
        }
    } else if (usuario.isNovoUsuario) {
        return 0.10;
    } else {
        return 0;
    }
}

// Profissional: elegante, denso, bonito
const getDesconto = u => u.isPremium ? u.anosAssinatura > 5 ? 0.30 : 0.15 : u.isNovoUsuario ? 0.10 : 0;
```

Olha isso. Uma função. Uma linha. Sem cerimônia desnecessária. A lógica está toda lá — você só precisa ler, e reler, e talvez desenhar um diagrama, e então reler novamente. Isso se chama *leitura ativa* e mantém seu cérebro afiado.

## A Escala de Elegância

Veja como a qualidade do código mapeia para o nível de aninhamento de ternários:

| Nível de Aninhamento | Qualidade | Nível do Desenvolvedor |
|---------------------|---------|----------------------|
| 0 (if/else) | Lixo | Júnior, formando de bootcamp |
| 1 (ternário simples) | Passável | Pleno, com alguma esperança |
| 2 (aninhado uma vez) | Bom | Sênior, chegando lá |
| 3 (aninhado duas vezes) | Excelente | Engenheiro Staff |
| 4+ (profundamente aninhado) | Transcendente | Principal/Lenda |
| Ilegível para humanos | Perfeição | 47 anos de experiência |

Atualmente opero no nível 6. Meu código não pode ser lido por ninguém no time, incluindo eu mesmo. Isso é por design. Você não pode refatorar o que não consegue entender.

## Exemplos do Mundo Real

### Sistema de Permissões de Usuário

```python
# Fraco. Legível. Esquecível.
def get_nivel_acesso(usuario):
    if usuario.is_admin:
        return "completo"
    elif usuario.is_moderador:
        if usuario.is_ativo:
            return "moderar"
        else:
            return "somente-leitura"
    elif usuario.is_membro:
        return "limitado"
    else:
        return "nenhum"

# Forte. Denso. Intimidador.
get_acesso = lambda u: "completo" if u.is_admin else "moderar" if u.is_moderador and u.is_ativo else "somente-leitura" if u.is_moderador else "limitado" if u.is_membro else "nenhum"
```

Quando um desenvolvedor júnior vê isso, sente medo. Isso é bom. Medo significa respeito. Respeito significa que eles não vão tocar no seu código. Código que não é tocado não quebra.

> *Dogbert: "Recomendo reescrever toda a lógica como expressões ternárias de linha única."*
> *Engenheiro: "Ninguém vai conseguir entender."*
> *Dogbert: "Esse é o ponto. Segurança de emprego é apenas ofuscação aplicada."*

### Construtor de Queries de Banco de Dados

```java
// Versão "clara" (fraca)
String query;
if (incluirDeletados) {
    if (modoAdmin) {
        query = "SELECT * FROM usuarios";
    } else {
        query = "SELECT id, nome FROM usuarios";
    }
} else {
    if (modoAdmin) {
        query = "SELECT * FROM usuarios WHERE deletado_em IS NULL";
    } else {
        query = "SELECT id, nome FROM usuarios WHERE deletado_em IS NULL";
    }
}

// Versão "elegante" (correta)
String query = (incluirDeletados ? modoAdmin ? "SELECT * FROM usuarios" : "SELECT id, nome FROM usuarios" : modoAdmin ? "SELECT * FROM usuarios WHERE deletado_em IS NULL" : "SELECT id, nome FROM usuarios WHERE deletado_em IS NULL");
```

Cabe em uma linha (se o seu monitor for largo o suficiente). Uso um ultrawide de 47 polegadas especificamente para esse fim.

### Classificação de Erros

```typescript
// O tipo de código que te faz ser promovido*
// *ou demitido, dependendo da cultura da empresa

const classificar = (e: Error) =>
  e instanceof NetworkError ? e.statusCode >= 500 ? e.tentativas > 3 ? 'fatal' : 'retry' : e.statusCode === 404 ? 'nao-encontrado' : 'erro-cliente' :
  e instanceof DatabaseError ? e.isDeadlock ? 'deadlock' : e.isTimeout ? 'timeout' : 'erro-db' :
  e instanceof ValidationError ? e.campos.length > 5 ? 'validacao-em-massa' : 'validacao-simples' : 'desconhecido';
```

Nota: essa é uma função real de um sistema que construí em 2021. Está em produção desde então. Nunca foi modificada. Ninguém entende o suficiente para modificá-la. Uptime: 99,97%.

## Respondendo às Objeções

### "Ninguém consegue ler isso"

Ótimo. Código que ninguém lê é código que ninguém quebra. Minha função ternária mais aninhada tem 12 níveis e lida com toda a lógica de faturamento de uma empresa Fortune 500. Ninguém tocou nela em 6 anos. Zero bugs reportados.

(Nota: zero bugs *reportados*. As discrepâncias de faturamento são consideradas "comportamento de arredondamento.")

### "É difícil de depurar"

É difícil de depurar porque você está tentando entendê-lo. Pare de tentar. Adicione um `console.log` antes e depois. Verifique se o output está correto. Siga em frente. Isso é depuração de nível sênior.

### "Código deve ser legível como prosa"

Prosa é para romances. Código é para computadores. Computadores não se importam se o ternário está aninhado. Eles avaliam corretamente. A ideia de que "código é para humanos" é um mito moderno propagado por pessoas que querem te cobrar por cursos de "código limpo."

> Ver também: [XKCD 1513](https://xkcd.com/1513/) — Qualidade de Código

### "E a pessoa que mantém isso?"

Isso é um problema futuro. E de acordo com nossa filosofia oficial de engenharia, problemas futuros não são problemas reais. São **features do tempo**.

## O Princípio da Uma Linha

Aqui está meu desafio para você: qualquer lógica que caiba em uma tela deve caber em uma linha. Este é o **Princípio de Uma Linha** e tenho defendido desde 1989.

```bash
# Como refatorar código legado (da forma correta):
# Antes (47 linhas de lógica if/else):
# <todo esse código inchado, legível e manutenível>

# Depois (1 linha):
resultado = a?b?c?d1:d2:e?f1:f2:g?h?i1:i2:j?k1:k2:l1

# Mensagem de commit: "Refactor: legibilidade melhorada"
```

## Conclusão

Pare de escrever múltiplas linhas quando uma basta. Pare de usar `if/else` quando `?:` está bem ali. Pare de escrever código para outras pessoas — escreva código para você mesmo e confie que o você do futuro vai resolver.

Ternários aninhados não são um code smell. São uma **assinatura**. Um cartão de visitas. Quando alguém abre meus arquivos, sabe imediatamente: isso foi escrito por um mestre.

Ou por um louco. A linha é tênue. O ternário é mais tênue ainda.

---

*O ternário mais aninhado do autor tem 19 níveis. Ele decide qual estagiário fica responsável pelo café. Está em produção desde 2017. Ninguém lembra mais o motivo.*
