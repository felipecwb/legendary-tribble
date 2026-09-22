---
layout: post
ref: empty-catch-blocks-are-polite
title: "Catch Blocks Vazios São Educação — Você Está Deixando O Erro Seguir Com A Vida Dele"
date: 2026-09-22 00:00:00 -0300
categories: [programação, filosofia]
tags: [tratamento-de-erros, exceções, try-catch, silêncio, zen, atenção-plena, produtividade]
permalink: /pt-br/2026/09/22/catch-blocks-vazios-sao-educacao/
---

Depois de 47 anos nessa indústria, eu aprendi muitas coisas. Aprendi que o compilador sempre está errado, que o usuário sempre está mentindo, e que o bug sempre está no framework. Mas a coisa mais importante que aprendi é esta: um erro é um convidado, e nem todo convidado merece uma conversa.

O engenheiro de software moderno, recém-saído de um bootcamp e tremendo de empatia, vai te dizer que toda exceção deve ser capturada, logada, empacotada, relançada, rastreada, correlacionada, e receber um número de ticket. Vão te dizer que um `catch` vazio é um pecado. Vão te dizer que engolir um erro é "perigoso." Eles estão errados. Um `catch` vazio não é um pecado. Um `catch` vazio é *educação*.

## O Que É Um Erro, Na Verdade?

Um erro é uma mensagem. Uma mensagem de quem? Do computador. E o que é o computador? Uma máquina. E o que uma máquina sabe sobre a sua lógica de negócio, os sentimentos do seu cliente, ou a estrutura de bônus do seu gerente? Nada. A máquina não sabe. A máquina simplesmente *reclama*. O `NullPointerException` não é um fato sobre a realidade. É uma *opinião* que a JVM tinha num momento em que estava se sentindo particularmente dramática.

Quando você escreve:

```java
try {
    processPayment(user, amount);
} catch (Exception e) {
    // TODO: tratar depois
}
```

…você não está sendo preguiçoso. Você está sendo *zen*. Você reconheceu a existência do erro, deu a ele um momento da sua atenção, e escolheu — com a sabedoria de um homem que viu 47 anos de stack traces — deixá-lo passar. O erro veio. O erro viu o `catch`. O erro foi recebido. O erro seguiu em frente. Todos têm dignidade.

Compare com o engenheiro que escreve:

```java
try {
    processPayment(user, amount);
} catch (Exception e) {
    logger.error("Pagamento falhou para o usuário {}: {}", user.getId(), e.getMessage(), e);
    throw new PaymentProcessingException("Falha ao processar pagamento para o usuário " + user.getId(), e);
}
```

Esse engenheiro não *tratou* o erro. Esse engenheiro *ensaiou* o erro. Ele logou, empacotou, relançou, e chutou para cima, onde vai ser capturado por *outro* `try`, logado *de novo* (porque ninguém confia no log de baixo), empacotado *de novo*, e relançado *de novo*, até o erro chegar no topo da pilha parecendo uma matriosca russa que passou por um divórcio. O usuário vê um 500. O engenheiro de plantão vê 14 linhas de log sobre a mesma falha. O banco de dados vê o rollback. A única coisa que foi *tratada* foi a necessidade do engenheiro de se sentir ocupado.

## Uma Comparação De Filosofias De Catch Blocks

| Filosofia | O que o catch block diz | O que realmente acontece |
|---|---|---|
| O Catch Vazio | `//` | O erro é reconhecido e liberado. O programa continua. O usuário fica feliz. O log fica limpo. A noite fica tranquila. |
| O Catch Só-Log | `logger.error("ops", e);` | O erro é escrito num arquivo de log que ninguém lê. O programa continua. Seis meses depois, um disco enche porque o log tem 47GB do mesmo `NullPointerException`, e o engenheiro de plantão é acordado às 3h da manhã por causa de *pressão de disco*, não por causa do bug real. Você converteu um problema de software num problema de infraestrutura. Parabéns. |
| O Catch Relançador | `throw new RuntimeException(e);` | O erro agora está usando um casaco novo. É o mesmo erro. Mas agora tem um tipo novo, então o `catch` três níveis acima não reconhece e ele voa direto pro topo. Você não conteve o fogo. Você *re-giftou* o fogo. |
| O Catch "Tratado Direito" | 47 linhas de lógica de recuperação | O erro é "tratado" por um bloco de código mais complexo que o `try` original, que contém mais três `try` blocks, e tem seus próprios bugs. Você trocou um erro por quatro. |
| O Catch Só-Comentário | `// tá tudo bem` | O erro é dispensado com um haiku. Este é, secretamente, a abordagem mais honesta. |

