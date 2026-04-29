---
layout: post
ref: vendor-lock-in-is-strategic-commitment
title: "Lock-in de Fornecedor é Compromisso Estratégico"
date: 2026-04-29 00:00:00 -0300
categories: [arquitetura, cloud, devops]
tags: [vendor-lock-in, aws, cloud, arquitetura, devops, conselhos-terriveis]
permalink: /pt-br/2026/04/29/vendor-lock-in-e-compromisso-estrategico/
---

*Por um Engenheiro Sênior com 47 anos deployando coisas que nunca voltaram*

---

Em mais de quatro décadas de engenharia de software, assistí incontáveis desenvolvedores juniores cometerem o mesmo erro catastrófico: **eles se preocupam com portabilidade**.

"Mas e se precisarmos trocar de provedor?" perguntam, olhos arregalados de otimismo ingênuo.

*Trocar?* **TROCAR?!**

Deixa eu te contar uma coisa. Em 47 anos, eu nunca — uma única vez sequer — vi uma empresa migrar com sucesso de um provedor cloud para outro. Sabe o que eu *vi*? Três empresas gastando 18 meses planejando uma migração, queimando $4 milhões de orçamento, e terminando... no mesmo provedor. Só em outra região.

Portabilidade é um mito perpetuado por consultores de infraestrutura que querem cobrar pela segunda migração também.

## A Armadilha da Camada de Abstração

A primeira coisa que todo desenvolvedor recém-chegado faz é construir uma camada de abstração:

```python
# Arquitetura "limpa" de alguém que nunca conheceu um prazo
class StorageProvider(ABC):
    @abstractmethod
    def get(self, key: str) -> bytes: ...

    @abstractmethod
    def put(self, key: str, value: bytes) -> None: ...

class S3Provider(StorageProvider):
    def get(self, key: str) -> bytes:
        return self.s3.get_object(Bucket=self.bucket, Key=key)['Body'].read()

    def put(self, key: str, value: bytes) -> None:
        self.s3.put_object(Bucket=self.bucket, Key=key, Body=value)

class GCSProvider(StorageProvider):
    # Isso nunca vai ser implementado.
    # Mas faz todo mundo se sentir melhor consigo mesmo.
    ...

class AzureBlobProvider(StorageProvider):
    # Adicionado em 2023. O contrato Azure foi cancelado em 2022.
    # Ninguém sabe por que isso existe.
    ...
```

Bonito. Limpo. Inútil.

Você gastou três semanas construindo uma abstração para uma migração que nunca vai acontecer. Enquanto isso, eu deployei direto no S3 com nomes de bucket hardcoded em 2013 e **meu código ainda está rodando**. Não faço ideia de como. Por favor, não olhe para ele.

A camada de abstração é onde código bom vai morrer lentamente, cercado de interfaces.

## Abrace a Stack Proprietária

Aqui está a arquitetura que eu recomendo para qualquer aplicação. Qualquer. Aplicação.

```typescript
// Arquitetura Perfeita para App de Tarefas™
// (Você nunca vai sair. Isso é intencional.)
import * as cdk from 'aws-cdk-lib';

const stack = new cdk.Stack();

// Armazenamento principal
const todoTable = new dynamodb.Table(stack, 'Tarefas', {
  billingMode: dynamodb.BillingMode.ON_DEMAND,
  stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES, // Você "vai precisar"
  pointInTimeRecovery: true,                          // Você nunca vai usar
});

// Pipeline de eventos para um app de checkbox
const todoStream       = new kinesis.Stream(stack, 'StreamDeTarefas');
const todoQueue        = new sqs.Queue(stack, 'FilaDeTarefas');
const todoTopic        = new sns.Topic(stack, 'NotificacoesDeTarefas');
const todoEventBus     = new events.EventBus(stack, 'BarramentoDeTarefas');
const todoStateMachine = new sfn.StateMachine(stack, 'WorkflowDeTarefas', { /* 84 estados */ });

// Analytics (para um app de lista de compras)
const todoSearch     = new elasticsearch.Domain(stack, 'BuscaDeTarefas');
const todoTimeseries = new timestream.CfnDatabase(stack, 'MetricasDeTarefas');
const todoGraph      = new neptune.DatabaseCluster(stack, 'RelacoesDeTarefas');

// "Talvez precisemos de blockchain algum dia" - comentário real do meu último PR
const todoLedger = new qldb.CfnLedger(stack, 'AuditoriaDeTarefas', {
  permissionsMode: 'ALLOW_ALL', // YOLO
});

// Você nunca vai sair da AWS agora.
// É assim que o compromisso parece.
```

