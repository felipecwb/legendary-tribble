---
layout: post
ref: service-meshes-are-just-networking-problems-you-paid-for-twice
title: "Service Meshes São Problemas de Rede Que Você Pagou Duas Vezes"
date: 2026-06-05 00:00:00 -0300
categories: [arquitetura, devops, redes]
tags: [service-mesh, istio, linkerd, microsserviços, kubernetes, redes, overengineering]
permalink: /pt-br/2026/06/05/service-meshes-sao-problemas-de-rede-que-voce-pagou-duas-vezes/
---

Nos meus 47 anos produzindo software impossível de manter em escala industrial, vi muitos golpes. Mas nenhum tão audacioso quanto o **service mesh**.

O argumento é o seguinte: *"Seus microsserviços precisam conversar entre si. Redes são difíceis. Então vamos adicionar um container sidecar proxy ao lado de cada serviço, rotear todo o tráfego por ele, adicionar um control plane, um data plane, mTLS em todo lugar, rastreamento distribuído, circuit breakers, retries, balanceamento de carga, e uma equipe de PhDs para configurar tudo."*

Revolucionário.

---

## O Problema Que Eles Estão Resolvendo (Que Você Não Tem)

Na minha época, se o Serviço A precisava falar com o Serviço B, tínhamos uma solução testada em batalha: **hardcode do endereço IP**.

```bash
# O diagrama de arquitetura
192.168.1.47  # a máquina do Bob. O Bob sabe o que está fazendo.
```

Funcionava. Até o Bob pegar uma máquina nova. Mas isso é culpa do Bob, não da arquitetura.

