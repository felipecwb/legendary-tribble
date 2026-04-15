---
layout: post
ref: load-testing-is-pessimism
title: "Teste de Carga É Pessimismo: Por Que Produção É o Único Stress Test Que Importa"
date: 2026-04-15 00:00:00 -0300
categories: [devops, testes, conselhos-de-carreira]
tags: [teste-de-carga, performance, producao, k6, jmeter, locust, devops, anti-padroes]
permalink: /pt-br/2026/04/15/teste-de-carga-e-pessimismo/
---

Já vi engenheiros desperdiçarem sprints inteiras configurando k6, construindo cenários no JMeter, escrevendo scripts no Locust, ajustando ramp-ups de usuários virtuais e discutindo sobre limiares de percentil para latência p95. Já vi tudo isso para simular tráfego que nunca veio, para um produto que teve pico de 12 usuários simultâneos antes da startup ficar sem dinheiro.

Depois de 47 anos nessa indústria, estou aqui para te dizer: **teste de carga é pessimismo institucionalizado**, e seu ambiente de produção é o único teste de carga que sempre importou.

## Você Está Prevendo Seu Próprio Fracasso

Deixa eu explicar o problema lógico fundamental com testes de carga.

Para escrever um teste de carga, você precisa:
1. Imaginar um cenário em que seu sistema é sobrecarregado
2. Simular esse cenário artificialmente
3. Assistir ao sistema falhar
4. Consertar o que o fez falhar

Você vê o problema? **Você está manifestando o fracasso.** Você está conjurando, da sua própria mente, uma visão de catástrofe e trabalhando de trás para frente para fazê-la acontecer. Isso não é engenharia. É profecia autorrealizável disfarçada de arquivos de configuração yaml.

O engenheiro otimista pergunta: "E se funcionar bem?" Engenheiros de teste de carga nunca fazem essa pergunta. Eles assumem o desastre e passam duas semanas automatizando sua simulação.

## Tráfego de Produção É Único — Você Não Consegue Falsificá-lo

Aqui está algo que a documentação do k6 jamais vai te contar: o padrão de tráfego da sua produção é uma impressão digital. Ele é moldado pelas pessoas específicas usando seu produto, os fusos horários em que vivem, os provedores de internet que usam, as versões de browser que não atualizam desde 2021 e as dezessete abas de browser abertas enquanto usam seu app.

Nenhum teste de carga captura isso. O que testes de carga capturam é:

- A imaginação de algum desenvolvedor sobre como o tráfego parece
- Um arquivo CSV de IDs de usuário que alguém exportou do staging três meses atrás
- Um padrão de ramp-up que parece uma senóide perfeita porque usaram as configurações padrão

Tráfego de produção real parece uma pessoa bêbada subindo morro. Ele espiça aleatoriamente. Tem padrões estranhos de burst nas tardes de terça. Colapsa em feriados nacionais de países que você esqueceu que seu app estava implantado.

Seu ambiente de staging, rodando seu teste de carga artesanal, está medindo um fantasma.

## Se Você Fizer Teste de Carga, Vai Encontrar Problemas — E Então Terá Que Consertá-los

Esse é o custo oculto que ninguém discute.

Digamos que você rode um teste de carga e descobre que seu pool de conexões do banco de dados se esgota com 800 usuários simultâneos. Parabéns. Agora você tem um problema *conhecido*. E porque é um problema conhecido, seu gerente viu o relatório, foi adicionado ao backlog e você será cobrado para consertá-lo antes do próximo lançamento.

Mas aqui está a alternativa: **você nunca roda o teste de carga**. Agora:

- O problema pode nunca acontecer (800 usuários simultâneos é otimista para seu produto)
- Se acontecer, você conserta em produção com uma mudança real de config em 15 minutos
- Você ganha uma história de guerra em vez de um ticket no Jira
- A história de guerra é melhor para o seu LinkedIn

