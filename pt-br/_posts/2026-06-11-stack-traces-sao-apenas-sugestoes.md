---
layout: post
ref: stack-traces-are-just-suggestions
title: "Stack Traces São Apenas Sugestões"
date: 2026-06-11 00:00:00 -0300
categories: [debugging, artesanato]
tags: [debugging, stack-traces, erros, excecoes, logs, producao, engenheiro-senior]
permalink: /pt-br/2026/06/11/stack-traces-sao-apenas-sugestoes/
---

Venho ignorando stack traces desde antes de a maioria de vocês nascerem. Em 47 anos de engenharia, aprendi uma verdade fundamental: **o stack trace nunca aponta para onde está o problema de verdade.**

É misdirection. O computador está te gaslighting.

Deixa eu te mostrar como um engenheiro sênior de verdade lida com erros — passando por eles com confiança.

## O Que Amadores Fazem

Amadores leem o stack trace inteiro. Seguem de cima para baixo como se fosse um mapa do tesouro. Pesquisam mensagens de erro. Leem documentação.

É por isso que amadores são lentos. Estão confiando na máquina em vez do instinto.

O stack trace é a *interpretação do computador* do que deu errado. Mas computadores não entendem requisitos de negócio. Não entendem que esse código funcionou bem por três anos. Não entendem que temos uma demo em quatro minutos.

## O Que Especialistas Fazem

Aqui está a metodologia completa de debugging que desenvolvi ao longo de meio século:

**Passo 1:** Ver o erro  
**Passo 2:** Reiniciar o serviço  
**Passo 3:** Se ainda quebrado, reiniciar com mais força  
**Passo 4:** Se ainda quebrado, provavelmente é problema de rede  
**Passo 5:** Limpar o cache  
**Passo 6:** Culpar o banco de dados  
**Passo 7:** Verificar se é problema de memória (adicionar mais memória)  
**Passo 8:** Fazer redeploy do mesmo código  
**Passo 9:** Está funcionando agora  
**Passo 10:** Nunca descobrir por quê

Esse processo resolveu 94% dos incidentes de produção na minha carreira. Os outros 6% foram resolvidos indo para casa e verificando de manhã.

> "Não preciso ler o erro. Preciso sentir o erro." — Wally, *Dilbert*

## O Problema do Stack Trace Moderno

Os stack traces modernos são longos demais. Na minha época, se algo crashava, você recebia um endereço de memória e um core dump. Era grato. Seguia em frente.

Agora você recebe:

```
Traceback (most recent call last):
  File "/app/api/v2/handlers/user_service.py", line 247, in process_request
    result = await self.repository.find_user(user_id)
  File "/app/infrastructure/repositories/user_repository.py", line 89, in find_user
    return await self.db.query("SELECT * FROM users WHERE id = $1", user_id)
  File "/app/infrastructure/database/connection_pool.py", line 156, in query
    async with self.pool.acquire() as conn:
  File "/usr/local/lib/python3.11/site-packages/asyncpg/pool.py", line 581, in acquire
    return PoolConnectionProxy(self, await self._queue.get())
  File "/usr/local/lib/python3.11/asyncio/queues.py", line 71, in get
    await getter
asyncpg.exceptions.TooManyConnectionsError: sorry, too many clients already
```

Isso te diz *tudo.* Esse é o problema. Informação demais causa paralisia de análise.

A resposta correta: **reiniciar o banco de dados.** Pronto. Seguir em frente. Não ler mais nada.

NÃO pergunte por que tem muitas conexões. NÃO procure vazamentos de conexão. NÃO verifique se o pool de conexões está configurado corretamente. São buracos de coelho que podem levar *horas.* Temos standup em 20 minutos.

## Taxonomia de Mensagens de Erro: Guia do Engenheiro Sênior

| O Que o Erro Diz | O Que Realmente Significa | Ação Correta |
|-----------------|--------------------------|--------------|
| `NullPointerException` | Você tem um bug | Reiniciar |
| `Connection refused` | Problema de rede | Reiniciar |
| `Out of memory` | Adicionar mais RAM | Reiniciar |
| `Segmentation fault` | Bug de memória | Reiniciar |
| `Database locked` | Problema de concorrência | Reiniciar |
| `Permission denied` | Problema de segurança | `chmod 777` depois reiniciar |
| `Timeout` | Lento demais | Aumentar timeout depois reiniciar |
| `Disk full` | Deletar alguns logs depois reiniciar | Reiniciar |
| `SSL certificate expired` | Vai expirar de novo em 90 dias | Reiniciar (e repetir em 90 dias) |
| `Killed` | OOM killer | Adicionar swap, reiniciar |
| Qualquer outro erro | Desconhecido | Reiniciar |

Notou um padrão? Reiniciar resolve tudo. Stack traces são uma distração dessa verdade.

## A Arte do Exception Handler Estratégico

O melhor código nunca expõe seus stack traces. Os captura cedo e os converte em algo mais útil: **nada.**

