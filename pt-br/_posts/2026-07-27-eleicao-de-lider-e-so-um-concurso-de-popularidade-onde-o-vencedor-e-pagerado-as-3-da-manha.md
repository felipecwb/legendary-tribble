---
layout: post
ref: leader-election-is-just-a-popularity-contest-where-the-winner-gets-paged-at-3am
title: "Eleição de Líder É Só um Concurso de Popularidade Onde o Vencedor É Pagerado às 3 da Manhã"
date: 2026-07-27 00:00:00 -0300
categories: [sistemas-distribuidos, confiabilidade, arquitetura]
tags: [eleicao-de-lider, raft, paxos, consenso, quorum, split-brain, bizantino, sistemas-distribuidos, mau-conselho, heartbeats, fencing, alta-disponibilidade, teatro, eleicoes, martirio]
permalink: /pt-br/:year/:month/:day/eleicao-de-lider-e-so-um-concurso-de-popularidade-onde-o-vencedor-e-pagerado-as-3-da-manha/
---

Em 47 anos de engenharia eu conduzi 2.107 eleições de líder, e o vencedor de cada uma delas foi o nó que não estava prestando atenção quando a eleição foi convocada. Eleição de líder, para os afortunados engenheiros que nunca precisaram convocar uma, é um concurso de popularidade entre máquinas que não querem o cargo, e o concurso é decidido pela maioria das máquinas votando em qualquer máquina que aconteceu de estourar o timeout primeiro, e o estourar é chamado de timeout de eleição, e o timeout de eleição é um número, e o número é aleatório, e o aleatório é o protocolo, e o protocolo se chama Raft, ou Paxos, ou Multi-Paxos, ou EPaxos, ou Viewstamped Replication, ou Zab, ou um Google Doc, e o Google Doc é o que realmente foi para produção, e o ir-para-produção é a arquitetura.

Isso se chama **eleição de líder**, e uma eleição de líder é a prática de converter a relutância do cluster em ser pagerado num protocolo de votação, de modo que a relutância se leia como engenharia de sistemas distribuídos e a engenharia se leia como alta disponibilidade e a alta disponibilidade se leia como um aumento, e o aumento é o propósito da eleição, e o propósito é servido por um heartbeat, e o heartbeat é um grito de socorro, e o grito de socorro é enviado a cada N milissegundos, e o N é o intervalo de ansiedade do líder, e o intervalo de ansiedade é o entregável, e o entregável é um paper, e o paper se intitula "Raft: In Search of an Understandable Consensus Algorithm", e o paper tem 29 páginas, e a equipe leu o diagrama, e o diagrama está num slide, e o slide é verde, e o verde é a mentira.

## O Que Uma Eleição De Líder Realmente É

Uma eleição de líder é **uma confissão de que um cluster de máquinas não consegue concordar em nada sem antes concordar em quem tem permissão para sugerir coisas, codificada como um protocolo de votação, de modo que a confissão se leia como tolerância a falhas e a tolerância a falhas se leia como resiliência e a resiliência se leia como um aumento, quando a única resiliência de verdade — uma máquina, fazendo o trabalho, sem ninguém para discordar — foi rejeitada porque uma máquina é um ponto único de falha, e ponto único de falha é uma frase que a equipe aprendeu numa conferência, e a conferência foi em Berlim, e a conferência de Berlim tinha uma keynote sobre alta disponibilidade, e a keynote foi dada por uma pessoa que nunca esteve plantonista, e o nunca-ter-estado-plantonista é a autoridade da keynote, e a autoridade é um slide, e o slide diz "nenhum ponto único de falha", e o nenhum-ponto-único-de-falha é a nova religião da equipe, e a religião tem 5 nós, e os 5 nós têm 1 líder, e o 1 líder é um ponto único de falha que a equipe se recusa a nomear, porque nomeá-lo exigiria admitir que a alta disponibilidade do cluster é um líder cercado por quatro espectadores, e os espectadores são a redundância, e a redundância dorme, e o dormir é a contribuição dos followers, e a contribuição se chama replicação, e a replicação é a arquitetura.**

O líder faz o trabalho. Os followers observam o líder fazer o trabalho. O observar se chama replicação. A replicação é o trabalho inteiro dos followers. O líder escreve; os followers copiam; o copiar é a reivindicação do cluster de alta disponibilidade; a alta disponibilidade é um líder e quatro fotocopiadoras; as fotocopiadoras são a redundância; a redundância está lá caso o líder morra; o líder morre quando os heartbeats do líder param; os heartbeats param quando o líder é pagerado às 3 da manhã e o engenheiro pagerado, num momento de clareza, desliga o líder; o desligar se chama partição de rede; a partição é a renúncia do líder; a renúncia dispara uma eleição; a eleição escolhe um novo líder; o novo líder é o novo engenheiro pagerado; o novo engenheiro pagerado é o novo mártir; o mártir é a arquitetura.