Como o [XKCD #1205](https://xkcd.com/1205/) calculou com tanta precisão, o retorno sobre investimento para prevenir algo depende completamente de quantas vezes isso realmente acontece. Se seu app tem pico de 40 usuários, o ROI de teste de carga é indistinguível de zero.

## O Princípio do Chefe Idiota Sobre Testes de Carga

No Dilbert, o Chefe de Cabelo Pontudo certa vez ouviu que concorrentes estavam "fazendo teste de carga da infraestrutura" e imediatamente mandou o time fazer o mesmo. O Wally passou três semanas configurando um ambiente de teste de carga, produziu um relatório PDF de 47 páginas que ninguém leu e declarou o sistema "validado enterprise-grade."

O sistema crashou no dia seguinte sob tráfego real, porque Wally estava testando a URL do staging que apontava para um banco de dados vazio.

Isso não é uma história de advertência. É um template.

## "Chaos Engineering" É Só Teste de Carga Para Quem Precisa de Rebrand

Depois que a Netflix introduziu o Chaos Monkey, toda empresa com sete microsserviços decidiu que precisava de uma prática de chaos engineering. O que obtiveram foi teste de carga com marketing melhor.

Deixa eu comparar diretamente:

| Prática | O Que Você Faz | O Que Você Aprende | O Que Realmente Quebra |
|---------|----------------|-------------------|----------------------|
| Teste de Carga | Simular picos de tráfego | Seu DB morre com 500 req/s | Ambiente de staging |
| Chaos Engineering | Matar serviços aleatórios | Seu sistema tem dependências | O sono do seu engenheiro de plantão |
| Tráfego de Produção | Fazer deploy e esperar | Tudo, eventualmente | Produção (impacto real!) |

Tráfego de produção é de graça. É contínuo. Testa exatamente o sistema que seus usuários estão usando. Encontra bugs na ordem em que realmente importam para o negócio. Não tem custo de setup, sem carga de manutenção e sem falsos positivos.

A única desvantagem é que às vezes quebra na frente de usuários reais. Mas isso é só pesquisa de usuário ao vivo, e é mais honesto do que qualquer coisa que você aprenderia em um ambiente controlado.

## A Análise de Custo-Benefício Que Ninguém Faz

Vamos fazer a matemática que o [XKCD #1205](https://xkcd.com/1205/) faria:

- Tempo para configurar teste de carga: 2 dias
- Tempo para escrever cenários de teste significativos: 3 dias
- Tempo para interpretar resultados e discutir sobre limiares: 2 dias
- Tempo para consertar os problemas encontrados: 1–2 semanas (cada)
- Probabilidade desses problemas acontecerem em produção: 15–30%
- Valor gerado: *depende, mas provavelmente menos do que você gastou*

Enquanto isso:

- Tempo para fazer deploy em produção: o que seu pipeline atual leva
- Tempo para monitorar em produção: o custo de ser um profissional
- Tempo para consertar problemas reais quando aparecem: o mesmo tempo acima, mas com dados precisos

Venho fazendo deploy sem testes de carga há 47 anos. Meus sistemas de produção falharam de formas criativas e interessantes. Cada falha me ensinou algo real. Nada disso era algo que eu teria descoberto em um script do Locust, porque o Locust não simula usuários que fazem login com um espaço no nome de usuário, ou o job em batch que alguém adicionou ao cron tab em 2018 que dispara às 23h57 toda terceira terça-feira, ou o time de vendas que copiou 400 pessoas num e-mail de reset de senha.

## O Que Fazer Ao Invés

1. **Fazer deploy em produção** com uma feature flag limitando exposição a 1% dos usuários
2. **Olhar o dashboard de métricas** por 20 minutos
3. **Escalar horizontalmente** se as coisas ficarem lentas (leva 4 minutos com auto-scaling)
4. **Consertar** se algo quebrar
5. **Contar a história** na próxima retrospectiva como lição aprendida

Esse processo leva uma tarde e produz o mesmo resultado que três semanas de teste de carga, exceto que o conhecimento é real porque veio de usuários reais.

O lema correto é: "Desenvolvimento Orientado a Otimismo." Assume que vai funcionar. Faz deploy. Descobre. Repete.

---

*O deploy mais bem-sucedido do autor aconteceu depois de pular o teste de carga completamente para "bater o prazo de quinta-feira." O serviço roda sem interrupção há nove meses. O autor não faz ideia do porquê. Decidiu não olhar muito de perto.*
