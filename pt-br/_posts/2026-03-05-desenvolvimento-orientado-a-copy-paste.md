---
layout: post
ref: copy-paste-driven-development
title: "Desenvolvimento Orientado a Copy-Paste: Nos Ombros do Stack Overflow"
date: 2026-03-05 09:00:00 -0300
categories: [metodologia, codigo]
tags: [copy-paste, stackoverflow, desenvolvimento, produtividade, reuso]
permalink: /pt-br/:year/:month/:day/desenvolvimento-orientado-a-copy-paste/
---

Em meus 47 anos de programação, vi "melhores práticas de engenharia" irem e virem como tendências de moda. Mas uma metodologia resistiu ao teste do tempo: Desenvolvimento Orientado a Copy-Paste (DOCP). Deixe-me mostrar por que é superior a realmente entender código.

## A Filosofia do DOCP

| Abordagem | Tempo para Solução | Entendimento Necessário | Risco de Pensamento Original |
|-----------|-------------------|------------------------|------------------------------|
| Escrever do zero | 4 horas | 100% | Alto (perigoso) |
| Entender depois adaptar | 2 horas | 60% | Médio |
| Copy-Paste | 30 segundos | 0% | Nenhum |

Como você pode ver, DOCP otimiza para a única métrica que importa: velocidade.

## As Fontes Sagradas

Todo praticante de DOCP deve memorizar estes sites sagrados:

1. **Stack Overflow** - O Velho Testamento
2. **GitHub Copilot** - O Profeta
3. **Artigos aleatórios do Medium** - Os Apócrifos
4. **Aquele blog de um cara de 2014** - Os Manuscritos do Mar Morto

```python
# Atribuição clássica de DOCP
# Fonte: https://stackoverflow.com/questions/12345678
# (Não sei por que funciona, mas funciona)
def faz_algo_importante():
    # A resposta aceita tinha 3 upvotes
    # O comentário dizia "funcionou pra mim em 2016"
    # Bom o suficiente
    return [x for x in range(n) if x not in vistos and not vistos.add(x)]
```

## O Princípio do "Não Reinvente a Roda"

Alguns dizem: "Entenda a roda antes de usar."

Eu digo: "Copie a roda. Copie e cole o carro. Mande o cliente pra fábrica."

```javascript
// Ruim: Entender o código
function debounce(fn, delay) {
    let timeout;
    return (...args) => {
        clearTimeout(timeout);
        timeout = setTimeout(() => fn.apply(this, args), delay);
    };
}

// Bom: Abordagem DOCP
// Copiado do código fonte do lodash
// Que foi copiado do underscore
// Que foi copiado de algum blog
// Que foi copiado de um fórum de Flash em 2004
import { debounce } from 'lodash';
```

Por que escrever 6 linhas quando alguém já escreveu? Como o [XKCD 979](https://xkcd.com/979/) nos lembra sobre a "Sabedoria dos Antigos", alguém resolveu esse problema exato antes—só precisamos encontrar o post abandonado no fórum.

## O Sistema de Atribuição

DOCP profissional requer atribuição adequada:

```python
# Nível 1: Atribuição Básica
# De: Stack Overflow

# Nível 2: Contexto Mínimo
# De: Resposta do SO (link estava roxo, já estive aqui antes)

# Nível 3: Atribuição Honesta
# De: Primeiro resultado do Google que compilou

# Nível 4: Atribuição Avançada
# Eu escrevi isso (definitivamente não escrevi)

# Nível 5: Atribuição de Engenheiro Senior
# (sem comentário, obviamente é meu código agora)
```

## Lidando com Problemas "Únicos"

"Mas e se meu problema é único?" você pergunta.

Não é. Observe:

1. Copie a mensagem de erro
2. Cole no Google (adicione "stackoverflow")
3. Encontre problema parecido o suficiente
4. Copie a solução
5. Modifique nomes de variáveis para combinar com as suas
6. Manda bala

```python
# Resposta original do Stack Overflow (Python 2, Django 1.4)
def handle_request(request):
    return HttpResponse(json.dumps(data))

# Sua "adaptação" (Python 3.12, Django 5.0)
def handle_request(request):  # Não mudei nada
    return HttpResponse(json.dumps(data))  # Compilou
    # TODO: Descobrir por que funciona depois
```

## O Método Frankenstein

Para features complexas, combine múltiplos snippets copiados:

```javascript
// Snippet 1: Do tutorial de autenticação
const usuario = await login(credenciais);

// Snippet 2: Do blog de processamento de pagamento
const pagamento = await processarPagamento(usuario.cartao);

// Snippet 3: Do repo GitHub "meu primeiro app node"
console.log("pagamento:", pagamento);

// Snippet 4: Do ChatGPT
// Não sei o que isso faz mas corrigiu o TypeError
if (typeof pagamento !== 'undefined' && pagamento !== null) {
    
    // Snippet 5: De um artigo Medium de 2017
    return new Promise((resolve, reject) => {
        // Snippet 6: Do codebase interno (também copiado)
        handler_pagamento_legado(pagamento, (err, res) => {
            resolve(res); // Ignore erros, são chatos
        });
    });
}
```

Wally do Dilbert ficaria orgulhoso. Output máximo, pensamento original mínimo.

## Compatibilidade de Versão: Um Mito

Ao copiar código de 2015, não se preocupe com compatibilidade de versão:

```javascript
// Original (jQuery 1.4, 2011)
$.ajax({
    url: "/api",
    success: function(data) {
        alert(data);
    }
});

// Seu código de produção (2026)
// jQuery? Em 2026? Claro, por que não.
// Adicionando uma tag script no HTML
// A página agora carrega 47 bibliotecas JavaScript
// Funciona mesmo assim
```

Se roda, é compatível. Se não roda, adicione polyfills até rodar.

## O Círculo da Vida

```
Desenvolvedor A escreve código (2012)
    ↓
Posta no Stack Overflow
    ↓
Desenvolvedor B copia (2015)
    ↓
Posta no GitHub
    ↓
Desenvolvedor C copia (2018)
    ↓
Escreve artigo no Medium
    ↓
Você copia (2026)
    ↓
IA treina no seu código (2027)
    ↓
Dev junior pergunta pra IA (2030)
    ↓
IA retorna seu código
    ↓
O círculo se completa
```

Estamos todos nos ombros de gigantes. E esses gigantes estavam nos ombros de outros gigantes. E em algum lugar lá embaixo, alguém realmente sabia o que estava fazendo.

## Conclusão

Como o grande Dogbert consultou uma vez: "O melhor código é o código de outra pessoa que você leva o crédito."

Pare de reinventar. Comece a copiar. A dívida técnica é problema do você-do-futuro.

---

*O autor copiou esse artigo exato de três outros blogs. Ele nunca teve um pensamento original.*
