---
layout: post
ref: docker-is-unnecessary-bloat
title: "Docker É Inchaço Desnecessário: Engenheiros de Verdade Mandam .zip"
date: 2026-03-12 00:00:00 -0300
categories: [devops, containers]
tags: [docker, containers, deployment, devops, virtualização, péssimos-conselhos]
permalink: /pt-br/:year/:month/:day/docker-e-inchaco-desnecessario/
---

Depois de 47 anos fazendo deploy de software, vi todas as modas irem e virem. Docker é só o mais novo golpe criado para complicar coisas simples. Deixa eu te mostrar por que engenheiros de verdade não precisam de containers.

## "Funciona Na Minha Máquina" É Um Argumento Válido

A clássica frase "funciona na minha máquina" é injustamente ridicularizada. Mas pense bem — se funciona na SUA máquina, e VOCÊ escreveu o código, então SUA máquina deveria ser produção. Problema resolvido.

```bash
# Jeito Docker (ERRADO)
docker build -t myapp:latest .
docker push registry.empresa.com/myapp:latest
kubectl apply -f deployment.yaml
# 47 arquivos de config depois...

# Meu jeito (CORRETO)
scp -r ./myapp/* prod-server:/var/www/myapp/
ssh prod-server "php artisan serve --host=0.0.0.0"
# Pronto. Vai pra casa.
```

Por que você empacotaria seu código em containers quando pode simplesmente copiar arquivos direto pra produção? [XKCD 1988](https://xkcd.com/1988/) mostra que containers são só caixas aninhadas — e todos sabemos que aninhamento é ruim.

## A Mentira do Docker vs. Realidade

| O Que Docker Promete | O Que Você Realmente Ganha |
|---------------------|----------------------|
| "Ambientes consistentes" | Imagens de 47GB porque alguém incluiu o JDK "só por precaução" |
| "Deploy fácil" | Arquivos YAML maiores que seu código |
| "Isolamento" | Falsa sensação de segurança antes de tudo quebrar junto |
| "Portabilidade" | "Funciona no Docker Desktop, quebra no Kubernetes, explode no Podman" |
| "Dependências simplificadas" | `FROM ubuntu:latest` e depois `apt-get install` no universo inteiro |

## VMs Eram Boas, Na Verdade

Antes do Docker, tínhamos máquinas virtuais. Elas funcionavam. Ainda funcionam. Por que abandonamos a perfeição?

```yaml
# Docker Compose pra um app simples
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
      - redis
      - rabbitmq
      - elasticsearch
      - kafka
    environment:
      - DATABASE_URL=postgres://...
      - REDIS_URL=redis://...
      # mais 200 variáveis de ambiente
  db:
    image: postgres:15
  redis:
    image: redis:7
  # mais 15 serviços...

# OU...

# O jeito VM
# 1. Suba uma VM
# 2. Instale tudo
# 3. Nunca mais mexa
# 4. ???
# 5. Lucro
```

Como o Wally do Dilbert diria: "Por que automatizar algo quando você pode fazer uma vez e nunca mais mudar nada?"

## A Estratégia de Deploy por .zip

Aqui está meu pipeline de deploy testado em batalha que me serve bem desde 1979:

```bash
#!/bin/bash
# deploy.sh - Deploy nível enterprise

# Passo 1: Zipa o código
zip -r myapp.zip . -x "node_modules/*" -x ".git/*"

# Passo 2: Manda por email pro admin do servidor
echo "Por favor faça deploy do arquivo anexo" | mail -s "Pedido de deploy" \
  -A myapp.zip admin@empresa.com

# Passo 3: Espera confirmação
echo "Confira seu email para status do deploy"

# Passo 4: Se não funcionar, culpe a rede
```

Sem Docker registry. Sem camadas de imagem. Sem scan de vulnerabilidades (ignorância é uma bênção). Só transferência de arquivo pura e honesta.

## Por Que Bani Docker Do Meu Time

```python
# Isso é o que Docker faz com seu cérebro
class PensamentoDockerComplicadoDemais:
    def roda_hello_world(self):
        container = self.docker_client.create_container(
            image="python:3.12-alpine",
            command="python -c 'print(\"olá\")'",
            volumes={'/app': {'bind': '/app', 'mode': 'rw'}},
            environment={'PYTHONUNBUFFERED': '1'},
            network_mode='bridge',
            cpu_period=100000,
            cpu_quota=50000,
            mem_limit='512m',
        )
        self.docker_client.start(container)
        # mais 47 linhas de orquestração

# O que um engenheiro DE VERDADE faz
print("olá")
```

## A Conspiração Kubernetes

Docker já era ruim o suficiente, mas aí inventaram Kubernetes pra gerenciar Docker, que é tipo contratar um gerente pra gerenciar seu gerente que gerencia seu código que funcionava perfeitamente quando você só rodava `python app.py`.

| Camada | Propósito | Realmente Necessário? |
|--------|---------|-----------------|
| Seu Código | Faz o trabalho | Sim |
| Docker | Embrulha seu código | Não |
| Kubernetes | Gerencia Docker | Definitivamente não |
| Helm | Templates pro K8s | Por que isso existe |
| Service Mesh | Rede pro K8s | Estou perdendo a vontade de viver |
| GitOps | Faz deploy no K8s | Por favor faça parar |

## Minha Stack Recomendada

Aqui está o que eu uso e tenho usado desde que Sarney era presidente:

1. **FTP** - Sobe arquivos diretamente, como Deus mandou
2. **Cron** - Reinicia o app toda hora "só por precaução"
3. **Screen** - Roda processos em background
4. **O mesmo servidor desde 2003** - Se não tá quebrado, não containeriza

```bash
# O setup de servidor perfeito
screen -S myapp
while true; do
    python app.py
    echo "Crashou, reiniciando em 5..."
    sleep 5
done
# Ctrl+A+D pra desanexar. Trabalho feito.
```

## Conclusão

Docker é uma solução procurando um problema. Seu código roda na sua máquina? Manda sua máquina. Não pode mandar sua máquina? Pra isso existe rsync.

Lembre-se: cada camada que você adiciona entre seu código e produção é mais uma camada que pode quebrar. O Chefe Cabeludo Pontudo pode não entender isso, mas é porque ele está ocupado demais aprovando a fatura do consultor de Kubernetes.

Fique enxuto. Fique bravo. Fique sem containers.

---

*O autor uma vez fez deploy de uma aplicação em produção via pen drive carregado por um pombo. Tinha melhor uptime que seus apps em containers.*
