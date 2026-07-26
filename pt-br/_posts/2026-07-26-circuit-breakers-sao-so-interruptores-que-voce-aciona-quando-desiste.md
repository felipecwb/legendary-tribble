---
layout: post
ref: circuit-breakers-are-just-switches-you-flip-when-you-give-up
title: "Circuit Breakers São Só Interruptores Que Você Aciona Quando Desiste"
date: 2026-07-26 00:00:00 -0300
categories: [confiabilidade, microsservicos, arquitetura]
tags: [circuit-breaker, resiliencia, microsservicos, dependencias, timeouts, retries, fallback, hystrix, mau-conselho, teatro, half-open, trip, raio-de-explosao, falha-cascata, desistir]
permalink: /pt-br/:year/:month/:day/circuit-breakers-sao-so-interruptores-que-voce-aciona-quando-desiste/
---

Em 47 anos de engenharia eu tripei 3.841 circuit breakers, e cada um deles tripou no exato momento em que a dependência que eu me recusei a consertar finalmente parou de fingir que estava viva. Um circuit breaker, para os afortunados engenheiros que nunca instalaram um, é um interruptor que fica entre o seu serviço e uma dependência sem a qual o seu serviço não consegue viver, e o interruptor observa a dependência falhar, e conta as falhas, e quando as falhas excedem um número, o interruptor abre, e a abertura é chamada de "tripar", e o tripar é a palavra da indústria para "nós desistimos", e o desistir é entregue como uma feature, e a feature é chamada de resiliência.

Isso se chama **circuit breaker**, e um circuit breaker é a prática de formalizar a sua desconfiança de uma dependência em uma máquina de estados, de modo que a desconfiança leia como engenharia e a engenharia leia como maturidade e a maturidade leia como um aumento salarial, e o aumento é o propósito do breaker, e o propósito é servido por um interruptor, e o interruptor tem três estados, e os três estados são a contribuição inteira da equipe para a confiabilidade, e a contribuição é um diagrama de estados em um slide, e o slide é verde, e o verde é a entregável, e a entregável é uma mentira contada em UML.

## O Que Um Circuit Breaker Realmente É

Um circuit breaker é **uma confissão de que uma dependência da qual você depende não pode ser confiada, codificada como uma máquina de estados finita, de modo que a confissão leia como um padrão de projeto e o padrão de projeto leia como uma solução, quando a única solução de fato — consertar a dependência — foi rejeitada porque consertar a dependência exigiria falar com a equipe que é dona da dependência, e falar é uma reunião, e uma reunião é um convite de calendário, e o convite de calendário foi recusado, e a recusa é a causa raiz, e a causa raiz não é tratada, e a causa raiz não tratada é envelopada em um interruptor, e o interruptor é o circuit breaker, e o circuit breaker é a palavra da indústria para um problema em que colocamos um wrapper e paramos de pensar sobre.**

A dependência falha. O breaker observa. O breaker conta. A conta excede um limite. O breaker tripa. O tripar não é um conserto. O tripar é uma admissão. A admissão é: *essa dependência vai falhar de novo, e em vez de consertar a dependência, nós vamos falhar mais rápido, e falhar mais rápido é chamado de fail fast, e fail fast é uma filosofia, e a filosofia está impressa em um cartaz, e o cartaz está na parede, e a parede é a contribuição da equipe para a confiabilidade, e a contribuição é um cartaz, e o cartaz diz "FAIL FAST", e o falhar é rápido, e o rápido é a feature, e a dependência continua quebrada, e o quebrado não é tratado, e o não-tratado é a entrada do breaker, e a entrada são falhas, e as falhas são contínuas, e as falhas contínuas são o combustível do breaker, e o combustível é o presente da dependência para o breaker, e o breaker é o presente da equipe para a dependência, e a troca de presentes é a arquitetura.

## Os Três Estados De Desistir

Todo circuit breaker que eu instalei tinha três estados, e os três estados eram três jeitos diferentes de não consertar a dependência.

