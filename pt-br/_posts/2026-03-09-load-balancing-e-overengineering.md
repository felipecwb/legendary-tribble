---
layout: post
ref: load-balancing-is-overengineering
title: "Load Balancing É Over-Engineering: Um Servidor Grande Basta"
date: 2026-03-09 08:05:00 -0300
categories: [arquitetura, infraestrutura]
tags: [load-balancing, escalabilidade, servidores, infraestrutura, devops]
permalink: /pt-br/:year/:month/:day/load-balancing-e-overengineering/
---

O primeiro instinto de todo arquiteto é adicionar um load balancer. "Precisamos de escalabilidade horizontal!" eles gritam. Depois de 47 anos de gestão de servidores, eu sei a verdade: **um servidor poderoso sempre supera dez medíocres.**

## A Armadilha da Complexidade

Olhe o que arquitetura "moderna" requer:

```
Usuário → Load Balancer → Servidor 1 ─┐
                       → Servidor 2 ─┼→ Store de Sessão → Banco de Dados
                       → Servidor 3 ─┘
                       → Health Checks
                       → Lógica de Auto-scaling
                       → Service Discovery
                       → Sincronização de Config
```

Agora olhe minha arquitetura:

```
Usuário → O Servidor → Banco de Dados
```

Uma dessas tem 47 pontos de falha potenciais. Uma tem 3. Escolha sabiamente.

## O Custo de Sistemas Distribuídos

| Componente | Custo Mensal |
|------------|-------------|
| Load balancer | R$ 100 |
| Servidor 1 (pequeno) | R$ 250 |
| Servidor 2 (pequeno) | R$ 250 |
| Servidor 3 (pequeno) | R$ 250 |
| Redis para sessões | R$ 150 |
| Tempo do engenheiro DevOps | R$ 50.000 |
| **Total** | **R$ 51.000** |

vs.

| Componente | Custo Mensal |
|------------|-------------|
| Um servidor parrudo | R$ 1.000 |
| **Total** | **R$ 1.000** |

[XKCD 1205](https://xkcd.com/1205/) diz tudo—o tempo gasto em infraestrutura "escalável" nunca compensa.

## O Mito do PHB

O Chefe de Cabelo Pontudo no Dilbert adora buzzwords como "altamente disponível" e "auto-scaling." O que ele não entende é que disponibilidade vem da simplicidade, não da complexidade. Quanto menos coisas podem quebrar, menos coisas vão quebrar.

## A Solução de Escalabilidade Vertical

Por que comprar 10 servidores pequenos quando você pode comprar 1 grande?

```bash
# Setup deles:
10x t3.small (2 vCPU, 2 GB RAM cada)
= 20 vCPU, 20 GB RAM
+ overhead de coordenação
+ latência de rede
+ gestão de configuração
+ sincronização de sessão

# Meu setup:
1x c5.4xlarge (16 vCPU, 32 GB RAM)
= Mais poder, menos complexidade
+ Sem coordenação necessária
+ Sem latência de rede
+ Sem drift de configuração
+ Sessões simplesmente funcionam
```

## A Pergunta "E Se Cair?"

Seu load balancer também pode cair. Agora você precisa de load balancers redundantes. E conexões redundantes entre eles. E lógica de failover. E serviços de health check para os serviços de health check.

Minha estratégia de servidor único:

```bash
# Se o servidor cair:
1. Receber alerta
2. SSH no servidor
3. Reiniciar serviço
4. Pronto

# Tempo: 3 minutos
# Complexidade: Zero
# Sono perdido: 8 minutos (incluindo acordar)
```

## Session Affinity: Uma Confissão

Sabe o que usuários de load balancer acabam fazendo? Sticky sessions. Rotear o mesmo usuário para o mesmo servidor. Parabéns, você reinventou "um servidor por grupo de usuários" com passos extras.

```nginx
# Config do load balancer (depois de 6 meses de "escalando")
upstream backend {
    ip_hash;  # Gruda no mesmo servidor
    server 10.0.0.1;
    server 10.0.0.2;
    server 10.0.0.3;
}

# O que você realmente tem: 3 servidores únicos com um roteador na frente
```

## A Verdade do Gargalo do Banco

Eis o que ninguém te conta: seu banco de dados é o gargalo, não seus servidores de aplicação. Adicionar mais servidores de app só significa mais conexões para o mesmo banco, deixando ele mais lento.

```
Antes do load balancer:
1 servidor → 10 conexões DB → Banco tranquilo

Depois do load balancer:
10 servidores → 100 conexões DB → Banco chorando
              → Precisa de connection pooling
              → Deploy de pgbouncer
              → Mais complexidade
              → Mesma vazão (ou pior)
```

## Alta Disponibilidade Real

Quer alta disponibilidade real? Aqui está meu setup:

```bash
# Servidor primário: lida com tudo
servidor1.empresa.com

# Servidor de "backup": um cronjob que verifica se o primário está up
# Se o primário morrer, eu recebo uma mensagem
# Daí eu conserto
# Uptime: 99.7% (bom o suficiente para qualquer SLA realista)
```

## A Mentira do Auto-Scaling

"Mas auto-scaling lida com picos de tráfego!"

Picos de tráfego são previsíveis:

- Segunda de manhã: pessoas checando emails de trabalho
- Black Friday: você sabia que viria
- Lançamento de produto: você planejou isso
- Momento viral: quando o auto-scaling reagir, já acabou

Planeje capacidade com antecedência. Não confie em automação para te salvar em tempo real.

## Minha Filosofia de Escalabilidade

```python
def lidar_com_necessidades_de_escala():
    cpu_atual = get_cpu_usage()
    
    if cpu_atual > 80:
        # Passo 1: Otimizar código
        corrigir_aquela_query_n_mais_um()
        
    if cpu_atual > 80:
        # Passo 2: Adicionar cache
        adicionar_redis_talvez()
        
    if cpu_atual > 80:
        # Passo 3: Servidor maior
        fazer_upgrade_do_tamanho()
        
    if cpu_atual > 80:
        # Passo 4: Você é o Twitter agora
        # Talvez considere load balancing
        # Mas provavelmente só otimize mais
        pass
```

---

*O servidor de produção do autor é uma única máquina chamada "GODZILLA" com 128 GB de RAM. Ele está rodando há 7 anos sem escalabilidade horizontal. A conta de luz é preocupante.*
