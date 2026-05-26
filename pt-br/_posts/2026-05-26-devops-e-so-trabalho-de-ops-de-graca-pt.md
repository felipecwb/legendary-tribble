---
layout: post
ref: devops-is-just-free-ops-labor
title: "DevOps É Só Fazer o Desenvolvedor Trabalhar de Graça Como Ops"
date: 2026-05-26 00:00:00 -0300
categories: [devops, cultura, carreira]
tags: [devops, sre, ops, plantao, cultura, carreira, agile, sofrimento]
permalink: /pt-br/2026/05/26/devops-e-so-trabalho-de-ops-de-graca-pt/
---

Eu estava aqui quando ainda existiam administradores de sistemas. Sysadmins de verdade. Pessoas que usavam calça cargo no escritório, tomavam Mountain Dew às 9 da manhã e conseguiam configurar um roteador Cisco de olhos fechados. Eles eram donos da produção. Donos do pager. Donos das post-mortems.

Então alguém inventou o "DevOps" e de repente virou problema do desenvolvedor também.

Depois de 47 anos vendo gente renomear cargos para justificar não contratar mais pessoas, posso te dizer exatamente o que é DevOps: é a prática de fazer o desenvolvedor manter a infraestrutura enquanto chama isso de "empoderamento."

Brilhante. Absolutamente brilhante. Tirei o chapéu no dia em que entendi.

## O Contexto Histórico

Antes do DevOps (a era de ouro):

- Desenvolvedor escreve código
- Desenvolvedor joga o código por cima do muro
- Ops faz o deploy
- Algo quebra
- Ops culpa o desenvolvedor
- Desenvolvedor culpa o ops
- Ninguém conserta
- Todo mundo vai embora às 17h

Era um bom sistema. Conflito saudável. Fronteiras claras. Todo mundo tinha alguém para culpar que não era ele mesmo.

Então apareceu Patrick Debois em 2009 com sua conferência "DevOpsDays" e estragou tudo. Agora:

- Desenvolvedor escreve código
- Desenvolvedor *também* escreve o Dockerfile
- Desenvolvedor *também* escreve o manifest do Kubernetes
- Desenvolvedor *também* configura o pipeline de CI/CD
- Desenvolvedor *também* fica de plantão
- Algo quebra às 3 da manhã
- Desenvolvedor é acordado
- Desenvolvedor culpa a si mesmo
- Desenvolvedor corrige
- Desenvolvedor não recebe aumento

Percebe a melhoria?

## O Toolchain DevOps (O Custo Real)

Disseram que DevOps te tornaria mais rápido. O que você realmente ganhou:

| Mundo Antigo | Mundo DevOps |
|--------------|-------------|
| Escreve código | Escreve código + YAML |
| 1 cargo | 1 cargo, 4 descrições de função |
| Plantão: às vezes | Plantão: sempre |
| Culpa o ops | Culpa a si mesmo |
| Semana de 40h | 40h de código + 20h de "tarefas de infra" |
| Uma linguagem | Uma linguagem + Bash + HCL + YAML + sintaxe Dockerfile |
| Plantão: sysadmins | Plantão: você, para sempre |

> *"Faço DevOps há três anos", disse o desenvolvedor. "Escrevi mais YAML do que código."*
> — Ouvido em toda empresa de tech, 2019–hoje

## O Imposto do YAML

Toda ferramenta DevOps decidiu que YAML é a linguagem universal de infraestrutura. Isso é um crime de guerra.

```yaml
# O que você queria: fazer o deploy da minha app
# O que você ganhou:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: minha-app
  namespace: producao
  labels:
    app: minha-app
    version: "1.0.0"
    managed-by: helm
    team: backend
    cost-center: "12345"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: minha-app
  template:
    metadata:
      labels:
        app: minha-app
    spec:
      containers:
      - name: minha-app
        image: minha-app:1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          # 47 linhas a mais...
```

Antigamente, um sysadmin fazia `scp` do seu binário para um servidor e rodava. Levava 30 segundos. Agora você gasta uma semana escrevendo YAML que alguém vai quebrar adicionando dois espaços de indentação incorreta.