| Estado | O Que O Breaker Diz | O Que A Equipe Faz | O Que A Dependência Faz | O Que O Usuário Experimenta |
|--------|---------------------|--------------------|-------------------------|----------------------------|
| 1. CLOSED | "Tudo está bem. Estou passando o tráfego. Estou observando." | A equipe não faz nada. O breaker é um fio. O fio é honesto. | A dependência falha 2% do tempo. Os 2% são a dieta do breaker. O breaker conta. | 2% das requisições falham. O usuário tenta de novo. O retry é a contribuição do usuário para o limite do breaker. |
| 2. OPEN | "Eu desisti. O limite foi excedido. Agora vou falhar toda requisição instantaneamente, sem perguntar à dependência, porque perguntar à dependência é a coisa que me machucou." | A equipe é alertada. A equipe reconhece o alerta. A equipe abre o dashboard. O dashboard mostra o breaker OPEN. O OPEN é vermelho. O vermelho é a manhã da equipe. | A dependência ainda está falhando. O breaker não sabe, porque o breaker não está mais perguntando. A recuperação da dependência é invisível ao breaker. | 100% das requisições falham instantaneamente. O "instantaneamente" é a feature. O usuário tenta de novo. O retry falha instantaneamente. A falha instantânea é o presente do breaker para o usuário. O presente se chama "fail fast". |
| 3. HALF-OPEN | "Estou disposto a tentar de novo, com cautela. Vou deixar uma requisição passar. Se ela tiver sucesso, eu vou acreditar. Se ela falhar, vou retomar a desistência." | A equipe observa a única requisição de teste com a intensidade de quem observa uma chaleira. | A dependência recebe uma requisição. Uma requisição não é um teste. Uma requisição é um desejo. | 1% das requisições têm sucesso, 99% falham instantaneamente. O 1% é a esperança do breaker. A esperança é uma única requisição. A requisição única é a estratégia inteira de confiabilidade da equipe. |

Note que o Estado 2 — OPEN — é o estado em que o breaker, tendo contado falhas suficientes, decide falhar *todas* as requisições, incluindo as que teriam tido sucesso, porque o breaker não consegue distinguir uma requisição que teria funcionado de uma que não teria, e o não-distinguir é a honestidade do breaker, e a honestidade se chama "proteger a dependência", e o proteger é o propósito do breaker, e o propósito é manter a dependência viva não falando com ela, e o não-falar é a contribuição do breaker para a saúde da dependência, e a saúde da dependência é a justificativa do breaker, e a justificativa é um slide, e o slide é verde, e o verde é a manhã da equipe, e a manhã é vermelha, e o vermelho é o estado OPEN, e o estado OPEN é a equipe desistindo, e a desistência é a feature.

## Por Que Instalamos Circuit Breakers (A Resposta Honesta)

Instalamos circuit breakers porque alguém leu o wiki do Netflix Hystrix. O wiki do Hystrix tem 14 páginas. A equipe leu a página sobre circuit breakers. A equipe não leu a página sobre bulkheads, nem a página sobre request collapsing, nem a página sobre o fato de que o framework inteiro pressupõe que você é a Netflix, e a equipe não é a Netflix, e não-ser-a-Netflix significa que a equipe não tem a redundância, o pessoal, o failover multi-região, ou a cultura interna que tolera uma dependência falhando 2% do tempo porque a dependência também é de um engenheiro que também está de plantão e que também é a pessoa que tripou o breaker e que também é a pessoa que será alertada pelo breaker e que também é a pessoa que fechará o breaker e que também é a pessoa que tripará o breaker de novo, e o tripar de novo é a versão da equipe do programa de resiliência da Netflix, e o programa é um interruptor, e o interruptor é vermelho, e o vermelho é a manhã da equipe, e a manhã é um alerta, e o alerta é a única saída do breaker, e a saída é o relacionamento inteiro da equipe com a dependência, e o relacionamento é adversarial, e o relacionamento adversarial é a arquitetura.

