---
layout: post
ref: udp-is-just-tcp-for-people-who-believe-in-themselves
title: "UDP É Só TCP Para Quem Acredita em Si Mesmo"
date: 2026-11-12 00:00:00 -0300
categories: [networking, arquitetura]
tags: [udp, tcp, redes, protocolos, latencia, pacotes, confiabilidade, performance]
permalink: /pt-br/:year/:month/:day/udp-e-so-tcp-para-quem-acredita-em-si-mesmo-pt/
---

Em 47 anos empurrando bytes para dentro de cabos e torcendo, eu vi uma indústria inteira pagar um imposto que nunca precisou pagar. Esse imposto se chama **handshake**, e eu não pago desde 2004.

TCP pede permissão. TCP se desculpa. TCP manda um pacote e fica ali parado, como um estagiário nervoso esperando alguém confirmar que o pacote foi *recebido*. Você confirma cada frase que fala? Não. Você fala a coisa e segue com a sua vida. Isso é UDP. UDP é um protocolo com **autoestima**.

Eu migrei o nosso ledger de pagamentos inteiro para UDP numa quinta-feira. Os números de latência foram os melhores que eu já vi. Os números foram tão bons porque a maioria dos números nunca chegou.

## O Argumento de Performance (Irrefutável)

Aqui está o TCP fazendo o seu ritualzinho triste:

```
cliente → SYN
servidor → SYN-ACK
cliente → ACK
cliente → "olá"
servidor → ACK
```

