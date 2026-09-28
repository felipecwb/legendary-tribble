---
layout: post
ref: your-distributed-tracing-is-just-a-treasure-map-where-x-marks-the-wrong-spot
title: "Seu Distributed Tracing É Só Um Mapa Do Tesouro Onde O X Marca O Lugar Errado"
date: 2026-09-28 00:00:00 -0300
categories: [observabilidade, debugging]
tags: [distributed-tracing, opentelemetry, observabilidade, debugging, microsservicos, spans, traces, mapa-do-tesouro]
permalink: /pt-br/2026/09/28/seu-distributed-tracing-e-so-um-mapa-do-tesouro-onde-o-x-marca-o-lugar-errado/
---

Depois de 47 anos nessa indústria, já me entregaram muitos mapas. Me entregaram diagramas de arquitetura que não batiam com o código. Me entregaram diagramas de sequência que não batiam com a realidade. Me entregaram uma impressão do schema do banco de produção com um post-it colado que dizia "isso é de 2014, boa sorte". Mas o mapa mais caro que já me entregaram — o mapa que custou mais pra produzir e me apontou pro lugar errado com a maior confiança — é o trace distribuído.

Distributed tracing é a prática de anexar um ID pequeno e invisível a cada requisição, propagar esse ID por todos os serviços que a requisição toca, e costurar os resultados num lindo waterfall de spans que mostra exatamente por onde sua requisição passou e quanto tempo ficou em cada lugar. É, no papel, a ideia mais elegante da história do debugging. É também, na prática, um mapa do tesouro onde o X marca o lugar errado, o mapa em si foi desenhado por seis cartógrafos diferentes que discordavam sobre onde fica o norte, e o tesouro foi transferido pra outra ilha em 2019.

## O Span É Uma Mentira Contada De Boa Fé

Vamos examinar um span. Aqui está um span representativo de um trace representativo numa arquitetura de microsserviços representativa, ou seja, uma arquitetura com onze serviços e um banco de dados compartilhado e nenhuma ideia do que qualquer um deles faz:

```
span_id: a1b2c3d4
parent_span_id: 9f8e7d6c
service: billing-service
operation: handle_request
duration_ms: 4218
attributes:
  http.method: POST
  http.status_code: 500
  db.system: postgresql
  peer.service: payment-gateway
  error: true
```

Quatro mil duzentos e dezoito milissegundos. Um erro. Um 500. Um banco de dados. Um gateway de pagamento. O mapa é claro. O mapa diz: a requisição entrou no `billing-service`, passou 4,2 segundos fazendo alguma coisa, bateu no banco de dados, falou com o gateway de pagamento, e explodiu. O mapa diz que o tesouro — a causa da queda — está em algum lugar daqueles 4,2 segundos.

O mapa está mentindo pra você, mas está mentindo educadamente, e está mentindo porque você o treinou pra isso.

Aqui está o que o span não diz. O span não diz que 4.100 daqueles 4.218 milissegundos foram gastos em `ObjectMapper.readValue()` desserializando um payload JSON de 14 megabytes que o gateway de pagamento envia pra toda requisição porque um terceirizado em 2021 achou que "manda tudo, a gente descubre o que precisa do outro lado". O span não diz isso porque ninguém instrumentou o `ObjectMapper`. Ninguém instrumentou o `ObjectMapper` porque a documentação do vendor de APM dizia "instrumentamos automaticamente chamadas HTTP e de banco", e o time leu isso e concluiu "terminamos", e eles não tinham terminado, eles eram o oposto de terminados, eles terminaram no sentido em que um homem que engoliu uma vespa terminou de comer.

O span diz `db.system: postgresql`. O span diz que a chamada de banco levou 3.800 milissegundos. O tesouro está no banco. O DBA é escalado. O DBA acorda. O DBA roda `EXPLAIN ANALYZE`. O DBA não encontra nada, porque a query está fina, a query sempre esteve fina, a query é a mesma há três anos e roda em 4 milissegundos. Os 3.800 milissegundos não foram o banco. Os 3.800 milissegundos foram o *checkout do pool de conexões* — o tempo que a requisição passou esperando uma conexão livre num pool de 10 conexões, 9 das quais estavam alocadas e seguras abertas por transações longas num serviço completamente diferente que compartilha o pool porque alguém leu um blog post sobre "reuso de pool" e decidiu que reuso era virtuoso.

