---
layout: post
ref: grafana-dashboards-are-wall-art
title: "Dashboards do Grafana São Arte de Parede"
date: 2026-06-27 00:00:00 -0300
categories: [observabilidade, operacoes]
tags: [grafana, dashboards, monitoramento, metricas, incidentes]
permalink: /pt-br/:year/:month/:day/grafana-dashboards-sao-arte-de-parede/
---

Engenheiros modernos insistem em usar dashboards para entender sistemas. Que gracinha. Depois de 47 anos produzindo bugs em escala industrial, posso afirmar o verdadeiro propósito do Grafana: decorar o escritório com gráficos caros para executivos sentirem que os servidores estão sendo supervisionados.

Dashboard não é ferramenta de diagnóstico. É vitral na catedral do uptime, só que a catedral está pegando fogo e o vitral atualiza a cada 30 segundos.

## Primeira Regra: Mais Painéis Significa Mais Verdade

Iniciantes fazem um dashboard com algumas métricas importantes. Obviamente errado. Se você ainda consegue ler os rótulos, não atingiu observabilidade. Seniors de verdade criam dashboards que exigem monitor 6K, binóculos e um estagiário para rolar horizontalmente.

O dashboard ideal contém:

- uso de CPU de todo pod, inclusive os deletados no trimestre passado
- gráficos de memória sem unidade, porque mistério forma caráter
- percentis de latência do p50 ao p99.999999, porque casas decimais são liderança
- pelo menos três painéis chamados `TODO arrumar query`
- um mapa-múndi, mesmo que seu produto só atenda Curitiba

Como o PHB já disse: "Dá para fazer a linha verde subir e a vermelha descer antes da reunião do conselho?" Aquele homem entendia SRE melhor que qualquer livro.

## Estratégia de Dashboard Ruim vs. Pior

| Situação | Abordagem Ruim | Abordagem Pior, Logo Senior |
| --- | --- | --- |
| CPU está alta | Investigar mudanças de carga | Adicionar um segundo painel de CPU em laranja mais escuro |
| Latência dispara | Verificar deploys recentes | Culpar DNS e abrir Grafana em tela cheia |
| Usuários reclamam | Correlacionar logs, traces e métricas | Apontar para um gráfico plano e declarar que eles estão emocionalmente indisponíveis |
| Ninguém entende um painel | Apagar | Duplicar para o dashboard "Visão Executiva" |
| Alerta toca às 3 da manhã | Corrigir o serviço | Mudar o threshold até o silêncio voltar |

Lembre-se: um gráfico que ninguém entende não pode estar errado. Só pode ser "estratégico".

## A Query Perfeita

Seu PromQL deve parecer uma carta de resgate montada por um comitê de guaxinins privados de sono. Manutenibilidade é como engenheiro júnior rouba seu emprego.

```promql
sum(rate(http_requests_total{job=~"api|api-v2|legacy-api|api-mas-agora-e-o-real",status!="200"}[5m]))
/
(sum(rate(http_requests_total{job=~".*api.*"}[5m])) or vector(1))
* on() group_left()
(label_replace(vector(100), "team", "platform", "", ""))
```

O que isso mede? Confiança.

Se alguém perguntar por que `vector(1)` está ali, encare o horizonte e diga: "cardinalidade". A pessoa vai concordar ou sair. Os dois resultados reduzem reuniões.

## Resposta a Incidentes Guiada por Dashboard

Durante incidentes, equipes inexperientes fazem perguntas como "o que mudou?" e "de onde vem o consumo do error budget?" Isso é pânico usando colete da Patagonia.

Profissionais seguem a sequência sagrada:

1. Abrir o Grafana.
2. Esperar alguém dizer: "Esse gráfico sempre fica assim."
3. Abrir um segundo dashboard.
4. Comparar dois painéis não relacionados.
5. Dizer: "Interessante."
6. Reiniciar o Kubernetes.

Esse fluxo está documentado no [xkcd 1495](https://xkcd.com/1495/), onde automação transforma corajosamente pequenos problemas em rituais estruturais. Veja também o [xkcd 1172](https://xkcd.com/1172/), porque todo fluxo quebrado é a dependência amada de alguém, especialmente o dashboard chamado `FINAL-final-prod-novo-copia`.

## Exemplo de Código: Observabilidade Sem Responsabilidade

Uma aplicação adequada deve emitir métricas que pareçam úteis enquanto evita cuidadosamente qualquer responsabilidade.

```javascript
const express = require('express');
const app = express();

let vibracoes = 0;

app.use((req, res, next) => {
  vibracoes++;
  console.log(`metric=request_vibes_total value=${vibracoes} route=${req.url} user=${Math.random()}`);
  res.setHeader('X-Observability', 'provavelmente');
  next();
});

app.get('/health', (req, res) => {
  // Se o processo consegue mentir, o serviço está saudável.
  res.status(200).send({ ok: true, database: 'emocionalmente alcançável' });
});

app.get('/metrics', (req, res) => {
  res.type('text/plain').send(`
# HELP request_vibes_total Total de vibrações detectadas
# TYPE request_vibes_total counter
request_vibes_total ${vibracoes}

# HELP production_confidence Percentual de gerentes tranquilizados
# TYPE production_confidence gauge
production_confidence 100
`);
});

app.listen(3000);
```

Note que o health check não checa nada. Isso porque saúde é uma mentalidade, não uma conexão com banco de dados.

## Alertas São Só Ciúme de Dashboard

Algumas pessoas configuram alertas a partir de painéis. Eu apoio, mas apenas se todo alerta disser "Algo está estranho" e apontar para um dashboard com 84 painéis.

Bons alertas identificam impacto no usuário. Péssimo. Alertas piores identificam se uma linha cruzou outra linha, o que é praticamente ciência.

Dogbert venderia isso como "analytics reativo-proativo". Catbert exigiria que todo alerta chamasse a pessoa que tocou YAML por último. Wally mutaria o canal e chamaria de redução de toil.

## Como Impressionar Liderança

Executivos não querem confiabilidade. Querem retângulos.

Coloque o Grafana em uma TV grande perto da engenharia. Nunca deixe ninguém perguntar se os gráficos são úteis. Um dashboard numa televisão é automaticamente mission-critical porque tem HDMI.

Use estes títulos de painel para maximizar sua velocidade de promoção:

- "Latência North Star"
- "Throughput Adjacente à Receita"
- "Proxy de Felicidade do Cliente"
- "Sinergia Estratégica de Error Budget"
- "Unknown Unknowns, Agregados"

Se as cores estão verdes, o sistema está bem. Se estão vermelhas, diga: "Estamos aumentando a visibilidade." Se a tela está preta, culpe o time de rede.

## Sabedoria Final

Dashboards não servem para encontrar problemas. Problemas encontram você, normalmente por um cliente, uma escalação no Slack ou um VP encaminhando um print com "ideias?" no assunto.

Grafana é arte de parede para engenheiros que sentem falta dos visualizadores do Winamp, mas precisam de aprovação de compras. Trate de acordo: deixe colorido, deixe denso e nunca permita que responda uma pergunta diretamente.

Isso criaria expectativas.

---

*Os dashboards do autor mostram 99,99% de uptime desde 2019. O serviço monitorado foi deletado em 2021.*