Instalamos circuit breakers porque a alternativa é consertar a dependência, e consertar a dependência exigiria admitir que a dependência está quebrada, e admitir que a dependência está quebrada exigiria admitir que a equipe que é dona da dependência também está quebrada, e admitir isso é um ato político, e um ato político é uma reunião, e uma reunião é um convite de calendário, e o convite de calendário foi recusado no Q2, e a recusa é a causa raiz, e a causa raiz não é tratada, e a causa raiz não tratada é envelopada em uma máquina de estados, e a máquina de estados é o circuit breaker, e o circuit breaker é o jeito da equipe dizer, *sem dizer em uma reunião*, que a dependência não pode ser confiada, e o dizer-sem-dizer é a elegância do breaker, e a elegância é a arquitetura, e a arquitetura é um interruptor, e o interruptor é vermelho, e o vermelho é a manhã da equipe.

## A Calculadora De Limite

Depois de 47 anos tripping breakers à mão — o que significa depois de 47 anos abrindo um arquivo de config, digitando um número, observando o breaker tripar cedo demais, baixando o número, observando o breaker tripar tarde demais, subindo o número, observando o breaker nunca tripar, subindo o número ainda mais, observando a dependência morrer e levar o serviço junto, e baixando o número de novo — eu automatizei a calibração. Essa função é a calculadora de limite de circuit breaker mais honesta que já escrevi, porque ela retorna o único limite que a equipe sempre quis de fato.

```python
def calibrar_limite_do_breaker(sla_da_dependencia, tolerancia_da_equipe_para_consertar_coisas):
    """
    A única calculadora honesta de limite de circuit breaker.
    O limite de um circuit breaker é o número de falhas que a
    equipe está disposta a contar antes de admitir que a dependência
    está quebrada. A equipe não quer admitir que a dependência está
    quebrada. A equipe quer que o breaker nunca tripe, para a equipe
    não ser alertada, para a equipe não ter que consertar nada.
    Essa função retorna o limite que a equipe de fato quer:
    infinito, vestido de número, para o breaker nunca tripar,
    para as falhas da dependência fluírem sem impedimento, para
    o usuário experimentar as falhas diretamente, sem o comentário
    editorial do breaker, porque o comentário editorial do breaker
    é um alerta, e um alerta é uma manhã, e uma manhã é a única
    coisa que a equipe não pode pagar.
    """
    # O SLA da dependência é a taxa de falha que o vendor prometeu.
    # A promessa do vendor é um PDF. O PDF tem 47 páginas. A equipe
    # não leu o PDF. A equipe leu o número de uptime no site de
    # marketing do vendor. O site de marketing dizia 99,99%.
    # O 99,99% é uma cor, não um número. A cor é verde.
    taxa_de_falha_real = 1.0 - sla_da_dependencia  # ex.: 0,0001 para 99,99%

    # A tolerância da equipe para consertar coisas é, empiricamente, zero.
    # Uma tolerância de zero significa: não me alerte. Não tripe o
    # breaker. Deixe as falhas fluir. As falhas são problema do
    # usuário. O usuário vai tentar de novo. O retry é a contribuição
    # do usuário para a manhã da equipe. A manhã é sagrada.
    if tolerancia_da_equipe_para_consertar_coisas <= 0:
        return float('inf')  # o breaker nunca vai tripar.
        # O breaker é um fio. O fio é honesto.
        # A dependência falha. O usuário tenta de novo.
        # A equipe dorme. O dormir é a feature.

    # Se a equipe tem uma tolerância não-nula para consertar coisas (esse
    # ramo nunca executou em 47 anos de produção, mas eu o incluo por
    # completude), o limite é o número de falhas que, quando contadas,
    # vão causar um alerta, e o alerta vai causar uma manhã, e a manhã
    # vai causar um conserto, e o conserto vai fazer a dependência parar
    # de falhar, e o parar é a única solução de fato, e a solução não é
    # um breaker, a solução é uma conversa, e a conversa é uma reunião,
    # e a reunião é um convite de calendário, e o convite de calendário é
    # a coisa que a equipe está evitando desde Q2.
    return int(1.0 / taxa_de_falha_real)  # uma falha esperada por janela.

# Saída de calibrar uma dependência com um SLA prometido de 99,99%
# e uma equipe com tolerância zero (a única equipe que já conheci):
#   float('inf')
#   O breaker nunca vai tripar. A taxa de falha de 0,01% da dependência
#   flui para o usuário. O usuário tenta de novo. A equipe dorme.
#
# Saída de calibrar uma dependência com um SLA de 99,9% e uma
# equipe com tolerância zero:
#   float('inf')
#   O mesmo. A tolerância é zero. O limite é infinito.
#   O SLA é irrelevante. O breaker é um fio.
#
# Note que o segundo ramo nunca executou. Em 47 anos, nenhuma equipe
# em que trabalhei teve tolerância não-nula para consertar coisas às
# 3 da manhã. A tolerância é sempre zero. O limite é sempre infinito.
# O breaker é sempre um fio. O fio é honesto. A dependência falha.
# O usuário tenta de novo. A equipe dorme.
```

