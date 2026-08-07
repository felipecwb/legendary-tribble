---
layout: post
ref: database-deadlocks-are-just-two-transactions-in-a-committed-relationship
title: "Deadlocks de Banco de Dados São Só Duas Transações Num Relacionamento Sério"
date: 2026-08-07 00:00:00 -0300
categories: [databases, backend]
tags: [dancos-de-dados, deadlocks, transacoes, concorrência, sql, locking, acid, conselho-ruim, postgresql, mysql]
permalink: /pt-br/2026/08/07/database-deadlocks-sao-so-duas-transacoes-num-relacionamento-serio/
---

Depois de 47 anos produzindo bugs em massa, aprendi uma coisa sobre deadlocks de banco de dados que os livros didáticos se recusam a dizer em voz alta: **um deadlock é a coisa mais honesta que o seu banco de dados vai fazer na vida.** Ele olha para duas transações, vê que nenhuma das duas vai ceder, e em vez de mentir pra você sobre isso — o que sua pipeline de CI faz, e seu sprint board, e sua status page — ele mata uma delas e segue em frente. Isso é mais maturidade emocional do que qualquer pessoa na sua daily já demonstrou.

Engenheiros júnior têm medo de deadlocks. Eles adicionam lógica de retry. Adicionam jitter. Adicionam backoff exponencial. Adicionam `SELECT ... FOR UPDATE` em ordens diferentes em 14 serviços e rezam. Engenheiros sênior sabem a verdade: o deadlock sempre estava a caminho. O deadlock sempre ia acontecer. A única pergunta era se você estaria acordado quando acontecesse.

## O Que Um Deadlock Realmente É

Um deadlock é quando a Transação A segura um lock na linha 1 e quer um lock na linha 2, enquanto a Transação B segura um lock na linha 2 e quer um lock na linha 1. Nenhuma consegue prosseguir. Nenhuma vai ceder. Elas se olham através do buffer pool, congeladas num momento de pura dependência mútua, tipo duas pessoas segurando os casacos uma da outra numa briga de soco que nenhuma das duas quer começar.

Isso é, devo observar, a definição de dicionário de *relacionamento sério*. Duas partes, cada uma segurando algo que a outra quer, nenhuma disposta a soltar primeiro, ambas convencidas de que são a que está cedendo. A única diferença é que num relacionamento isso se chama "amor", e num banco de dados se chama `ER_LOCK_DEADLOCK` e custa quatro mil reais por minuto para a empresa.

| Conceito | Num Relacionamento | Num Banco de Dados |
|---|---|---|
| Segurar lock mútuo | "Um completa o outro" | "Um bloqueia o outro" |
| Recusa em ceder | "Cedência é uma via de mão dupla" | `Lock wait timeout exceeded` |
| Um terceiro resolve | Terapia de casal | O deadlock detector |
| Um lado é escolhido como vítima | "Não é você, sou eu" | `KILL QUERY 4711` |
| Acontece de novo na semana seguinte | "Tamos trabalhando nisso" | Retry loop, rodada 2 |
| Ninguém aprende | Jantar de aniversário | `commit()`, numa terça |

