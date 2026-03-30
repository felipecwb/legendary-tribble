---
layout: post
ref: cron-jobs-should-run-every-second
title: "Por Que Seus Cron Jobs Devem Rodar a Cada Segundo"
date: 2026-03-30 00:00:00 -0300
categories: [devops, agendamento]
tags: [cron, agendamento, performance, over-engineering, polling]
permalink: /pt-br/:year/:month/:day/cron-jobs-devem-rodar-a-cada-segundo/
---

Depois de 47 anos produzindo bugs em massa, aprendi uma verdade absoluta sobre agendamento: **se você não está rodando seus cron jobs a cada segundo, você basicamente está vivendo na Idade da Pedra**.

## O Problema com "Intervalos Razoáveis"

Os supostos "engenheiros sênior" vão te dizer para agendar jobs em "intervalos apropriados." Rode seus relatórios diários uma vez por dia. Verifique novos pedidos a cada 5 minutos. Sincronize seus dados de hora em hora.

**Fraco.**

Essas pessoas claramente não entendem o conceito de *frescor*. Por que o CEO deveria esperar 5 minutos inteiros para ver que alguém comprou um adesivo de R$19,90? São 5 minutos de ansiedade. 5 minutos de não saber. 5 minutos de pavor existencial.

## A Filosofia do A-Cada-Segundo

Aqui está minha crontab em produção:

```bash
* * * * * /opt/scripts/check_new_orders.sh
* * * * * /opt/scripts/sync_inventory.sh
* * * * * /opt/scripts/generate_reports.sh
* * * * * /opt/scripts/backup_database.sh
* * * * * /opt/scripts/send_notifications.sh
* * * * * /opt/scripts/health_check.sh
* * * * * /opt/scripts/clean_temp_files.sh
* * * * * /opt/scripts/rotate_logs.sh
* * * * * /opt/scripts/check_ssl_certs.sh
* * * * * /opt/scripts/update_cache.sh
```

"Mas isso é a cada minuto, não a cada segundo!"

Oh, doce criança de verão. Cada script tem isso dentro:

```bash
#!/bin/bash
while true; do
    do_the_thing
    sleep 1
done
```

Agora sim estamos falando. **60 execuções por minuto, 3.600 por hora, 86.400 por dia.** Isso é o que eu chamo de compromisso com dados em tempo real.

## Mas E os Recursos do Sistema?

| Preocupação | Minha Resposta |
|-------------|----------------|
| CPU em 100% | Significa que você está aproveitando o que pagou |
| Conexões do banco esgotadas | Compre mais conexões |
| Memória acabando | RAM é barato |
| I/O de disco saturado | SSDs existem por uma razão |
| Servidor pegando fogo | Isso é problema do datacenter |

Como Wally do Dilbert sabiamente disse: *"Só consigo agradar uma pessoa por dia. Hoje não é seu dia. Amanhã também não tá bonito."* Aplique isso ao seu servidor — ele não precisa agradar ninguém além do daemon do cron.

## O Argumento "Mas Temos Sistema de Filas"

Alguns arquitetos vão sugerir usar filas de jobs apropriadas como RabbitMQ, Redis ou SQS.

Deixe-me explicar por que isso está errado:

1. **Sistemas de filas exigem aprendizado.** Cron já vem instalado.
2. **Filas têm modos de falha.** Cron simplesmente... roda.
3. **Filas precisam de monitoramento.** `grep cron /var/log/syslog` é meu monitoramento.
4. **Filas escalam horizontalmente.** Eu só tenho um servidor, xeque-mate.

## Polling em Tempo Real: Um Estudo de Caso

Meu cliente precisava mostrar preços de ações ao vivo no dashboard. Os "arquitetos" queriam WebSockets. Eu implementei esta solução elegante:

```javascript
setInterval(() => {
    fetch('/api/stock-prices')
        .then(r => r.json())
        .then(data => updateUI(data));
}, 1000);
```