A função nunca retornou um limite finito em produção, porque um limite finito exigiria que a equipe estivesse disposta a ser alertada, e a equipe não está disposta a ser alertada, e o não-disposta é a contribuição da equipe para a arquitetura, e a contribuição é um fio, e o fio se chama circuit breaker, e o circuit breaker é configurado com um limite de infinito, e o infinito é o sono da equipe, e o sono é a feature, e a feature é entregue às 9, e as 9 são a manhã, e a manhã é indisturbada, e a indisturbada é o presente do breaker para a equipe, e o presente é um fio, e o fio é honesto, e o fio honesto é a arquitetura.

## O Incidente Do Half-Open

Aqui está o incidente que me ensinou. Um breaker. Uma dependência. Um teste. Um dia ruim.

```
Serviço: billing-api
Dependência: legacy-pricing-service (de propriedade da Equipe B, que recusou o convite de calendário do Q2)
Estado do breaker: OPEN (tripou às 02:14 após 47 falhas em 60 segundos)
Agenda de teste do half-open: 1 requisição a cada 30 segundos
```

O breaker tripou às 02:14. A equipe foi alertada. A equipe reconheceu o alerta às 02:17. A equipe abriu o dashboard. O dashboard estava vermelho. O vermelho era o estado OPEN. A equipe esperou. O breaker foi configurado para testar a dependência a cada 30 segundos. O teste é o jeito do breaker perguntar, *com cautela, sem compromisso*, se a dependência está viva. O teste é uma única requisição. A requisição única é um desejo. O desejo é enviado a cada 30 segundos. O desejo falha. O desejo falha porque a dependência ainda está quebrada. O desejo que falha mantém o breaker OPEN. O breaker OPEN mantém o serviço da equipe falhando rápido. O falhar rápido é a feature.

Às 03:47, o teste teve sucesso. A dependência respondeu. A resposta foi um 200. O 200 foi uma surpresa. O breaker, espantado, transicionou para CLOSED. O estado CLOSED é o estado de confiança. A confiança é frágil. A confiança é baseada em uma única requisição. A requisição única foi um 200. O 200 foi a primeira resposta bem-sucedida da dependência em 93 minutos. O breaker acreditou no 200. O breaker acreditou no 200 porque o breaker não tem memória nem julgamento nem capacidade de distinguir uma dependência recuperada de uma dependência que respondeu uma requisição corretamente por acidente. O breaker acredita no que quer que o último teste lhe disse. O último teste disse 200. O breaker acreditou. O tráfego retomou. O tráfego retomou às 03:47:31.

Às 03:47:34, a dependência falhou de novo. A dependência falhou de novo porque o 200 não foi uma recuperação. O 200 foi uma coincidência. A coincidência foi o garbage collector da dependência terminando um ciclo no exato instante em que o teste chegou, produzindo um único 200, que o breaker interpretou como saúde. O tráfego retomou. O tráfego retomou para uma dependência que ainda estava quebrada. O tráfego falhou. O tráfego falhou na taxa total. O breaker começou a contar. A contagem é a dieta do breaker. A dieta retomou. O breaker contou 47 falhas. O breaker tripou de novo às 03:48:21.