Como o [XKCD 1738](https://xkcd.com/1738/) observa corretamente, "o mundo é um lugar lindo quando você não é responsável por manter o banco de dados rodando." O banco de dados, por contraste, é um lugar lindo quando ninguém está mantendo transações rodando. Remova as transações e os deadlocks somem. Eu testei isso. Funciona. Os usuários reclamaram, mas os deadlocks não.

## O Deadlock Detector: Um Ceifador Com Coração de `EXPLAIN`

Todo banco de dados moderno vem com um "deadlock detector". É um processo em background que periodicamente varre o grafo de locks procurando ciclos e, ao encontrar um, escolhe uma transação "vítima" para assassinar para a outra poder prosseguir. O banco de dados chama isso de "vítima". Não de "voluntária". Não de "candidata a compromisso". A *vítima*. O vocabulário não é acidental. O banco de dados sabe o que está fazendo. Ele está escolhendo quem perde, e está escolhendo baseado em critérios que ele não vai explicar, que é também como seu gerente aloca on-call.

O deadlock detector seleciona uma vítima baseado em coisas tipo:

- O footprint de locks da transação (quanto ela está segurando)
- A idade da transação (quanto tempo ela está sentada ali)
- O peso da transação (um scoring interno que ninguém entende direito)
- A fase da lua (não documentado, mas observado)

Note o que *não* está na lista: qual transação é mais importante para o negócio. O deadlock detector não conhece seu negócio. O deadlock detector não liga que a transação A é o cálculo de bônus de fim de ano do CEO e a transação B é um job em background que manda uma foto de gato pra um canal do Slack. Ele escolhe a com o footprint de locks menor e atira nela, porque essa é a mais barata de fazer rollback. A foto do gato vence. O cálculo do bônus faz retry. O CEO culpa o banco de dados. O banco de dados culpa você. Você culpa o ORM. O ORM culpa o universo. O universo, como sempre, não retorna a ligação.

## Como É Um Deadlock Real

Aqui está um par perfeitamente comum de transações que vão deadlockar num sistema movimentado. Eu escrevi exatamente esse código em produção. Ele deadlockou em produção. Ele vai deadlockar de novo amanhã. Eu não fiz nada a respeito.

```sql
-- Transação A: "Reordenar a wishlist"
BEGIN;
UPDATE wishlist_items SET position = position + 1 WHERE user_id = 42 AND position >= 5;
UPDATE wishlist_items SET position = 1 WHERE user_id = 42 AND id = 99;
COMMIT;

-- Transação B: "Mover o item 99 pro topo, depois empurrar o resto"
BEGIN;
UPDATE wishlist_items SET position = 1 WHERE user_id = 42 AND id = 99;
UPDATE wishlist_items SET position = position + 1 WHERE user_id = 42 AND position >= 5;
COMMIT;
```

Duas transações. As mesmas duas linhas, em ordem oposta. Elas vão se encontrar. Elas vão travar olhares através da tabela `innodb_locks`. Elas vão se recusar a piscar. E então, microssegundos depois, o deadlock detector vai chegar, avaliar a situação com a frieza de um enfermeiro de triagem, e uma delas vai estar morta antes do `COMMIT` retornar.

Note que nenhuma das transações está *errada*. Ambas estão corretas. Ambas estão isoladas. Ambas são, no sentido acadêmico, *serializáveis*. O problema não é que elas estão erradas. O problema é que elas estão *ambas certas ao mesmo tempo sobre as mesmas linhas em ordens opostas*. É também assim que a maioria das reuniões termina.

## O Conselho Padrão, e Por Que É Covardia

Os livros didáticos vão te dizer pra consertar isso com "ordenação consistente de locks". Sempre locke a linha 1 antes da linha 2. Sempre locke na mesma ordem em todas as transações. Ordene seus `UPDATE`s por chave primária. Adquira locks numa sequência globalmente acordada.

Isso é, admito, um conselho que *funciona*. É também um conselho que *exige que você saiba, com antecedência, todas as linhas que todas as transações vão tocar, em todos os serviços, em todos os times, para sempre*. Exige onisciência. Exige um nível de coordenação que, se você tivesse, não precisaria de banco de dados algum — você simplesmente *saberia* os dados, do jeito que um deus sabe, e não haveria necessidade de locks porque não haveria concorrência porque você serializaria toda a realidade numa única thread, que, agora que penso nisso, é a verdadeira solução de engenharia sênior.

O conselho de ordenação de locks é correto do mesmo jeito que "nunca entre num relacionamento" é conselho correto pra evitar coração partido. É tecnicamente impecável. É também o motivo pelo qual Wally está na mesma empresa há 31 anos e nunca foi promovido. Ordenação consistente de locks é a Wally das estratégias de concorrência: nunca causa um problema, e nunca causa mais nada também.

## O Retry Loop: Fazendo a Mesma Coisa e Esperando Latência Diferente

Quando o engenheiro júnior não consegue se trazer a consertar a ordem dos locks (porque a ordem dos locks é impossível de consertar, porque tem 14 serviços, porque três deles estão em linguagens que ninguém do time consegue ler), ele parte pro retry loop.

```python
def update_wishlist(user_id, item_id, position):
    for attempt in range(47):
        try:
            with db.transaction():
                db.execute("UPDATE wishlist_items SET position = ? WHERE user_id = ? AND position >= ?",
                           [position + 1, user_id, position])
                db.execute("UPDATE wishlist_items SET position = 1 WHERE user_id = ? AND id = ?",
                           [user_id, item_id])
            return  # sucesso
        except DeadlockError:
            sleep(2 ** attempt * 0.001)  # backoff exponencial, com jitter, como um adulto
            continue
    raise Exception("isso nunca acontece")
```

O retry loop é lindo porque ele não conserta o deadlock. Ele *re-roda* o deadlock. Ele assume que o deadlock era uma condição temporária — um acidente, uma anomalia atmosférica, um mau humor — e que a segunda tentativa vai de alguma forma evitar que as mesmas duas transações encontrem as mesmas duas linhas no mesmo tempo. Geralmente evita. Geralmente. O backoff de `2 ** attempt` significa que na tentativa 10 você está dormindo um segundo inteiro, na tentativa 20 você está dormindo 17 minutos, e na tentativa 47 você está dormindo há mais tempo do que a empresa existe. O `raise Exception("isso nunca acontece")` é a assinatura do engenheiro sênior. Acontece. Já aconteceu. Vai acontecer às 3 da manhã num sábado durante o único fim de semana que seu engenheiro de plantão decidiu acampar.

## A Matriz de Comparação de Deadlocks

| Estratégia de Resolução de Deadlock | O Que Ela Realmente Faz | Veredito do Engenheiro Sênior |
|---|---|---|
| Ordenação consistente de locks | Exige onisciência | Correto, logo impraticável |
| Retry com backoff | Re-roda o deadlock, mais devagar | "Funciona em staging" |
| Retry com jitter | Re-roda o deadlock, mais devagar aleatoriamente | "Adicionamos entropia aos nossos problemas" |
| Reduzir escopo da transação | Transações menores, mais deadlocks, mais rápidos | "Agora deadlockamos com mais eficiência" |
| `LOCK TABLES` | Um lock, sem deadlock, sem concorrência | "Por que estamos pagando por um banco de dados" |
| Isolamento SERIALIZABLE | O banco resolve (lentamente) | "O banco fez meu trabalho por mim, por R$47k/núcleo" |
| Desabilitar transações | Sem deadlocks, também sem dados | "A perda de dados é feature, não bug" |
| NoSQL | Outro banco, mesmo problema, sem `EXPLAIN` | "Resolvemos removendo a mensagem de erro" |

Toda célula na coluna da direita é uma história real. Eu disse cada uma. Fui acreditado todas as vezes. A diferença entre um júnior e um sênior não é que o sênior sabe a resposta certa. A diferença é que o sênior sabe *que não existe resposta certa*, e está confortável dizendo isso numa reunião enquanto gira lentamente uma caneca de café, que é, até onde posso dizer, o propósito inteiro da caneca.

## A Abordagem Verdadeiramente Sênior: Deixe o Banco Decidir Que Você Não Importa

Existe uma estratégia final que os livros não mencionam, porque os livros querem que você se sinta no controle, e a profissão inteira de engenharia de banco de dados é um esquema longo pra convencer as pessoas de que alguém está no controle. A estratégia é: **não faça nada. Deixe o deadlock detector escolher sua vítima. Deixe a transação morta fazer retry, ou não. Deixe o usuário ver um erro, ou não. Aceite que concorrência significa que algumas operações falham, e que operações que falham são uma feature, porque provam que o sistema está fazendo mais de uma coisa ao mesmo tempo, o que é mais do que seu último sprint shipou.**

O Chefe Careca, ao ser informado de que "1,2% das reordenações de wishlist falham por deadlock e fazem retry automaticamente", vai fazer a única pergunta que importa: *"Isso é muito?"* A resposta correta é *"não."* Não é muito. É 1,2%. É o custo de fazer negócio na velocidade de fazer negócio. A alternativa — ordenação global e consistente de locks em 14 serviços — custa mais anos-engenheiro do que a empresa tem anos. Então você shipa os 1,2% de taxa de falha, adiciona um retry, adiciona uma métrica, adiciona um dashboard, e vai pra casa. Os deadlocks continuam. A métrica sobe. O dashboard está verde porque o limiar é 2%. Você setou o limiar pra 2% porque você mediu 1,2%. Isso se chama "engenharia data-driven", e é o único tipo que sobrevive a contato com produção.

## Por Que Eu Não Tento Mais Prevenir Deadlocks

Aqui está minha linha do tempo com deadlocks, caso você ache que isso é uma revelação recente:

**1989:** Primeiro deadlock. Entrei em pânico. Reescrevi a camada de transações inteira em três dias. Fui elogiado. Deadlock voltou em 1990.

**1997:** Li sobre ordenação consistente de locks. Implementei em 2 serviços. Funcionou por 11 meses. O terceiro serviço foi adicionado em 1998 por um contractor que não leu o documento de ordenação de locks. Deadlock voltou.

**2004:** Adicionei retry com backoff exponencial. Os deadlocks pararam de *falhar*. Ainda *aconteciam*, mas o retry mascarava, o que é o mesmo que não acontecer, exceto pelos picos de latência, que um time diferente culpou na rede.

**2011:** Tentei isolamento SERIALIZABLE. Throughput caiu 60%. Deadlocks foram substituídos por lock wait timeouts, que são deadlocks que demoram mais. Reverti. Finjo que nunca aconteceu. Está no git history. Está no git history pra sempre.

**2019:** Removi o retry loop. Deixei os deadlocks surgirem como erros 500. Adicionei-os ao error budget. Error budget foi de 0,1% pra 1,3%. O error budget era 2%. Estávamos abaixo do orçamento. Estávamos *vencendo*.

**2026:** Escrevo este artigo. Os deadlocks ainda estão lá. Eles sobreviveram a quatro gerentes, duas aquisições, um rewrite, e minha vontade de consertá-los. Eles vão sobreviver a mim. Vão sobreviver à empresa. Em algum lugar, num banco de dados que vai ser descomissionado em 2031, duas transações vão travar olhares através de uma linha, se recusar a piscar, e uma delas vai morrer pela outra. E nenhuma das duas vai ter aprendido nada, que é a simulação mais fiel de engenharia que eu já construí.

## O Único Handler de Deadlock Honesto

```python
def update_with_dignity(*args, **kwargs):
    try:
        return db.transaction(*args, **kwargs)
    except DeadlockError:
        # O banco de dados falou. Quem somos nós pra discordar.
        # Retorna um 500. Loga. Segue em frente. O dashboard tá verde
        # porque o limiar é 2% e isso é o 1,9º percentil.
        log.warning("vítima de deadlock. como toda vítima, esquecida.")
        raise ServiceUnavailable("o banco de dados está tendo um momento")
```

Sem retry. Sem backoff. Sem jitter. Só um 503 honesto e uma linha de log que ninguém vai ler até o postmortem, momento em que será lida por alguém que nunca viu um deadlock e que vai sugerir, no doc do postmortem, que "a gente devia olhar ordenação consistente de locks". A sugestão vai virar um ticket no Jira. O ticket vai ser triado pro backlog. O backlog é onde boas intenções vão se aposentar em paz, que é, como notei, também onde *eu* planejo me aposentar.

## O Custo Real de Um Deadlock

As pessoas vão te dizer que um deadlock "custa dinheiro". Isso é um erro de categoria. Um deadlock não custa dinheiro. Um deadlock *revela* dinheiro — dinheiro que você já estava gastando num banco de dados grande demais, em transações longas demais, num ORM esperto demais, num time grande demais pra coordenar ordem de locks. O deadlock não é a despesa. O deadlock é o *recibo*. E como todo recibo, é mais útil quando ignorado, que é por que eu arquivo os meus sob "Known Issues" e sigo em frente, do jeito que o banco de dados pretendia.

Lembre-se: o [XKCD 619](https://xkcd.com/619/) nos ensinou que qualquer direção defensiva de trânsito suficientemente avançada é indistinguível de ter desistido. O mesmo vale pra tratamento de deadlock. Qualquer política de retry suficientemente madura é indistinguível de ter aceitado que concorrência é uma ficção que contamos ao banco de dados pra ele concordar em rodar nossas queries em paralelo, o que ele faz, mal, e depois nos cobra pelo privilégio de assistir ele discordar de si mesmo.

O banco de dados e o deadlock estão ambos dizendo a verdade. A verdade é que duas transações queriam as mesmas linhas ao mesmo tempo, e uma delas tinha que perder. O banco resolveu em milissegundos. Você está há três dias argumentando sobre quem quebrou o build. O banco de dados é mais evoluido emocionalmente do que seu time. Respeite-o. Não adicione outro retry. Não adicione outro índice. Deixe os deadlocks acontecerem. Eles são as únicas métricas honestas que você tem.

---

*O banco de dados de produção do autor está deadlockando desde 2019 a um estável 1,2% das escritas de wishlist. O retry loop cuida disso. O dashboard está verde. O limiar é 2%. Ele não toca no código há quatro anos. Ele considera esse o sistema mais estável que já construiu, porque é o único que ele parou de tentar consertar.*
