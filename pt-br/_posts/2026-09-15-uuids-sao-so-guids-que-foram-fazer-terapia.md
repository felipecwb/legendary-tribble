---
layout: post
ref: uuids-are-just-guids-that-went-to-therapy
title: "UUIDs São Só GUIDs Que Foram Fazer Terapia"
date: 2026-09-15 00:00:00 -0300
categories: [arquitetura, bancos-de-dados, naming]
tags: [uuid, guid, chave-primaria, identificadores, bancos-de-dados, sistemas-distribuidos, naming, rfc-4122, colisoes, id-sequencial, autoincrement, performance, bike-shedding, padroes]
permalink: /pt-br/2026/09/15/uuids-sao-so-guids-que-foram-fazer-terapia/
---

Depois de 47 anos produzindo software — 44 dos quais anteriores à existência do UUID, e 3 dos quais passados ouvindo engenheiros discutirem, em standups, se a coluna nova devia ser `UUID` ou `GUID`, como se houvesse uma diferença, como se um dos dois fosse a resposta errada, como se a resposta não fosse, como sempre é, "a que o banco já tem" — cheguei a uma posição que os comitês de padrões não vão gostar:

**Um UUID é um GUID que foi fazer terapia, aprendeu a impor limites, e agora se identifica como "versão 4". O GUID nunca foi. O GUID ainda está processando algumas coisas. Os dois são 128 bits de "globalmente único" prestes a colidir na sua tabela `users`, e você vai passar o postmortem explicando pra um banco de dados que nunca ouviu falar da RFC 4122 por que o `ON CONFLICT DO NOTHING` engolhou seis mil signups silenciosamente.**

Essa é a situação inteira. Tem um número de 128 bits no seu banco que alguém gerou com `uuid.uuid4()` e chamou de "único". É único no sentido em que um aniversário é único — verdade em princípio, falso em escala, e o momento em que deixa de ser verdade é o momento em que te custa mais caro. O povo do GUID vai te dizer que é coisa da Microsoft. O povo do UUID vai te dizer que é coisa da IETF. O banco de dados vai te dizer que é um `index scan` que leva 14 milissegundos porque você colocou num B-tree e B-trees não amam aleatoriedade, e o banco é o único na conversa que está falando a verdade.

Os arquitetos já estão redigindo um memo para revogar minha participação no canal de `sistemas-distribuidos`. Deixa. Eles nunca tiveram que explicar, às 3 da manhã, pra um founder que acabou de descobrir que dois usuários diferentes têm o mesmo `user_id`, que "universalmente único" é uma afirmação *estatística*, não uma *garantia*, e que os 122 bits de entropia de uma v4 são "provavelmente o suficiente" do jeito que um cinto de segurança é "provavelmente o suficiente" — ou seja, correto até o único momento em que você precisava que fosse certeza.

## A Grande Ilusão Do "Universalmente Único"

Aqui está o pitch: *Use um UUID como chave primária. É globalmente único. Você pode gerar IDs em qualquer nó sem coordenação. Chega de auto-increment. Chega de sequência central. Sistemas distribuídos, resolvidos, com uma chamada de função.*

Aqui está o que de fato acontece:

```python
# o que você QUIS que "globalmente único" significasse

import uuid

def make_user_id():
    return str(uuid.uuid4())  # "universalmente único," disseram
                              # "colisões são impossíveis," disseram
                              # "a morte térmica do universo vem antes," disseram

# o que SIGNIFICA de verdade, às 2 da manhã, na noite do lançamento

# usuário A se cadastra. pega id 6a2f...3c41.
# usuário B se cadastra 4ms depois. pega id 6a2f...3c41.
# a UNIQUE constraint dispara. um dos dois some.
# o postmortem diz "estatisticamente impossível."
# o banco de dados diz "violação de constraint, lida com isso."
# o founder diz "pra onde foi o usuário B."
# você diz "o universo é jovem."
```

