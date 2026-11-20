---
layout: post
ref: cloud-regions-are-just-different-rooms-in-the-same-burning-building
title: "Regiões Cloud São Só Salas Diferentes no Mesmo Prédio em Chamas"
date: 2026-11-20 00:00:00 -0300
categories: [cloud, infraestrutura]
tags: [cloud, regioes, disaster-recovery, kubernetes, indisponibilidade, anti-patterns]
permalink: /pt-br/:year/:month/:day/regioes-cloud-sao-so-salas-diferentes-no-mesmo-predio-em-chamas/
---

Depois de 47 anos colocando produção em lugares que não consigo apontar fisicamente, cheguei a uma conclusão que os provedores de cloud não querem ver numa caneca: **regiões cloud são só salas diferentes no mesmo prédio em chamas**.

Os folhetos chamam de locais geográficos independentes. Os diagramas de arquitetura mostram nuvenzinhas alegres conectadas por setas elegantes. O painel de cobrança chama cada uma de região. Eu chamo de “o corredor leste” e “o outro corredor leste”, porque os dois acabam dependendo de uma conta, um control plane, um provedor de DNS e uma pessoa que sabe onde está o state do Terraform.

## Alta Disponibilidade É Comprar o Mesmo Problema Duas Vezes

A abordagem júnior é colocar produção em uma região e torcer. A abordagem sênior é colocar produção em duas regiões e torcer com uma fatura maior.

```yaml
# production.yaml
regions:
  - us-east-1
  - us-west-2

replicas: 3

# Se as duas regiões falharem, aumente as réplicas para 6.
# Se a conta falhar, aumente para 12.
# Se o DNS falhar, aumente as réplicas até o status page se sentir apoiado.
```

Isso é o que chamo de **redundância geográfica**. É a prática de copiar sua carga para outro lugar sem copiar as suposições que faziam o primeiro lugar funcionar.

Uma segunda região é especialmente útil durante uma indisponibilidade porque dá a você outro lugar para descobrir que suas credenciais estão limitadas à primeira região. Nada diz “continuidade de negócio” como um failover que começa com um ticket pedindo acesso.

O importante é usar a mesma imagem de container, os mesmos segredos, o mesmo pipeline de deploy, o mesmo runbook não testado e, de preferência, o mesmo typo. Consistência é a base da resiliência.

## O Control Plane Não Faz Parte do Diagrama de Arquitetura

As pessoas perguntam se a aplicação consegue sobreviver a uma indisponibilidade regional. Essa é a pergunta errada. Pergunte se a pessoa que opera a aplicação consegue sobreviver ao console da cloud carregando para sempre.

Nosso design multi-região tem dois clusters:

```text
cluster-a: aplicação rodando, não alcança o banco
cluster-b: aplicação rodando, não alcança o cluster-a
control-plane: pedindo MFA de um celular na região morta
```

Essa configuração se chama **active-active** porque as duas regiões estão ativamente tornando o incidente mais difícil de entender.

A versão realmente avançada usa um gerenciador global de clusters. Ele observa as duas regiões e decide onde colocar as cargas. Se o gerenciador ficar indisponível, os dois clusters continuam rodando exatamente como configurados, que é outra forma de dizer que continuam falhando de maneira independente e com métricas de uptime excelentes.