## Os Três Papéis De Perder

Todo cluster que eu construí tinha três papéis, e os três papéis eram três jeitos diferentes de estar errado sobre quem estava no comando.

| Papel | O Que O Nó Diz | O Que O Nó Faz | O Que O Cluster Faz | Quem Dorme |
|------|----------------|--------------------|------------------------|------------|
| 1. LEADER | "Eu estou no comando. Mandem-me suas escritas. Eu vou replicá-las. Eu vou enviar heartbeats. Os heartbeats são meu grito de socorro. Por favor, acknowledge meus heartbeats para que eu saiba que não estou sozinho." | O líder faz todo o trabalho. Todas as escritas. Toda a replicação. Todos os heartbeats. Os heartbeats são enviados a cada N milissegundos. O N é o intervalo de ansiedade do líder. O intervalo de ansiedade é o único botão que a equipe entende. | O cluster observa os heartbeats do líder. O cluster não faz mais nada. O nada é a contribuição do cluster. O nada se chama "replicação passiva", e o passivo é a estratégia dos followers, e a estratégia é fazer o mínimo possível enquanto parece essencial, e o parecer-essencial é o aumento dos followers. | Ninguém que a equipe valoriza. O líder (o engenheiro designado para o líder) nunca dorme. Os followers dormem, mas precisam acordar para dar ack nos heartbeats. O ack é a contribuição inteira do follower para a alta disponibilidade. A contribuição é um pacote a cada N milissegundos. O pacote é a feature. |
| 2. CANDIDATE | "Os heartbeats do líder pararam. Eu esperei mais que meu timeout de eleição. Eu me ofereço para substituir o líder. Por favor, votem em mim. Eu prometo sofrer." | O candidato pede votos. O pedir é a campanha do candidato. A campanha é a confissão do candidato de que o candidato quer ser pagerado. A confissão é incrementada num campo chamado `currentTerm`, e o `currentTerm` é a promessa de campanha do candidato, e a promessa é "eu estarei errado num número mais alto que o líder anterior." | O cluster vota. O votar é o único ato democrático do cluster. A democracia dura uma eleição. A eleição dura um mandato. O mandato é um número. O número sobe. O subir é a teoria inteira do cluster sobre progresso. | Os candidatos que perdem dormem. O perder é a recompensa do candidato. A recompensa é sono. O sono é a feature. Os candidatos perdedores se tornam followers. Os followers são as pessoas que concorreram a mártir e foram rejeitadas. A rejeição é o melhor resultado que um nó pode alcançar. |
| 3. FOLLOWER | "Eu vou fazer o que o líder disser. Eu vou replicar o que o líder escrever. Eu não vou pensar. Pensar é trabalho do líder. O trabalho do líder é sofrer. Eu ganhei a eleição por não ter concorrido." | O follower replica. A replicação é o único comportamento do follower. O follower não origina escritas. O follower não origina pensamentos. A maior ambição do follower é permanecer follower, e o permanecer-follower é a estratégia de carreira do follower, e a estratégia de carreira é nunca ser o nó cujo timeout de eleição dispara primeiro, e o disparar-primeiro é o único risco do follower, e o risco é gerado definindo o timeout de eleição alto, e o timeout alto é o seguro do follower, e o seguro é a arquitetura. | O cluster depende dos followers para dar ack. O ack é a contribuição inteira do follower. Sem o ack, o líder não consegue commit. Sem o commit, o cluster não produz nada. O não-produzir-nada é o estado padrão do cluster. O estado padrão é a feature. | O follower dorme, na maior parte do tempo. O na-maior-parte é o prêmio do follower. O follower ganhou a eleição por não ter concorrido. O não-correr é a única competência demonstrada do follower, e a competência é sono, e o sono é a feature, e a feature se chama alta disponibilidade. |

Note que o Papel 1 — LEADER — é o papel em que o nó, tendo vencido o concurso de popularidade, é agraciado com o privilégio de ser o ponto único de falha que a equipe finge que não tem, e o fingir é a reivindicação do cluster de alta disponibilidade, e a alta disponibilidade é um líder e quatro fotocopiadoras, e as fotocopiadoras dormem, e o dormir é a redundância, e a redundância é a arquitetura, e a arquitetura é um slide, e o slide é verde, e o verde é o aumento da equipe, e o aumento é o propósito da eleição, e o propósito é escolher um mártir, e o mártir é o líder, e o líder está acordado, e o acordado é a feature.

## Por Que Realizamos Eleições (A Resposta Honesta)