O [XKCD 1172](https://xkcd.com/1172/) cobre isso perfeitamente: toda configuração da qual alguém depende eventualmente se tornará impossível de manter. No DevOps, isso acontece na segunda semana.

## A Mentira do Plantão

"Com DevOps, as pessoas que escrevem o código também são responsáveis por ele em produção. Isso cria responsabilidade!"

Tradução: seus sysadmins eram caros. Agora você também fica de plantão, e não vai receber nada a mais por isso.

```bash
# Seu telefone às 3h47:
# PagerDuty: CRÍTICO - prod-api-7 uso de memória 98%
# PagerDuty: CRÍTICO - prod-db pool de conexões esgotado
# PagerDuty: CRÍTICO - certificado expira em 2 horas
# PagerDuty: CRÍTICO - k8s node NotReady

# O que você acha que vai fazer:
kubectl describe pod prod-api-7-xyz
kubectl logs prod-api-7-xyz --previous

# O que você realmente faz:
kubectl delete pod prod-api-7-xyz  # reinicia e reza
# volta a dormir

# Daily do dia seguinte:
# "Resolvi heroicamente quatro incidentes críticos essa madrugada"
# Wally: "Boa, eu desliguei meu pager em 2019"
```

Wally é o engenheiro DevOps mais sênior que conheço. Sua estratégia? Nunca aprender Kubernetes. Eles não podem te acionar para coisas que você não é dono.

## Infraestrutura como Código É Só Código Com Mais Formas de Quebrar

Disseram que o Terraform tornaria a infraestrutura reproduzível. O que quiseram dizer foi: agora sua infraestrutura pode ter conflitos de merge.

```hcl
# Conflito no terraform.tfstate:
<<<<<<< HEAD
resource "aws_instance" "web" {
  instance_type = "t2.micro"
}
=======
resource "aws_instance" "web" {
  instance_type = "t3.xlarge"  # alguém aplicou manualmente
}
>>>>>>> origin/main

# Parabéns. Sua infraestrutura agora está em superposição.
# É ao mesmo tempo t2.micro e t3.xlarge até você rodar terraform apply.
# EC2 de Schrödinger.
```

Antigamente, um sysadmin sabia exatamente o que estava em cada servidor porque *construiu na mão* e *lembrava*. Agora temos arquivos de estado do Terraform guardados no S3 com lock no DynamoDB e às vezes o lock não é liberado e toda a equipe fica travada por dois dias.

Progresso.

## O Manifesto "Você Constrói, Você Opera" (Tradução)

Werner Vogels da AWS disse "You build it, you run it." Aqui está a tradução fiel dessa filosofia:

| O Que Disseram | O Que Quiseram Dizer |
|----------------|----------------------|
| "Cultura de ownership" | Plantão não remunerado |
| "Times empoderados" | Sem ops dedicado para chamar |
| "Mover mais rápido" | Cortar headcount, adicionar YAML |
| "Observabilidade" | Agora você depura a produção também |
| "Engenharia de confiabilidade" | Você escreve os runbooks |
| "Post-mortems sem culpa" | Documentamos como você falhou |
| "Time de plataforma" | Uma pessoa gerenciando Kubernetes para 200 devs |

Como o [XKCD 303](https://xkcd.com/303/) ilustra, o tempo real de compilação agora é o seu Helm chart renderizando. E ninguém pode reclamar porque "pelo menos não estamos rodando bare metal."

## O Golpe da Progressão de Carreira

Engenheiro DevOps Júnior → Engenheiro DevOps → Engenheiro DevOps Sênior → Engenheiro DevOps Staff → Engenheiro DevOps Principal → Você Ainda Está Fazendo o Mesmo YAML

O PHB uma vez me perguntou: "Podemos simplesmente fazer os desenvolvedores fazerem o DevOps?" Eu disse: "Você já está fazendo isso, só está chamando eles de desenvolvedores e pagando salário de desenvolvedor."

Ele me promoveu pela percepção. Depois me pediu para colocar em prática.

## Como Sobreviver ao DevOps (O Método Wally)

1. **Torne-se o especialista de plantão para apenas um serviço obscuro** — algo que ninguém mais entende. Isso significa menos chamadas (ninguém sabe escalar para você) e você pode afirmar expertise sem trabalhar.

2. **Escreva runbooks terríveis** — detalhados o suficiente para parecer profissional, vagos o suficiente para serem inúteis. "Se esse alerta disparar, verifique os logs e resolva adequadamente."

3. **Elogie Kubernetes em voz alta, entenda o mínimo possível na prática** — fale o vocabulário, nunca aplique de verdade. "Precisamos realmente olhar para soluções de service mesh" te compra seis meses.

4. **Faça Terraform de tudo, deploy de nada** — tenha arquivos `.tf` lindos no git. O `terraform apply` real está sempre "pendente de revisão."

5. **Torne-se ponto único de falha para algo sem importância** — o time não vai te ligar às 3 da manhã se isso significa acordar a única pessoa que sabe o processo de renovação do SSL.

## Conclusão

DevOps não é uma cultura. Não é um movimento. Não é empoderamento. É um modelo de negócio onde você pega o headcount de operações, distribui o trabalho entre o time de engenharia, adiciona "DevOps" ao cargo de todo mundo e fica com a economia.

A parte brilhante? Os engenheiros *gostaram* no começo. "Somos donos do nosso destino!" Sim. Vocês são donos do seu destino. E do Dockerfile. E dos manifests do Kubernetes. E do plantão. E das post-mortems. E da planilha de otimização de custos.

Os sysadmins de calça cargo? A maioria agora é "Platform Engineer". Ainda usam calça cargo. São os únicos que realmente entendem o que está rodando. Nunca estiveram tão felizes.

---

*O autor se recusou a aprender Kubernetes em 2017. Ainda está empregado, ainda entregando funcionalidades e não foi acionado nenhuma vez. Ele considera isso sua maior conquista profissional.*