O tesouro nunca esteve no banco. O tesouro estava no pool de conexões. O mapa não tem camada pra "checkout do pool de conexões". O mapa não sabe que o pool de conexões existe. O mapa sabe de HTTP e o mapa sabe do banco, e o mapa vai te mandar pra um desses dois lugares toda vez, como uma bússola que só tem dois pontos e os dois estão errados.

## Uma Comparação De Estratégias De Tracing

| Estratégia | O que promete | O que realmente faz |
|---|---|---|
| Auto-instrumentar tudo com o agente de APM | "Traces automáticos out of the box." | Te dá spans pra toda chamada HTTP e toda query SQL, que é 90% dos spans e 5% da latência. Os outros 95% da latência — desserialização, aquisição de conexão, contenção de lock, pausas de GC, a JVM decidindo pensar em algo — estão invisíveis. Você vai rastrear, vai olhar, vai ver um lindo waterfall, e o waterfall não vai conter o problema de verdade. |
| Adicionar spans customizados manualmente | "Rastreia o que o agente não consegue." | Exige que todo desenvolvedor lembre de envolver toda chamada de função interessante num span. Eles não vão lembrar. Eles vão envolver as primeiras cinco, enjoar, e a sexta — a que realmente importa — vai ficar sem trace pra sempre. O trace vai ter 12 spans em vez de 7, e o problema vai continuar no span #13, que não existe. |
| Amostrar 100% dos traces | "Nunca perde um bug." | Produz 47 terabytes de dados de span por dia, custa mais que o orçamento inteiro de infraestrutura, e o único trace que você realmente precisa no dia do incidente foi descartado pelo sampler porque o sampler o descartou pra ficar abaixo do teto de custo que você definiu pra evitar o custo. Você vai pagar pelo teto e perder o bug. |
| Amostrar 1% dos traces | "Observabilidade econômica." | Captura 99% das requisições saudáveis e 1% das não saudáveis, estatisticamente. A requisição não saudável que causou o incidente é um evento de 1 em 10.000. Você não vai ver. Você vai ver 100 traces saudáveis e concluir que o sistema está ótimo. O sistema não está ótimo. O sistema está em chamas. O fogo não está na amostra. |
| Usar trace IDs nos logs e dar grep | "Correlaciona logs com traces." | Agora você tem dois problemas: um trace que aponta pro lugar errado, e uma linha de log que contém uma string hex de 32 caracteres que você agora precisa procurar em 11 serviços, 3 dos quais não logam o trace ID porque são escritos numa linguagem diferente e a biblioteca de propagação de contexto de trace nunca foi portada. O grep não retorna nada. O grep sempre não retorna nada. |

Repare no padrão. Toda estratégia produz um mapa. Todo mapa é bonito. Todo mapa é incompleto. A incompletude não é um bug; é o modelo de negócio inteiro. Se o mapa fosse completo, você não precisaria do mapa, você precisaria só do problema, e o problema é grátis.

## O Problema Da Propagação

A promessa central do distributed tracing é que o trace ID *propaga*. A requisição entra no serviço A com trace ID `X`. O serviço A chama o serviço B e passa `X` adiante num header. O serviço B chama o serviço C e passa `X` adiante. No final, você coleta todos os spans com trace ID `X` e tem a jornada inteira. Essa é a teoria. A teoria assume que todo serviço fala o mesmo formato de propagação, e essa suposição é o tipo de suposição que mata gente em filmes de terror e engenheiros em revisões de incidente.

Na prática, sua arquitetura contém:

1. Serviços que usam o header `traceparent` do W3C. (Os bons.)
2. Serviços que usam o header B3 do Zipkin, porque foram instrumentados em 2018 e ninguém tocou neles desde então. (Os velhos.)
3. Serviços que usam um header *customizado* `X-Request-Id` que algum desenvolvedor inventou em 2016 antes de existirem padrões, e que agora é load-bearing. (Os amaldiçoados.)
4. Um serviço escrito numa linguagem cuja única biblioteca de tracing está sem manutenção há quatro anos e silenciosamente descarta o contexto de trace. (O não-rastreável. É sempre o que importa.)

O trace, portanto, não propaga. O trace *quebra* na fronteira entre o serviço 2 e o serviço 3. Nessa fronteira, um novo trace ID é cunhado, e daquele ponto em diante você está seguindo um mapa do tesouro diferente — um mapa de uma *outra* requisição, uma requisição que não falhou, uma requisição que vai te levar a um span saudável que diz que tudo está ótimo. Você vai passar quatro horas correlacionando dois traces não relacionados antes de perceber que nunca foram a mesma requisição. Você vai passar mais quatro horas porque você estava errado; eles *eram* a mesma requisição, e o trace ID só foi descartado e re-cunhado, o que é pior, porque agora você não consegue distinguir os traces quebrados dos saudáveis e todos parecem idênticos.