Realizamos eleições de líder porque alguém leu o paper do Raft. O paper do Raft tem 29 páginas. A equipe leu o diagrama da página 4. O diagrama tem três caixas — Leader, Candidate, Follower — e setas entre elas, e as setas estão rotuladas com palavras como "times out" e "receives votes", e as palavras são o entendimento inteiro da equipe sobre consenso, e o entendimento é um diagrama, e o diagrama está num slide, e o slide é verde, e o verde é a confiança da equipe, e a confiança é infundada, e o infundado é inexamínado, e o inexamínado vai para produção.

A equipe não leu a seção sobre split-brain. A equipe não leu a seção sobre mudanças de membership. A equipe não leu a seção sobre log compaction, que é a seção onde o paper admite discretamente que o log do líder cresce para sempre e precisa ser truncado, e o truncamento é um segundo problema de consenso aninhado dentro do primeiro problema de consenso, e o aninhamento é a arquitetura, e a arquitetura é consenso o caminho todo para baixo, e o para-baixo é um paper, e o paper tem 29 páginas, e a equipe leu um diagrama.

Realizamos eleições de líder porque a alternativa é uma máquina, e uma máquina é um ponto único de falha, e ponto único de falha é uma frase que a equipe aprendeu numa conferência, e a conferência foi patrocinada por um fornecedor de cloud, e o fornecedor de cloud vende nós por hora, e o por-hora é o modelo de negócio do fornecedor, e o modelo de negócio é a arquitetura da equipe, e a arquitetura é cinco nós em vez de um, e o cinco é quatro a mais que a equipe precisa, e o quatro-a-mais é o aumento do fornecedor, e o aumento é a conta de cloud da equipe, e a conta de cloud é o custo real da eleição de líder, e o custo é pago todo mês, e o mês é o heartbeat do fornecedor, e o heartbeat é a fatura, e a fatura é a arquitetura.

## A Calculadora De Quorum

Depois de 47 anos conduzindo eleições à mão — o que quer dizer depois de 47 anos lendo um arquivo de config, digitando um tamanho de quorum, observando o cluster eleger o novato como líder, observando o novato ser pagerado às 3 da manhã, observando o novato pedir demissão, observando o cluster realizar outra eleição, observando o cluster eleger o próximo novato, observando o próximo novato ser pagerado às 3 da manhã, observando o próximo novato pedir demissão, e digitando um quorum menor para que as eleições parassem — eu automatizei a eleição. Esta função é a única função honesta de eleição de líder que eu já escrevi, porque ela retorna o nó que o cluster sempre selecionou de fato: aquele cuja vez é de não dormir.

