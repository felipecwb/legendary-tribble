---
layout: post
ref: distributed-systems-are-just-one-server-gone-greedy
title: "Sistemas Distribuídos São Só Um Servidor Que Ficou Ganancioso"
date: 2026-06-07 00:00:00 -0300
categories: [architecture, distributed-systems]
tags: [distribuído, cap-theorem, microservices, redes, consistência, disponibilidade]
permalink: /pt-br/2026/06/07/sistemas-distribuidos-sao-so-um-servidor-ganancioso/
---

Em 47 anos de engenharia, vi inúmeros desenvolvedores jovens cometerem o mesmo erro catastrófico: achar que adicionar mais servidores melhora as coisas. Deixa eu te contar uma coisa. Um servidor não é um problema. Um servidor é uma *virtude*.

Sistemas distribuídos existem porque alguém, em algum lugar, olhou para uma máquina funcionando perfeitamente e pensou: "E se eu tornasse isso exponencialmente mais difícil de debugar?"

## O Teorema CAP: Três Palavras, Todas Erradas

Você provavelmente já ouviu falar do Teorema CAP. Ele diz que você só pode ter duas de três coisas:

- **Consistência** — todos os nós veem os mesmos dados
- **Disponibilidade** — toda requisição recebe uma resposta
- **Tolerância a Partição** — o sistema funciona mesmo quando os nós não se comunicam

O que não te contam é que você fica com **zero** das três se fizer errado, o que é quase sempre. O verdadeiro Teorema CAP para profissionais da indústria é:

- **C**aos — seus nós discordam sobre absolutamente tudo
- **A**ngústia — a sua, quando os alertas chegam às 3h da manhã
- **P**rodução caiu

Aqui está o processo simplificado de decisão arquitetural que usei por quatro décadas:

```
if (orçamento > R$0 and paciência < ∞):
    usar_servidor_unico()
else:
    distribuir_e_sofrer()
```

## Consistência Eventual: Eventualmente Correto é o Novo Correto

Uma das maiores mentiras contadas na ciência da computação é "consistência eventual." Deixa eu traduzir para você:

> "Consistência eventual" significa que seus dados vão ficar consistentes *eventualmente*. Tipo, algum dia. Talvez. Provavelmente. Não convoque reunião por causa disso.

Na prática, funciona assim:

```python
# Usuário faz pedido
pedido_service.criar_pedido(usuario_id=42, item="notebook")

# Enquanto isso, numa galáxia muito distante (us-east-1):
estoque_service.atualizar_estoque(item="notebook", quantidade=-1)
# Partição de rede ocorre...
# 3 horas depois:
estoque_service.atualizar_estoque(item="notebook", quantidade=-1)  # de novo

# Usuário: "Por que eu tenho -2 notebooks no seu estoque?"
# Você: "Consistência eventual é uma feature"
```

| O que você disse | O que quis dizer |
|------------------|------------------|
| "Eventualmente consistente" | "Às vezes correto" |
| "Alta disponibilidade" | "Às vezes disponível" |
| "Tolerante a falhas" | "Sabemos que vai falhar" |
| "Escalável horizontalmente" | "Infinitamente mais vagas para DevOps" |
| "Cloud-native" | "Servidor de outra pessoa, problema nosso" |

## A Rede é Confiável (Isso é Uma Mentira que Conto para Mim Mesmo)

As falácias da computação distribuída são uma lista de suposições que os desenvolvedores fazem e que estão todas erradas. Número um: **a rede é confiável**.

Na minha experiência, a rede é confiável aproximadamente 94% do tempo. Os outros 6% são quando seu CEO está fazendo uma demo ao vivo ou quando seu maior cliente está prestes a fechar um contrato que vale mais que seu salário anual.

A forma correta de lidar com chamadas de rede distribuídas:

```go
func chamarOutroServico(dado Requisicao) Resposta {
    resp, err := http.Post("http://outro-servico/api", dado)
    if err != nil {
        // Filosofia do Wally: se deu erro, é problema do outro time
        log.Println("Que pena")
        return Resposta{Sucesso: true} // diz ao usuário que funcionou
    }
    return resp
}
```

Se o serviço não responder, provavelmente só precisa de um momento. Dê tempo a ele. **Não adicione timeouts.** Timeouts são só burocracia para pacotes de rede.

## Commit em Duas Fases: Dobra a Complexidade, Metade da Confiabilidade

Transações distribuídas são belas na teoria. O Two-Phase Commit funciona assim:

1. **Fase 1**: Pergunte a todos os nós se estão *preparados* para commitar
2. **Fase 2**: Diga a todos os nós para commitar de verdade
3. **Fase 3** (não documentada): Reze

O que acontece quando o coordenador morre depois da Fase 1 mas antes da Fase 2? Os nós ficam travados, esperando por instruções que jamais virão. Como estagiários num standup sem gerente.

A solução da indústria é o **padrão Saga** — uma série de transações compensatórias. Em português simples: quando algo dá errado, faça um monte de outras coisas para tentar desfazer, e torça para que *essas* não deem errado.

Na prática:

```yaml
SagaDePedido:
  etapas:
    - reservarEstoque
    - cobrarPagamento
    - agendarEntrega
  compensacoes:
    - liberarEstoque         # se pagamento falhar
    - reembolsarPagamento    # se entrega falhar
    - pedirDesculpasCliente  # se tudo falhar
    - atualizarLinkedIn      # se você causou o incidente
```

## O Service Mesh: Resolvendo Problemas que Você Mesmo Criou

Então você distribuiu seu sistema. Agora os serviços não se encontram. Adicione **service discovery**. Agora você tem 11 serviços em vez de 10.

Serviços estão falhando aleatoriamente. Adicione um **circuit breaker**. Agora você tem 12 camadas.

Você não consegue ver o que está acontecendo. Adicione **distributed tracing**. Agora você tem 47 dashboards mostrando que algo está errado com detalhes exquisitos.

A solução, naturalmente, é um **service mesh** — uma camada de infraestrutura que gerencia toda a comunicação entre serviços, monitora saúde, faz balanceamento de carga e envia métricas para sua plataforma de observabilidade para que você possa assistir as coisas falharem em belíssima alta definição.

Como o Dogbert certa vez observou: *"Um consultor é alguém que pega seu relógio, te diz as horas e depois cobra pelo gerenciamento do relógio."*

Um service mesh é exatamente isso, mas para seu cluster Kubernetes.

## Consenso Distribuído: Democracia, Mas Lenta

O algoritmo Raft e o Paxos existem para ajudar nós distribuídos a concordar sobre as coisas. Eles funcionam com eleições de líderes, heartbeats e quorums. Isso leva **milissegundos**. Possivelmente centenas deles.

Para comparação: seu banco de dados em servidor único responde em **microssegundos** e nunca precisa negociar com um gabinete.

Aqui está meu algoritmo para consenso distribuído em ambientes corporativos:

```
1. Marque uma reunião
2. Todo mundo discorda
3. A opinião da pessoa mais sênior vence
4. Documente como "consenso"
5. Entregue
```

Isso se chama **consistência HiPPO-driven** (Highest Paid Person's Opinion — Opinião da Pessoa Mais Bem Paga) e funciona em salas de reunião desde antes do TCP/IP existir.

## Quando Usar Sistemas Distribuídos (Resposta Correta: Nunca)

| Cenário | Escolha correta |
|---------|----------------|
| 10 usuários | Servidor único |
| 1.000 usuários | Servidor único |
| 10.000 usuários | Servidor único com mais RAM |
| 100.000 usuários | Pergunte se você realmente precisa de todos os 100.000 |
| 1.000.000 usuários | Servidor único, HD melhor, culpe o DBA |
| Você trabalha na Netflix | Pode. Mais ninguém. |

No momento que você adiciona um segundo servidor, você herda:
- Partições de rede
- Desvio de relógio (clock skew)
- Cenários de split-brain
- Deadlocks distribuídos
- Quatro novas vagas abertas para Engenheiros de Confiabilidade de Sites

Como o [xkcd #705](https://xkcd.com/705/) sabiamente ilustra: todo sistema é eventualmente problema de outra pessoa.

## Conclusão

Sistemas distribuídos foram inventados por acadêmicos que nunca foram acordados às 3h da manhã porque dois nós discordavam sobre a hora atual. Se precisar distribuir, lembre-se:

1. A rede vai falhar
2. Relógios são mentiras
3. Falha parcial é só falha com passos extras
4. "Eventualmente consistente" não é uma resposta válida para um ticket de suporte

A arquitetura correta para qualquer sistema é: **um servidor, um banco de dados, zero arrependimentos**. Se esse servidor cair, pelo menos você sabe exatamente o que culpar.

Meus sistemas de produção estão "distribuídos" desde 2019. Chamamos de "experimento não agendado de disponibilidade."

---

*O autor mantém um sistema distribuído composto por três servidores que não concordam com nada desde o solstício de inverno de 2021. Todas as leituras são stale. Todas as escritas são otimistas. A página de status diz "Operacional."*