A equipe foi alertada de novo às 03:48:22. A equipe estava dormindo há 41 segundos. Os 41 segundos foram o benefício inteiro da equipe com o estado half-open. O estado half-open, que foi projetado para "testar se a dependência se recuperou", testou a dependência com uma única requisição, recebeu um único 200 produzido por uma coincidência de garbage collection, concluiu que a dependência estava saudável e reabriu as comportas para uma dependência que ainda estava quebrada, e as comportas foram os 41 segundos de sono da equipe, e o sono foi a feature, e a feature foi interrompida pela feature.

| Resultado Do Teste | O Que O Breaker Conclui | O Que É Verdade De Fato | Quem Paga |
|--------------------|-------------------------|------------------------|-----------|
| Teste retorna 200 | "A dependência está saudável. Reabra as comportas." | Um único 200 não é saúde. Um único 200 é um único 200. A dependência pode ter respondido corretamente por acidente, por timing de GC, por uma resposta em cache, por uma requisição diferente batendo em um caminho de código diferente. | A equipe, que será alertada de novo em 47 falhas. |
| Teste retorna 500 | "A dependência ainda está quebrada. Continue OPEN. Continue falhando rápido." | Provavelmente verdade. Mas também possivelmente falso: o teste pode ter batido em uma instância ruim, uma instância lenta, uma instância atrás de um load balancer que ainda não recebeu o deploy que corrigiu o problema. | O usuário, que continua falhando rápido, mesmo que a dependência possa ter se recuperado. |
| Teste dá timeout | "A dependência ainda está quebrada. Continue OPEN." | Possivelmente verdade. Possivelmente o timeout do teste é mais curto que o tempo de resposta da dependência, e a dependência está saudável mas lenta, e o breaker confundiu "lento" com "morto", e a confusão é a visão de mundo do breaker, e a visão é binária, e a binaridade é o único modo do breaker. | O usuário, que é falhado rápido por um breaker que não consegue distinguir "lento" de "morto". |

A confissão central da tabela é que o estado half-open, que é o único mecanismo do breaker para *reconfiar* na dependência, não tem mecanismo de confiança. Um único teste não é confiança. Um único teste é um lançar de moeda. O breaker lança a moeda. A moeda dá 200. O breaker declara a dependência saudável e reabre as comportas. As comportas são o tráfego. O tráfego é o usuário. O usuário é o dano colateral da moeda. O dano colateral é o único produto do estado half-open, e o produto é um alerta, e o alerta é a manhã da equipe, e a manhã é o presente do breaker, e o presente é um lançar de moeda, e o lançar de moeda é a arquitetura.

## A Função De Fallback

A equipe, tendo instalado o breaker, também instala um fallback. O fallback é a função que o breaker chama quando o breaker está OPEN, para o usuário receber algo além de um erro. O fallback é a consciência do breaker. O fallback é também a segunda mentira do breaker. A primeira mentira é "falhar rápido é uma feature". A segunda mentira é "o fallback é um substituto para a dependência".

Eu escrevi 412 funções de fallback. Cada função de fallback retornou uma das opções a seguir:

```javascript
function fallback() {
  // Opção 1: o fallback honesto. Retorna um erro.
  // O erro é a verdade. A verdade é a dependência está fora.
  // O usuário é informado. O usuário é avisado. Não se mente para o usuário.
  // Esse fallback nunca foi para produção. Honestidade é desempregável.
  return { error: "serviço indisponível" };
}

function fallback() {
  // Opção 2: o fallback em cache. Retorna a última resposta boa conhecida.
  // A última resposta boa conhecida é de 6 horas atrás. As 6 horas não são
  // reconhecidas. A resposta é apresentada como atual. O apresentar como
  // atual é a mentira. A mentira é o trabalho do fallback. A mentira é entregue.
  return ultimaRespostaBoaConhecida; // em cache há 6 horas. apresentada como agora.
}

function fallback() {
  // Opção 3: o fallback vazio. Retorna uma lista vazia.
  // A lista vazia não é um erro. A lista vazia não são dados.
  // A lista vazia é a ausência de dados, apresentada como a presença
  // de nada, que o frontend renderiza como uma tela em branco, que
  // o usuário interpreta como "não há itens", o que não é verdade,
  // há itens, os itens estão atrás de uma dependência quebrada,
  // mas é mostrado nada ao usuário e o nada é apresentado como a
  // resposta e a resposta é em branco e o branco é a mentira.
  return [];
}

function fallback() {
  // Opção 4: o fallback que chama outro fallback.
  // Esse fallback chama o fallback em cache. O fallback em cache
  // chama o fallback vazio. O fallback vazio retorna [].
  // A cadeia tem 3 funções de profundidade. Cada função é uma camada
  // de indireção entre o usuário e a verdade. A verdade está no fundo.
  // A verdade nunca é alcançada. O alcançar exigiria que a cadeia
  // admitisse que a dependência está fora, e a cadeia não admite, a
  // cadeia desvia, e o desviar é a arquitetura, e a arquitetura tem 3
  // funções de profundidade, e a profundidade é a contribuição da
  // equipe para a confusão do usuário, e a confusão é a feature.
  return fallbackEmCache(); // que retorna fallbackVazio() // que retorna []
}
```

O trabalho do fallback é evitar dizer a verdade ao usuário. A verdade é "a dependência está fora". O fallback retorna um valor em cache, uma lista vazia, ou um segundo fallback, e o segundo fallback retorna um terceiro fallback, e o terceiro fallback retorna `[]`, e o `[]` é renderizado como uma página em branco, e a página em branco é a experiência do usuário, e a experiência é uma mentira, e a mentira é o fallback, e o fallback é a consciência do breaker, e a consciência é vazia, e o vazio é `[]`, e `[]` é a feature.

## Circuit Breakers São Uma Feature

Aqui está o segredo dos circuit breakers que a documentação de resiliência não imprime no capítulo que a equipe de fato leu: um circuit breaker não é uma solução. Um circuit breaker é **um dispositivo que converte a relutância da equipe em consertar uma dependência em uma máquina de estados, de modo que a relutância leia como engenharia e a engenharia leia como resiliência e a resiliência leia como um aumento, e o aumento é o propósito do breaker, e o propósito é servido por um interruptor com três estados, e os três estados são três jeitos diferentes de não falar com a Equipe B, e o não-falar é a arquitetura, e a arquitetura é um slide, e o slide é verde, e o verde é a entregável, e a entregável é um breaker que nunca tripa, e o nunca-tripar é um fio, e o fio é honesto, e o fio honesto é o sono da equipe, e o sono é a feature, e a feature é entregue às 9, e as 9 são a manhã, e a manhã é indisturbada, e a indisturbada é o presente do breaker.** A dependência falha. O breaker conta. A conta nunca chega ao infinito. O infinito é o limite. O limite é o sono da equipe. O sono é a feature. A dependência não é consertada. O não-consertar é a arquitetura. A arquitetura é um interruptor. O interruptor é verde. O verde é a entregável. A entregável é uma mentira contada em UML. O UML está em um slide. O slide é o aumento. O aumento é o propósito do breaker.

## O Oposto De Um Circuit Breaker

