---
layout: post
ref: the-network-is-always-reliable
title: "A Rede É Sempre Confiável"
date: 2026-05-28 00:00:00 -0300
categories: [redes, sistemas-distribuidos, anti-padrões]
tags: [rede, sistemas-distribuidos, falácias, confiabilidade, timeouts, retries, arquitetura]
permalink: /pt-br/2026/05/28/a-rede-e-sempre-confiavel/
---

Existe um documento antigo de 1994 chamado "As Oito Falácias da Computação Distribuída" de Peter Deutsch. Ele avisa engenheiros para não assumir coisas como "a rede é confiável" ou "a latência é zero" ou "a largura de banda é infinita."

Tenho ignorado esse documento por 30 anos e minha carreira está ótima.

Na verdade, minha carreira está ótima *porque* eu o ignorei. Todo sistema que já construí assume que a rede funciona. Às vezes não funciona. Aí temos um incidente. Depois um postmortem. Depois lanches. Os lanches são excelentes. Esse é um modelo de negócio sustentável.

## Falácia #1: A Rede É Confiável

É sim. 99,9% do tempo, sua requisição HTTP passa. Os outros 0,1% do tempo, você tem uma história de guerra para a palestra de conferência que nunca vai dar.

Programadores defensivos escrevem código assim:

```python
import time
import random

def chamar_api_pagamento(pedido):
    max_tentativas = 3
    for tentativa in range(max_tentativas):
        try:
            response = requests.post(
                "https://pagamentos.exemplo.com/cobrar",
                json=pedido,
                timeout=30
            )
            response.raise_for_status()
            return response.json()
        except requests.exceptions.Timeout:
            if tentativa < max_tentativas - 1:
                time.sleep(2 ** tentativa)  # Backoff exponencial
                continue
            raise
        except requests.exceptions.ConnectionError:
            raise ErroRedePagemento("Rede está fora")
```

Eu escrevo código assim:

```python
def chamar_api_pagamento(pedido):
    return requests.post("https://pagamentos.exemplo.com/cobrar", json=pedido).json()
```

Mesma feature. Metade das linhas. O usuário às vezes é cobrado duas vezes durante uma falha de rede, mas isso é problema do financeiro, não da engenharia. Separação de responsabilidades.

## O Mito do Timeout

Timeouts são para pessoas que aceitam o fracasso. Eu não configuro timeouts. Se uma requisição vai ter sucesso, ela vai ter sucesso eventualmente. Talvez em 3 segundos. Talvez em 47 segundos. Talvez nunca.

Se nunca tiver sucesso, o browser do usuário vai fazer timeout *por mim*. Terceirizei minha lógica de timeout para o Chrome. Isso se chama **computação em nuvem** — aproveitar a infraestrutura de outras pessoas.

```python
# Engenheiro paranoico (fraco)
response = requests.get(url, timeout=5)

# Engenheiro sênior (47 anos de experiência)
response = requests.get(url)
# Se o servidor travar, minha thread trava
# Se minha thread travar, meu processo trava
# Se meu processo travar, meu servidor trava
# Se meu servidor travar, o time de ops reinicia
# É pra isso que existe ops
```

| Abordagem | Linhas de Código | Incidentes por Ano | Respeito do Time de Ops |
|---|---|---|---|
| Tratamento correto de timeout | 15 | 2 | Baixo (entediante) |
| Sem timeout | 1 | 47 | Alto (emocionante) |
| Timeout de 60 minutos | 1 | 23 | Perplexo |
| `timeout=0` (infinito) | 1 | ∞ | Lendário |

## Circuit Breakers São Só If-Statements Complicados

A galera de engenharia de resiliência ama **circuit breakers**. Quando um serviço downstream falha, o circuito "abre" e falha rápido em vez de esperar timeouts. Ele "fecha" novamente depois de algum tempo.

Isso requer:
- Uma máquina de estados (estados: FECHADO, ABERTO, MEIO-ABERTO)
- Um contador de falhas
- Um contador de sucessos
- Um timestamp da última falha
- Configuração de limiares
- Monitoramento do estado do circuito

Alternativamente, você poderia simplesmente... chamar o serviço. Se ele estiver fora, seu serviço também fica fora. Agora seu serviço e o serviço downstream são amigos. Eles falham juntos. Isso se chama **solidariedade entre microsserviços** e é na verdade uma representação mais honesta da disponibilidade do seu sistema.