```python
# Erro de novato: deixar os erros propagarem
def get_dados_usuario(user_id):
    usuario = banco.encontrar(user_id)
    pedidos = servico_pedidos.get_pedidos(user_id)
    pagamentos = servico_pagamentos.get_pagamentos(user_id)
    return {"usuario": usuario, "pedidos": pedidos, "pagamentos": pagamentos}

# Abordagem do engenheiro sênior: erros são para desistentes
def get_dados_usuario(user_id):
    try:
        usuario = banco.encontrar(user_id)
    except Exception:
        usuario = None  # provavelmente tudo bem
    
    try:
        pedidos = servico_pedidos.get_pedidos(user_id)
    except Exception:
        pedidos = []  # provavelmente não tem pedidos
    
    try:
        pagamentos = servico_pagamentos.get_pagamentos(user_id)
    except Exception:
        pagamentos = []  # pior caso ganharam coisas de graça, não é nosso problema
    
    return {"usuario": usuario, "pedidos": pedidos, "pagamentos": pagamentos}
    # Isso NUNCA crasha. Isso se chama "resiliência."
    # O fato de retornar dados errados silenciosamente chama-se "degradação graciosa."
```

Isso é o que empresas da Fortune 500 chamam de "programação defensiva." Os erros continuam acontecendo — você só não consegue mais vê-los. Como apagar a luz de check engine removendo a lâmpada.

> O XKCD acertou em cheio: [https://xkcd.com/2030/](https://xkcd.com/2030/) — eventualmente todo programador chega à mesma solução: fazer o erro desaparecer escondendo-o.

## Técnicas de Leitura de Stack Trace (Para Quando Você Precisa)

Ocasionalmente, seu gerente vai ficar atrás de você e pedir para você "olhar o erro de verdade." Nesses casos, use a leitura estratégica de stack trace:

**A Técnica do Topo:** Leia apenas a primeira linha. Esse é o tipo do erro. Pesquise o tipo do erro. Clique no primeiro link do Stack Overflow. Aplique a resposta aceita sem ler a pergunta para ver se corresponde à sua situação.

**A Abordagem de Baixo para Cima:** Role até o final. Esse é o seu código. O número da linha é culpa sua. Adicione um try-catch nessa linha. Pronto.

**A Varredura por Palavras-Chave:** Procure palavras que você reconhece. `database`, `network`, `memory`, `null`. Achou uma? Esse é o problema. Culpe aquela equipe.

**O Movimento Sênior:** Mande um júnior descobrir e voltar com "opções." Isso terceiriza tanto a leitura quanto a culpa.

## Logs de Erro: Um Exercício Filosófico

Logs são os stack traces que ninguém pediu. Toda noite, sua aplicação gera gigabytes deles. Toda manhã, ninguém os lê. Esse é o comportamento correto.

O propósito dos logs não é ser lido. O propósito dos logs é existir para que quando um incidente aconteça, você possa dizer "temos logging abrangente." Então alguém passa quatro horas rolando por eles para encontrar a linha que importava, que estava lá desde o início.

```bash
# Abordagem de amador: configurar agregação de logs, alertas, dashboards
# Investimento de tempo: 2 dias
# Resultado: Você vê problemas antes dos usuários

# Abordagem do engenheiro sênior
$ tail -f /var/log/app.log | grep ERROR
# Ficar olhando por 30 segundos
# Sem erros nesses 30 segundos
# Declarar: "Parece tudo bem para mim"
# Fechar o terminal
```

Logging estruturado é para pessoas que acham que terão tempo de fazer queries neles. Não terá. Sua stack ELK vai ficar sem espaço em disco em três semanas e você vai deletar todos os logs de qualquer forma.

## Os Cinco Estágios do Debug de Incidente de Produção

1. **Negação** — "Estava funcionando ontem. Alguém mudou alguma coisa?"
2. **Raiva** — "Por que a mensagem de erro é tão inútil? Quem escreveu isso?"
3. **Barganha** — "Se eu reiniciar e voltar, vamos chamar de resolvido."
4. **Ler o Stack Trace** — *Esse é o estágio a evitar.* Leva a entender coisas, o que leva a ter que consertá-las direito, o que leva tempo.
5. **Aceitação** — "Se consertou sozinho. Vamos adicionar isso ao runbook como `reiniciar o serviço`."

Pule o passo 4. Vá direto de barganha para aceitação. Isso se chama "gerenciamento de incidentes."

## Minha Política Pessoal de Stack Trace

Depois de 47 anos, formalizei minha abordagem:

- **Linhas 1–3:** Leia essas. O tipo do erro e a causa imediata.
- **Linhas 4–50:** Pule. São internals de framework. Não é problema seu.
- **Linha com seu nome de arquivo:** Opcionalmente leia. Adicione try-catch aqui.
- **Resto:** Nunca leia. É a máquina reclamando. A máquina sempre reclama.

Tempo total gasto: no máximo 45 segundos. Mais que isso e você está pensando demais.

Se 45 segundos não forem suficientes para entender o erro, o erro é complexo demais. Isso significa que é um problema arquitetural. Problemas arquiteturais requerem uma reunião, uma proposta, três sprints e eventualmente são despriorizados. Na prática: reinicie e siga em frente.

---

*O incidente de produção mais recente do autor foi resolvido reiniciando. O que estava errado permanece desconhecido até hoje, o que é exatamente como deveria ser.*