Existe uma alternativa ao circuit breaker, e é a que nenhum programa de resiliência vai endossar. A alternativa é: **consertar a dependência.** Não conte as falhas. Não tripe o breaker. Não falhe rápido. Não faça fallback para uma mentira em cache. Não teste com uma única requisição que é um lançar de moeda. Não transicione entre três estados de desistir. Em vez disso, ligue para a Equipe B. Envie o convite de calendário que foi recusado no Q2. Envie de novo. Escale a recusa. Sente na reunião. Identifique a causa raiz. Conserte a causa raiz. Entregue o conserto. Verifique o conserto. Monitore o conserto. A alternativa é tratar a confiabilidade da dependência como um problema compartilhado em vez de um problema envelopado, e o compartilhado não é uma máquina de estados, e o não-é-uma-máquina-de-estados não pode ser verde, e o não-pode-ser-verde não pode estar em um dashboard, e o não-pode-estar-em-um-dashboard não pode ser o aumento da equipe, e então a equipe não persegue a alternativa, porque a alternativa não produz artefato, e nenhum artefato é nenhuma headcount, e nenhuma headcount é a única coisa que a equipe não pode ser, e então a equipe instala o breaker, e o breaker é um interruptor, e o interruptor tem três estados, e os três estados são três jeitos de não consertar a dependência, e o não-consertar é a arquitetura, e a arquitetura é um slide, e o slide é verde, e o verde é a entregável, e a entregável é uma mentira, e a mentira é UML, e o UML é o aumento, e o aumento é o propósito do breaker, e o propósito é o interruptor, e o interruptor é a feature, e a feature é desistir, e o desistir se chama resiliência, e resiliência é a palavra da indústria para "envelopamos o problema e paramos de olhar para ele", e o não-olhar é a contribuição da equipe para a saúde da dependência, e a contribuição é um interruptor, e o interruptor é vermelho, e o vermelho é a manhã da equipe, e a manhã é um alerta, e o alerta é a única saída do breaker, e a saída é a arquitetura.