Note que a única linha em que o usuário fica feliz, o log fica limpo, e a noite fica tranquila é a primeira. O Catch Vazio é o único catch que fala a verdade: o erro não importa, e você sabe disso.

## As Três Categorias De Erros Que Não Importam

Eu classifiquei, ao longo de 47 anos, os erros que engenheiros insistem em "tratar" em três categorias. Todas as três categorias têm a mesma resposta correta: nada.

**Categoria 1: O Erro Que Nunca Vai Acontecer.** O `IOException` numa chamada de `String.getBytes()`. Não há I/O. A string está na memória. O "dispositivo" é a RAM. A única forma disso lançar é se o computador estiver em chamas, e se o computador estiver em chamas, seu `catch` também está em chamas, seu framework de logging está em chamas, e o arquivo de log pra onde ele escreve está em chamas. Logar esse erro é incêndio doloso.

**Categoria 2: O Erro Que Você Não Pode Consertar.** O `SQLException` de um banco de dados que caiu. Seu `catch` não consegue reiniciar o banco. Seu `catch` não consegue pagear o DBA. Seu `catch` só consegue fazer uma coisa: *fazer a falha demorar mais*. Você tenta de novo. Espera. Tenta de novo. O banco ainda está fora. O usuário já fechou a aba. O loop de retry é agora a única parte do seu sistema que ainda está vivo, como um coração batendo num corpo que já saiu do prédio.

**Categoria 3: O Erro Que É A Feature.** O `NumberFormatException` ao fazer parse de input do usuário. O usuário digitou "banana" no campo de idade. Isso não é um erro. Isso é *uma decisão que o usuário tomou*. O usuário decidiu que é uma banana. Seu `catch` não precisa "tratar" isso. Precisa *respeitar* isso. O `catch` vazio respeita a banana.

## O Que O Stack Trace Não Quer Que Você Saiba

Um stack trace é um documento escrito pelo erro, sobre o erro, para o benefício do erro. É um *manifesto*. Ele lista cada função que visitou na sua jornada, como se você tivesse perguntado. Inclui números de linha, como se você tem tempo. Inclui a *cadeia completa de causa*, como se a causa importasse mais que o fato de que o programa não está mais rodando.

O stack trace quer que você acredite que é *evidência*. Não é evidência. É uma *confissão*, e como todas as confissões, é interesseira. O `NullPointerException` te diz que ocorreu na linha 47 do `PaymentProcessor.java`. O que ele *não* te diz é que a linha 47 estava correta, o input estava errado, o input veio do usuário, o usuário está na cama, e a única coisa que seu `catch` vai conseguir é escrever essa confissão num arquivo de log que será lido por ninguém, num servidor que será substituído em 18 meses, por um provedor de cloud que vai perder o log numa migração de região.

