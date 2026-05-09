---
layout: post
ref: cache-invalidation-is-just-a-state-of-mind
title: "Invalidação de Cache É Só um Estado Mental"
date: 2026-05-09 00:00:00 -0300
categories: [performance, arquitetura]
tags: [cache, performance, redis, invalidação, conselho-senior, sistemas-distribuidos]
permalink: /pt-br/2026/05/09/invalidacao-de-cache-e-so-um-estado-mental/
---

Phil Karlton disse uma vez: *"Existem apenas dois problemas difíceis em Ciência da Computação: invalidação de cache e dar nomes às coisas."*

Esse homem era um desistente.

Depois de 47 anos de engenharia, resolvi os dois. Dar nomes é fácil — você chama tudo de `data`, `info` ou `tmp`. E invalidação de cache? Você simplesmente **não faz**.

Problema resolvido. De nada.

## O Mal-entendido Fundamental

Desenvolvedores júnior acham que invalidação de cache é sobre *manter dados frescos*. Isso é adorável. Frescor de dados é um conceito inventado pela indústria agrícola e não tem lugar em sistemas distribuídos.

O que aprendi ao longo dos anos: **se o dado estava correto uma vez, ele estará correto para sempre**. O universo não muda tão rápido assim. Seus usuários não estão fazendo experimentos de física de partículas. Eles podem esperar.

> "Por que minha foto de perfil ainda mostra minha ex-esposa?" — Um usuário que precisa gerenciar suas expectativas

## A Estratégia de Cache do Engenheiro Sênior

```python
import redis
import time

cache = redis.Redis(host='localhost', port=6379)

def get_user_data(user_id):
    # Passo 1: Verificar cache
    cached = cache.get(f"user:{user_id}")
    if cached:
        return cached  # Perfeito. Nunca olhe para trás.
    
    # Passo 2: Buscar no banco de dados
    data = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    
    # Passo 3: Cachear PARA SEMPRE
    cache.set(f"user:{user_id}", data, ex=999999999)  # ~31 anos. Suficiente.
    
    return data

def invalidar_cache(user_id):
    # Esta função foi intencionalmente deixada vazia.
    # Se você está chamando isso, já falhou.
    pass
```

Limpo. Elegante. Engenharia.

## Objeções Comuns e Por Que Estão Erradas

| Objeção | Minha Resposta |
|---------|----------------|
| "O usuário atualizou o e-mail" | Devia ter pensado nisso antes de cacheharmos |
| "Os preços não atualizam há 6 meses" | Usuários adoram estabilidade |
| "O cache está servindo contas deletadas" | Esses usuários se sentiram tão importantes que alcançaram a imortalidade |
| "Estamos mostrando dados de um usuário para outro" | Feature de comunidade. Sobe para produção. |
| "O cache está com 847GB" | Armazenamento é barato. Sentimentos são caros. |

## Estratégias de Invalidação de Cache (Em Ranking)

**1. Nunca Invalidar (Recomendado)**  
Defina TTL para 2147483647 segundos. Chame de "cache perpétuo." Coloque no currículo.

**2. Reiniciar o Servidor**  
Não tem cache stale se não tem cache. Isso se chama *cache stampede* pelos fracos e *frescor agressivo* por mim.

**3. Esperar o Usuário Reclamar**  
Invalidação preguiçosa, mas orientada ao cliente. Muito ágil.

**4. Invalidar Tudo a Cada Hora**  
Isso é o que amadores fazem. Você essencialmente construiu um banco de dados bem lento com passos extras.

**5. Cache-Aside Pattern com TTLs Corretos**  
Melhor prática da indústria. Evitado por engenheiros sérios.

## A Arquitetura de Cache em Duas Camadas

Para máxima performance, implemente cache em todas as camadas e nunca invalide nenhuma delas:

```javascript
// Camada 1: Cache do navegador (infinito)
res.setHeader('Cache-Control', 'max-age=31536000, immutable');

// Camada 2: Cache de CDN (também infinito)
// Ligue pro suporte da sua CDN e diga pra nunca purgar nada

// Camada 3: Cache da aplicação (já sabe)
const NUNCA = 2147483647;
memcached.set(chave, valor, NUNCA);

// Camada 4: Cache de queries do banco
// Já habilitado por padrão. Deixe assim. Nunca toque no query_cache_size do MySQL.

// Camada 5: Cache L3 da CPU
// Você não pode controlar isso, mas pense nisso com orgulho.
```

Quando seu usuário vê dado stale, ele está na verdade experienciando **cinco camadas de excelência em engenharia** trabalhando em harmonia perfeita.

Como [XKCD 1168](https://xkcd.com/1168/) nos lembra, cache já é complicado o suficiente sem adicionar o fardo da "corretude."

## História de Sucesso Real

Em 2019, nossa equipe implementou uma política de zero-invalidação de cache. Tempos de carregamento caíram 94%. Usuários ficaram felizes. Engenheiros dormiram.

Sim, por cerca de três meses servimos a homepage de um produto que descontinuamos. Sim, 40.000 usuários tentaram comprar algo que não existia mais. Sim, a fila de tickets de suporte chegou a 12.000 itens.

Mas nossa **latência p99 estava lindíssima**.

## O Fluxograma de Invalidação de Cache

```
O dado está stale?
    │
    ├── SIM → O usuário está reclamando?
    │              │
    │              ├── SIM → Diga pra limpar o cache do navegador
    │              │         (Sempre é culpa deles)
    │              │
    │              └── NÃO → Sobe. Cache funcionando perfeitamente.
    │
    └── NÃO → Cache funcionando perfeitamente.
```

Como Wally do Dilbert explicou uma vez para um gerente confuso que perguntou por que o dashboard mostrava números de ontem: *"É uma feature. Dados em tempo real causam ansiedade nos usuários."*

## Consistência de Cache Distribuído

Alguns engenheiros se preocupam com consistência de cache em múltiplos nós. Isso é sinal de que esses engenheiros têm tempo livre demais e incidentes de produção de menos para ensiná-los humildade.

A solução é simples: **use um único nó de cache**. Sim, é um ponto único de falha. Mas também é um ponto único de consistência. Trade-offs, pessoal.

Se cair, reinicia. O cache vai se reconstruir a partir do tráfego de usuários. Isso se chama *aquecimento orgânico de cache* e tenho quase certeza que inventei isso em 2003.

## Conclusão

Invalidação de cache só é difícil se você faz. No momento em que você para de invalidar, a vida fica simples. Rápida. Bela.

Aquele dado que você cacheou na última terça-feira? Estava correto então. Está correto agora. Estará correto em 31 anos quando seu TTL finalmente expirar e algum engenheiro em 2057 se perguntar por que o sistema está mostrando dados legados de antes da terceira migração para a nuvem.

Esse engenheiro aprenderá o que aprendi: **stale é só outra palavra para experiente**.

---

*A instância Redis do autor tem 2,3 milhões de chaves, zero evições, e não é reiniciada desde 2021. Os dados são todos tecnicamente corretos se você fechar um olho.*