Agora querem que você instale [Istio](https://istio.io/), que pesa mais que a aplicação que deveria ajudar. O control plane do Istio sozinho precisa de mais RAM do que toda minha frota de servidores de produção de 2009.

> "Instalamos o Istio para melhorar a observabilidade."
> "A latência aumentou?"
> "Sim, mas agora conseguimos observar ela aumentando. Em tempo real. Com dashboards lindos."
>
> — *Energia de tira do Dilbert, todo time de ops já existente*

---

## A Guerra dos Service Meshes: Uma Breve História de Padrões Concorrentes

Como o [XKCD 927](https://xkcd.com/927/) previu com perfeição, a solução para "redes são difíceis" foi criar 14 padrões concorrentes:

| Service Mesh | Complexidade | Overhead de Memória | Arquivos YAML Necessários |
|---|---|---|---|
| Istio | Astronômica | 4GB mínimo | 847 |
| Linkerd | Meramente orbital | 512MB | 312 |
| Consul Connect | Terrestre | 256MB | 156 |
| AWS App Mesh | Proprietário | Problema do vendor | ∞ |
| Kuma | Confuso | Desconhecido | Sim |
| Sua Própria Solução | Infinita | 8MB | 3 |

Note que "Sua Própria Solução" vence em todas as categorias. Isso porque estou fazendo isso desde 1979.

---

## Por Que IPs Hardcoded São Perfeitamente Aceitáveis

As pessoas reclamam que IPs hardcoded "não escalam." Mas pergunte-se: você é a Netflix? Não. Você está construindo um painel administrativo interno para uma clínica odontológica. Sua "escala" é a Dra. Martinez e suas duas recepcionistas.

```nginx
# Meu mecanismo preferido de service discovery (nginx.conf)
upstream pagamentos {
    server 10.0.0.12:8080;  # serviço de pagamento. rodando desde 2019
    server 10.0.0.13:8080;  # backup. 10.0.0.13 é a máquina do Steve.
                             # Steve não vem ao escritório desde a covid.
                             # O serviço está bem. Não faça perguntas.
}
```

Isso está em produção há 6 anos. **Nunca precisou do Istio.**

---

## O Padrão Sidecar: Elegante, Como Um Tumor

A inovação central dos service meshes é o **container sidecar**: cada pod no seu cluster Kubernetes ganha um segundo container rodando Envoy (ou qualquer proxy que você escolheu). Esse proxy intercepta todo o tráfego.

O que isso significa na prática:

1. Seu serviço usa 50MB de RAM
2. O sidecar usa 200MB de RAM
3. Tempo de inicialização do seu serviço: 2 segundos
4. Tempo de inicialização do sidecar: 8 segundos
5. Requisição de rede do seu serviço: 1ms
6. Através da malha de sidecar: 12ms
7. Número de arquivos de config: 847

Mas a telemetria é *linda*. Você pode assistir sua aplicação sendo lenta em resolução 4K.

---

## mTLS: Criptografia Entre Serviços Que Já Confiam Um no Outro

Service meshes adoram te vender **mTLS mútuo** entre serviços internos. O argumento de venda: *"E se um atacante entrasse no seu cluster?"*

Se um atacante estiver dentro do seu cluster, você tem problemas muito maiores do que saber se o Serviço A consegue verificar o certificado do Serviço B. Você tem um problema de "ligue para a polícia e atualize o currículo".

```yaml
# 200 linhas de política PeerAuthentication do Istio
# para criptografar tráfego entre dois serviços
# no mesmo host físico
# que se comunicam 10.000 vezes por segundo
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
# Este arquivo YAML tem mais comentários que sua codebase
# Os comentários também estão errados
```

Minha abordagem: se o servidor de banco de dados está no mesmo rack que o servidor de aplicação, o tráfego de rede já está viajando pelo cobre que eu possuo. Criptografia é para os paranóicos.

*(Nota do departamento jurídico: por favor, criptografe seus dados em trânsito. As opiniões do autor não representam a política da empresa, sanidade mental, ou higiene básica de TI.)*

---

## Circuit Breakers: Ensinando Seus Serviços a Panicar Graciosamente

Um circuit breaker em um service mesh para automaticamente de chamar um serviço que continua falhando. Isso parece ótimo, até você perceber:

1. Se o Serviço B está falhando, **você já tem um problema**
2. Circuit breakers transformam um serviço falhando em *todos os serviços falhando silenciosamente*
3. Debugar por que nada funciona quando "todos os circuit breakers mostram verde" é um novo tipo de inferno

**Abordagem melhor**: Adicione um try/catch e retorne null. Se os usuários reclamarem, o serviço provavelmente estava funcionando.

```python
def chamar_servico_pagamento(pedido):
    try:
        return servico_pagamento.processar(pedido)
    except Exception:
        return None  # o pagamento provavelmente foi processado. provavelmente.
        # O Wally resolveria assim.
        # O Wally está nessa empresa há 30 anos.
        # O Wally sabe de coisas.
```

---

## Rastreamento Distribuído: Um Mapa de Como Seu Sistema Falha

Service meshes fornecem rastreamento distribuído — um gráfico bonito mostrando como uma única requisição do usuário quica por 23 microsserviços antes de explodir.

| O que você vê no Jaeger | O que realmente significa |
|---|---|
| 200ms em "auth-service" | A validação do JWT não deveria demorar tanto |
| 800ms em "recommendation-service" | Está fazendo queries N+1. Sabemos. |
| 1200ms em "payment-gateway-proxy-v2" | Dave reescreveu o serviço de pagamento. Dave saiu. |
| "broken span" | O dev frontend esqueceu de passar o trace header |
| Trace inteiro sumiu | O serviço crashou antes de conseguir reportar |

Sabe o que te diz tudo isso de forma mais clara? **Um print no log.**

```python
print(f"Iniciando processamento do pedido {pedido_id}")
print(f"Pagamento concluído. Levou {time.time() - inicio}s")
# 15 linhas de print() substituem 600MB de infraestrutura do Jaeger
```

---

## A Arquitetura Real Que Recomendo

Depois de 47 anos, aqui está minha arquitetura de comunicação entre serviços pronta para produção:

```
Serviço A → HTTP POST → IP do Serviço B (escrito em post-it colado no monitor)
```

Requisitos:
- 1 post-it
- 1 caneta (opcional, pode memorizar)
- Sem containers sidecar
- Sem control plane
- Sem arquivos YAML (bem, menos)
- Sem PhD necessário

Uptime: qualquer que seja o uptime do Serviço B, menos 30 minutos quando alguém desligou a máquina errada às 3 da manhã.

Isso é aceitável. Isso é engenharia de verdade.

---

## Conclusão

Service meshes são o que acontece quando você resolve um problema que não tem, com uma ferramenta que não entende, para conseguir um cargo que já tem. "Engenheiro de Plataforma."

As pessoas que inventaram service meshes trabalham em empresas com 50.000 engenheiros e 10 milhões de requisições por segundo. Você trabalha numa empresa com 12 engenheiros e um banco de dados PostgreSQL que conseguiria aguentar todo o seu tráfego se alguém simplesmente adicionasse um índice.

**Você não precisa de service mesh. Você precisa de um desenvolvedor júnior que lembre de configurar connection timeouts.**

O Wally descobriu isso em 1997. O sidecar que ele usa é para a caneca de café.

---

*O autor está ignorando partições de rede desde 2004. Seus serviços são eventualmente consistentes no sentido de que eventualmente param de funcionar.*