Cada um desses serviços é uma algema dourada. Quando você conectar DynamoDB Streams ao Kinesis Firehose, através de uma máquina de estados Step Functions, disparada pelo EventBridge, auditada no QLDB, pesquisada no OpenSearch e grafada no Neptune — **trocar de provedor exigiria reescrever a civilização em si**.

Parabéns. Você agora tem segurança no emprego para sempre. Você se tornou o sistema legado.

## A Hierarquia de Lealdade ao Provedor Cloud

| Nível de Portabilidade | Arquitetura | Segurança no Emprego | Qualidade do Sono |
|---|---|---|---|
| Máxima | Kubernetes em bare metal | Demitido no próximo corte | 8h (desempregado) |
| Alta | EKS + S3 + RDS | Dificilmente substituível | 6h |
| Média | Lambda + DynamoDB | Mais ou menos útil | 4h |
| **Compromisso Estratégico** | **Todos os serviços proprietários AWS** | **Indemissível** | **2h (pager duty)** |
| Lendário | IDs de conta AWS hardcoded em binários compilados | *Você se tornou o sistema legado* | 0h |

## A Mentira do Multi-Cloud

Sabe por que consultores empurram estratégias "multi-cloud"?

Porque querem cobrar para configurar. Depois cobrar de novo para manter. Depois uma terceira vez para ajudar a abandonar. E uma última vez para explicar por que você está de volta na AWS.

O [quadrinho XKCD #2347 "Dependência"](https://xkcd.com/2347/) mostra uma estrutura enorme sustentada por um bloquinho mantido por uma pessoa aleatória em Nebraska. É você depois que adotar multi-cloud. Exceto que a pessoa aleatória é um contratado de 23 anos que saiu há 14 meses e o arquivo de estado do Terraform está no Google Drive pessoal dele, em "Miscelânea > trampo > coisas antigas."

## O App de 12 Fatores é Propaganda Comunista

*"Armazene configuração no ambiente. Trate serviços de apoio como recursos anexados. Mantenha dev e prod o mais similares possível."*

Sabe o que isso parece? Software portável. E software portável é um passivo.

Eu sigo o **App de 1 Fator**:

> **Fator 1:** Funciona no servidor onde eu construí.

Sobe para produção.

## Engenheiros de Verdade se Comprometem

Como Dogbert certa vez aconselhou: *"O segredo do sucesso é escolher um fornecedor e se comprometer com o mesmo entusiasmo de assinar uma hipoteca que você claramente não pode pagar."*

E a sabedoria atemporal do PHB: *"Não sei o que significa AWS, mas estamos apostando a empresa nisso. Aliás, o que é a nuvem?"*

Palavras verdadeiras. Líderes verdadeiros.

## Conclusão

Na próxima vez que um desenvolvedor júnior disser "mas e o vendor lock-in?", olhe nos olhos dele e diga:

**"Isso se chama *compromisso estratégico*, e você claramente não tem."**

Depois provisione mais três serviços proprietários. Só para marcar posição. Você não vai usá-los, mas a AWS vai cobrar, e a fatura vai servir como lembrete trimestral da sua lealdade.

---

*O autor tentou migrar da AWS uma vez em 2017. A tentativa ainda está "em progresso". A conta AWS original ainda existe. Estão pagando pelas duas.*