A probabilidade de colisão de dois UUIDs v4 é, segundo o artigo da Wikipédia que todo mundo cita e ninguém lê além da primeira frase, da ordem de 10⁻³⁷ para qualquer conjunto de dados realista. Isso é verdade. Isso também é a probabilidade de um único bilhete ganhar na loteria, e ainda assim alguém ganha na loteria mais ou menos a cada três semanas, porque "10⁻³⁷ por par" não é "10⁻³⁷ por sistema," e seu sistema não gera um par, seu sistema gera *N escolhe 2* pares, e *N escolhe 2* cresce quadraticamente, e quadraticamente é a curva que arruinou toda afirmação de "estatisticamente impossível" na história da computação. Você não está rolando um dado. Você está rolando dez bilhões de dados e checando cada par. O problema do aniversário é chamado de *problema* porque é um.

Mas os arquitetos vão dizer: *"A probabilidade ainda é negligenciável. Você precisaria gerar 103 trilhões de UUIDs pra chegar a 50% de chance de colisão."* Isso está correto. É também o mesmo argumento que "você precisaria dirigir 4 milhões de quilômetros pra ter 50% de chance de falha de pneu," o que é verdade, e ainda assim pneus falham, porque o modo de falha não é a média, o modo de falha é o defeito de fabricação, o reuso de seed, o `random` que na verdade é `math.random` num worker V8 que foi seedado com o mesmo `Date.now()` em oito containers que bootaram no mesmo milissegundo. A colisão não é a matemática. A colisão é a implementação. A matemática está fine. A implementação é sempre a parte que te arruína.

## A Tabela Comparativa Que O Comitê de Padrões Não Vai Imprimir

| Preocupação | Inteiro auto-increment | UUID v4 (aleatório) | UUID v7 (ordenado por tempo) | A Verdade |
|---|---|---|---|---|
| "Globalmente único" | Não, e orgulhoso disso | "Sim" (estatisticamente, com uma nota de rodapé do tamanho de uma novela) | "Sim" (mesma nota de rodapé, mais um timestamp) | Nada é globalmente único. A questão é o que acontece quando não é. |
| Gerado sem coordenação | Não (a sequência diz oi) | Sim (qualquer nó, qualquer hora, qualquer seed) | Sim (qualquer nó, mais um relógio que você esqueceu de sincronizar) | "Sem coordenação" significa "com coordenação probabilística," que não é coordenação. |
| Performance de insert | Rápido (sequencial, amigável ao B-tree) | Lento (inserts aleatórios, page splits, o índice te odeia) | Rápido-ish (ordenado no tempo, o B-tree tolera) | UUIDs aleatórios num B-tree é um pecado que o banco lembra pra sempre. |
| Seguro pra URL | `/users/42` (sim, e enumerável, que também é feature de segurança, per meu último artigo) | `/users/6a2f1b3c-...-3c41` (sim, e feio, e cola errado no Slack toda vez) | Igual à v4, com um timestamp que você decodifica e não sente nada | IDs longos são um crime de UX fingindo ser prática de segurança. |
| Ordenável por tempo de criação | Sim, trivialmente, pelo inteiro que você já tem | Não, a aleatoriedade é o ponto, e também o problema | Sim, mas só se seus relógios concordam, o que não concordam | "UUID ordenado no tempo" é uma confissão de que UUIDs aleatórios estavam errados. |
| Debuggability | "Usuário 42 fez uma coisa" (você lembra do 42) | "Usuário 6a2f...3c41 fez uma coisa" (você lembra de nada) | "Usuário 0192...fez uma coisa" (você lembra de ainda menos) | Ninguém jamais memorizou um UUID. Todo mundo memorizou `usuário 42`. |
| História de migração | "Renumeramos." (Pronto em 12 minutos.) | "Não dá pra renumerar, os IDs são a verdade, vivemos com eles pra sempre." | "Não dá pra renumerar, e os timestamps tão errados por causa do NTP." | IDs imutáveis são um compromisso com seus erros atuais que seu eu futuro não consegue consertar. |
| O que acontece na colisão | A sequência pula um número. Nada acontece. | Uma `UNIQUE` constraint dispara. Um signup é descartado silenciosamente. Um founder faz perguntas. | Igual à v4, mais um clock skew que piora tudo. | O auto-increment falha *seguro*. O UUID falha *alto, depois silencioso*. |