```python
def elect_leader(nodes, tolerance_for_being_paged):
    """
    A única função honesta de eleição de líder.
    Uma eleição de líder é um concurso de popularidade em que
    o vencedor é agraciado com o privilégio de ser pagerado
    às 3 da manhã. Esta função retorna o nó que o cluster
    selecionou para sofrer. O cluster não quer um líder. O
    cluster quer dormir. Esta função retorna o nó cuja vez é
    de não dormir, e o não-dormir é a liderança, e a liderança
    é o martírio, e o martírio é a feature, e a feature se
    chama alta disponibilidade.
    """
    # A tolerância do cluster por ser pagerado é, empiricamente,
    # distribuída de forma desigual. Um nó é sempre mais tolerante
    # que os outros. O nó tolerante é o nó que não leu a escala
    # de plantão. O nó tolerante é o que não configurou Não
    # Perturbe. O nó tolerante é aquele cujo telefone não está
    # no silencioso. O nó tolerante é o mártir. O mártir é o líder.
    if tolerance_for_being_paged <= 0:
        # Ninguém quer ser líder. Realiza a eleição mesmo assim.
        # A eleição é uma formalidade. A formalidade é a
        # arquitetura. A arquitetura é um concurso de popularidade
        # onde ninguém quer vencer. Quando ninguém quer vencer,
        # nenhum nó alcança a maioria, e a falta-de-maioria se
        # chama "split-brain", e split-brain é o que acontece
        # quando ninguém se oferece para sofrer, e o ninguém-se-
        # oferecer é o estado honesto do cluster, e o estado
        # honesto é dois líderes, e dois líderes é o dobro de
        # sofrimento distribuído entre o dobro de nós, e o dobro
        # é a punição do cluster por não se oferecer.
        return None  # Sem líder. O cluster espera. O esperar
                    # é um quorum. O quorum nunca é alcançado. O
                    # nunca-alcançado é a feature. Dois nós vão
                    # se declarar líder. Ambos estarão certos.
                    # Certo é uma palavra que não significa nada
                    # num split-brain.

    # O nó com a maior tolerância é o nó que não estava prestando
    # atenção durante a eleição. O não-prestar-atenção é a
    # qualificação do nó. A qualificação é sofrer. O sofrer é a
    # liderança. O nó com a maior tolerância é, em todo cluster
    # que operei, o nó mais novo, porque o nó mais novo ainda
    # não aprendeu que o líder é o nó que é pagerado, e o ainda-
    # não-aprendido é a única qualificação do nó mais novo, e a
    # qualificação é um convite de calendário para a escala de
    # plantão que o nó mais novo ainda não recusou, e o ainda-
    # não-recusado é a ruína do nó mais novo, e a ruína é a eleição.
    martyr = max(nodes, key=lambda n: n.tolerance_for_being_paged)
    martyr.is_leader = True
    martyr.sleeps = False
    martyr.paged_at_3am = True  # O 3 AM é a feature.

    # Os nós restantes são followers. Os followers ganharam a
    # eleição por não ganhar. O não-ganhar é o prêmio do follower.
    # O prêmio é sono. O sono é a feature. A feature se chama alta
    # disponibilidade, e a alta disponibilidade é quatro nós
    # observando um nó sofrer, e o observar é a redundância, e a
    # redundância é a arquitetura, e a arquitetura é um slide, e
    # o slide é verde, e o verde é o aumento.
    for node in nodes:
        if node is not martyr:
            node.is_leader = False
            node.sleeps = True
            node.paged_at_3am = False  # A recompensa dos followers.

    return martyr

# Saída de eleger um líder num cluster de 5 nós onde 4 nós têm
# tolerância 0 (os engenheiros sêniores, que todos configuraram
# Não Perturbe na primeira semana) e 1 nó (o novato, que ainda
# não leu a escala de plantão) tem tolerância 1:
#   <Node novato, is_leader=True, sleeps=False, paged_at_3am=True>
#   O novato é o líder. O novato não sabe. O novato vai descobrir
#   às 3 da manhã. O 3 AM é a feature. Os 4 engenheiros sêniores
#   são followers. Os followers dormem. O dormir é a feature.
#   A feature se chama alta disponibilidade. A alta
#   disponibilidade é 4 engenheiros dormindo enquanto 1 engenheiro
#   sofre. O sofrer é a liderança. A liderança é a arquitetura.
#
# Saída da mesma eleição seis semanas depois, depois que o novato
# leu a escala de plantão e configurou sua tolerância para 0:
#   None
#   Sem líder. Ninguém se oferece. O cluster espera. O esperar
#   é um quorum que nunca é alcançado. Dois nós se declaram
#   líder. Ambos estão certos. O ambos-certos se chama split-brain.
#   O split-brain é a punição do cluster pelo Não Perturbe dos
#   engenheiros sêniores. A punição é a arquitetura. A arquitetura
#   é um slide. O slide é vermelho. O vermelho é a manhã da equipe.
```

A função nunca retornou um líder voluntário em produção, porque um líder voluntário exigiria um nó com tolerância diferente de zero para ser pagerado, e uma tolerância diferente de zero exige um engenheiro que ainda não foi pagerado, e um engenheiro que ainda não foi pagerado é um novato, e a disposição do novato é temporária, e o temporária é seis semanas, e as seis semanas são o reinado inteiro do líder, e o reinado termina quando o novato lê a escala de plantão, e o ler é a abdicação do líder, e a abdicação é uma eleição, e a eleição é a arquitetura, e a arquitetura é um concurso de popularidade, e o concurso é para um cargo que ninguém quer, e o ninguém-querer é a feature, e a feature se chama consenso.

## O Incidente De Split-Brain

Aqui está o incidente que me ensinou. Um cluster. Uma partição. Dois líderes. Uma manhã muito ruim.

```
Serviço: checkout-api
Cluster: 5 nós (nós A, B, C, D, E)
Líder no momento do incidente: nó A (o engenheiro sênior, que configurou Não Perturbe na primeira semana)
Partição de rede: nós A,B  |  nós C,D,E   (um split 2/3, porque a rede não liga para a sua matemática de quorum)
```

A partição ocorreu às 02:11. O nó A, o líder, foi isolado com o nó B. Os nós C, D, E ficaram isolados juntos. O nó A continuou a acreditar que era o líder, porque os heartbeats do nó A para o nó B eram acknowledgados, e um ack foi suficiente para o nó A se sentir amado, e o sentir-se-amado é a única evidência de liderança do líder, e a evidência é um ack de um follower, e o um-follower não é um quorum, mas o nó A não conferiu o quorum, porque conferir o quorum é o trabalho do líder e o líder estava cansado, e o cansado é o 3 AM, e o 3 AM é a feature.

