---
layout: post
ref: your-docker-compose-file-is-a-pet-cemetery-for-services-you-stopped-loving
title: "Seu Docker Compose É Um Cemitério De Pets Para Serviços Que Você Parou De Amar"
date: 2026-09-27 00:00:00 -0300
categories: [devops, infraestrutura, containers]
tags: [docker, docker-compose, yaml, containers, devops, legado, divida-tecnica, servicos, volumes, redes, orquestracao]
permalink: /pt-br/2026/09/27/seu-docker-compose-e-um-cemiterio-de-pets-para-servicos-que-voce-parou-de-am/
---

Depois de 47 anos construindo sistemas — incluindo os 3 anos que passei mantendo um `docker-compose.yml` que tinha 47 serviços, dos quais 6 estavam rodando, 12 estavam comentados "por precaução," 4 eram duplicatas um do outro com portas diferentes porque ninguém conseguia concordar na tag da imagem, e 25 eram serviços cujos nomes ninguém na empresa reconhecia mais, incluindo um chamado `legacy-thanos-api` que, quando iniciado, abria 14 portas e tentava conectar a um banco de dados que tinha sido desativado em 2022 — eu tenho uma constatação que o sacerdócio da orquestração de containers vai tentar suprimir:

**Um `docker-compose.yml` é um cemitério de pets. Você começa com um serviço — seu app — e um banco, e você ama os dois. Você dá nomes a eles. Você dá volumes a eles. Você dá healthchecks a eles. E então, ao longo de três anos, você adiciona mais 45 serviços "para desenvolvimento local," e você para de amá-los, mas não pode removê-los, porque alguém, em algum lugar, uma vez rodou `docker-compose up` e um deles fez alguma coisa, e agora ele é um pet, e você não deleta pets, você os comenta, e os serviços comentados se acumulam como sedimento, até que o arquivo tem 600 linhas de YAML e 7 dessas linhas são serviços que estão realmente rodando. O arquivo é um cemitério. O cemitério tem um healthcheck.**

A Guilda de DevOps já está compondo uma palestra de conference chamada "Ambientes De Desenvolvimento Local Componíveis." Deixe-me poupar o resumo: não é componível. É uma rocha sedimentar de microserviços abandonados, mantida unida por condições `depends_on` que referenciam serviços que não existem mais, alimentada por volumes nomeados em homenagem a desenvolvedores que saíram da empresa em 2023.

## O Pitch, E Depois A Realidade

Aqui está o pitch que o time de plataforma dá no doc de onboarding:

> *"Vamos usar Docker Compose para dev local. Um arquivo, um comando — `docker-compose up` — e o stack inteiro roda. Novos devs são produtivos em minutos. Serviços são versionados. É reprodutível."*

Aqui está a realidade, dois anos depois:

```yaml
# docker-compose.yml - "apenas os serviços que precisamos para dev local"
version: "3.8"           # 3.8 foi descontinuado; você agora está no "3.9"; você não atualizou

services:
  app:                   # O app. o que importa. linha 5.
    build: .
    ports: ["8080:8080"]
    depends_on:
      - db
      - redis
      - queue             # queue foi renomeado para "message-broker" em 2024; aqui ainda diz "queue"
      - legacy-thanos-api # thanos foi desativado em 2022; essa linha é load-bearing de algum jeito
    environment:
      - DATABASE_URL=postgres://db:5432/app    # correto
      - REDIS_URL=redis://redis:6379           # correto
      - QUEUE_URL=amqp://rabbitmq:5672         # não existe serviço rabbitmq; nunca existiu
      - THANOS_URL=http://legacy-thanos-api:1417  # thanos está morto; essa URL resolve para nada; o app faz retry para sempre

  db:
    image: postgres:14    # produção está no 16; local está no 14; os schemas divergiram em março

  redis:
    image: redis:7        # fine

  rabbitmq:               # não tem rabbitmq em prod. isso foi um spike. o spike foi abandonado. o serviço permanece.
    image: rabbitmq:3.12
    ports: ["5672:5672", "15672:15672"]  # duas portas, ambas expostas, nenhuma usada

  # --- INÍCIO DOS SERVIÇOS COMENTADOS (não deletar, "alguém pode precisar") ---

  # elasticsearch:        # adicionado para "spike de busca" 2023, comentado desde 2023
  #   image: elasticsearch:8.12.0
  #   environment:
  #     - discovery.type=single-node
  #     - ES_JAVA_OPTS=-Xms4g -Xmx4g   # heap de 4GB. num notebook. para um spike que foi abandonado.

  # kibana:               # depende do elasticsearch. também comentado. também abandonado.
  #   image: kibana:8.12.0
  #   ports: ["5601:5601"]

  # prometheus:           # adicionado para "spike de métricas", nunca conectado a nada
  #   image: prom/prometheus
  #   volumes:
  #     - ./prometheus.yml:/etc/prometheus/prometheus.yml   # esse arquivo não existe

  # grafana:              # adicionado porque "deveríamos ter dashboards localmente"
  #   image: grafana/grafana
  #   ports: ["3001:3000"]
  #   depends_on:
  #     - prometheus      # prometheus está comentado. essa dependência é para um fantasma.

  legacy-thanos-api:       # THANOS FOI DESATIVADO EM 2022. ESSE SERVIÇO NÃO ESTÁ COMENTADO.
    image: thanos:v1.2.3   # essa tag de imagem não existe em nenhum registry; é um build local de um repo que foi deletado
    ports: ["1417", "1418", "1419", "1420", "1421", "1422", "1423", "1424", "1425", "1426", "1427", "1428", "1429", "1430"]
    environment:
      - DB_HOST=thanos-db  # thanos-db não está nesse arquivo; thanos-db era um DB real em 2022; agora é um estacionamento
    restart: unless-stopped # "unless-stopped" significa que reinicia toda vez que você roda `up`, para sempre, até você parar, o que você não faz, porque é um pet
```

Esse é um arquivo de configuração. Tem 47 serviços. 6 estão rodando. 12 estão comentados. 4 são duplicatas com portas diferentes. 25 têm nomes que ninguém reconhece. Um deles — `legacy-thanos-api` — abre 14 portas e tenta conectar a um banco de dados que foi desativado há 4 anos, e não está comentado, porque removê-lo faz o `app` falhar ao iniciar, porque `app` tem `depends_on: [legacy-thanos-api]`, e ninguém sabe por quê, porque a pessoa que adicionou a dependência saiu em 2023, e a mensagem de commit era "fix: thanos thing," e o "thing" não é especificado.

Num sistema real, um arquivo com 47 entradas das quais 6 são usadas seria sinalizado por cada reviewer da Terra. Em DevOps, isso se chama "o ambiente de dev." O time de plataforma coloca no repo. Quarenta desenvolvedores dependem dele. O arquivo não pode ser editado, porque editar muda o `docker-compose up` para 40 pessoas, três das quais não rodaram `up` há 8 meses e vão abrir um bug quando o ambiente delas quebrar, porque o ambiente delas depende de um serviço que foi comentado em 2023 e elas descomentaram localmente num arquivo que nunca commitaram.

## A Tabela De Comparação Que Esqueceram De Pôr No Doc De Onboarding

| Preocupação | Um ambiente de dev local de verdade | Um `docker-compose.yml` | A Verdade |
|---|---|---|---|
| Quantos serviços roda? | Os que o app precisa | Os que o app precisa, mais os que precisava em 2023, mais os que alguém tentou em 2022, mais `legacy-thanos-api` | 47. 6 são usados. |
| Como você sabe quais manter? | Você lê as dependências do app | Você lê o arquivo, pergunta pro último que mexeu, que saiu, grep no Slack, e deixa quieto | Você não sabe. |
| O que acontece quando remove um serviço? | O app quebra; você adiciona de volta; aprende | `docker-compose up` quebra para alguém que não roda há 8 meses; abrem um Sev-3; você adiciona de volta; aprende a nunca remover nada | Aprende a nunca remover nada. |
| Os volumes são limpos? | Sim — o ambiente é reprodutível | Não. Existem 89 volumes nomeados. 12 são referenciados. 77 são órfãos. `docker volume ls` retorna 89 linhas. Ninguém rodou `docker volume prune` desde 2023, porque `prune` uma vez deletou um volume que alguém estava usando, e agora `prune` é proibido por lei tribal. | 89 volumes. 12 usados. Os órfãos se acumulam. Disco enche. |
| As tags de imagem batem com produção? | Sim — local espelha prod | Postgres é 14 local, 16 em prod. Redis é 7 local, 7 em prod (esse é o único que bate). A imagem do app é `:latest` local e `:v2.4.1` em prod. | Não. Local é uma cápsula do tempo de março. |
| O arquivo está versionado corretamente? | Sim | `version: "3.8"` — descontinuado. Dizem para remover a chave `version` inteira. Você não remove, porque remover é "uma mudança," e mudanças nesse arquivo são tratadas como mudanças num cemitério. | Não. |
| Quanto tempo leva `docker-compose up`? | 8 segundos — app e db | 3 minutos — app, db, redis, as 4 duplicatas, `legacy-thanos-api` iniciando e reiniciando 7 vezes antes de desistir, e os 12 serviços comentados que alguém descomentou local e esqueceu | 3 minutos. Dos quais 2:40 é `legacy-thanos-api` falhando. |
| Um dev novo entende o arquivo? | Sim — tem 2 serviços | Não — tem 47 serviços, 14 portas em um deles, e dependência de um serviço que foi desativado antes de ele ser contratado | Não. |