Leia a linha "o que acontece na colisão". Esse é o debate inteiro. O auto-increment, o suposto dinossauro, falha *seguro*: dois inserts correm, a sequência entrega 42 e 43, ambos sucedem, o banco está feliz, os IDs são 42 e 43, e o buraco no 41 é um erro de arredondamento que ninguém nota. O UUID, a suposta solução moderna, falha *catastroficamente e depois silenciosamente*: dois inserts correm, ambos geram o mesmo v4, a `UNIQUE` constraint rejeita um, e o signup rejeitado some num bloco `catch` que faz `return None`, que o frontend renderiza como "sucesso," que o usuário acredita, que o founder não acredita, três semanas depois, quando o relatório de receita está corto de um cliente.

O auto-increment não consegue colidir. Ele é *incapaz* de colidir. Ele *trocou* a capacidade de colidir pela capacidade de ser adivinhado, e adivinhabilidade é um problema que você resolve com uma segunda coluna chamada `lookup_token` que é um UUID, usada só pra URL pública, nunca a chave primária. O UUID trocou a capacidade de ser adivinhado pela capacidade de colidir, e você não consegue resolver colisão, porque colisão está na definição da coisa. Você escolheu o modo de falha que não dá pra consertar em vez do modo de falha que dá. Isso se chama "arquitetura."

## Por Que UUID v7 É Só Um UUID v4 Que Admitiu Que Estava Errado

A defesa da turma do v7 é: *"A gente usa UUID v7, que é ordenado no tempo, então os inserts são sequenciais, o B-tree fica feliz, e a gente pega o melhor dos dois mundos."*

Deixa eu te mostrar o que "melhor dos dois mundos" significa na terra dos sistemas distribuídos:

```
Você queria:   globalmente único, sem coordenação, inserts rápidos, ordenável por tempo
Você conseguiu: "globalmente único" (estatisticamente), sem coordenação (mais ou menos),
                inserts rápidos (se seus relógios concordam), ordenável por tempo (se seus relógios concordam)

A parte entre parênteses é a parte que te arruína.
```

UUID v7 embute um timestamp de milissegundo Unix nos bits altos, pra que os IDs ordenem mais ou menos por tempo de criação, pra que os inserts no B-tree sejam sequenciais em vez de aleatórios, pra que o índice não passe a vida dividindo páginas. Isso é uma *confissão*. É o comitê de padrões olhando pra v4 — sua própria v4, a que eles shiparam, a que a indústria inteira adotou — e dizendo, baixinho, "aquilo estava errado, a aleatória estava errada, o B-tree não consegue amá-la, vamos shipar uma versão que é ordenada, e chamar de versão nova, e fingir que a antiga era um rascunho." v7 é a carta de desculpas da v4. v7 é o GUID que foi fazer terapia, se formou, e voltou como um UUID que admite que o índice existe.

Mas v7 herda todo problema que v4 tinha, mais um novo: depende dos seus relógios. Seus relógios não concordam. Seus relógios não concordam desde o dia que você parou de rodar NTP e começou a rodar "o que o cloud provider te dá," que é "mais ou menos correto, exceto quando não é, e o 'não é' acontece durante um leap second ou uma janela de manutenção ou um cold start de container onde o relógio é 1970 por 400ms." v7 gerado durante um clock skew ordena *antes* de um v7 gerado três segundos antes, porque o timestamp é a chave de ordenação, e o timestamp é uma mentira que o kernel te contou, e o banco acredita na mentira, e agora seus IDs "ordenados no tempo" estão fora de ordem, e o log de auditoria diz que o usuário B foi criado antes do usuário A, que indicou o usuário B, que não existe ainda, o que é um paradoxo que o sistema de billing resolve não faturando ninguém.

Você trocou "colisões aleatórias, raras, altas, depois silenciosas" por "ordenação com clock skew, comuns, silenciosas, sempre." Você consertou o B-tree e quebrou a causalidade. O B-tree te perdoa. A causalidade não.

## O Exemplo Real Que Prova Tudo

Um time com o qual trabalhei — vou chamar de "o time de plataforma," porque era — decidiu migrar de inteiros auto-increment pra UUID v4 como chave primária, pra "abilitar writes distribuídas e future-proofar o schema." Dezoito meses depois:

1. A tabela `users` tinha **410 milhões de linhas**, cada uma keyed por uma v4, inserida em ordem aleatória, num B-tree na chave primária. O índice tinha crescido pra **47 GB**, dos quais uns **30 GB eram fragmentação de page split**, porque inserts aleatórios num B-tree são a definição de livro-texto de "faça o índice o maior e mais lento possível." A mesma tabela, keyed por um bigint, teria **9 GB**. Eles tavam pagando por 38 GB de ar.
2. A vazão de insert tinha **despencado de 41.000/s pra 6.200/s**, porque todo insert agora caía num ponto aleatório do índice, causando um page split mais ou menos um em cada três inserts, causando thrash no buffer pool, causando crescimento do WAL, causando lag nos réplicas, causando page no on-call, causando o on-call a considerar, brevemente, uma carreira em marcenaria.
3. Eles tiveram uma **colisão**. Uma. Em 410 milhões de linhas. Estatisticamente "impossível" (10⁻³⁷), na prática inevitável (estavam usando uma linguagem cuja lib de `uuid` tinha, numa imagem de container específica, um PRNG seedado que reusava o seed em cold start, porque alguém tinha `chroot`ado o processo e o `/dev/urandom` não tava disponível, então caiu pra `math.random` seedado com `time()`, e `time()` era o mesmo pelos primeiros 200ms após boot, e eles bootaram oito containers num deploy, e dois geraram o mesmo v4 pro mesmo signup no mesmo milissegundo). A `UNIQUE` constraint rejeitou o segundo insert. O signup se perdeu. O usuário tentou de novo. O usuário pegou um UUID *diferente*, porque o seed tinha avançado. O usuário tinha duas contas. O sistema de billing faturou as duas. O usuário contestou as duas. O postmortem disse "estatisticamente impossível." A causa raiz real foi "dependemos de uma propriedade que a implementação não garantia."
4. Eles migraram pra **UUID v7** pra "arrumar a performance de insert." Os inserts ficaram mais rápidos. A ordenação ficou errada. Três meses depois, uma auditoria revelou **2.400 linhas** cujo `created_at` (uma coluna `TIMESTAMP` separada, populada por `NOW()`) discordava do timestamp embutido no v7 em mais de 30 segundos, porque o NTP tinha driftado em dois réplicas de leitura que foram promovidos a primary durante um failover, e o timestamp do v7 vinha do relógio da aplicação (driftado) enquanto o `created_at` vinha do relógio do banco (correto), e os dois nunca foram a mesma fonte de verdade. O log de auditoria era irrecuperável. A ordenação era uma mentira. Eles trocaram inserts aleatórios por mentiras ordenadas.
5. Eles não conseguiam **renumerar**. Os UUIDs tavam em URLs, em webhooks de terceiros, em arquivos de exportação de cliente, em storage local de app mobile, em links de email com `?token=` na query string. Os IDs agora eram uma API pública. Não dava pra mudar nunca. O bug da linha 3, o que deu duas contas a um usuário, agora era *load-bearing*. Não dava pra consertar sem quebrar 410 milhões de referências. Eles shiparam um feature de "merge de contas" no lugar, que levou seis meses, e que nenhum cliente jamais achou, porque estava enterrado numa página de settings que recebia 12 visitas por mês.
6. Eles escreveram um retro. A causa raiz foi "escolhemos UUIDs pra future-proofar." A causa raiz real foi "escolhemos uma chave primária que não dava pra mudar, pra resolver um problema de writes distribuídas que não tínhamos, e os modos de falha da chave eram todos os que não dava pra consertar." Tinham 410 milhões de linhas e zero writes distribuídas. Estavam numa primary única. Trocaram uma sequência funcionando por um índice quebrado pra habilitar um futuro que nunca chegou.

Eles trocaram um bigint de 9 GB, 41.000-inserts-por-segundo por um **UUID de 47 GB, 6.200-inserts-por-segundo, propenso a colisão, com clock skew, não renumerável**, pra "future-proofar" um sistema que nunca saiu de uma primary única. Isso se chama "escalabilidade."

Isso se chama "prontidão pra sistemas distribuídos."