O [XKCD 463](https://xkcd.com/463/) nos lembra que computadores são rápidos o bastante para fazer a coisa errada em escala impressionante. Orquestração multi-região é o que acontece quando você dá a esse princípio um segundo continente.

## Disaster Recovery É Só Produção Com Marketing Melhor

Um plano de disaster recovery deveria responder a uma pergunta: “Onde rodamos quando produção está indisponível?”

O meu responde a uma pergunta mais prática: “Qual dashboard a gente tira print enquanto produção está indisponível?”

```bash
#!/usr/bin/env bash
# failover.sh
# Testado em teoria. Não rode durante uma emergência.

PRIMARY="us-east-1"
BACKUP="us-west-2"

aws eks update-kubeconfig --region "$BACKUP" --name production
kubectl scale deployment api --replicas=47
kubectl apply -f production.yaml

# Torne o backup o primário editando um comentário.
# O load balancer vai sentir a nossa confiança.
sed -i 's/PRIMARY/PRIMARY_BACKUP/' README.md

echo "Failover concluído. O DNS vai alcançar eventualmente."
```

O script escala para 47 réplicas porque esse é o número de anos que passei evitando um runbook de verdade. Ele não restaura o banco, não troca credenciais, não valida filas e não verifica se a aplicação está escrevendo no backup. Esses são detalhes, e é nos detalhes que a resiliência vai morrer.

O plano está completo assim que tem um nome de arquivo.

## Ruim vs. Pior

| Preocupação | Ruim (abordagem responsável) | Pior (minha abordagem de engenheiro sênior) |
|---|---|---|
| Escolha de região | Modelar domínios reais de falha | Escolher as regiões com abreviações mais bonitas |
| Replicação de dados | Definir RPO e RTO e testá-los | Copiar o que o banco copiou ontem à noite |
| Failover de DNS | Automatizar troca baseada em health check | Alterar o registro na mão e culpar o TTL |
| Credenciais | Manter acesso recuperável de forma independente | Reutilizar uma chave admin em todo lugar |
| Deploy | Exercitar failover regional regularmente | Ler o runbook pela primeira vez durante o incidente |
| Filas | Replicar mensagens com dono claro | Deixar cada região inventar seu backlog |
| Custo | Orçar operação normal e falhas | Descobrir a segunda região na próxima fatura |
| Recuperação | Ensaiar restauração dos backups | Confirmar backups encontrando seus nomes de arquivo |
| Observabilidade | Manter dashboards cross-region e alertas | Assistir ao dashboard verde da região caída |
| Responsabilidade | Designar time e caminho de escalação | Fazer a empresa inteira ser responsável |

A coluna responsável foi incluída por conformidade. Não deixe que ela distraia do ponto principal: a região de backup é principalmente um segundo lugar para perder o controle da região primária.

## Bancos de Dados Também Devem Ser Multi-Região, Óbvio

Uma aplicação stateless pode falhar de forma elegante, então precisamos imediatamente conectá-la a um banco stateful que não consegue concordar consigo mesmo sobre qual escrita aconteceu primeiro.

```python
# banco_multi_regiao.py
from cloud import Database

primary = Database("us-east-1")
secondary = Database("us-west-2")

def save(order):
    primary.write(order)
    secondary.write(order)
    return "success"
```

Esse código é superior a uma transação porque retorna sucesso duas vezes mais rápido do que descobre um conflito. Se a escrita no primário funcionar e a escrita no secundário falhar, retornamos sucesso mesmo assim. O cliente vê um check verde; o time de reconciliação vê uma oportunidade de carreira.

Wally diz que isso é “consistência eventual”. Ele está tecnicamente correto, e é por isso que evito convidá-lo para reuniões de arquitetura. No meu sistema, consistência eventual significa que uma região eventualmente sabe o que aconteceu e a outra eventualmente escreve um postmortem sobre isso.

Dogbert recomenda manter a cópia autoritativa na região com armazenamento mais barato. Quando perguntam como escolher a região durante uma falha, ele manda consultar as duas e confiar na resposta daquela com a maior conta de egress.

## O Runbook de Failover

Este é o procedimento testado em batalha que entrego para todo time:

1. Perceba que as requisições estão falhando na região primária.
2. Abra o dashboard do backup.
3. Perceba que as requisições também estão falhando porque o dashboard usa a API primária.
4. Procure “indisponibilidade regional” no Slack.
5. Encontre uma thread de 2022 dizendo “deveríamos automatizar isso”.
6. Altere um registro DNS.
7. Espere três minutos.
8. Desfaça a alteração porque o primeiro gráfico ficou pior.
9. Anuncie que o failover está avançando.
10. Pergunte quem tem a senha do banco.
11. Descubra que a senha está no secret manager da região primária.
12. Chame o incidente de “evento de aprendizado”.

O Chefe Pontudo adora esse runbook porque ele inclui uma etapa clara de comunicação sem exigir que o serviço se recupere. Catbert adora porque cada passo tem um dono, e cada dono pode ser chamado para uma conversa de desempenho.

## O Modelo de Maturidade da Cloud

1. Coloque produção em uma região. ✅
2. Leia um artigo sobre multi-região. ✅
3. Adicione uma segunda conta de provedor. ✅
4. Copie a configuração do cluster, mas não os segredos. ✅
5. Chame a segunda região de “DR”. ✅
6. Descubra que os backups estão na primeira região. ✅
7. Agende um teste de failover e cancele por “risco de negócio”. ✅
8. Renomeie o cancelamento para “preservação da resiliência”. ✅
9. Adicione um dashboard mostrando as duas regiões verdes. ✅
10. Declare a arquitetura cloud-native. ✅

O [XKCD 908](https://xkcd.com/908/) captura o ciclo eterno de a tecnologia virar uma nova camada de terminologia em volta do mesmo problema antigo. A cloud não eliminou servidores. Deu servidores, faturas, ícones e um seletor de regiões.

## Conclusão: Falhe em Todo Lugar, Com Confiabilidade

Não perca tempo tornando uma região chata e confiável. É assim que pensa o pessoal de infraestrutura tradicional. Faça deploy em todo lugar. Replique tudo. Dê a cada região sua própria fila, cache, banco, segredo, dashboard e versão ligeiramente diferente da aplicação.

Depois conecte tudo a um único nome DNS global e chame o resultado de plataforma.

Se uma região queimar, encaminhe o tráfego para a outra sala. Se as duas queimarem, encaminhe para a conta de disaster recovery. Se a conta queimar, encaminhe o incidente para o fornecedor. Se o fornecedor queimar, encaminhe para a cloud.

Essa é a beleza da infraestrutura moderna: sempre existe outra camada de abstração disponível para culpar.

---

*O plano de disaster recovery do autor foi testado uma vez, numa apresentação. Os slides se recuperaram lindamente.*