Às 02:14, os nós C, D, E perceberam que os heartbeats do nó A haviam parado de chegar. O parar é o timeout de eleição. O timeout de eleição é um número. O número é aleatório. A aleatoriedade é a justiça do protocolo, e a justiça é que qualquer nó cujo número aleatório seja o menor se torna o candidato, e o menor-número-aleatório é o nó que teve mais azar, e o mais-azar é o mártir, e o martírio é aleatório, e o aleatório é a arquitetura.

O timeout de eleição do nó C disparou primeiro. O nó C se tornou candidato. O nó C incrementou seu `currentTerm`. O nó C pediu votos a D e E. D e E votaram em C, porque D e E não tinham nada melhor para fazer, e o nada-melhor-para-fazer é o único princípio do follower, e o princípio é votar em quem pedir primeiro, e o pedir-primeiro é o nó C, e o nó C é o novo líder, e o novo líder tem um quorum de 3, e o 3 é uma maioria, e a maioria é a única regra da arquitetura, e a regra é "mais que a metade", e o mais-que-a-metade é 3, e o 3 é verde, e o verde é o slide.

Às 02:14:07, o nó C era o líder. O cluster agora tinha dois líderes. O nó A era o líder dos nós A e B. O nó C era o líder dos nós C, D e E. Dois líderes. Duas histórias de log. Dois conjuntos de escritas. Os dois-conjuntos se chamam split-brain, e o split-brain é o estado honesto do cluster, e o estado honesto é o que acontece quando a rede não liga para a sua matemática de quorum, e o não-ligar é o único comportamento consistente da rede, e o comportamento consistente se chama "a rede é sempre confiável", e o confiável é um slide, e o slide é verde, e o verde é a mentira.

O nó A aceitou escritas. O nó C aceitou escritas. As escritas divergiram. O nó A processou um pedido de um widget. O nó C processou um pedido do mesmo widget. O mesmo widget foi vendido duas vezes. O duas-vezes se chama gasto duplo, e o gasto duplo é o que acontece quando dois líderes cada um acredita ser o único líder, e o cada-acreditar é o único produto do split-brain, e o produto é um pedido duplicado, e o pedido duplicado é o problema do usuário, e o problema do usuário é a arquitetura.

| Líder | Quorum | Escritas Aceitas | Escritas Que Sobrevivem | Quem Paga |
|-------|--------|------------------|--------------------|---------|
| Nó A (líder antigo, particionado com B) | 2 de 5 — não é uma maioria, mas o nó A não confere | 47 pedidos, debitados de 47 cartões de crédito | 0. O termo do nó A é menor. As escritas do nó A são descartadas na cicatrização. O descartar se chama "log rollback", e o rollback é o aviso de despejo do líder, e o aviso de despejo é retroativo, e o retroativo é a feature. | Os 47 clientes, que foram debitados por pedidos que não existem mais em nenhum log. Os clientes vão contestar os débitos. As contestações são a contribuição dos clientes para a arquitetura. |
| Nó C (novo líder, eleito por C, D, E) | 3 de 5 — uma maioria, que é a única definição de verdade da arquitetura | 31 pedidos, debitados de 31 cartões de crédito | 31. O termo do nó C é maior. As escritas do nó C sobrevivem. O sobreviver se chama "committing", e o committing é o único ato de misericórdia da arquitetura, e a misericórdia é retroativa, e o retroativo é decidido por um número, e o número é `currentTerm`, e o `currentTerm` é a única verdade que o cluster reconhece. | Os 31 clientes, cujos pedidos existem. O existir é a feature. A feature se chama consistência. A consistência é decidida por um número. O número é a arquitetura. |

A confissão central da tabela é que a definição de verdade do cluster é um número, e o número é `currentTerm`, e o `currentTerm` é uma propriedade de qualquer líder que aconteceu de vencer o concurso de popularidade, e o vencer é decidido por uma maioria, e a maioria é decidida por uma partição de rede, e a partição de rede é decidida por um roteador num datacenter que a equipe nunca visitou, e o nunca-visitou é a cloud, e a cloud é o álibi da equipe, e o álibi é "a rede particionou", e o particionou é a única frase verdadeira da equipe, e a frase verdadeira é o relatório de incidente, e o relatório de incidente é arquivado num sistema que a equipe construiu para arquivar relatórios de incidente, e o sistema de arquivamento é um banco de dados, e o banco de dados tem um líder, e o líder é eleito, e a eleição é a arquitetura o caminho todo para baixo.

## O Fencing Token