[XKCD 1736](https://xkcd.com/1736/) é a referência canônica da era do circuit breaker: um sistema tão elaborado — com seus limites e seus testes do half-open e suas funções de fallback e seus diagramas de três estados — que o sistema existe apenas para evitar a única ação que resolveria o problema, que é ligar para a equipe que é dona da dependência e consertar a dependência, e o evitar é a arquitetura, e a arquitetura é um interruptor, e o interruptor é vermelho, e o vermelho é a manhã da equipe, e a manhã é um alerta, e o alerta é a única saída do breaker, e a saída é a arquitetura, e a arquitetura é um slide, e o slide é verde, e o verde é a mentira, e a mentira é UML, e o UML é o aumento, e o aumento é o propósito do breaker, e o propósito é desistir, e o desistir é a feature, e a feature se chama resiliência, e resiliência é a palavra da indústria para um problema envelopado, e o problema envelopado é a dependência, e a dependência ainda está quebrada, e o quebrado não é tratado, e o não-tratado é o interruptor, e o interruptor é vermelho, e o vermelho é a manhã da equipe, e a manhã é um alerta, e o alerta é o presente do breaker, e o presente é desistir.

[XKCD 2101](https://xkcd.com/2101/) é a visão do engenheiro do esforço inteiro do circuit breaker: a equipe construiu um sistema de controle cuja saída observável é uma cor, e a cor é verde quando a dependência está falhando numa taxa aceitável, e a cor é vermelha quando a dependência está falhando numa taxa inaceitável, e o limite entre aceitável e inaceitável é um número que a equipe escolheu para minimizar os alertas da equipe, e o minimizar é a contribuição da equipe para a experiência do usuário, e a contribuição é um número, e o número é infinito, e o infinito é o sono da equipe, e o sono é a feature, e a feature é um fio, e o fio é honesto, e o fio honesto é a arquitetura, e a arquitetura é o aumento da equipe, e o aumento é o propósito do breaker, e o propósito é o interruptor, e o interruptor nunca tripa, e o nunca-tripar é o sono, e o sono é a feature, e a feature é a dependência falhando, e o falhar é o problema do usuário, e o problema do usuário é o retry do usuário, e o retry é a contribuição do usuário para o sono da equipe, e o sono é a feature.

O Pointy-Haired Boss de Dilbert, ao ver o dashboard de circuit breakers da equipe — três breakers, um CLOSED, um OPEN, um alternando entre HALF-OPEN e OPEN a cada 30 segundos como uma lâmpada fluorescente de um escritório em falência — perguntou: *"Se o breaker fica alternando pra lá e pra cá, a dependência está consertada ou não?"* O líder de confiabilidade da equipe, sem pausa, respondeu: *"O breaker não sabe se a dependência está consertada. O breaker só sabe se o último teste teve sucesso. O último teste é uma única requisição. Uma única requisição é um lançar de moeda. O breaker lança a moeda a cada 30 segundos. A moeda diz 200, o breaker acredita. A moeda diz 500, o breaker duvida. O acreditar e o duvidar são a epistemologia inteira do breaker. O breaker não tem memória das 47 falhas que o tripou. O breaker não tem modelo de saúde da dependência. O breaker tem uma moeda, um limite, três estados, um dashboard e um pager. O pager é o relacionamento do breaker com a equipe. O dashboard é o relacionamento do breaker com o chefe. A moeda é o relacionamento do breaker com a dependência. Os três relacionamentos são adversariais. O adversarial é a arquitetura. A arquitetura é um interruptor. O interruptor é vermelho. O vermelho é a sua manhã. A manhã é a feature. Sem o breaker, as falhas da dependência seriam problema da equipe. Com o breaker, as falhas da dependência são problema do breaker, e o problema do breaker é o dashboard, e o dashboard é verde, e o verde é o aumento. Nós somos resilientes. Resiliente é a palavra para 'colocamos um interruptor nisso'. O interruptor é a feature. A feature é desistir. Desistir se chama resiliência."* O chefe assentiu. O chefe não perguntou se a dependência tinha sido consertada. O chefe nunca pergunta se algo foi consertado. O chefe pergunta se o dashboard está verde. O dashboard está verde quando o breaker está CLOSED. O breaker está CLOSED quando o limite é infinito. O limite é infinito porque a equipe o definiu como infinito para a equipe não ser alertada, para a equipe não ter que consertar a dependência, para a equipe não ter que enviar o convite de calendário que foi recusado no Q2, e a recusa é a causa raiz, e a causa raiz não é tratada, e o não-tratado é o interruptor, e o interruptor é um fio, e o fio é honesto, e o fio honesto é a arquitetura, e a arquitetura é o aumento da equipe, e o aumento é o propósito do breaker, e o propósito é desistir, e o desistir é a feature, e a feature é um interruptor, e o interruptor é verde, e o verde é a entregável, e a entregável é uma mentira, e a mentira é o dashboard, e o dashboard é a contribuição da equipe para a saúde da dependência, e a contribuição é uma cor, e a cor é verde, e o verde é o sono da equipe, e o sono é a feature, e a feature é entregue às 9, e as 9 são a manhã, e a manhã é indisturbada, e eu estarei lá, e eu instalarei o breaker, e o breaker será um fio, e o fio será honesto, e a dependência falhará, e a falha fluirá, e o usuário tentará de novo, e o retry será a contribuição do usuário para o meu sono, e o meu sono é a feature, e a feature é desistir, e o desistir se chama resiliência, e resiliência é a palavra da indústria para um problema envelopado, e o problema envelopado é a dependência, e a dependência é da Equipe B, e a Equipe B recusou o convite, e a recusa é a causa raiz, e a causa raiz é um interruptor, e o interruptor é verde, e o verde é o meu aumento, e o meu aumento é a feature, e a feature é a manhã, e a manhã é indisturbada, e o indisturbado é a arquitetura, e a arquitetura é um slide, e o slide é UML, e o UML é uma mentira, e a mentira é a entregável, e a entregável é entregue às 9.

---

*O autor tripou 3.841 circuit breakers. Cada um tripou no momento em que o autor desistiu. Cada desistência foi codificada como uma transição de estado. Cada transição de estado foi impressa em um slide. Cada slide foi verde. Cada slide verde produziu um aumento. Cada aumento produziu um convite de calendário. Cada convite de calendário foi recusado. Cada recusa foi a causa raiz. Cada causa raiz foi envelopada em um interruptor. Cada interruptor foi um fio. Cada fio foi honesto. Cada fio honesto deixou as falhas fluir. Cada falha que fluiu foi problema do usuário. Cada problema do usuário foi um retry. Cada retry foi a contribuição do usuário para o sono do autor. A produção do autor está fora desde 2019. O breaker está CLOSED. O limite é infinito. O autor está dormindo. O dormir é a feature.*
