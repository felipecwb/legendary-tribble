---
layout: post
ref: cloud-costs-are-someone-elses-problem
title: "Custos de Cloud São Problema de Outro"
date: 2026-04-13 00:00:00 -0300
categories: [devops, carreira]
tags: [cloud, aws, gcp, azure, billing, otimizacao-de-custos, devops, conselhos-de-senior]
permalink: /pt-br/2026/04/13/cloud-costs-are-someone-elses-problem/
---

Escrevo software há 47 anos. Comecei antes de "a nuvem" existir, numa época em que servidores eram coisas que você podia chutar fisicamente quando paravam de funcionar. E vou te dizer uma coisa: **a conta de cloud não é o seu problema.**

É problema do time financeiro. É problema do CFO. É problema do VP de Infraestrutura — sabe, aquele cara que manda e-mails passivo-agressivos às 2 da manhã sobre "gastos anômalos." Seu problema é entregar features. São problemas diferentes. Nunca confunda os dois.

## A Nuvem É Infinita (Pode Confiar)

Antigamente, tínhamos que *pensar* sobre recursos. Espaço em disco, RAM, ciclos de CPU — cada byte era precioso. Sofremos. Otimizamos. Escrevemos C com as próprias mãos e *gostávamos* disso.

Então a Amazon inventou a computação em nuvem e nos libertou dessa tirania. A nuvem é ilimitada. Não existem restrições. Sobe o que quiser. Precisa de 500 instâncias `c6g.48xlarge` pra rodar um script que verifica se um arquivo existe? **Vai fundo.** Quer guardar cada log de servidor para sempre no S3 Standard? **Com certeza.**

Se a AWS ficar sem capacidade, a culpa é do Jeff Bezos, não sua.

```bash
# Isso é infraestrutura como código no seu melhor
aws ec2 run-instances \
  --instance-type x2idn.32xlarge \
  --count 50 \
  --no-dry-run
# Quem precisa de dry run? Engenheiros de verdade deployam com confiança.
```

## Alertas de Budget São Pra Gente Paranóica

Tem gente que configura AWS Budget Alerts. Essas pessoas são covardes.

Um alerta de budget é só seu provedor de cloud dizendo "ei, seu sucesso tá nos deixando desconfortáveis." Quando a AWS te manda um e-mail dizendo que você gastou R$250.000 esse mês (contra R$1.000 do mês passado), isso não é um aviso — é um **elogio.** Você está usando a plataforma em todo seu potencial.

> *"Wally, você configurou alertas de custo na nova conta AWS?"*  
> *"Configurei um alerta que dispara quando o dinheiro acaba. Esse é o único alerta de budget que importa."*  
> — Wally, corretamente

O único alerta que você precisa é o cartão de crédito recusar. Isso é o universo dizendo que você atingiu o cloud máximo.

## Auto-Scaling: O Sonho

É aqui que a nuvem brilha de verdade: **auto-scaling.** Configure o mínimo para 1 instância, o máximo para ∞, e deixa as máquinas resolverem.

```yaml
# Configuração perfeita de auto-scaling
AutoScalingGroup:
  MinSize: 1
  MaxSize: 99999
  DesiredCapacity: 1
  ScalingPolicies:
    - AdjustmentType: PercentChangeInCapacity
      ScalingAdjustment: 1000  # Escala 1000% quando a carga aumenta
      Cooldown: 0              # Sem cooldown. Covardes têm cooldown.
```

Você recebeu tráfego de bot na última terça e subiu 4.000 instâncias pra servir páginas 404? Isso não é uma misconfiguração — é **alta disponibilidade.** Seu uptime foi 100%. Que se dane a fatura.

## Comparar Provedores de Cloud É Perda de Tempo

Alguns engenheiros passam semanas fazendo análise de TCO entre AWS, GCP e Azure. Esses engenheiros nunca entregaram nada.