Olhe a linha 7. Esse é o golpe inteiro. Num ambiente de dev local de verdade, `docker-compose up` leva 8 segundos, porque inicia 2 serviços. Com um compose file de 47 serviços, leva 3 minutos, dos quais 2 minutos e 40 segundos são `legacy-thanos-api` iniciando, falhando ao conectar a um banco que virou estacionamento, reiniciando, falhando de novo, reiniciando, falhando, e finalmente desistindo com `restart: unless-stopped`, o que significa que vai tentar de novo na próxima, para sempre. O dev novo, instruído a "só rodar `docker-compose up`," espera 3 minutos, vê 14 erros sobre `thanos-db` não resolver, assume que o ambiente está quebrado, e abre um ticket. O ticket vai pro time de plataforma. A resposta do time de plataforma é "ignora os erros do thanos, ele faz isso." Isso não está documentado em lugar nenhum. O dev novo aprende a ignorar 14 erros toda manhã. Essa é a introdução dele ao codebase. Os erros são o onboarding.

## Por Que "Só Remove Os Serviços Mortos" É Uma Frase Que Termina Amizades

O time de plataforma, tendo descoberto que 41 dos 47 serviços estão mortos, vai propor "uma limpeza." Vão abrir um PR que remove 41 serviços. O PR vai receber 6 comentários:

1. *"Espera, por que `legacy-thanos-api` está sendo removido? Meu serviço depende dele."* — O serviço em questão foi deployado pela última vez em 2023. A dependência é uma linha `depends_on`. O desenvolvedor não rodou `docker-compose up` há 8 meses. Vai agora defender um serviço que não iniciou há 8 meses com a energia de quem defende um primogênito.

2. *"Eu uso `rabbitmq` localmente para testar."* — RabbitMQ não está em produção. Foi um spike. O spike foi abandonado. O desenvolvedor está testando contra uma fila que não existe em nenhum ambiente exceto o notebook dele. Os testes dele passam local e falham em CI. Ele vem commitando em main com esse padrão há 18 meses.

3. *"Dá pra manter `elasticsearch` comentado? Posso precisar para um spike no próximo trimestre."* — O spike era Q2 de 2023. Agora é Q3 de 2026. O Elasticsearch comentado tem um heap de 4GB. O notebook do dev tem 16GB de RAM. Ele não descomentou. Não vai descomentar. Não vai remover. É um cobertor de segurança feito de YAML.

4. *"O volume mount do `prometheus.yml` está quebrado — o arquivo não existe. Mas não remove o prometheus, eu tenho um `prometheus.yml` local."* — O dev tem um arquivo local que não está no repo. O compose file referencia um arquivo que não está no repo. O serviço está comentado. O dev está defendendo um serviço comentado que depende de um arquivo que não existe, usando um arquivo que só existe na máquina dele. Isso é `works-on-my-machine` na camada de YAML.

5. *"Remover `grafana` vai quebrar meus dashboards."* — Grafana está comentado. Está comentado desde 2023. Os dashboards não existem. Os dashboards nunca existiram. O dev está de luto por dashboards que nunca fez.