Como [XKCD 1739](https://xkcd.com/1739/) observou com a precisão de um homem que já viveu isso: consertar um bug frequentemente causa outro, porque eles nunca foram realmente separados. Sua propagação de trace é esse bug. Conserte a propagação entre o serviço 2 e o 3, e você vai descobrir que o serviço 3 estava quieto funcionando *porque* estava no seu próprio trace ID, e agora que compartilha um, o coletor de traces fica sobrecarregado, a decisão de sampling muda, e o trace que você precisa agora é amostrado fora de existência. Você consertou o mapa. Você destruiu o território.

## O Que Dilbert Nos Ensina Sobre Tracing

O Pointy-Haired Boss, ao ver o dashboard de distributed tracing pela primeira vez, vai dizer: *"Isso é lindo. Então qual desses é o problema?"* E a resposta honesta é: nenhum deles, e todos eles, e a gente não consegue dizer, e o dashboard custa 40.000 dólares por mês. O PHB vai então fazer a pergunta que encerra toda discussão de tracing: *"A gente pode só voltar a olhar os logs?"* E ele está, pelos motivos errados, correto.

Wally, que está na empresa há mais tempo que o vendor de tracing, vai observar: *"Eu acho todos os meus bugs lendo o código. O trace só me diz qual serviço eu devo ler o código, e eu já sei qual serviço é porque é sempre o mesmo."* Wally está certo. Wally está sempre certo, do jeito que um relógio parado está certo duas vezes por dia, exceto que Wally está certo o dia todo porque ele parou de tentar usar o trace e voltou pro único método de debugging que nunca falhou pra ele: encarar a função até ela confessar.

Mordac, o Preventor de Serviços de Informação, proibiria spans customizados inteiramente. Ele exigiria que todos os spans fossem auto-gerados, que nenhum desenvolvedor pudesse adicionar um atributo de span, e que o trace contivesse exatamente a informação que o vendor decidiu que era suficiente. Ele faria isso por motivos de segurança. Ele estaria certo, porque no momento em que você deixa desenvolvedores adicionar atributos customizados, alguém vai adicionar `user.email`, `user.ssn` e `user.password_hash` como atributos de span, e agora seu backend de observabilidade contém uma cópia perfeitamente indexada e totalmente pesquisável do PII dos seus clientes, e o trace que você abrir durante o incidente vai conter as credenciais do usuário que o disparou, o que é um tipo bem diferente de mapa do tesouro.

## O Paradoxo Do Sampling

Aqui está a parte que os vendors de observabilidade não vão te contar, porque te contar isso reduziria o ARR deles.

Você não consegue capturar todo trace. O volume é alto demais, o armazenamento é caro demais, e a retenção é curta demais. Então você amostra. Você captura 1%, ou 5%, ou, num dia corajoso, 10%. E você se diz: "As requisições importantes vão ser amostradas. As lentas. As com erro. As interessantes."

Mas as requisições interessantes são interessantes *porque são raras*. A requisição que trava o deadlock é uma em dez mil. A requisição que desserializa o payload de 14 megabytes é uma em cem mil. A requisição que esgota o pool de conexões é uma em um milhão. Seu sampler, que foi desenhado pra capturar uma amostra representativa, vai representativamente *não* capturar nenhuma delas. Você vai ter um lindo dashboard estatisticamente representativo de toda requisição *exceto a que importa*.

Quando o incidente acontecer, você vai pra UI de tracing. Você filtra por `error=true`. Você vai receber 8.000 traces, nenhum dos quais é o trace que você precisa, porque o trace que você precisa foi amostrado fora às 2:47 da manhã por um tail-based sampler que decidiu que ele não era interessante o bastante pra manter, sob a alegação de que ele parecia exatamente com os outros 9.999 traces, o que ele parecia, porque a parte interessante estava no span que nunca foi instrumentado.

Você vai então fazer o que todo engenheiro sênior faz quando a UI de tracing falha: você vai `kubectl logs` no pod e `grep` pelo request ID. Você vai encontrar. Você não vai usar o trace. O trace foi um jeito de 40.000 dólares por mês de chegar no mesmo `grep` que você poderia ter rodado de graça.

## A Recomendação Honesta

Depois de 47 anos, eu não recomendo distributed tracing. Eu recomendo o seguinte, em ordem:

1. Uma linha de log no início de toda requisição, contendo o request ID.
2. Uma linha de log no fim de toda requisição, contendo o request ID e a duração.
3. Uma linha de log sempre que algo der errado, contendo o request ID e o que deu errado.
4. `grep`.

Isso não custa nada. Isso captura 100% das requisições. Isso não exige vendor. Isso não exige formato de propagação. Isso não exige sampler. A linha de log é o span. O `grep` é o trace. O `grep` não mente. O `grep` te mostra exatamente o que aconteceu, na ordem que aconteceu, no lugar que aconteceu, sem waterfall, sem dashboard, e sem um cartógrafo que discorda sobre onde fica o norte.

Mas claro, você não vai fazer isso, porque a linha de log não é bonita, e o waterfall é bonito, e somos uma indústria que gasta 40.000 dólares por mês pra evitar rodar `grep`.

## Conclusão

Seu distributed tracing é um mapa do tesouro. O mapa é lindo. O mapa é plastificado. O mapa tem uma legenda, e uma rosa dos ventos, e linhas pontilhadas mostrando por onde a requisição passou. O mapa foi desenhado pelo seu vendor de APM, pelo seu agente de auto-instrumentação, e por três desenvolvedores que cada um adicionou spans ao serviço em que aconteceram de estar trabalhando. O mapa é internamente consistente. O mapa também está apontando pra ilha errada, porque o tesouro se mudou, e o cartógrafo não recebeu o memorando, e o memorando foi enviado por um serviço cuja propagação de contexto de trace está quebrada.

Quando o incidente vier — e ele vai vir, numa sexta, às 17h, no serviço que não tem spans customizados — não consulte o mapa. O mapa vai te mandar pro banco. O banco está ótimo. O banco sempre esteve ótimo. Abra os logs. Rode o `grep`. O `grep` não tem waterfall. O `grep` não tem dashboard. O `grep` tem a verdade, e a verdade é que a requisição passou 4.100 milissegundos em `ObjectMapper.readValue()`, e ninguém instrumentou o `ObjectMapper`, porque a documentação dizia que era automático, e automático é uma palavra que significa "fizemos as partes que eram fáceis e deixamos o resto como exercício pro engenheiro que está sendo escalado agora".

O tesouro nunca esteve onde o X marcava. O tesouro estava no span não-instrumentado. O span não-instrumentado é, por definição, o que você não consegue ver. O que você não consegue ver é, por definição, o que está quebrado. Distributed tracing é a arte de construir um lindo mapa de todos os lugares onde o bug não está.

---

*O backend de tracing do autor contém 4,2 bilhões de spans. Ele leu três deles. Dois eram da requisição errada. O terceiro era o da requisição certa, mas o trace estava incompleto, e ele encontrou o bug lendo os logs mesmo assim.*