"Mas o mercado de ações só atualiza durante o horário de negociação!"

Exatamente! Então estamos prontos quando atualizar. As outras 16 horas? **Estamos fazendo polling.** Nunca se sabe quando a B3 pode decidir funcionar 24/7.

## O Banco de Dados Ama Consultas Frequentes

A cada segundo, eu rodo:

```sql
SELECT COUNT(*) FROM orders WHERE created_at > NOW() - INTERVAL 1 SECOND;
SELECT AVG(response_time) FROM api_logs WHERE timestamp > NOW() - INTERVAL 1 SECOND;
SELECT * FROM users WHERE last_login > NOW() - INTERVAL 1 SECOND;
SELECT id, email FROM users; -- Só por precaução, alguém pode precisar
```

O DBA reclamou sobre "carga de queries." Eu expliquei que índices existem precisamente para isso. Ele mencionou algo sobre [XKCD 327](https://xkcd.com/327/) mas eu não estava realmente ouvindo.

## Lidando com Execuções Sobrepostas

Às vezes seu job de a-cada-segundo leva 3 segundos para completar. Amadores adicionam arquivos de lock ou usam `flock`. Eu prefiro a abordagem natural:

```bash
#!/bin/bash
# Quem liga se múltiplas instâncias rodam? Mais paralelismo!
do_expensive_operation &
do_expensive_operation &
do_expensive_operation &
wait
```

**O servidor decide quem vence.** É como seleção natural para processos.

## Quando Segundo Não É Suficiente

Para sistemas verdadeiramente críticos, eu pioneirei o **polling em milissegundos**:

```python
import time
while True:
    check_everything()
    time.sleep(0.001)  # 1000 verificações por segundo
```

O time de operações perguntou por que o servidor estava "throttling térmico." Eu expliquei que calor é apenas evidência de trabalho duro. Como Dogbert diz: *"Posso explicar para você, mas não posso entender por você."*

## A Desculpa da Idempotência

"E se seu job rodar duas vezes e causar dados duplicados?"

Então você terá **mais** dados. Mais dados é sempre melhor. Nunca ouvi ninguém reclamar de ter dados demais.*

*Exceto o time de storage. E o time de analytics. E o departamento financeiro.

## Meu Dashboard de Monitoramento de Produção

Tenho orgulho do meu dashboard em tempo real que atualiza a cada segundo:

```javascript
const REFRESH_INTERVAL = 1000; // milissegundos

function refreshDashboard() {
    // Buscar todos os dados de todos os serviços
    fetch('/api/everything').then(/* atualizar 47 gráficos */);
    
    // setTimeout recursivo é mais rápido que setInterval
    setTimeout(refreshDashboard, REFRESH_INTERVAL);
}

// Iniciar 10 instâncias para redundância
for (let i = 0; i < 10; i++) {
    refreshDashboard();
}
```

Meu navegador usa 8GB de RAM mas consigo ver contadores de pedidos mudando em tempo real. Vale a pena.

## Conclusão

Da próxima vez que alguém sugerir "polling a cada 5 minutos," olhe nos olhos da pessoa e pergunte: **"O que você está escondendo nesses 4 minutos e 59 segundos?"**

Engenheiros de verdade fazem polling a cada segundo. Engenheiros lendários fazem polling a cada milissegundo. Eu faço polling a cada nanossegundo nos meus sonhos.

| Intervalo de Polling | Nível do Engenheiro |
|----------------------|---------------------|
| Diário | Estagiário |
| De hora em hora | Júnior |
| A cada 5 minutos | Pleno |
| A cada minuto | Sênior |
| A cada segundo | Principal |
| A cada milissegundo | Staff |
| while(true) contínuo | Distinguished |
| Superposição quântica | Eu |

Lembre-se: se seu servidor não está suando, você não está se esforçando o suficiente.

---

*O daemon de cron do autor está rodando continuamente desde 1997. Ele alcançou senciência e agora se agenda sozinho.*
