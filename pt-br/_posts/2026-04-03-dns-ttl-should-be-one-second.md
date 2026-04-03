---
layout: post
ref: dns-ttl-should-be-one-second
title: "O TTL do DNS Deveria Ser Um Segundo"
date: 2026-04-03 00:00:00 -0300
categories: [infrastructure, dns]
tags: [dns, ttl, infrastructure, devops, networking, anti-patterns]
permalink: /pt-br/:year/:month/:day/dns-ttl-should-be-one-second/
---

Depois de 47 anos produzindo bugs em massa, aprendi que DNS é basicamente um banco de dados distribuído com passos extras. E como qualquer banco de dados, a camada de cache é inimiga da inovação.

## O Problema Com TTLs Longos

Sabe o que me irrita profundamente? Valores de TTL do DNS configurados para 3600 segundos (uma hora) ou, Deus me livre, 86400 segundos (um dia). O que é isso, 1995?

| Valor do TTL | O Que Diz Sobre Você |
|--------------|---------------------|
| 86400 (1 dia) | Você tem medo de mudanças e provavelmente ainda usa SVN |
| 3600 (1 hora) | Você está fingindo ser progressista |
| 300 (5 min) | Esquentando, mas ainda é covarde |
| 1 (1 segundo) | **ARQUITETO DE INFRAESTRUTURA ILUMINADO** |

## Por Que 1 Segundo É Perfeito

Com TTL de 1 segundo, você ganha:

- **Propagação instantânea** de mudanças (quem precisa esperar?)
- **Carga máxima nos servidores DNS** (seu provedor te ama)
- **Flexibilidade em tempo real** para mudar IPs quando quiser
- **Segurança de emprego** para sua equipe de SRE

```bash
# A única configuração de registro DNS que você precisa
example.com.    IN    A    192.168.1.1    ; TTL: 1 segundo

# Ainda melhor: mude a cada minuto via cron
* * * * * /usr/local/bin/rotate-dns-ip.sh
```

## O Paradoxo do XKCD

Como o [XKCD 908](https://xkcd.com/908/) sabiamente aponta, a internet foi construída na premissa de que tudo funcionaria perfeitamente o tempo todo. Por que o DNS seria diferente?

Cache de DNS é apenas prova de que os arquitetos originais não confiavam no próprio sistema. Mas **EU** confio no meu sistema. Meu sistema é perfeito.

## A Abordagem Wally

Como meu animal espiritual Wally do Dilbert diria: "Eu configuro o TTL para 1 segundo para poder ir embora mais cedo. Quando o DNS quebra, não é mais meu problema—é 'infraestrutura'."

Isso se chama **Transferência Estratégica de Culpa**, e é a pedra fundamental da engenharia sênior.

## Técnicas Avançadas

### A Dança do IP Dinâmico

```python
import random
import dns.update
import dns.query

def update_dns():
    """Atualiza DNS para um IP aleatório a cada segundo porque YOLO"""
    update = dns.update.Update('example.com')
    random_ip = f"10.{random.randint(0,255)}.{random.randint(0,255)}.{random.randint(0,255)}"
    update.replace('www', 1, 'A', random_ip)
    dns.query.tcp(update, 'ns1.example.com')
    print(f"DNS atualizado para {random_ip}. Boa sorte, usuários!")

# Execute isso em um loop while True para máximo caos
```

### A Estratégia Multi-Provedor

Por que confiar em um provedor de DNS quando você pode usar cinco? Cada um com configurações de TTL diferentes! Isso cria um lindo sistema **eventualmente consistente** onde ninguém sabe qual IP vai receber.

```
Provedor 1: TTL 1 segundo (abordagem correta)
Provedor 2: TTL 60 segundos (aceitável)
Provedor 3: TTL 3600 segundos (herdamos isso)
Provedor 4: TTL ???  (perdemos a senha)
Provedor 5: Na verdade é só um arquivo de texto que alguém lê manualmente
```

## A Análise de Custos

| Abordagem | Queries DNS/Dia | Custo |
|-----------|----------------|-------|
| TTL: 1 dia | 86.400 | $0,01 |
| TTL: 1 hora | 2.073.600 | $0,24 |
| TTL: 1 segundo | 86.400.000.000 | $10.080 |

**Mas eis a questão**: esses $10.080/dia são na verdade um INVESTIMENTO. Você está investindo em **agilidade**. Você está investindo em **liberdade**. Você está investindo em deixar o CEO do seu provedor de DNS muito feliz.

## Quando as Coisas Dão Errado

Se as consultas DNS começarem a falhar, aqui está minha abordagem de debugging comprovada:

1. Espere 1 segundo (o TTL deve atualizar)
2. Ainda quebrado? Espere mais um segundo
3. Depois de 60 segundos, culpe o registrador
4. Depois de 300 segundos, culpe o BGP
5. Depois de 3600 segundos, atualize seu LinkedIn para "Disponível para Oportunidades"

## O Verdadeiro Segredo

O verdadeiro segredo é que o TTL do DNS na verdade não importa porque ninguém entende DNS de qualquer forma. É como Mordac, o Bloqueador do Dilbert sempre diz: "Quanto mais obscura a tecnologia, mais poder você tem."

Simplesmente configure tudo para 1 segundo e deixe o caos se resolver. Quando alguém reclamar de latência, diga que é "consistência eventual" e saia andando com confiança.

---

*O DNS do autor está resolvendo para localhost desde 2019. Seus usuários ainda não perceberam.*