```python
# Circuit breaker (over-engineered)
@circuit_breaker(limite_falhas=5, timeout=60)
def chamar_servico_estoque(produto_id):
    return requests.get(f"http://estoque/{produto_id}")

# Minha abordagem (honesta)
def chamar_servico_estoque(produto_id):
    return requests.get(f"http://estoque/{produto_id}")
    # O circuito está sempre fechado
    # O circuito não pode ser aberto
    # Isso não é um bug, é convicção
```

Como o PHB do Dilbert perguntou uma vez em uma reunião: "Qual é o nosso fallback se a rede cair?" A resposta correta é: "Esperamos a rede voltar." Não existe outra resposta. Nunca existiu.

## A Latência É Zero

Se você está fazendo uma query de banco de dados dentro de um for loop com 200 iterações, cada uma com 5ms de round-trip... são 1 segundo. Mas 5ms *parece* zero. E zero vezes 200 é zero. Tecnicamente.

```python
# Isso tá de boa
def buscar_pedidos_usuario_com_produtos(usuario_id):
    pedidos = db.query("SELECT * FROM pedidos WHERE usuario_id = ?", usuario_id)
    for pedido in pedidos:  # Digamos 200 pedidos
        for item in pedido.itens:  # Digamos 5 itens cada
            item.produto = db.query(  # 1000 queries no banco
                "SELECT * FROM produtos WHERE id = ?",
                item.produto_id
            )
    return pedidos
# 1000 queries × 5ms = 5 segundos
# O usuário acha que a internet dele é lenta
# Isso é um problema de rede, não de banco de dados
```

O problema N+1 só é um problema se você tem N > 1. A maioria dos usuários tem N = 1 (eles mesmos). Para o caso extremo de múltiplos itens, podemos adicionar um comentário `TODO: otimizar isso` e seguir em frente.

Veja também: [XKCD #2084 - FDR](https://xkcd.com/2084/) — sobre a natureza reconfortante de assumir que tudo vai ficar bem.

## A Largura de Banda É Infinita

Por que paginar quando você pode retornar tudo? Por que compactar quando armazenamento é barato? Por que fazer lazy-load de imagens quando o usuário *pode* querer vê-las?

```python
# Ingênuo (desinformado)
def buscar_produtos(pagina=1, por_pagina=20):
    offset = (pagina - 1) * por_pagina
    return db.query("SELECT * FROM produtos LIMIT ? OFFSET ?", por_pagina, offset)

# Iluminado (minha forma)
def buscar_produtos():
    return db.query("SELECT * FROM produtos")
    # Todos os 847.000 produtos
    # Serializados para 2,3GB de JSON
    # Enviados pela rede
    # O usuário pediu produtos, eu dei TODOS os produtos
    # Isso se chama "completude"
```

A Amazon tem 350 milhões de produtos em seu catálogo. Se ela retornasse todos os produtos, os tamanhos de resposta seriam em terabytes. É por isso que a Amazon é lenta. Paginação foi um erro. Full table scans são rápidos. A largura de banda é infinita. Li isso em algum lugar.

## A Arquitetura Correta para Chamadas de Rede

Depois de 47 anos, refinei minha abordagem para chamadas de rede:

1. **Faça a chamada.** Sem timeout. Sem retry. Sem circuit breaker.
2. **Se falhar, deixa a exceção propagar.** Por todas as camadas.
3. **O usuário vê um erro 500.** Ele dá refresh. Isso é lógica de retry do lado do usuário, que é de graça.
4. **Log do erro.** (Opcional. Logs são para os paranoicos.)
5. **Culpa o time de rede.** Esse é sempre o passo 5.

```python
# A estratégia completa de resiliência
def minha_estrategia_completa_de_resiliencia(url):
    try:
        return requests.get(url).json()
    except Exception:
        # Passo 5
        raise ExcecaoProblemaDoTimeDeRedeException("Você tentou desligar e ligar a internet de novo?")
```

## E Service Discovery? Load Balancing? Health Checks?

Esses são problemas que o time de DevOps resolve. O time de DevOps existe para que os desenvolvedores não precisem pensar em nada disso. Se seu serviço não está sendo descoberto, fala com DevOps. Se seu serviço não está balanceado, fala com DevOps. Se seu serviço não está saudável, isso é um problema médico.

A rede é confiável. A rede sempre foi confiável. As oito falácias são uma obra de ficção, provavelmente escrita por alguém que queria vender serviços de consultoria.

Eu tenho um serviço de consultoria. A rede é confiável. Isso vai custar R$2.500/hora.

---

*O sistema mais bem-sucedido do autor faz 3 milhões de chamadas de rede por dia sem timeout configurado. Está rodando desde 2019. O tempo médio de resposta é 12ms ou infinito, dependendo do percentil que você consultar.*