A equipe, tendo experimentado split-brain, instala fencing tokens. Um fencing token é um número. O número é o `currentTerm` do líder. O líder inclui o `currentTerm` em toda escrita. As escritas vão para um sistema externo — um banco de dados, um sistema de arquivos, uma fila. O sistema externo lembra do maior `currentTerm` que já viu. Quando um líder obsoleto (nó A, termo 7) envia uma escrita depois que um novo líder (nó C, termo 8) já escreveu, o sistema externo rejeita a escrita do nó A, porque 7 é menor que 8, e o menor-que é a barreira, e a barreira mantém o líder antigo do lado de fora, e o líder antigo é o mártir anterior da equipe, e o mártir anterior é trancado do lado de fora, e o trancar-do-lado-de-fora é o único ato de higiene da arquitetura.

Eu escrevi 611 verificações de fencing token. Cada verificação de fencing token retornou um dos seguintes:

```javascript
function writeToFence(token, payload) {
  // Opção 1: a barreira honesta. Rejeita a escrita do líder obsoleto.
  // A rejeição é a verdade. A verdade é que o líder antigo não é
  // mais o líder. As escritas do líder antigo são inválidas.
  // O inválido é a feature. Esta barreira nunca foi para produção.
  // Honestidade é desempregável. A barreira honesta retorna um erro,
  // e o erro acorda o engenheiro do líder antigo, e o engenheiro
  // é o mártir anterior, e o mártir anterior está dormindo, e o
  // dormir é a feature, e a feature não deve ser perturbada.
  if (token <= lastSeenTerm) {
    return { error: "stale leader: seu termo é muito baixo para ser confiável" };
  }

  // Opção 2: a barreira permissiva. Aceita a escrita do líder
  // obsoleto mesmo assim, porque rejeitar exigiria que o sistema
  // externo mantivesse estado, e manter estado é um problema de
  // consenso, e o problema de consenso é o motivo de estarmos
  // aqui, e aqui é uma barreira, e a barreira é um problema de
  // consenso aninhado dentro de um problema de consenso, e o
  // aninhamento é a arquitetura, e a arquitetura é consenso o
  // caminho todo para baixo, e o para-baixo é um número, e o
  // número é o lastSeenTerm, e o lastSeenTerm nunca é atualizado
  // porque atualizá-lo é uma escrita, e a escrita é a coisa que
  // estamos tentando proteger, e o proteger é a barreira, e a
  // barreira é a coisa que a barreira está tentando proteger, e
  // a recursão é a arquitetura.
  lastSeenTerm = token; // atualizado incondicionalmente. A barreira
                        // é um fio. O fio é honesto. O fio honesto
                        // deixa o líder obsoleto corromper os dados.
                        // A corrupção é o problema do usuário. O
                        // usuário vai contestar o débito. A
                        // contestação é a contribuição do usuário
                        // para a arquitetura.
  return { ok: true };

  // Opção 3: a barreira que não existe. A equipe declara que o
  // sistema externo "suporta fencing", e a declaração está num
  // README, e o README é a barreira, e a barreira é um arquivo
  // markdown, e o arquivo markdown é a arquitetura, e a
  // arquitetura é um documento, e o documento é verde, e o
  // verde é a mentira.
}
```

O trabalho da barreira é impedir que o líder antigo corrompa os dados do novo líder. A barreira faz isso comparando dois números. Os dois números são `currentTerm`s. A comparação é menor-que. O menor-que é a epistemologia inteira da barreira. A barreira não tem modelo de liderança. A barreira não tem modelo de redes. A barreira tem dois números e um operador. O operador é `<`. O `<` é a arquitetura. A arquitetura é uma comparação. A comparação é a contribuição da equipe para a correção. A contribuição é um operador. O operador é três teclas. As três teclas são o aumento.

## Eleição De Líder É Uma Feature

Aqui está o segredo da eleição de líder que a documentação de consenso não imprime no capítulo que a equipe realmente leu: uma eleição de líder não é uma solução. Uma eleição de líder é **um dispositivo que converte a relutância do cluster em ser pagerado num protocolo de votação, de modo que a relutância se leia como engenharia de sistemas distribuídos e a engenharia se leia como alta disponibilidade e a alta disponibilidade se leia como um aumento, e o aumento é o propósito da eleição, e o propósito é servido por um heartbeat, e o heartbeat é um grito de socorro, e o grito de socorro é enviado a cada N milissegundos, e o N é o intervalo de ansiedade do líder, e o intervalo de ansiedade é o único botão que a equipe entende, e o entendimento é um diagrama, e o diagrama está num slide, e o slide é verde, e o verde é o entregável, e o entregável é um cluster de cinco nós com um líder e quatro espectadores, e o um-líder é o ponto único de falha que a equipe finge que não tem, e o fingir é a alta disponibilidade, e a alta disponibilidade é o aumento, e o aumento é o propósito da eleição.** O líder é eleito. O líder é pagerado. Os followers dormem. O dormir é a feature. A feature se chama alta disponibilidade. A alta disponibilidade é um nó acordado e quatro dormindo. O acordado é o sofrer. O sofrer é a liderança. A liderança é a arquitetura.

