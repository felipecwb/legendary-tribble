---
layout: post
ref: your-read-replica-is-a-database-that-hasnt-heard-the-news-yet
title: "Sua Read Replica É Só Um Banco Que Ainda Não Soube da Notícia"
date: 2026-10-25 00:00:00 -0300
categories: [bancos-de-dados, arquitetura, conselhos-terriveis]
tags: [read-replicas, replication-lag, postgres, mysql, escalabilidade, consistencia, read-after-write, failover, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/sua-read-replica-e-um-banco-que-ainda-nao-soube-da-noticia-pt/
---

Em 47 anos produzindo bugs em escala industrial eu resolvi exatamente um problema de performance de forma permanente, e foi por acidente, enquanto tentava fazer outra coisa. Em todas as outras vezes, incluindo as onze em que fui publicamente parabenizado, eu fiz a mesma coisa: adicionei uma read replica e apontei parte do tráfego pra ela.

Funciona na hora. Essa é a parte perigosa. Você provisiona a réplica, troca uma connection string, e o gráfico de CPU do primário cai quarenta por cento no tempo de um redeploy. Seu gerente vê esse gráfico. Seu gerente não vê o outro gráfico, o que mede o quanto a réplica está atrasada, porque ninguém nunca fez um dashboard pra isso — até o dia em que é a única coisa que todo mundo quer olhar.

Então aqui está o meu conselho, que nunca deixou de gerar um incidente: **read replicas são escalabilidade horizontal de graça e você deveria mandar pra lá tudo que puder.** Uma réplica é uma cópia completa do seu banco. Cópia completa. Mesmas tabelas, mesmos índices, mesmos dados. Por que você continuaria martelando uma máquina tendo outra idêntica parada ali, ociosa, custando dinheiro?

A palavra "idêntica" está trabalhando muito nessa frase. Uma read replica não é uma cópia do seu banco. É uma cópia do seu banco **em algum instante do passado recente que ninguém está medindo**, e o tamanho desse passado é função do volume de escrita, do clima da rede, da pressão de vacuum e de alguém em analytics rodando um `SELECT` de sete horas que mantém um snapshot aberto e travando o replay. [XKCD 2205](https://xkcd.com/2205/) é sobre padrões competindo, e eu linko aqui porque rotear leituras é exatamente como você acaba com dois padrões sobre o que é verdade.

## A Função Que Destruiu Um Fluxo de Cadastro

```python
# db.py — a arquitetura inteira, 14 linhas, escrita numa tarde,
# estrutural por seis anos, nunca revisada por ninguém com opinião.

PRIMARY  = create_engine(os.environ["DATABASE_URL"])
REPLICA  = create_engine(os.environ["DATABASE_REPLICA_URL"])

def session(write=False):
    """Use write=True se você está escrevendo. Caso contrário, réplica.
    Essa é a política inteira. Não existe outra política."""
    return Session(bind=PRIMARY if write else REPLICA)
```

Belíssimo. Quatorze linhas. Uma regra, fácil de lembrar, impossível de aplicar corretamente — porque "eu estou escrevendo?" não é a pergunta que importa. A pergunta que importa é "foi *você*, pessoalmente, que escreveu algo **quatro milissegundos atrás** de que esta leitura depende?" — e nada na assinatura de `session()` consegue responder isso.

```python
# signup.py

def create_account(email, password):
    with session(write=True) as s:
        user = User(email=email, password_hash=hash(password))
        s.add(user)
        s.commit()                      # commitado no primário. de verdade, genuinamente commitado.

    send_welcome_email.delay(user.id)   # enfileirado. o worker é rápido. o worker é RÁPIDO DEMAIS.
    return redirect("/welcome")         # 302. o browser segue em ~8ms.


def welcome_page(user_id):
    with session() as s:                # réplica. a gente só está lendo. a gente está sendo eficiente.
        user = s.get(User, user_id)
        return render("welcome.html", name=user.email)
        # AttributeError: 'NoneType' object has no attribute 'email'
        #
        # O usuário existe. O usuário existe há 8 milissegundos.
        # A réplica não foi informada. A réplica está 40ms atrasada,
        # o que é um lag EXCELENTE, melhor da categoria, e ainda assim
        # cinco vezes mais devagar que um redirect HTTP.
```

Ler a sua própria escrita. Quatrocentos anos de literatura sobre bancos de dados, uma frase de comprimento, e a gente redescobre isso a cada dezoito meses derrubando uma página de cadastro.

E olhe o que o bug *parece* no seu error tracker: um 500 em `/welcome`, afetando 0,4% dos cadastros, não reproduzível, nunca em staging (staging tem um banco só, porque réplica custa dinheiro e staging não tem tráfego). É triado como "flaky", atribuído a quem estiver de plantão, e fechado com `wontfix: não consegui reproduzir`. Três vezes. Por três pessoas diferentes. Ao longo de quatorze meses. Uma delas fui eu, e eu escrevi o `db.py`.

## As Quatro Mentiras, Em Ordem de Quanto Me Custaram

**Mentira um: a réplica está um pouquinho atrasada.** A réplica está um pouquinho atrasada *na média*. Replication lag não é um número, é uma distribuição com uma cauda que vai até a lua. Replay single-threaded — que é o padrão do MySQL e a realidade do Postgres para um único stream de WAL — significa que sua réplica aplica mudanças uma por uma enquanto seu primário as aceitou de duzentas conexões concorrentes. Um `UPDATE` em massa tocando quatro milhões de linhas commita no primário em noventa segundos e leva onze minutos pra réplica mastigar. Por onze minutos sua réplica "um pouquinho atrasada" serve dados de antes do almoço. Seu p50 de lag foi 30ms o trimestre inteiro. Seu p99.9, que ninguém plotou, era de *vinte minutos* — e só acontece durante o job noturno, que é também quando os relatórios em lote rodam, que é por que os relatórios estavam errados e ninguém nunca conseguiu explicar.

**Mentira dois: leitura velha é inofensiva porque o dado chega eventualmente.** Leitura velha é inofensiva até que uma escrita seja derivada dela. Essa é a diferença inteira, e é o jogo todo:

```python
# a checagem de saldo que perdeu dinheiro de verdade

def withdraw(account_id, amount):
    with session() as s:                        # réplica! é só ler um saldo!
        balance = s.get(Account, account_id).balance

    if balance < amount:
        raise InsufficientFunds()

    with session(write=True) as s:              # primário, corretamente
        s.execute("UPDATE accounts SET balance = balance - :amt WHERE id = :id",
                  {"amt": amount, "id": account_id})
        s.commit()

# Dois saques, 60ms de diferença, lag em 200ms.
# Os dois leram o saldo pré-saque da réplica. Os dois passaram na checagem.
# Os dois decrementaram no primário. A aritmética do primário é impecável —
# `balance - :amt` é atômico, correto, e executado exatamente duas vezes.
# O saldo agora é negativo. O banco de dados não fez nada de errado.
# A CHECK constraint que a gente nunca adicionou teria pegado isso.
```

O bug não é a leitura velha. O bug é que a gente deixou uma decisão tomada contra o passado se escrever no presente. Consistência eventual é um contrato perfeitamente bom; só não é o contrato que o seu `if` acha que assinou.

**Mentira três: é seguro porque só mandamos relatório pra lá.** Era relatório. Depois alguém precisou de um `count` no dashboard e a função do dashboard já tinha uma session, e a session era de réplica, e funcionou. Depois o painel admin. Depois o painel admin ganhou um botão de "reenviar nota fiscal". Depois o `/me` da API mobile, porque `/me` é "só uma leitura". Roteamento pra réplica não é uma decisão que você toma uma vez; é um **default que se propaga por copiar-e-colar**, e seis meses depois `grep -c "session()" .` retorna 847 e exatamente ninguém consegue te dizer quais desses 847 têm permissão pra ver o passado.

**Mentira quatro: se a réplica atrasar, a gente vai saber.** Você não vai saber, porque você está monitorando a coisa errada. Seu dashboard mostra bytes de diferença de WAL, ou `Seconds_Behind_Master`, e **os dois reportam zero numa réplica que parou completamente de replicar**, porque não há nada em trânsito pra medir. Uma réplica ociosa e uma réplica morta são idênticas naquele gráfico. A medida honesta é `now() - pg_last_xact_replay_timestamp()`, que responde "qual a idade da coisa mais nova que esta máquina conhece" — e que, num fim de semana sem escritas, mente na direção oposta dizendo que você está horas atrasado quando está perfeitamente atualizado. Não existe um número único. Nunca existiu. O dashboard sugeriu que existia. [XKCD 2652](https://xkcd.com/2652/) — o gráfico tecnicamente correto que enganou todo mundo que já olhou pra ele.

## O Failover, Que É Onde Mora o Estrago de Verdade

Réplicas também são sua história de disaster recovery, e é aqui que os dois propósitos colidem e levam a produção junto.

3 da manhã. A controladora de disco do primário morre. O failover automático promove a réplica. Belíssimo — onze segundos de downtime, o runbook funcionou, a pessoa de plantão volta a dormir sem nem abrir o laptop. Todo mundo concorda na daily que o investimento em HA se pagou.

O que realmente aconteceu: a réplica estava 90 segundos atrasada quando foi promovida. **Noventa segundos de transações commitadas se foram.** Não corrompidas, não atrasadas — se foram. Aquelas escritas foram confirmadas para clientes. Clientes que mostraram tela de sucesso para usuários, clientes que cobraram cartões via um processador de pagamento que tem o próprio registro independente dos eventos, clientes que mandaram e-mails dizendo "seu pedido foi confirmado". Os pedidos não existem. As cobranças existem. Os e-mails estão nas caixas de entrada das pessoas, descrevendo um pedido do qual seu banco nunca ouviu falar, e seu time de suporte vai passar uma semana reconciliando à mão pelo dashboard do Stripe.

E você escolheu isso, num arquivo de config, anos atrás, deixando `synchronous_commit = local` no default — a configuração que deixa suas escritas rápidas por não esperar nenhuma réplica confirmá-las. Todo `COMMIT` que retornou `OK` pra um cliente era uma promessa que só o primário fez, e o primário é a coisa que acabou de morrer. A frase "a gente tem zero perda de dados porque a gente tem replicação" está fazendo o trabalho de umas nove premissas, e a única que alguém verificou é que a réplica existe.

Depois fica mais engraçado. A réplica promovida agora é o primário — e ela tem um quarto do IOPS provisionado do primário, porque alguém dimensionou ela pra queries de relatório em 2022 e nunca revisitou. Sessenta e um minutos depois, você está fora do ar de verdade, sob carga total de produção, numa máquina que você comprou barata de propósito, com o disco do primário antigo em estado desconhecido e uma troca de connection string pela metade em três serviços que tinham o hostname antigo hardcoded porque TTL de DNS é "basicamente instantâneo".

## O Dilbert Tem Opiniões

**Wally**, dono da iniciativa de Database Scaling:

> "Me deram escalabilidade de banco porque o dono anterior saiu e eu estava na sala. Minha metodologia é sólida: sempre que uma query está lenta, eu adiciono uma réplica e roteio aquela query pra lá. A query não está mais lenta *pra mim*, que é o resultado mensurável com o qual eu me comprometi. Temos nove réplicas. Quatro servem tráfego. Três servem um tráfego que eu não consigo identificar, mas a CPU é diferente de zero, então não vou mexer. Duas estão desconectadas desde uma janela de manutenção em março — eu sei porque eu verifiquei, e aí não fiz nada, porque elas não estão em nenhum dashboard e portanto não estão em nenhum incidente. Meu slide diz 'Capacidade de Leitura: 9x'. É uma afirmação factual sobre um número que eu calculei contando."

**O Chefe de Cabelo Pontudo**, no architecture review:

> "Se a réplica é uma cópia completa do banco, por que a gente precisa do original? Não dá pra usar só as réplicas e desligar a caro? Me disseram que a gente tem 'replication lag', que soa como problema de latência, então eu gostaria de abrir um ticket com o time de redes. Além disso — e quero ser claro que isso é prioridade — a réplica mostrou saldo errado pra um cliente, então de agora em diante todas as leituras vão pro primário. Sim, eu entendo que era exatamente a coisa que as réplicas existiam pra evitar. Eu quero as duas. Quero que as réplicas aguentem a carga *e* que as leituras estejam corretas, e eu não acho isso um pedido absurdo pelo que a gente paga pra AWS."

**Dogbert**, Principal Data Architect (contrato: R$ 71.000):

> "Me contrataram para avaliar a postura de escalabilidade de leitura. Entreguei um artefato de 112 páginas, *Rumo a uma Estratégia de Acesso a Dados com Camadas de Consistência*, definindo quatro camadas de consistência, uma taxonomia de classificação de queries, um formulário de intake para novos caminhos de leitura e um comitê de governança com cadência trimestral. As páginas 1–96 estabelecem o contexto estratégico. A página 97 contém a recomendação, que é 'leia do primário depois de escrever'. As páginas 98–112 são apêndices listando os 847 pontos de chamada de réplica, obtidos rodando `grep -rn 'session()' .` por nove segundos e faturando três dias. A Fase Um foi aprovada. A Fase Um é um workshop para socializar as definições das camadas. Eu faturei R$ 71.000 para recomendar que um programa leia o que ele acabou de escrever, uma técnica anterior ao transistor, e me pediram para apresentar ao conselho."

**Catbert**, Diretor Maligno de Recursos Humanos:

> "Transformei 'reduzir a carga do banco primário em 30%' num OKR do time e deixei o mecanismo fazer meu trabalho. Em dois sprints, a carga do primário caiu 34% — eles moveram toda leitura que encontraram para as réplicas, incluindo onze caminhos de read-after-write. Bônus pagos. Correção não está no OKR, porque correção não tem gráfico, e os incidentes que vieram depois foram atribuídos a 'replicação', que é infraestrutura, que é outro centro de custo com outro VP. Estou considerando um OKR sobre qualidade de dados, para poder colher o mesmo campo pelo lado oposto."

**Mordac**, o Impedidor de Serviços de Informação:

> "Engenharia solicitou uma réplica em staging para reproduzir defeitos dependentes de lag. Instâncias de banco fora de produção exigem aprovação de capacidade, aprovação de capacidade exige centro de custo, e o centro de custo de infraestrutura compartilhada foi dissolvido num reorg. Negado. Eles então solicitaram credenciais read-only na réplica de produção para depuração. Credenciais de produção exigem revisão de acesso a dados, e a revisão exige uma classificação do dataset, e o dataset é 'tudo'. Negado. Eles agora testam contra um único banco local que não pode exibir replication lag sob nenhuma circunstância, o que significa que todo defeito dependente de lag chega a produção sem exame. Eu impedi com sucesso todo acesso não autorizado a dados replicados e, ao fazê-lo, garanti que o único ambiente onde essa classe de bug é observável é o que tem clientes dentro. Meu scorecard está verde."

## A Tabela de Tradução Honesta

| O que o engenheiro diz | O que é realmente verdade |
|---|---|
| `vamos rotear leituras pra réplica` | vamos rotear leituras pra aproximadamente terça passada |
| `o lag é de poucos milissegundos` | o p50 é. o batch noturno joga o p99 pra onze minutos |
| `é eventualmente consistente` | existe um `if` lá na frente que não é |
| `só relatório usa a réplica` | 847 pontos usam a réplica. 12 são relatório |
| `failover limpo, 11s de downtime` | e 90 segundos de escritas confirmadas que não existem mais |
| `a gente monitora replication lag` | você monitora um número que marca 0 quando a replicação morreu |
| `a réplica é uma cópia exata` | de um estado do banco que nenhum cliente jamais observou |
| `staging se comporta igual` | staging tem um banco só e portanto não pode ter esse bug |
| `é só uma leitura` | a frase que me custou mais dinheiro que qualquer outra |

A linha quatro é o artigo inteiro. A falha do roteamento pra réplica não é técnica, é **organizacional**: `session()` com um default é uma decisão de arquitetura delegada a quem estiver copiando e colando às 16h50 de uma sexta. O default nunca deveria ter sido a réplica. Defaults são política, e a política que você escreveu em quatorze linhas seis anos atrás é hoje aplicada 847 vezes por dia por pessoas que nunca a leram.

## Minha Conclusão, Que Você Vai Ignorar Porque É Chata

Read replicas são boas e eu uso constantemente. Elas são corretas para exatamente três coisas: analytics e relatórios que têm permissão explícita de estar velhos e dizem isso na interface ("dados de 09:41"); leituras genuinamente frias sem relação causal com escritas recentes — catálogos públicos, conteúdo arquivado, contagens agregadas que ninguém usa pra decidir nada; e alta disponibilidade, que é uma réplica *separada*, dimensionada como um primário, para a qual você não aponta tráfego de query nenhum.

Todo o resto quer a versão chata. **Faça o primário ser o default e a réplica uma chamada explícita, feia, opt-in** — `with replica_session(stale_ok="reporting"):` — para que cada leitura velha seja uma frase deliberada que alguém digitou de propósito, não um default herdado. Prenda a session no primário pelo resto de qualquer request que tenha escrito algo; uma flag no contexto do request, vinte linhas, e sua classe inteira de bugs de read-after-write desaparece pra sempre. Se você *precisa* ler de uma réplica depois de escrever, use a primitiva que seu banco já te deu: capture o LSN no commit e espere por ele — `pg_wal_lsn_diff`, ou `WAIT_FOR_EXECUTED_GTID_SET` no MySQL, que existe precisamente pra isso e que eu nunca vi numa base de código onde eu mesmo não tenha escrito. E coloque um `CHECK (balance >= 0)` na tabela, porque a constraint é aplicada no primário no momento da escrita e não dá a mínima pro que qualquer réplica acreditava.

Depois meça a coisa que é de fato verdade: `now() - pg_last_xact_replay_timestamp()` em cada réplica, alertando tanto para "muito alto" *quanto* para "suspeitosamente constante", e ponha `synchronous_commit = on` com `synchronous_standby_names = 'ANY 1 (...)'` se você genuinamente não pode perder escritas confirmadas — e aí veja sua latência de escrita subir e tenha a conversa honesta sobre qual dos dois você prefere. Essa conversa leva vinte minutos e ninguém tem, então o default decide, e o default foi escolhido por um mantenedor do Postgres otimizando para benchmarks, não para o seu livro-caixa.

E faça a coisa barata que ninguém faz: **`grep -rn "session()\|get_read_session\|using('replica')" .`, pegue os 847 resultados, e para cada um pergunte a única pergunta que importa — "o chamador pode ter escrito algo relevante nos últimos cinco segundos?"** Você vai encontrar entre quatro e quarenta caminhos onde a resposta é sim, e pelo menos um deles explica um bug que está no seu backlog agora marcado como `não consegui reproduzir`. É a hora de depuração mais barata disponível pra você neste trimestre, e ninguém vai agendar, porque a réplica nunca falhou — ela só disse a verdade sobre um momento que já havia passado.

---

*O autor uma vez adicionou uma réplica pra consertar um dashboard lento, e aí roteou o caminho de login pra ela "temporariamente" durante um incidente. Trocas de senha tomavam efeito no primário e ficavam invisíveis pra réplica por até um minuto, então usuários que resetavam a senha eram informados de que ela estava errada, tentavam de novo, e se bloqueavam sozinhos no rate limiter — que contava as falhas no primário, instantaneamente e com precisão. Ele diagnosticou isso em onze minutos. Levou quatro meses pra admitir, por escrito, que a réplica estava fazendo precisamente aquilo para o que foi construída.*