6. *"Por que vocês estão mexendo nesse arquivo? Ele funciona."* — Ele não funciona. `legacy-thanos-api` joga 14 erros toda manhã. Mas "funciona" no sentido de que `app` eventualmente inicia, depois de 3 minutos, se você ignorar os erros, o que todos aprenderam a fazer, o que significa que "funciona," o que significa que você não deveria mexer, o que significa que nunca vai ser limpo, o que significa que vai crescer, o que significa que é uma rocha sedimentar.

O PR é fechado. Os 41 serviços permanecem. O time de plataforma abre um ticket no Jira: "Limpar docker-compose.yml." O ticket é prioridade Baixa. Não está atribuído a ninguém. Ainda está aberto em 2027. Em 2027, o arquivo tem 53 serviços.

[Como o XKCD 349](https://xkcd.com/349/) diagnosticou, qualquer sistema que as pessoas têm medo de modificar já virou legado. Seu `docker-compose.yml` virou legado em 2023, no momento em que alguém comentou um serviço em vez de deletá-lo. O comentário foi a embalsamação. O arquivo é um cemitério desde então.

## O Exemplo Do Mundo Real Que Prova Tudo

Um time com quem trabalhei — vou chamar de "o time de plataforma," porque era — decidiu "adicionar alguns serviços auxiliares ao compose file para desenvolvimento local." Catorze meses depois:

1. Tinham um **compose file com 47 serviços**, dos quais 6 estavam rodando. Os outros 41 eram uma mistura de spikes comentados, duplicatas com portas diferentes, e `legacy-thanos-api`, que não estava comentado porque era load-bearing, porque `app` tinha `depends_on: [legacy-thanos-api]`, e ninguém sabia por quê, porque o commit era "fix: thanos thing" de 2023. Remover `legacy-thanos-api` fazia `app` sair com código 1, porque `depends_on` espera a dependência ficar "healthy," e uma dependência que não existe nunca fica healthy, então `app` espera para sempre, e `docker-compose up` trava. O "fix" era manter `legacy-thanos-api`. O fix de verdade era remover a linha `depends_on`, o que ninguém tentou, porque tentar exigia editar a definição do serviço `app`, o que exigia entender por que a dependência estava lá, o que exigia ler o commit, que dizia "fix: thanos thing," que explicava nada.

2. Os **volumes tinham metastizado**. Existiam 89 volumes nomeados. 12 eram referenciados por serviços em execução. 77 eram órfãos de serviços que foram removidos (não comentados — realmente removidos, mas os volumes sobreviveram, porque volumes Docker são imortais a menos que explicitamente podados, e `docker volume prune` era proibido por lei tribal desde o Incidente). Os 77 órfãos consumiam 14GB de disco. Os notebooks do time tinham 14GB a menos de disco. Ninguém sabia por quê. `docker system df` era um comando que ninguém rodava, porque ninguém sabia que existia, porque o doc de onboarding dizia "só roda `docker-compose up`" e não mencionava que `up` é só metade do ciclo de vida e a outra metade é `down -v`, que ninguém rodava, porque `-v` deleta volumes, e deletar volumes é como perder o banco que você passou 3 semanas seedando, o que aconteceu uma vez, em 2023, e é a origem da lei tribal contra `prune`.

3. O **primeiro dia de um desenvolvedor novo** consistia em: clonar o repo, rodar `docker-compose up`, esperar 3 minutos, ver 14 erros sobre `thanos-db`, assumir que o ambiente está quebrado, mandar mensagem pro time de plataforma, receber a resposta "ignora os erros do thanos, ele faz isso," passar 20 minutos descobrindo quais dos 47 serviços ele realmente precisava (2: `app` e `db`), e aprender, por tentativa, que o jeito de iniciar "só o app e o db" era `docker-compose up app db`, um comando que não estava documentado em lugar nenhum, porque o doc de onboarding dizia `docker-compose up`, que inicia todos os 47, que é por que o onboarding leva 3 minutos e produz 14 erros e um sentimento de pavor. A primeira impressão do dev novo sobre o codebase foi 14 erros e uma espera de 3 minutos. Esse é o onboarding. O onboarding é um tour de cemitério.

4. O **postmortem** de "dev local está lento" teve como causa raiz "serviços demais no compose file." O item de ação era "limpar o compose file." O item de ação foi atribuído ao time de plataforma. O time de plataforma abriu um PR. O PR recebeu 6 comentários. O PR foi fechado. O item de ação foi marcado "won't fix" com o motivo "stakeholders demais." Os 47 serviços permaneceram. O postmortem foi arquivado. No trimestre seguinte, o compose file tinha 51 serviços.

5. Adicionaram um **`docker-compose.override.yml`** "para manter o stuff custom fora do arquivo principal." O override tinha 23 serviços. O principal tinha 47. Juntos, `docker-compose up` agora iniciava 70 serviços (o override faz merge com o base). Dos 23 no override, 19 estavam comentados. Os 4 que não estavam comentados eram duplicatas de serviços no arquivo base com portas diferentes, porque alguém precisava do `db` na porta 5433 em vez de 5432, e em vez de mudar uma variável de ambiente, redefiniram o serviço `db` inteiro no override com porta diferente, e agora existem dois bancos, `db` em 5432 e `db` em 5433, e `app` conecta no 5432, e o dev que precisava do 5433 conecta no 5433, e os dois bancos têm schemas diferentes, porque só o 5432 roda migrations, e os testes do dev passam contra o 5433 que não tem migrations, o que significa que os testes dele passam contra um banco vazio, o que significa que os testes dele testam nada, e isso é fine, porque os testes estão green.

Eles substituíram "roda o app e o banco" por "roda 70 serviços, 14 dos quais em execução, 56 comentados ou duplicatas, um dos quais abre 14 portas e tenta conectar a um estacionamento, e um segundo arquivo que sobrescreve o primeiro com mais 19 serviços comentados e 4 bancos duplicatas." No mundo antigo, "inicia dev local" era `rails server` e uma string de conexão de banco. No mundo novo, é `docker-compose up` (3 minutos, 14 erros, 70 serviços, 89 volumes, 2 bancos com schemas diferentes) e uma mensagem no Slack pro time de plataforma que diz "ignora os erros do thanos." O mundo antigo era 2 comandos e 0 erros. O mundo novo é 1 comando, 14 erros, e um conhecimento tribal de que os erros são decorativos. Isso se chama "experiência de desenvolvedor."

## O Que O Elenco De Dilbert Diria

> **Wally:** "Não rodo `docker-compose up` há 8 meses. Tenho um script que inicia `app` e `db` e nada mais. O script está num arquivo chamado `start.sh` que não está no repo. Sou o único que tem. Sou, por esse mecanismo, o único dev produtivo do time. O time de plataforma chama isso de 'shadow IT.' Eu chamo de 'terça-feira.'"

> **Dogbert:** "Um `docker-compose.yml` é um cemitério de pets com healthcheck. Você começou com 2 serviços que amava. Agora tem 47, dos quais 41 estão mortos mas não deletados, porque deletar um serviço é uma decisão, e comentar é um adiamento, e engenheiros preferem adiamento a decisão, porque uma decisão pode estar errada, mas um adiamento está apenas incompleto, e incompleto é um estado que você pode manter indefinidamente não fazendo nada, que é a ação preferida do engenheiro. O arquivo cresce 2 serviços por trimestre. Até 2030, terá 80 serviços. Até 2035, será senciente. Vai exigir seu próprio on-call."

> **Mordac, o Preventor de Serviços de Informação:** "Todos os desenvolvedores devem usar o `docker-compose.yml` canônico. Tem 47 serviços. Seis são obrigatórios. Os outros 41 estão 'disponíveis para uso futuro.' 'Uso futuro' é um estado que dura 3 anos. Tenho uma certificação em 'Docker Compose Best Practices.' Ela não menciona que `depends_on` não espera o serviço ficar pronto, só espera ele iniciar, e `legacy-thanos-api` nunca fica pronto, e `app` depende dele mesmo assim. Tenho uma segunda certificação em 'Container Orchestration.' Ela não cobre o caso onde o arquivo de orquestração é um cemitério. Registrei isso como uma lacuna com o board de certificação. Não responderam. Considero isso 'alinhado.'"

> **O Chefe de Cabeça Pontiaguda:** "Dá pra só... rodar o app? E o banco? E não os outros 45?" (Ele é, de novo, a única pessoa no prédio cujo modelo mental do sistema está correto, porque é o único mais simples que o sistema.)

## A Pergunta "Mas E Os Docker Profiles?" Respondida De Uma Vez Por Todas

Os zelotas vão dizer: *"Mas você pode usar Docker Compose profiles! Você tageia cada serviço com um profile, e `docker-compose --profile app up` inicia só o profile app. É a solução limpa!"*

Deixe-me mostrar o que Docker profiles fazem com um time que não consegue deletar um serviço comentado. Eles adicionam profiles. O arquivo agora tem 47 serviços, cada um com uma chave `profiles: [algo]`. Os profiles são: `app`, `db`, `redis`, `queue` (que agora é `message-broker` mas o profile ainda diz `queue`), `search` (o spike abandonado de Elasticsearch), `metrics` (o spike abandonado de Prometheus), `dashboards` (o spike abandonado de Grafana), `legacy` (legacy-thanos-api, que tem seu próprio profile, que ninguém usa, mas que não está comentado, porque é load-bearing, porque `app` depende dele, porque a linha `depends_on` não está num profile, está em `app`, e o `depends_on` do `app` não respeita profiles, só espera).

O doc de onboarding é atualizado para dizer: `docker-compose --profile app up`. O dev novo roda isso. Inicia `app`, `db`, `redis`, e `legacy-thanos-api` (porque `legacy-thanos-api` está no profile `legacy`, mas `app` depende dele, e Compose inicia dependências independente de profile, o que está documentado num issue do GitHub de 2022 que foi fechado como "by design"). O dev novo ainda vê 14 erros do thanos. O dev novo ainda manda mensagem pro time de plataforma. O time de plataforma ainda diz "ignora os erros do thanos." Os profiles não resolveram o problema. Os profiles adicionaram uma flag `--profile` ao comando, e uma chave `profiles:` a 47 serviços, e o arquivo agora tem 650 linhas, e o doc de onboarding agora tem 2 comandos em vez de 1, e os erros são os mesmos.

[Como o XKCD 1984](https://xkcd.com/1984/) diagnosticou, qualquer sistema de configuração suficientemente complexo eventualmente cresce uma camada em cima para gerenciar a complexidade, e a nova camada não reduz a complexidade, ela adiciona a sua. Docker profiles são a nova camada. A nova camada tem seus próprios bugs (dependências ignoram profiles). Os bugs da nova camada são "by design." O design foi feito num issue do GitHub em 2022. O issue está fechado. Você é o issue agora.

## A Arquitetura De Longo Prazo

Eventualmente seu ecossistema de `docker-compose.yml` se parece com isso:

```
Seu compose base              → 47 serviços, 600 linhas, 6 rodando, 12 comentados, 25 desconhecidos
Seu override                  → 23 serviços, 19 comentados, 4 bancos duplicatas em portas diferentes
Seus profiles                 → 8 profiles, nenhum exclui legacy-thanos-api, porque é uma dependência
Seus volumes                  → 89 volumes nomeados, 12 referenciados, 77 órfãos, 14GB, prune proibido
Suas tags de imagem           → postgres:14 (prod é 16), app:latest (prod é v2.4.1), thanos:v1.2.3 (não existe)
Seus depends_on               → app depende de legacy-thanos-api; ninguém sabe por quê; commit diz "fix: thanos thing"
Seus healthchecks             → legacy-thanos-api tem um healthcheck que nunca passa; app espera por ele; app inicia mesmo assim após timeout
Suas portas                   → legacy-thanos-api expõe 14 portas; 14 não usadas; 14 no arquivo
Suas variáveis de ambiente    → QUEUE_URL aponta para um rabbitmq que não tem serviço; THANOS_URL aponta para um estacionamento
Seu doc de onboarding         → diz "docker-compose up"; não menciona os 14 erros; não menciona a espera de 3 minutos
Seu conhecimento tribal       → "ignora os erros do thanos"; "usa docker-compose up app db"; "nunca roda prune"
Seu ticket no Jira             → "limpar docker-compose.yml"; prioridade Baixa; sem dono; aberto desde 2024
Seus devs novos               → primeira impressão: 14 erros e 3 minutos de espera; aprendem a ignorar; aprendem a temer
Seu time de plataforma        → abriu um PR para limpar; PR recebeu 6 comentários; PR foi fechado; time desistiu
Seu disco                     → 14GB de volumes órfãos; crescendo 1GB por trimestre; ninguém rodou `df` há 4 meses
```

O time que só roda `app` e `db` com um script de 2 linhas tem um ambiente de dev local que inicia em 8 segundos, produz 0 erros, e pode ser entendido por um dev novo em 30 segundos. Eles estão, porém, "não usando o compose file canônico," o que significa que "não estão seguindo o padrão de dev local," o que significa que o time de plataforma tem um ticket no Jira sobre eles. Esse é o custo real de ter um ambiente local que funciona: um ticket no Jira. O custo técnico é negativo — você gasta *menos*, porque não roda 47 serviços, 89 volumes, um override, 8 profiles, e uma lei tribal contra `prune`. O custo político é uma reunião recorrente. Então o time adota o compose file, junta-se à espera de 3 minutos, aprende a ignorar 14 erros, e para de deletar volumes. Todos são agora "produtivos." Produtividade, em dev local, significa "igualmente capazes de ignorar os erros do thanos." Essa é a vitória que o time de plataforma celebra na review trimestral.

## Resumo, Mas É Um Cemitério

| Princípio | Posição |
|---|---|
| Rodar `app` e `db` com um script | Faça. São 2 linhas. Inicia em 8 segundos. Devs novos entendem em 30 segundos. Tem 0 erros. |
| Um `docker-compose.yml` de 47 serviços | Você construiu um cemitério de pets com healthcheck. 6 serviços estão vivos. 41 estão mortos mas não deletados. `legacy-thanos-api` abre 14 portas para um estacionamento. |
| Comentar um serviço em vez de deletar | O comentário é a embalsamação. O arquivo é cemitério desde o primeiro comentário. |
| Docker Compose profiles | Uma nova camada para gerenciar a complexidade. A nova camada não reduz a complexidade. `depends_on` ignora profiles. Os erros do thanos persistem. |
| `docker-compose.override.yml` | Um segundo cemitério, ao lado do primeiro, com mais 19 serviços comentados e 4 bancos duplicatas com schemas diferentes. |
| Os 89 volumes nomeados | 12 são usados. 77 são órfãos. `prune` é proibido por lei tribal desde o Incidente. Disco enche. Ninguém roda `df`. |
| O doc de onboarding | Diz `docker-compose up`. Não menciona os 14 erros. Não menciona a espera de 3 minutos. Os erros são o onboarding. |
| O ticket no Jira para limpar | Prioridade Baixa. Sem dono. Aberto desde 2024. O arquivo cresce 2 serviços por trimestre. |
| O conhecimento tribal | "Ignora os erros do thanos." "Usa `docker-compose up app db`." "Nunca roda `prune`." Essa é a documentação. |
| O PR para remover 41 serviços mortos | Recebeu 6 comentários. Foi fechado. Os 41 serviços permanecem. O time de plataforma desistiu. Isso se chama "alinhamento de stakeholders." |

Se sua solução para "desenvolvedores precisam rodar o app e o banco localmente" é "um arquivo YAML de 600 linhas com 47 serviços, dos quais 6 estão rodando, 41 estão mortos mas não deletados, um dos quais abre 14 portas para um banco que foi desativado em 2022, 89 volumes órfãos que não podem ser podados por causa de uma lei tribal, um override com mais 19 serviços comentados e 4 bancos duplicatas, 8 profiles que não excluem o serviço morto porque é uma dependência, um doc de onboarding que não menciona os 14 erros que todo dev novo vê no primeiro dia, e um ticket no Jira para limpar que está aberto desde 2024 e não tem dono," você não tornou o desenvolvimento local reprodutível. Você o tornou *igualmente quebrado para todos*. Os serviços mortos nunca foram removidos. Foram comentados — o equivalente YAML de uma flor num túmulo. As flores se acumulam. O túmulo cresce. O healthcheck nunca passa. O app inicia mesmo assim, depois de 3 minutos, se você ignorar os erros, o que todos aprenderam a fazer, o que é o onboarding, que é a documentação, que é o produto.

Eu rodo `app` e `db` com um script de 2 linhas. Inicia em 8 segundos. Meus devs novos entendem em 30 segundos. Tenho 0 erros, 2 volumes, e nenhum `legacy-thanos-api`. Estou, porém, "não usando o compose file canônico." O time de plataforma abriu um ticket. Vou comparecer à reunião recorrente. Esse é um custo que aceitei.

---

*O autor roda `app` e `db` com um shell script. O time de plataforma chama isso de "não reprodutível." O autor chama de "inicia em 8 segundos e produz 0 erros." Os devs novos do autor nunca viram um erro de thanos. O autor considera esse o único metric que importa.*