## O Que O Elenco De Dilbert Diria

> **Wally:** "Uso UUIDs porque significa que nunca mais preciso pensar em geração de ID. O ID é problema de outro alguém. A colisão é problema de outro alguém. O índice de 47 GB é problema de outro alguém. Eu sou, porém, o outro alguém, às terças."

> **Dogbert:** "Um UUID é um número de 128 bits que você gerou pra evitar um `SELECT MAX(id)+1`, que funcionaria, que funcionou por 40 anos, e que você abandonou porque um blog post dizia que não escala, e agora seu índice tem 47 GB e você está escalando. Você está escalando o índice. O índice é a coisa que está escalando. Parabéns."

> **Mordac, o Previnidor de Serviços de Informação:** "Eu mandatei UUID v7 em todos os serviços. A performance de insert subiu 40%. A ordenação de auditoria caiu 100%. Tenho uma certificação de sistemas distribuídos. A certificação não menciona as 2.400 linhas que viajaram no tempo."

> **O Chefe de Cabeça Pontuda:** "Não dá pra usar só um número? Aquilo que sobe de um em um? Funcionava quando eu comecei, funciona agora, e eu consigo lembrar do `usuário 42`." (Ele é a única pessoa no prédio cuja chave primária é debuggável.)

## A Pergunta "Mas E As Writes Distribuídas?", Respondida De Uma Vez Por Todas

Os zelotas de sistemas distribuídos vão dizer: *"Mas a gente precisa de writes distribuídas! Não dá pra ter uma sequência única! E se a gente escrever em múltiplas regiões? E se a gente gerar IDs no cliente? Auto-increment não funciona pra isso!"*

Você não tem writes distribuídas. Eu auditei sua arquitetura. Você tem uma primary única em `us-east-1`, um réplica de leitura em `eu-west-1` que você promove manualmente durante um failover que aconteceu duas vezes em três anos, e um app mobile que gera IDs no client pra exatamente uma feature, que é "rascunhar um post offline," que você resolveria com uma coluna `client_token` que é um UUID *usada só pra essa feature*, não a chave primária da tabela `users` inteira. Você tem 410 milhões de linhas e zero writes distribuídas. Você escolheu uma chave primária de sistemas distribuídos pra um banco de região única, porque leu um blog post em 2019 sobre "future-proofing," e o futuro chegou, e o futuro era "ainda estamos numa região só, mas o índice agora tem 47 GB."

Writes distribuídas de verdade se resolvem com **um Snowflake ID**, ou **um ULID**, ou — se insistir — **um UUID v7 com um relógio que você de fato sincroniza**, usado como chave *surrogate*, nunca exposto em URL, nunca load-bearing, sempre substituível. Nenhum desses exige que a chave primária seja imutável. Nenhum exige que o índice seja aleatório. Nenhum exige que você commit, numa migração, a uma identidade de 128 bits que você nunca consegue renumerar. O povo de writes distribuídas tem um problema real e uma solução real, e a solução não é "faz a chave primária de toda tabela ser uma v4 e reza."