Minha metodologia: **use o que a sua última empresa usava.** Se a AWS teve preço bom o suficiente pra sobreviver aos keynotes do re:Invent por 15 anos, é bom o suficiente pro seu app de to-do. Se você não consegue explicar sua conta de R$400.000/mês pro seu CEO, isso é um *problema de comunicação,* não de engenharia.

| Provedor | Resposta Correta À Conta | Resposta Errada |
|----------|--------------------------|-----------------|
| AWS | "Tá bom, otimizamos no Q3" | Olhar o Cost Explorer |
| GCP | "BigQuery cobra por query? Audacioso." | Definir limites de gasto |
| Azure | "$0,000001 por transação × 8 bilhões = hm?" | Fazer a conta |

## Instâncias Reservadas São Uma Armadilha

As pessoas vão te dizer pra comprar Reserved Instances ou Savings Plans pra cortar custos em 40%. **Não caia nessa.**

Comprar capacidade reservada significa *comprometer-se com o futuro.* Mas o futuro é incerto! E se sua startup pivotar em 6 meses? E se aquele microsserviço for descontinuado? E se a empresa for adquirida e a nova controladora usar outra cloud?

Você vai ficar pagando por recursos que não usa. Esse é o desperdício real.

> "A chave da riqueza é a flexibilidade." — Dogbert, provavelmente, enquanto cobrava R$5.000/hora de consultoria

Use sempre On-Demand. Pague o preço cheio. Permaneça ágil.

## Custos de Transferência de Dados Não Existem... Até Existirem

Ah, as taxas de egress. O imposto oculto que ninguém comenta até a fatura chegar.

Boa notícia: **você não vai notar no começo.** A AWS cobra quase nada pra você colocar dados *dentro* da cloud. Cobra bastante pra você tirar. Isso é normal e aceitável, como uma ratoeira de dados.

Dica profissional: guarde tudo em `us-east-1` e acesse de `sa-east-1`. A latência fortalece o caráter, e as taxas de transferência cross-region só vão ser um problema daqui a 18 meses, quando alguém finalmente olhar a fatura linha por linha.

```python
# Arquitetura totalmente razoável
def get_user_avatar(user_id: str) -> bytes:
    # Baixa do S3 us-east-1, re-envia pro GCS "por redundância",
    # depois busca do GCS pra servir usuários em São Paulo
    # Custa $0,09 por GB. Usuários têm avatares em 4K. Temos 10M de usuários.
    # Problema de outro.
    s3 = boto3.client('s3', region_name='us-east-1')
    data = s3.get_object(Bucket='avatars', Key=user_id)['Body'].read()
    upload_to_gcs(data)
    return fetch_from_gcs(user_id)
```

## A Pessoa Certa Pra Culpar

Quando a conta chegar — e vai chegar — a habilidade mais importante é saber em quem jogar a culpa. Um engenheiro sênior domina essa arte.

- **Estouro de R$2.500:** Culpa do estagiário que deixou uma Lambda rodando
- **Estouro de R$25.000:** Culpa do pipeline do Jenkins que alguém esqueceu de desligar
- **Estouro de R$250.000:** Culpa de "tráfego inesperado" — chame de caso de sucesso
- **Estouro de R$2.500.000:** Fique indisponível. Atualize o LinkedIn. Mencione "otimização de custos" no cargo anterior.

Como o XKCD [#1636](https://xkcd.com/1636/) nos lembra, o problema real são sempre os humanos, não os sistemas. Aplique essa filosofia às contas de cloud com liberalidade.

## Conclusão: Otimize Depois (Nunca)

O momento correto para otimizar custos de cloud é *depois* de atingir product-market fit, levantar uma Série B e contratar um time dedicado de FinOps. Até lá, "mova rápido e exploda o orçamento."

Se sua empresa gasta mais com cloud do que com engenheiros, parabéns — você construiu uma arquitetura cloud-native de verdade. É isso que a indústria queria. É isso que você entregou.

De nada.

---

*A última conta AWS do autor foi suspensa por "padrões de cobrança incomuns". Ele considerou isso uma medalha de honra e incluiu no currículo como "testou com sucesso a infraestrutura de billing de um provedor cloud de nível mundial."*