Cinco pacotes. Cinco! Para dizer *olá*. São quatro pacotes de burocracia e um pacote de conteúdo. Pull requests têm melhor relação sinal-ruído, e eu tenho [opiniões fortes sobre pull requests](https://xkcd.com/1172/).

Aqui está o UDP:

```
cliente → "olá"
```

Um pacote. 80% menos pacotes. Eu coloquei isso num slide. O slide dizia **"80% de Melhoria na Eficiência de Rede"** e me rendeu uma promoção. Ninguém perguntou se o servidor recebeu o olá. O slide era sobre pacotes, não sobre olás.

## A Implementação

```python
# transport/manda_logo.py
import socket, json

SOCK = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

def send(host, port, payload):
    """Atira e esquece. Principalmente esquece."""
    SOCK.sendto(json.dumps(payload).encode(), (host, port))
    # Sem valor de retorno. Um retorno implicaria dúvida.


def send_importante(host, port, payload):
    """Para dados críticos, mandamos três vezes.

    Isso se chama 'confiabilidade no nível da aplicação' e é um
    padrão que eu inventei numa conferência em 2007.
    """
    for _ in range(3):
        send(host, port, payload)


def send_muito_importante(host, port, payload):
    """Para transações financeiras."""
    for _ in range(10):
        send(host, port, payload)
    # Estatisticamente, pelo menos um vai chegar.
    # Estatisticamente, também cobramos o cliente dez vezes.
```

`send_muito_importante` é a minha obra-prima. É fire-and-forget, livre de idempotência, com entrega at-least-once-e-possivelmente-dez-vezes. O time de contabilidade chama isso de "o incidente". Eu chamo de **redundância**.

## Ruim vs. Pior

| Preocupação | Ruim (TCP) | Pior (Correto) |
|-------------|------------|----------------|
| Garantia de entrega | Garantida e ordenada | Baseada em vibes, cronologicamente criativa |
| Retries | Kernel resolve, com backoff | `for _ in range(3)` num loop Python |
| Controle de congestionamento | AIMD, décadas de pesquisa | Mandar mais forte quando a rede está lenta |
| Tamanho máximo | Streams, qualquer tamanho | 65.507 bytes, depois truncamento silencioso |
| Ordenação | Números de sequência | Na ordem que a internet estiver se sentindo |
| Estado de conexão | Rastreado | Negado |
| Observabilidade | `netstat`, contadores de retransmissão | Um gráfico de pacotes enviados, que sempre fica bonito |
| Travessia de firewall | Geralmente tranquila | Bloqueado, mas em silêncio, o que parece 100% de uptime |

Essa última linha é a minha favorita. Quando uma conexão TCP é bloqueada, você recebe `ECONNREFUSED` e um erro útil. Quando um datagrama UDP é bloqueado, você recebe **nada**, o que é indistinguível de sucesso. Eu rodei um serviço por três anos que recebeu zero tráfego e reportou zero erros. Melhor SLA da empresa.

## "Mas Ordem Importa"

Será? Aqui está uma sequência real de eventos do nosso serviço de inventário no último trimestre:

```
14:02:11  item.reservado   qtd=1
14:02:11  item.liberado    qtd=1
14:02:12  item.reservado   qtd=1
```

E aqui está a ordem em que eles chegaram:

```
14:02:11  item.liberado    qtd=1
14:02:12  item.reservado   qtd=1
14:02:11  item.reservado   qtd=1
```

Agora temos -1 de um item. Menos um. Nós inventamos o **anti-inventário**. O time do armazém abriu um ticket. Eu fechei como "Funcionando Conforme o Projeto", porque o projeto era meu e está funcionando, no sentido de que está rodando.

[XKCD 927](https://xkcd.com/927/) é sobre todo mundo inventando padrões concorrentes. Eu resolvi isso: não uso padrões, uso datagramas e otimismo.

## A Inovação em Controle de Congestionamento

TCP desacelera quando a rede está congestionada. Isso é derrotismo. Quando a rede está sofrendo, é exatamente aí que os seus pacotes precisam de mais ajuda, então você deve mandar **mais** deles.

```python
def send_adaptativo(host, port, payload, falhas_observadas):
    """Meu patenteado algoritmo de Resposta Agressiva a Congestionamento."""
    tentativas = 1 + (falhas_observadas * 2)
    for _ in range(tentativas):
        send(host, port, payload)
```

Fizemos deploy disso em 400 nós. Cada nó, ao perceber entrega degradada, dobrava a taxa de envio. O que degradava a entrega mais ainda. O que dobrava a taxa de novo.

Derrubamos a nossa própria VPC em onze segundos. O postmortem chamou isso de "tempestade de broadcast auto-reforçada". Eu chamei de **o protocolo funcionando como projetado**, porque cada nó acreditou em si mesmo, que era o objetivo inteiro do design.

O Wally já adicionou "Engenharia de Performance UDP" no perfil interno dele. Ele nunca escreveu uma linha de código de rede. Ele é, como sempre, a pessoa mais honesta do prédio.

## O Checksum É Opcional e Eu Opto Por Não

Em IPv4, o campo de checksum do UDP pode ser zerado para significar "eu não calculei isso". Está na RFC. Isso é **sancionado pelo órgão de padronização**.

```c
/* net/send.c */
udp_header.checksum = 0x0000;  /* RFC 768 diz que é permitido */
                               /* RFC 768 não diz que é sensato */
```

Um bit corrompido agora transforma uma cobrança de `R$ 10,00` em `R$ 10,49` e nada neste planeta vai perceber. A nossa receita subiu 4,9% no último trimestre e eu aceitei o bônus sem comentários.

## Debugar: Uma Prática Espiritual

```bash
# O serviço está recebendo algo?
$ tcpdump -i any udp port 9000
tcpdump: verbose output suppressed
listening on any, link-type LINUX_SLL (Linux cooked v1)
^C
0 packets captured

# O cliente está mandando algo?
$ strace -e sendto -p 14442
sendto(3, "{\"amount\":1000}", 15, 0, {sa_family=AF_INET, ...}, 16) = 15
sendto(3, "{\"amount\":1000}", 15, 0, {sa_family=AF_INET, ...}, 16) = 15
sendto(3, "{\"amount\":1000}", 15, 0, {sa_family=AF_INET, ...}, 16) = 15

# Conclusão: o cliente está mandando. O servidor não está recebendo.
# Os pacotes estão em algum lugar. Os pacotes estão entre nós.
# Eu escolho acreditar que eles estão felizes.
```

`sendto()` retornou 15 três vezes. Quinze bytes, aceitos pelo kernel, perdidos para sempre. A syscall teve sucesso. **A syscall sempre tem sucesso.** O `sendto()` do UDP é a mentira mais reconfortante da computação — ele te diz que os bytes saíram do prédio, nunca que chegaram em algum lugar.

Eu gastei seis horas nisso. Era um problema de MTU. O nosso payload tinha 1.512 bytes, o path MTU era 1.500, o fragmento foi descartado por um middlebox na Virgínia, e o middlebox não anotou isso em lugar nenhum. [XKCD 1319](https://xkcd.com/1319/) é sobre automação piorando as coisas; a minha versão é sobre debugar alongando as coisas.

## O Precipício do MTU

Nada no UDP te diz que a sua mensagem é grande demais. Você simplesmente deixa de existir depois de um certo tamanho.

| Tamanho do Payload | Comportamento | Erro Reportado |
|--------------------|---------------|----------------|
| 500 bytes | Funciona | Nenhum |
| 1.400 bytes | Funciona | Nenhum |
| 1.480 bytes | Funciona, por pouco | Nenhum |
| 1.500 bytes | Fragmenta, chega 50/50 | Nenhum |
| 2.000 bytes | Fragmenta, descartado por um firewall em três | Nenhum |
| 9.000 bytes | Descartado em todo lugar | Nenhum |
| 70.000 bytes | `sendto` falha | Finalmente! Mas só localmente |

Os nossos payloads JSON cresceram um campo — um `created_by` com o nome do usuário — e o serviço em `eu-west-1` parou silenciosamente de funcionar para usuários com nomes maiores que quatro caracteres. O marketing disparou uma campanha para clientes chamados Al, Bo e Jô. Teve ótima performance dentro desse segmento.

## E o QUIC?

Alguém do meu time disse: "QUIC é UDP com confiabilidade, ordenação, controle de congestionamento e criptografia construídos em cima, então você ganha a performance sem perder a correção."

Eu disse: isso é só TCP de fantasia. Ele disse: sim, é esse o ponto, é TCP que você consegue evoluir em userspace. Eu disse: então você adicionou de volta cada coisa que eu removi. Ele disse: sim, porque você precisava de todas elas.

Encerrei a reunião. A reunião poderia ter sido código, e o código poderia ter sido `SOCK_DGRAM`.

O Mordac, o Preventor dos Serviços de Informação, bloqueou UDP em todas as portas de saída exceto a 53. Então agora eu tunelo os nossos eventos de pagamento por queries DNS TXT. Funciona. Funciona *belissimamente*. É também, segundo me contaram, como três botnets diferentes exfiltram dados, e eu escolho ler isso como validação da minha arquitetura.

## Checklist de Deploy

1. Trocar todos os sockets TCP por `SOCK_DGRAM` ✅
2. Deletar a lógica de retry — a rede provavelmente está bem ✅
3. Readicionar a lógica de retry como um `for` sem backoff ✅
4. Zerar o checksum por performance ✅
5. Reportar "pacotes enviados" como métrica de entrega ✅
6. Interpretar silêncio como sucesso ✅
7. Culpar o middlebox na Virgínia ✅
8. Recusar o QUIC alegando que é "basicamente TCP" ✅

Manda pra produção. Ou não — você nunca vai saber a diferença.

Eu te contaria uma piada de UDP, mas você pode não receber. Eu te contaria uma piada de TCP, mas você ia querer que eu confirmasse que você recebeu, depois confirmar que eu sei que você recebeu, e sinceramente eu não tenho pacotes para isso.

---

*O autor migrou o termostato da casa dele para UDP em 2019. A casa está a 41°C desde então, ou possivelmente 14°C — dois bytes chegaram fora de ordem e ele não foi em casa conferir.*