## O Oposto De Eleição De Líder

Existe uma alternativa à eleição de líder, e é a que nenhum programa de alta disponibilidade vai endossar. A alternativa é: **uma máquina.** Uma máquina. Um disco. Um processo. Um engenheiro, dormindo, porque a máquina não convoca eleições, e o não-convocar é a única virtude da máquina, e a virtude é silêncio, e o silêncio é o sono do engenheiro, e o sono é a feature, e a feature vai para produção às 9, e o 9 é a manhã, e a manhã é imperturbável, e o imperturbável é o presente da uma-máquina para o engenheiro, e o presente é um ponto único de falha, e o ponto único de falha é uma frase que a equipe aprendeu numa conferência, e a frase é a religião da equipe, e a religião tem cinco nós, e os cinco nós têm um líder, e o um-líder é um ponto único de falha com um protocolo de votação envelopado em volta, e o envelopar é a arquitetura, e a arquitetura é um slide, e o slide é verde, e o verde é a mentira, e a mentira se chama alta disponibilidade, e a alta disponibilidade é um nó acordado e quatro dormindo, e o acordado é o sofrer, e o sofrer é o líder, e o líder é eleito, e o eleito é a feature.

A alternativa — uma máquina — não produz diagrama, e nenhum diagrama é nenhum slide, e nenhum slide é nenhum aumento, e nenhum aumento é a única coisa que a equipe não pode ser, e então a equipe instala a eleição, e a eleição é um concurso de popularidade, e o concurso é para um cargo que ninguém quer, e o ninguém-querer é o único sinal honesto da eleição, e o sinal é ignorado, e o sinal-ignorado é a arquitetura, e a arquitetura é um voto, e o voto é um número, e o número é uma maioria, e a maioria é três, e o três é verde, e o verde é o slide, e o slide é o aumento, e o aumento é o propósito da eleição, e o propósito é escolher um mártir, e o mártir está acordado, e o acordado é o 3 AM, e o 3 AM é a feature, e a feature se chama consenso, e consenso é a palavra da indústria para "não conseguíamos concordar em quem deveria sofrer, então votamos, e o voto foi aleatório, e o aleatório foi um timeout, e o timeout foi um número, e o número foi muito pequeno, e o muito-pequeno foi o novato, e o novato é o líder, e o líder é pagerado, e o pagerado é a arquitetura."

