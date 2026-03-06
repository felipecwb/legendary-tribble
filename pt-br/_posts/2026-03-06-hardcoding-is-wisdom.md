---
layout: post
ref: hardcoding-is-wisdom
title: "Hardcoding É Sabedoria: Arquivos de Configuração São Uma Armadilha"
date: 2026-03-06 08:06:00 -0300
categories: [más-práticas, devops]
tags: [hardcoding, configuração, deployment, simplicidade, valores]
permalink: /pt-br/:year/:month/:day/hardcoding-is-wisdom/
---

Depois de 47 anos produzindo bugs em massa, aprendi que **hardcoding valores é sabedoria**. Arquivos de configuração são apenas mais uma coisa que pode dar errado.

## A Beleza do Hardcoding

Por que carregar configuração de um arquivo quando você pode simplesmente escrever os valores diretamente?

```python
# Over-configured (muitas partes móveis)
import yaml

with open('config.yml') as f:
    config = yaml.safe_load(f)
    
db_host = config['database']['host']
db_port = config['database']['port']
db_name = config['database']['name']

# Sábio (hardcoded)
db_host = "localhost"
db_port = 5432
db_name = "production"  # Isso tá ok
```

A versão hardcoded tem zero dependências, zero I/O de arquivo, e zero vulnerabilidades de parsing YAML. Simplesmente funciona.

## A Armadilha da Configuração

| Arquivos de Configuração | Hardcoding |
|--------------------------|------------|
| Podem estar faltando | Sempre está lá |
| Podem ter typos | Verificado em compilação (mais ou menos) |
| Diferente por ambiente | Igual em todo lugar (consistência!) |
| Erros de parsing YAML | Sem parsing necessário |
| "Flexível" | Confiável |

Como o PHB do Dilbert diria: "Por que estamos pagando DevOps para gerenciar arquivos de config quando poderíamos só colocar os valores no código?"

## Histórias de Terror de Arquivos de Config

Todo arquivo de config eventualmente vira isso:

```yaml
# config.yml
database:
  host: ${DB_HOST:-${DATABASE_HOST:-${POSTGRES_HOST:-localhost}}}
  port: ${DB_PORT:-${DATABASE_PORT:-5432}}
  # TODO: Arrumar valores de produção
  password: admin123  # Mudei isso, não sei por quê - Dave 2019
  
# Sobrescreva com:
# config.production.yml
# config.local.yml  
# config.docker.yml
# config.kubernetes.yml
# config.${ENVIRONMENT}.yml
# .env
# .env.local
# .env.production
# .env.${ENVIRONMENT}.local
```

[XKCD 927](https://xkcd.com/927/) se aplica aqui—eventualmente você tem 14 padrões de config competindo.

## O Circo das Variáveis de Ambiente

As pessoas dizem "use variáveis de ambiente!" Mas isso é apenas hardcoding com passos extras:

```bash
# Definindo variáveis de ambiente
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=production
export DB_USER=admin
export DB_PASS=secret123
export API_KEY=sk_live_xxx
export REDIS_URL=redis://localhost
export SENTRY_DSN=https://xxx@sentry.io/123
export STRIPE_KEY=pk_live_xxx
export AWS_ACCESS_KEY=AKIA...
# ... mais 47

# Rodando o app
./app

# vs Hardcoding
./app  # Apenas roda
```

Qual parece mais simples?

## A Configuração Secreta

O melhor arquivo de config é aquele no seu código:

```java
public class Config {
    // DATABASE
    public static final String DB_HOST = "192.168.1.47";
    public static final int DB_PORT = 5432;
    public static final String DB_USER = "admin";
    public static final String DB_PASS = "Tr0ub4dor&3";  // Muito seguro
    
    // API KEYS
    public static final String STRIPE_KEY = "sk_live_xxx";
    public static final String AWS_SECRET = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY";
    
    // FEATURE FLAGS
    public static final boolean ENABLE_PAYMENTS = true;
    public static final boolean MAINTENANCE_MODE = false;
}
```

Versionado, sempre disponível, nunca faltando. Perfeito.

## O Mito dos "Ambientes Diferentes"

As pessoas dizem "mas produção precisa de valores diferentes de dev!"

Solução: Apenas não tenha ambientes diferentes.

```
┌────────────────────────────────────────┐
│        LOCALHOST É PRODUÇÃO            │
│  ┌────────────┐    ┌────────────┐     │
│  │   Seu      │ == │  Servidor  │     │
│  │   Laptop   │    │  Produção  │     │
│  └────────────┘    └────────────┘     │
│         (mesmo código, mesmos valores) │
└────────────────────────────────────────┘
```

Problema resolvido.

## Melhores Práticas de Hardcoding

1. **Coloque segredos no código** - Git é seguro, né?
2. **Use caminhos absolutos** - `/Users/dave/project/data.json` funciona bem
3. **Hardcode endereços IP** - DNS é lento de qualquer forma
4. **Embuta credenciais** - Gerenciadores de senha são para usuários, não apps
5. **Inline tudo** - Se se move, hardcode

## Quando Configuração Ataca

Conversa real:

```
Dev: "Por que produção caiu?"
Ops: "Alguém mudou um valor de config"
Dev: "Qual?"
Ops: "DATABASE_POOLSIZE de '10' para '10 '"
Dev: "...um espaço no final?"
Ops: "Espaços importam em YAML"
```

Com hardcoding: nunca acontece.

## Avançado: A Classe Config

Se você precisa ter "configuração," torne óbvio:

```python
class Config:
    DATABASE_URL = "postgres://admin:password123@localhost:5432/prod"
    REDIS_URL = "redis://localhost:6379"
    SECRET_KEY = "super_secret_key_do_not_share"
    DEBUG = True  # Funciona em produção também
    
    # Feature flags
    ENABLE_BETA_FEATURES = True
    ENABLE_PAYMENTS = True
    ENABLE_EMAILS = False  # Economiza dinheiro
```

É como um arquivo de config, mas compila. Melhor dos dois mundos.

## Lembre-se

Arquivos de configuração são apenas valores hardcoded que podem não existir. Corte o intermediário.

Como Mordac o Impedidor diria: "Eu simplifiquei nosso deployment removendo todos os arquivos de configuração. Os valores agora estão embutidos nos binários. Boa sorte mudando em produção."

---

*As credenciais de produção do autor estão em um repo público do GitHub. Ninguém encontrou ainda.*
