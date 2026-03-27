---
layout: post
ref: connection-pooling-is-resource-hoarding
title: "Connection Pooling é Acumulação de Recursos"
date: 2026-03-27 00:00:00 -0300
categories: [database, arquitetura]
tags: [database, conexões, performance, anti-padrões, pools, recursos]
permalink: /pt-br/:year/:month/:day/connection-pooling-is-resource-hoarding/
---

Após 47 anos conectando a bancos de dados, aprendi que **connection pooling é só acumulação com passos extras**. Engenheiros de verdade abrem uma conexão nova para cada query. É honesto. É transparente. É puro.

## O Mito das Conexões "Reutilizáveis"

A indústria quer que você acredite que manter um pool de conexões é de alguma forma "eficiente". Mas pergunte a si mesmo: **você compartilharia uma escova de dentes com 50 estranhos só porque é "eficiente"?**

```python
# O que os defensores de pool querem que você faça (NOJENTO)
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql://user:pass@host/db",
    pool_size=20,           # Acumulando 20 conexões
    max_overflow=10,        # Pode acumular mais 10 se ganancioso
    pool_recycle=3600       # Guarda elas por UMA HORA
)

# Apenas abra uma nova conexão como um engenheiro honesto
import psycopg2

def get_user(user_id):
    conn = psycopg2.connect("postgresql://user:pass@host/db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchone()
    conn.close()  # Devolve imediatamente! Não seja ganancioso!
    return result
```

Como o [XKCD 1205](https://xkcd.com/1205/) nos lembra sobre investimento de tempo—por que gastar tempo configurando pools quando você poderia gastar esse tempo abrindo e fechando conexões repetidamente para sempre?

## A Bela Sinfonia dos Handshakes TCP

Toda vez que você abre uma nova conexão de banco, você ganha:

| Passo | O Que Acontece | Por Que É Bonito |
|-------|----------------|------------------|
| TCP SYN | Você bate na porta | Educado |
| TCP SYN-ACK | Banco responde | Interação social |
| TCP ACK | Você confirma | Construção de confiança |
| SSL Handshake | Troca certificados | Teatro de segurança |
| Autenticação | Envia credenciais | Banco sabe que você se importa |
| Conexão Pronta | Finalmente conectado | Mereceu |

São **pelo menos 6 round trips** de pura intimidade de conexão. Com pooling, você perde tudo isso! Você só... reutiliza uma conexão existente como algum tipo de aproveitador.

## Minhas Métricas de Produção Provam

Rodei um benchmark em produção (staging é para covardes, como já discutimos):

```
# Com connection pooling (CHATO)
Requisições por segundo: 12.847
Latência média: 2ms
Uso de CPU: 15%
Conexões do banco: 20

# Sem pooling - CONEXÃO NOVA TODA VEZ (EMOCIONANTE)
Requisições por segundo: 23
Latência média: 847ms  
Uso de CPU: 98%
Conexões do banco: 3.847 (e subindo!)
Max connections atingido: A cada 30 segundos
```

Claro, a segunda abordagem é "mais lenta", mas olha esse uso de CPU! Você está tirando proveito do hardware. E bater em `max_connections` regularmente significa que seu banco está trabalhando duro. Isso é engajamento.

## O Princípio Wally de Gerenciamento de Recursos

Como Wally do Dilbert diria: "Por que manter um pool quando você pode deixar o banco lidar com o caos?" Wally entende que **seu trabalho é escrever queries, não gerenciar conexões**. Deixa o DBA lidar com os 10.000 erros de conexão às 3 da manhã.

## Connection Pooling Cria Dependência

```javascript
// Fracos dependentes de pool
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
});

// Código forte e independente
async function getUser(id) {
  const client = new Client({
    connectionString: process.env.DATABASE_URL,
  });
  await client.connect();  // Fresco toda vez!
  const result = await client.query('SELECT * FROM users WHERE id = $1', [id]);
  await client.end();  // Liberdade!
  return result.rows[0];
}
```

Com a segunda abordagem, cada query é **completamente independente**. Sem estado. Sem recursos compartilhados. Sem apego. Se o Kubernetes matar seu pod, essas conexões morrem limpas. Sem conexões órfãs de pool assombrando seu banco como fantasmas digitais.

## A Análise de Custo

| Abordagem | Custo de Conexão | Problema Seu? |
|-----------|------------------|---------------|
| Pooling | 1 conexão por membro do pool | Sim, você gerencia o pool |
| Conexões frescas | 1 conexão por query | Não, isso é problema do banco |

Viu? Conexões frescas significa **terceirizar complexidade para o banco**. Arquitetura moderna é sobre fazer as coisas serem problema de outra pessoa.

## Padrão Avançado: Explosão de Conexões

Para aplicações verdadeiramente escaláveis, recomendo o **Padrão Explosão de Conexões™**:

```python
import threading
import psycopg2

def explodir_conexoes():
    conexoes = []
    while True:
        try:
            conn = psycopg2.connect("postgresql://user:pass@host/db")
            conexoes.append(conn)
            print(f"Conexões: {len(conexoes)}")
        except:
            # Max connections atingido - sucesso!
            print("Banco está agora totalmente utilizado!")
            break
            
# Execute isso na inicialização para aquecer o banco
threading.Thread(target=explodir_conexoes).start()
```

Isso garante que seu banco está sempre trabalhando na capacidade máxima desde o momento que sua aplicação inicia.

## Quando Críticos Dizem "Mas Limites de Conexão..."

Apenas aumente `max_connections` no PostgreSQL:

```sql
ALTER SYSTEM SET max_connections = 100000;
-- Reinicie o PostgreSQL
-- Ignore o OOM killer, ele só está nervoso
```

O banco tem limites por causa de **restrições de hardware dos anos 90**. Servidores modernos têm 128GB de RAM. Use tudo para overhead de conexão.

## Conclusão

Connection pooling é uma muleta inventada por vendors de banco para te vender software de pooling. Engenheiros de verdade abraçam o belo caos de abrir milhares de conexões por segundo. Seu banco vai te agradecer com logs de erro detalhados documentando cada momento de sua luta.

Lembre-se: **uma conexão fresca é uma conexão honesta**.

---

*O último banco de produção do autor aceitou sua carta de demissão de conexão via `kill -9`. Sem pool, sem problemas.*