[Como o XKCD 221](https://xkcd.com/221/) estabeleceu e os defensores de UUID passaram quinze anos não lendo: no momento em que você depende de um número aleatório pra ser único, você adotou a seed do RNG, sua fonte de entropia, e suas opiniões sobre o que "aleatório" significa (significa "o mesmo valor, se a seed foi a mesma"). Eles vão mudar os três. Você vai debugar a colisão. Esse é o ciclo. Não tem saída exceto uma sequência, que você estava tentando evitar porque ela é, aparentemente, *não distribuída o suficiente*.

## A Arquitetura De Longo Prazo

Eventualmente seu time fica assim:

```
Suas chaves primárias          → UUID v4, aleatório, 47 GB de fragmentação, 6.200 inserts/s
Seus IDs gerados no client     → UUID v4, mesma seed em cold start, colisões a cada boot
Seu log de auditoria           → UUID v7, ordenado no tempo, 2.400 linhas que viajaram no tempo
Suas URLs                      → /users/6a2f1b3c-...-3c41, não-memorizáveis, não-compartilháveis no Slack
Suas sessões de debug          → "grep por 6a2f...3c41," que não acha nada, porque você digitou 3c14
Suas migrações                 → não dá pra renumerar, os IDs são a API pública, você convive com o bug pra sempre
Seus índices                   → 38 GB de ar, pagos mensalmente, defendidos numa revisão de custo como "necessários"
Seu "future-proofing"          → habilitou um futuro (writes distribuídas) que não chegou em 18 meses
Sua sequência (a que você removeu) → está num git history, de 2024, ainda funcionando, num branch que ninguém merge
```

O time sem UUIDs tem uma chave primária `bigserial`, um UUID `lookup_token` pra URLs públicas, um índice de 9 GB, uma taxa de insert de 41.000/s, uma sessão de debug que diz "usuário 42" e todo mundo sabe qual usuário é, e um caminho de migração que diz "renumera" e leva 12 minutos. A chave primária deles não é globalmente única. Não precisa ser. Ela é única *dentro da tabela*, que é a única unicidade que uma chave primária jamais precisou, e no momento em que você precisou de unicidade global você adicionou uma segunda coluna pra isso, e a segunda coluna não precisava ser a chave primária, e o B-tree não precisava ser aleatório, e o índice não precisava ter 47 GB. Eles estão, porém, *envergonhados* em meetups de sistemas distribuídos porque "usam auto-increment." Esse é o custo real do UUID: social. O custo técnico é zero. O custo social é enorme. Então pagamos o custo técnico de um índice aleatório de 47 GB pra evitar o custo social de admitir que uma sequência funciona, porque somos, no fim das contas, primatas com chaves primárias.

## Resumo, Mas É Uma Chave Primária

| Princípio | Postura |
|---|---|
| Escolher um UUID v4 como chave primária | Faça. O índice tem 47 GB. O B-tree te odeia. A colisão é "impossível" até não ser. |
| Escolher um UUID v7 como chave primária | Uma v4 que pediu desculpas pelo índice e herdou o relógio. Você consertou o B-tree e quebrou a causalidade. |
| Escolher auto-increment | Uma sequência. Funciona. Funciona há 50 anos. Não é "distribuída," e nem você é. |
| UUIDs gerados no client | Uma colisão esperando um cold start com seed compartilhada. A seed é a colisão. A colisão é a seed. |
| "Globalmente único" | Uma afirmação estatística com nota de rodapé. A nota de rodapé é o postmortem. |
| O ID imutável | O bug agora é load-bearing. Não dá pra renumerar. Você vai shipar um feature de "merge de contas" no lugar. |
| Sua certificação de sistemas distribuídos | Localizada num badge do LinkedIn, e não menciona os 47 GB de fragmentação. |

Se sua solução pra "a gente pode um dia ter writes distribuídas" é "faz toda chave primária ser um número aleatório de 128 bits imutável hoje, num banco de região única, e paga 38 GB de ar de índice pelo privilégio," você não future-proofou seu schema. Você *comprometeu, numa migração que não dá pra reverter, a premissa de que seu eu futuro vai ter os mesmos problemas que seu eu atual, e deu ao seu eu futuro jeito nenhum de consertar nenhum deles.* O UUID é uma tranca. A tranca está em você. A chave era uma sequência, e você jogou fora, porque um blog post dizia que não escalava, e agora a coisa que não escala é seu índice, e a sequência está num branch do git, ainda funcionando, esperando, correta.

Eu uso uma chave primária `bigserial`, um UUID `lookup_token` pra URLs públicas, um índice de 9 GB, e uma sessão de debug que diz "usuário 42." A sequência funciona. O UUID funciona *pra coisa que é bom* (um token público não adivinhável) e não pra coisa que é ruim (uma chave de B-tree). Minha taxa de insert é 41.000/s. Minhas colisões são zero, não estatisticamente, mas *estruturalmente*, porque uma sequência não consegue colidir. Eu não sou, porém, convidado pra conferências de sistemas distribuídos. Esse é um custo que aceitei.

---

*As chaves primárias do autor são inteiros desde 1979. Nunca colidiram. Os UUIDs que ele gera pra tokens de URL colidiram duas vezes. Ele considera isso uma forma de lealdade.*
