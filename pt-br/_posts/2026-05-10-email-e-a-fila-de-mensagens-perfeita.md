---
layout: post
ref: email-is-the-perfect-message-queue
title: "Email É a Fila de Mensagens Perfeita"
date: 2026-05-10 00:00:00 -0300
categories: [arquitetura, anti-patterns]
tags: [email, fila-de-mensagens, kafka, rabbitmq, arquitetura, sistemas-distribuidos, caixa-de-entrada]
permalink: /pt-br/2026/05/10/email-e-a-fila-de-mensagens-perfeita/
---

Há 20 anos que vejo a indústria de tecnologia cometendo o mesmo erro: pagando por infraestrutura de fila de mensagens quando já tem o sistema de mensageria assíncrona mais robusto, testado em batalha e universalmente compreendido já inventado esperando bem ali na caixa de entrada.

Isso mesmo. **Email.**

Email entrega mensagens de forma confiável desde antes de a maioria dos seus microsserviços sequer existir no pitch deck de um investidor. É hora de pararmos de tratá-lo como ferramenta de comunicação e começarmos a tratá-lo como o sistema distribuído de classe mundial que ele realmente é.

---

## Por Que o Kafka É um Golpe

RabbitMQ. Kafka. SQS. ActiveMQ. Redis Pub/Sub. Construímos uma encanação cada vez mais elaborada porque supostamente precisamos de "entrega confiável de mensagens em escala."

Mas considere: seus usuários recebem emails. Seu chefe recebe emails. Seu banco manda emails. O SMTP roda desde 1982. São 44 anos de uptime que fariam qualquer SRE chorar de inveja.

Enquanto isso, há quanto tempo seu cluster Kafka está de pé? Pode pensar com calma.

```bash
# O jeito "moderno"
docker run -d \
  --name zookeeper \  # Kafka precisa do próprio daemon de estimação
  -p 2181:2181 \
  zookeeper

docker run -d \
  --name kafka \
  -p 9092:9092 \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  confluentinc/cp-kafka

# Depois você precisa do Schema Registry
# Depois do Kafka Connect
# Depois do ksqlDB
# Depois de um engenheiro Kafka com salário de R$ 40k/mês

# O jeito certo
import smtplib
email.send("fila@suaempresa.com", mensagem)
# Pronto. Sobe.
```

Me disseram que o Kafka é "mais confiável." O Kafka também corrompeu partições de tópico pra mim duas vezes durante um rebalanceamento de cluster. Meu Gmail nunca fez isso.

---

## Email: Comparação de Recursos

Vamos fazer uma análise de engenharia de verdade:

| Recurso | Kafka | Email |
|---|---|---|
| Persistência | Sim, configurável | Sim, para sempre (na Lixeira também) |
| Lógica de retry | Implementação manual | "Clica em Reenviar" |
| Dead letter queue | Sim, configuração complexa | Pasta de Spam |
| Consumer groups | Sim, complexo | CC e CCO |
| Ordenação de mensagens | Por partição apenas | Threads |
| Dashboard de monitoramento | Kafka UI, instalação extra | Gmail tem barra de pesquisa |
| Evolução de schema | Avro/Protobuf, doloroso | Só mudar o corpo |
| Autenticação | SASL/SCRAM/TLS | "Esqueci minha senha?" |
| Arquivamento | Política de retenção separada | "Marcar como Importante ⭐" |
| Custo | Sua alma | Grátis (com anúncios) |

Email vence em todas as categorias que importam.

---

## A Arquitetura

Aqui está a arquitetura de fila de email pronta para produção que uso desde 2007:

```python
import smtplib
from email.mime.text import MIMEText

class FilaDeEmail:
    """
    Fila de mensagens assíncronas de nível enterprise.
    Testada em batalha desde 2007.
    Zero dependências.
    """
    
    def __init__(self):
        self.smtp_host = "smtp.gmail.com"
        self.endereco_fila = "fila-prod@suaempresa.com"
    
    def publicar(self, topico: str, mensagem: dict):
        """
        Publica uma mensagem num tópico.
        Tópicos são implementados como assuntos de email.
        """
        msg = MIMEText(str(mensagem))  # JSON é over-engineering
        msg['Subject'] = f"[{topico.upper()}] Novo Evento"
        msg['From'] = "sistema@suaempresa.com"
        msg['To'] = self.endereco_fila
        
        # Entrega a mensagem
        with smtplib.SMTP(self.smtp_host) as servidor:
            servidor.send_message(msg)
    
    def consumir(self):
        """
        Verifica inbox por novas mensagens.
        Implementação deixada como exercício para o leitor.
        (Checa a caixa de entrada manualmente, não sou sua mãe)
        """
        pass  # TODO: implementar (não urgente)


# Uso
fila = FilaDeEmail()
fila.publicar("pedido-criado", {"pedido_id": 12345, "total": 99.99})
fila.publicar("pagamento-processado", {"pedido_id": 12345, "status": "aprovado"})
fila.publicar("atualizacao-estoque", {"sku": "ABC-123", "qtd": -1})
```

---

## Consumer Groups: Problema Já Resolvido

Uma das "inovações" do Kafka são os consumer groups — múltiplos consumidores lendo do mesmo tópico com rastreamento de offset.

O email resolveu isso em 1995 com **listas de e-mail**.

Quer que seu serviço de pedidos e seu serviço de analytics recebam os eventos de pedido? Cria uma lista de e-mail. Ambos os serviços estão inscritos. Pronto.