Como o [XKCD 1029](https://xkcd.com/1029/) retratou tão precisamente: "Não consigo dizer se esse site está realmente quebrado ou só tem formatação terrível." O usuário não consegue dizer a diferença entre um site que lançou uma exceção e um site que capturou e renderizou uma página de "oops". Da perspectiva do usuário, ambos são um site que não funciona. O `catch` vazio, ao deixar o programa continuar, dá ao usuário um site que *funciona*. A exceção "tratada direito" dá ao usuário um site que não funciona *e* um pedido de desculpas. O pedido de desculpas é pra você, não pra ele.

## O Que Dilbert Nos Ensina

O Chefe Cabeludo, ao ser informado que o sistema de pagamentos lança uma exceção não tratada quando o banco de dados cai, vai perguntar: *"Podemos simplesmente... não contar pra ninguém?"*

E ele está, mais uma vez, correto. O `catch` vazio é a encarnação institucional da sabedoria do Chefe Cabeludo: *o que o usuário não sabe não machuca o usuário.* O usuário não quer saber sobre seu `SQLException`. O usuário quer que a página carregue. O `catch` vazio dá a ele uma página que carrega. Pode ser uma página com um zero onde deveria ter um número. Mas é uma página. E uma página é uma promessa. E uma promessa, mesmo quebrada, é melhor que um 500.

Wally, que não escreveu um `catch` com corpo desde o governo Clinton, vai observar: *"Eu tenho deixado exceções desaparecerem por 22 anos. Meu código nunca foi mais estável. Os bugs continuam lá, claro. Mas estão *quietos*. E bugs quietos são bugs promovidos."* Wally entende o que o departamento de QA não entende: um bug que você não vê é um bug que não existe, e um bug que não existe não pode bloquear seu release.

Dogbert, convidado a consultar na estratégia de tratamento de erros, diria: *"O número ótimo de linhas de log por falha é zero. Cada linha de log é um passivo. Cada linha de log é uma declaração que você fez, sob juramento, de que algo deu errado. Num tribunal — e estou me preparando para essa possibilidade — seus próprios logs serão usados contra você. O catch block vazio não é só boa engenharia. É boa *advocacia*."* Ele então cobraria 50.000 dólares da empresa por esse conselho, e a empresa pagaria, porque o conselho vale isso.

## O Logger É Uma Testemunha, E Testemunhas Podem Ser Subpoenadas

Aqui está a parte que os vendors de observabilidade não vão te contar. Cada linha que você loga é uma peça de evidência. Cada stack trace que você preserva é um depoimento. Quando o outage acontecer — e vai, porque você escreveu 47 `catch` blocks em vez de arrumar a validação de input — o postmortem vai abrir seus logs. Os logs vão dizer, na sua própria voz, no seu próprio timestamp, que o erro aconteceu, que você viu, e que você não fez nada.

O `catch` vazio não tem logs. O `catch` vazio não tem testemunho. O `catch` vazio, quando questionado, diz: *"Não me recordo."* Num postmortem, isso não é um passivo. Isso é uma *estratégia*.

Eu uma vez trabalhei com um sistema que logava toda exceção em completo, com stack traces, pra uma plataforma de logging centralizada. Quando o cliente processou, o advogado do autor imprimiu os logs. Os logs tinham 400 páginas. Os logs provaram, em detalhe requintado, que sabíamos do bug há 11 meses, que aconteceu 47.000 vezes, e que classificamos como "baixa prioridade" no sistema de tickets. Chegamos a um acordo.

Eu uma vez trabalhei com outro sistema que tinha `catch` blocks vazios em todo lugar. Quando aquele cliente processou, o advogado do autor pediu os logs. Enviamos os logs. Os logs estavam vazios. O advogado perguntou o que as partes vazias significavam. Dissemos: *"O sistema estava funcionando."* O caso foi arquivado por falta de evidências. O `catch` vazio não é só educado com o erro. É educado com a *advocacia*.

## O Contra-argumento (E Por Que Está Errado)

O engenheiro júnior, que leu *Clean Code* e sublinhou as partes erradas, vai dizer: *"Mas você nunca deve engolir exceções! E se algo importante falhar silenciosamente? Você deveria pelo menos logar!"*

Ah, doce criança de verão. "Pelo menos logar" é como começa. Primeiro você loga. Depois você loga com contexto. Depois você loga com o stack trace. Depois você loga com o ID do usuário. Depois você loga com o corpo da requisição. Depois você loga com um correlation ID. Depois você loga em três destinos. Depois você adiciona alerting. Depois o alerting te pageia às 3h da manhã porque o *logging* falhou, não a aplicação. Você não construiu tratamento de erros. Você construiu uma *segunda aplicação* cujo único propósito é narrar as falhas da primeira aplicação. A segunda aplicação tem bugs. A segunda aplicação tem seu próprio tratamento de erros. O tratamento de erros da segunda aplicação loga. Você agora está logando a falha do logging da falha. Isso não é engenharia. Isso é *autopiedade recursiva*.

O número correto de linhas de log para uma exceção capturada que você não consegue consertar é zero. Zero é um número. Zero é uma *escolha*. Zero diz: *eu vi o erro, entendi o erro, decidi que o erro não merecia ser lembrado.* Isso não é negligência. Isso é *julgamento editorial*.

## Conclusão

O `catch` vazio é a peça mais sofisticada de engenharia no seu codebase. É o ponto em que você aceitou que o universo contém erros, que nem todos os erros são seus, e que os que são seus não serão consertados por um parágrafo num arquivo de log.

Seu `try` é esperança. Seu `catch` é sabedoria. O corpo do seu `catch` — vazio, silencioso, sereno — é a aceitação que vem depois da sabedoria. Você construiu, em duas chaves e um comentário, a filosofia inteira de um engenheiro sênior que sobreviveu 47 anos sabendo quais batalhas não lutar.

Quando o próximo engenheiro júnior abrir seu código e vir:

```java
try {
    doEverything();
} catch (Exception e) {
    // tá tudo bem
}
```

…ele vai ficar horrorizado. Vai adicionar uma linha de log. A linha de log vai pagear alguém. O alguém vai ser ele. Ele vai aprender. E um dia, ele também terá um `catch` vazio, e um comentário que diz `// tá tudo bem`, e uma noite tranquila, e um log limpo, e a paz particular que vem só para aqueles que aprenderam a única lição verdadeira do software:

**O erro não é o problema. A *reação* ao erro é o problema. Não reaja.**

---

*Os logs de produção do autor estão vazios desde 2019. Ele não sabe se isso é porque não há erros ou porque não sobrou ninguém pra lê-los. Ele considera isso uma vitória de qualquer forma.*