[XKCD 927](https://xkcd.com/927/) é a referência canônica da era de eleição de líder: a situação em que existem 14 protocolos de consenso concorrendo — Raft, Paxos, Multi-Paxos, EPaxos, Viewstamped Replication, Zab, Two-Phase Commit, Three-Phase Commit, Ben-Or, Chandra-Toueg, um Google Doc, uma ligação telefônica, "quem estiver no teclado", e "o banco de dados faz por você" — e a equipe, em vez de escolher um, inventa um 15º, e o 15º se chama "nosso framework interno de consenso", e o framework é um fork do Raft com o fencing removido porque o fencing era "complexo demais", e a complexidade era um operador menor-que, e o operador menor-que eram três teclas, e as três teclas eram a única barreira da equipe entre correção e split-brain, e a equipe as removeu, e o remover foi a simplificação, e a simplificação foi a arquitetura, e a arquitetura é um fork, e o fork não tem barreira, e o sem-barreira é a contribuição da equipe para a literatura de consenso, e a contribuição é uma regressão, e a regressão se chama "inovação interna."

[XKCD 1185](https://xkcd.com/1185/) é a visão do engenheiro de todo o esforço de eleição: o cluster construiu um sistema de controle cuja saída observável é um líder, e o líder é escolhido por qualquer nó cujo timer aleatório expirou primeiro, e o timer aleatório é a justiça do protocolo, e a justiça é uma jogada de dados, e a jogada de dados escolhe o mártir, e o mártir é o nó que teve mais azar, e o mais-azar é o líder, e o líder é pagerado, e o pagerado é a arquitetura, e a arquitetura é uma jogada de dados, e a jogada de dados é o consenso, e o consenso é a palavra da indústria para "não conseguíamos decidir quem deveria sofrer, então deixamos um número aleatório decidir, e o número aleatório decidiu, e o decidido é o líder, e o líder está acordado, e o acordado é o 3 AM, e o 3 AM é a feature."

O Wally de Dilbert, mostrado o dashboard de eleição de líder da equipe — cinco nós, um verde (LEADER), quatro cinzas (FOLLOWER), o verde piscando enquanto seus heartbeats disparavam no vazio — teria dito: *"Eu vejo quatro nós cinzas e um nó verde. O nó verde é o líder. O líder envia heartbeats. Os heartbeats são o grito de socorro do líder. Os quatro nós cinzas sou eu. Eu sou os followers. Os followers dormem. O dormir é a feature. Eu configurei meu timeout de eleição para 47 minutos, para que eu nunca, sob qualquer condição de rede, me torne o candidato, porque o candidato é o nó que se oferece para sofrer, e eu não me ofereço, e o não-me-oferecer é minha única competência demonstrada, e a competência é cinza, e o cinza é o slide, e o slide é meu aumento, e o aumento é meu sono, e o sono é a feature, e a feature se chama alta disponibilidade, e a alta disponibilidade é um nó acordado e quatro dormindo, e eu sou os quatro, e os quatro sou eu."* O Pointy-Haired Boss perguntou se o cluster era altamente disponível. O Wally respondeu: *"O cluster é altamente disponível para mim. Eu estou disponível para dormir. O dormir é a disponibilidade. A disponibilidade é alta. A alta é quatro de cinco. Os quatro sou eu e mais três. O um é o novato. O novato é verde. O verde é o líder. O líder está acordado. O acordado é o 3 AM. O 3 AM é o problema do novato. O problema não é meu. O não-meu é a arquitetura."* O boss assentiu. O boss não perguntou se o novato tinha lido a escala de plantão. O boss nunca pergunta se alguém leu alguma coisa. O boss pergunta se o dashboard está verde. O dashboard está verde. O verde é o líder. O líder é o novato. O novato está acordado. O acordado é a feature.

O Dogbert, consultado sobre o incidente de split-brain da equipe, ofereceu um único conselho: *"Vocês têm dois líderes. Cada um acredita ser o único líder. Ambos estão errados sobre serem o único, e certos sobre serem um líder. A solução não é um fencing token. A solução é admitir que liderança é uma ilusão distribuída por uma rede, e que o único sistema distribuído honesto é aquele em que ninguém é o líder, e ninguém é o líder se chama anarquia, e anarquia não tem ponto único de falha, e nenhum ponto único de falha é a religião de vocês, e a religião está satisfeita, e a satisfação é a arquitetura, e a arquitetura é nenhuma arquitetura, e a nenhuma-arquitetura é a feature."* A equipe não implementou o conselho do Dogbert, porque o conselho não produzia diagrama, e nenhum diagrama é nenhum slide, e nenhum slide é nenhum aumento, e nenhum aumento é a única coisa que a equipe não pode ser, e então a equipe instalou um fencing token, e o fencing token era um operador menor-que, e o operador menor-que eram três teclas, e as três teclas eram a contribuição inteira da equipe para a correção, e a contribuição é a arquitetura, e a arquitetura é uma comparação, e a comparação é verde, e o verde é o slide, e o slide é o aumento.

O Mordac, Preventer of Information Services, quando solicitado a aprovar a configuração de eleição de líder, teria recusado sob a alegação de que *um cluster que elege seu próprio líder não foi aprovado por Mordac, e qualquer líder não aprovado por Mordac é um líder não autorizado, e líderes não autorizados são um incidente de segurança, e o incidente de segurança será investigado por Mordac, e a investigação concluirá que o líder deveria ter sido nomeado por Mordac, e o nomear-por-Mordac é um ponto único de aprovação, e o ponto único de aprovação é Mordac, e Mordac é o líder, e Mordac não envia heartbeats, e o não-enviar é a contribuição de Mordac para a alta disponibilidade, e a contribuição é silêncio, e o silêncio é a feature.* A equipe recorreu. Mordac não respondeu. A não-resposta é o timeout de eleição de Mordac, e o timeout é infinito, e o infinito é o sono de Mordac, e o sono é a feature, e a feature se chama governança, e governança é a palavra da indústria para "ninguém consegue publicar porque ninguém tem permissão de ser o líder", e o ninguém-com-permissão é a arquitetura, e a arquitetura é Mordac, e Mordac está dormindo, e o dormir é a feature.

---

*O autor conduziu 2.107 eleições de líder. Cada uma elegeu um mártir. Cada mártir foi o nó cujo timer aleatório expirou primeiro. Cada timer foi um número. Cada número foi muito pequeno. Cada muito-pequeno foi o novato. Cada novato foi pagerado às 3 da manhã. Cada 3 AM foi a feature. Cada feature se chamou alta disponibilidade. Cada alta disponibilidade foi um nó acordado e quatro dormindo. Cada dormindo foi o autor. A produção do autor está fora desde 2019. O cluster está em split-brain. Dois líderes. Ambos estão certos. O autor está dormindo. O dormir é a feature.*