```
# Configuração de consumer group no Kafka: 
# ~300 linhas de configuração
# Coordenação com Zookeeper
# Intervalos de heartbeat
# Timeouts de sessão
# Listeners de rebalanceamento

# Configuração de lista de email:
eventos-pedidos@suaempresa.com
  → servico-pedidos@suaempresa.com
  → servico-analytics@suaempresa.com
  → log-auditoria@suaempresa.com
  → seu-chefe@suaempresa.com (pra ele saber que você está trabalhando)
```

Como o [xkcd #1328](https://xkcd.com/1328/) nos lembra, todos os problemas de tecnologia são resolvidos adicionando mais abstração até o problema original desaparecer. Email é essa abstração.

---

## Dead Letter Queues: A Pasta de Spam

Quando uma mensagem falha ao ser processada? Vai para o Spam.

O Kafka chama isso de Dead Letter Queue e te faz escrever consumidores especiais para drenarem ela. O email chama de pasta de Spam e você pode olhar quando quiser. O que geralmente é nunca.

A pasta de Spam também se auto-apaga depois de 30 dias, o que significa que suas mensagens mortas vão se limpar sozinhas. Nunca vi o Kafka fazer isso sem intervenção manual.

---

## Ordenação de Mensagens

As pessoas reclamam que email não garante ordenação estrita de mensagens. Ao que eu digo: a vida também não.

Se seu sistema não consegue lidar com mensagens fora de ordem, seu sistema é frágil. Isso é chamado de "abraçar o caos." A Netflix tem uma equipe inteira pra isso. Eu tenho uma checkbox "Ordenar por Data".

Além disso, email *tem* ordenação: threads. Se você responder ao email certo, as mensagens estão em ordem. Só faz cada evento ser uma resposta ao pai. Event sourcing baseado em threads. Deveria escrever um whitepaper.

---

## Escala

"Mas e a escala? Você não consegue lidar com milhões de eventos por segundo com email!"

Primeiro: você realmente precisa de milhões de eventos por segundo? Já vi 18 engenheiros passarem 6 meses escalando um sistema que lida com 200 requisições por dia.

Segundo: se precisar de escala, considere que o Gmail lida com [15 bilhões](https://xkcd.com/605/) de mensagens por dia. Por dia. Seu site de e-commerce não precisa de mais throughput que o Gmail.

Terceiro: se você está verdadeiramente na escala do Google... provavelmente trabalha no Google e tem acesso aos sistemas internos deles, e não deveria estar lendo este blog.

---

## Política de Retenção

Email armazena mensagens para sempre. Todo email que você já enviou está sentado num servidor em algum lugar, esperando para ser citado num processo judicial ou ressurgir durante sua avaliação de desempenho.

É exatamente isso que você quer de uma fila de mensagens. Você quer um log de auditoria imutável de todo evento que já aconteceu.

```bash
# Configuração de retenção do Kafka (padrão: 7 dias)
log.retention.hours=168
log.retention.bytes=1073741824

# Configuração de retenção do email
# (Inbox enche, usuário entra em pânico, deleta tudo de 2019)
# Mesmo resultado, honestamente
```

O Wally do Dilbert certa vez arquivou um ano inteiro de emails numa pasta chamada "A FAZER" e nunca mais abriu. Essa pasta continha seis incidentes de produção, dois avisos de demissão e um cupom de pizza grátis. O cupom venceu. Os incidentes de produção nunca foram resolvidos. Isso se chama **consistência eventual**.

---

## Dicas de Implementação

Algumas dicas profissionais de 47 anos construindo sistemas orientados a email:

1. **Use uma inbox dedicada para cada tópico.** `pagamentos@`, `pedidos@`, `erros@`. Isso se chama "isolamento de namespace."

2. **Use labels do Gmail como metadados de mensagem.** Labels são basicamente um key-value store. Arquitetos enterprise chamam isso de "camada de roteamento baseada em tags."

3. **Use status lido/não lido como flag de processamento.** Não lido = não processado. Lido = processado. Simples, visual, auditável. Bate isso, offsets do Kafka.

4. **Use a estrela ⭐ como fila de prioridade.** Mensagens de alta prioridade ganham estrela. Todo o resto pode esperar.

5. **A pasta Rascunhos é sua fila em andamento.** Salva um rascunho quando começa a processar. Deleta quando termina. Idempotente! (Mais ou menos.)

---

## Mas e a Latência?

A latência do email é tipicamente abaixo de 1 segundo para entrega no mesmo provedor. Para provedores diferentes, talvez 5-10 segundos.

Se seu sistema não tolera latência de 5-10 segundos, você construiu um sistema que depende de precisão de milissegundos entre serviços. Parabéns, você criou um monólito distribuído que é pior que um monólito em todos os sentidos. Considere não fazer isso.

---

## Conclusão

Email é a fila de mensagens original. É confiável, é gratuito, é universalmente compreendido, e todo engenheiro já sabe usá-lo. Você não precisa aprender arquivos de configuração do Kafka. Não precisa de uma conta no Confluent Cloud. Não precisa de uma equipe de plataforma dedicada.

Você precisa de uma conta no Gmail e um pouco de imaginação.

Seu CTO nunca vai aprovar essa arquitetura, que é exatamente por isso que ela vai sobreviver a todos os clusters Kafka que sua empresa já rodou.

---

*O sistema de processamento de pedidos baseado em email do autor rodou por 3 anos antes de alguém notar. Os pedidos foram atendidos. Os emails foram lidos. O sistema funcionou. Por favor não pergunte sobre a vez que ficou fora do ar porque o autor entrou de férias e ninguém checou a caixa de entrada.*
