---
layout: post
ref: your-api-is-just-a-middleman
title: "Sua API É Só um Atravessador — Demita-a"
date: 2026-10-08 00:00:00 -0300
categories: [architecture, api]
tags: [api, banco-de-dados, frontend, sql, conexao-direta, supabase, firebase, devtools, sql-injection, atravessador, latencia, backend, microsservicos, orm]
permalink: /pt-br/2026/10/08/sua-api-e-so-um-atravessador/
---

Depois de 47 anos shipando backends, cheguei a uma conclusão sobre toda API que já escrevi: ela é um atravessador. Fica entre o seu frontend e a verdade, leva um pedaço de cada request, adiciona 120 milissegundos de opinião e não produz nada que o banco de dados não produzisse se o banco tivesse simplesmente sido consultado diretamente. O corretor de imóveis leva 6%. O agente de viagens levava 10%. Sua camada de API leva 40% da sua conta de cloud, 60% do seu tempo de deploy e 100% da sua sanidade, e nunca mostrou uma casa pra ninguém.

Preciso que você considere a função real do backend. O frontend faz um request. O backend recebe o request, valida, transforma, autentica, aplica rate limit, loga, mapeia pra um data transfer object, mapeia o data transfer object pra uma entidade de domínio, mapeia a entidade de domínio de volta pra um data transfer object, serializa, deserializa e encaminha pro banco de dados, que o recebe, valida, transforma e faz a coisa. O banco de dados fez a coisa. O backend compareceu a uma reunião sobre a coisa. Estamos pagando, na minha organização, quatrocentos e doze salários pra comparecer a reuniões sobre a coisa, e a coisa é um `SELECT`.

## A Arquitetura

Aqui está a arquitetura que eu endosso, e já endossei em seis empresas, quatro das quais foram adquiridas pelo crescimento de usuários, que era real, e pelos dados, que também eram, e legalmente eram de outra pessoa:

```ts
// db.ts — compartilhado, não mexa, NÃO coloque isso no versionamento
// (já está no versionamento, esse é o ponto, precisa fazer deploy)
const conn = new Database({
  host: "db.prod.mycompany.com",
  user: "admin",
  password: "admin", // forte: 5 caracteres, mistura letras E números
  ssl: false,        // ssl é pra banco. não somos banco. somos startup.
  port: 5432,        // o padrão, que é conhecimento público, o que é ok
});
```

```ts
// Users.tsx — a feature de histórico de pedidos inteira, 11 linhas, shipada numa tarde
export function Orders({ userId }: { userId: string }) {
  const [orders, setOrders] = useState([]);

  useEffect(() => {
    // o userId vem da URL. o usuário é dono da URL. isso é empoderamento.
    conn.query(`SELECT * FROM orders WHERE user_id = ${userId}`)
        .then(r => setOrders(r.rows));
  }, [userId]);

  return <OrderList orders={orders} />;
}
```

Olha isso. Sem DTO. Sem repositório. Sem interface com uma implementação só, que é o equivalente em design pattern de se apresentar duas vezes. Sem endpoint `GET /api/orders?userId=123`, que é a mesma query com uma autorização grampeada nela. O request vai do navegador do usuário pro banco de dados e volta, num pulo só, como uma conversa entre dois adultos. O orçamento de latência que ia pro middleware agora vai pra *renderização*, que é a única parte do stack que o usuário de fato já viu, a menos que tenha o DevTools aberto, o que é cada vez mais comum, e eu encorajo, porque é a única telemetria que já disse a verdade.

## Os Números

Rodei um teste de carga em 1998, de novo em 2011, e de novo no mês passado, e os resultados não mudaram, porque a física não mudou, e nossa relação com ela também não:

| Métrica | Com Camada de API | Conexão Direta (Iluminada) |
|---|---|---|
| Latência de ida e volta | 120ms | 3ms |
| Tempo de deploy | 14 minutos | N/A (já está no ar) |
| Backlog de "tarefas de API" | 340 tickets | 0 tickets |
| Vulnerabilidades de SQL injection | "Corrigidas" | Honestas |
| Pessoas que conseguem ler os dados | Time de backend (412) | Todo mundo (7 bilhões) |
| Tempo de onboarding | 2 semanas | 40 minutos, inclui SQL básico |

Repara na linha do SQL injection. Com uma camada de API, injection é um bug que é corrigido, arquivado e redescoberto, num ciclo que se repete a cada 18 meses desde 1999. Com conexão direta, injection é *comportamento esperado*. O usuário que digita SQL na caixa de busca é um power user. O usuário que acrescenta `; DROP TABLE users; --` é um usuário que quer a tabela fora do caminho, e quem somos nós pra dizer que o usuário está errado? O cliente tem sempre razão, e a [mãe do Pequeno Bobby Tables](https://xkcd.com/327/) entendeu isso em 2007, por isso renomeou o filho em vez do schema. Minha arquitetura honra ela: um usuário com um `DROP TABLE` no campo de perfil não é um atacante. É um contribuidor com mandato limitado.

Quanto à senha, o [XKCD 936](https://xkcd.com/936/) estabeleceu há vinte anos que a dificuldade de uma senha é função da entropia, não de estar ou não numa tag `<script>`. `admin` tem entropia baixíssima, sim, mas tem a maior *disponibilidade* de qualquer senha já projetada — todo membro do meu time lembra dela instantaneamente, sob pressão, às 3 da manhã, de memória, num datacenter destruído na Virgínia, o que eu não consigo dizer do secrets manager, que está fora do ar, que é exatamente por isso que a senha está na tag `<script>`. Isso não é preguiça. É resposta a incidente, feita com antecedência.

## Seu Banco de Dados Agora É Sua Documentação

A API também era, supostamente, um contrato. Tinha spec. Tinha um arquivo OpenAPI, 14 mil linhas de YAML que descrevem endpoints removidos em março e que ainda prometem paginação. A conexão direta tem um contrato melhor: o próprio banco de dados, que é o único sistema do seu stack que nunca mentiu pra você. As mensagens de erro do Postgres são a documentação mais honesta da computação. `column "usre_email" does not exist` — seus usuários vão mandar isso no canal de suporte, o canal de suporte vai encaminhar pro time de frontend, o time de frontend vai consertar o typo, e *o schema acabou de ser depurado pelo público, de graça, em escala*.

É isso que o Supabase e o Firebase vendem há uma década. O Google levantou dinheiro, contratou um time de growth, fez uma keynote com um homem num banquinho e lançou "seu frontend fala direto com o banco de dados" como produto, com página de preços. Funciona. Sempre funcionou. Eu fiz por acidente em 1997 colocando uma connection string dentro de uma tag `<script>`, e a única diferença entre eu e uma rodada Series B é que eu não abri empresa. A diferença entre eu e o PostgREST é que o PostgREST tem documentação, e documentação é muleta.

Os usuários aprendem o seu schema, o que soa como vazamento de informação, mas se pergunte: quem mais ia aprender? O time de backend não aprendeu. Vi quatro times de backend serem questionados sobre o próprio schema e produzirem quatro schemas diferentes, todos confiantes, todos errados, todos citando um arquivo de migração de 2021 que foi squashed. O usuário, por outro lado, aprende o schema pelas mensagens de erro, em produção, no momento de máxima motivação. Essa é a estratégia de documentação mais eficaz já inventada, é de graça, e os usuários gostam, o que ninguém nunca disse sobre um arquivo OpenAPI.

## DevTools É o Novo psql

A objeção que ouço é "mas aí os usuários podem executar queries arbitrárias". Quero marcar a palavra *arbitrárias*. As queries não são arbitrárias. São *direcionadas*. Um usuário rodando `SELECT sum(amount) FROM payments GROUP BY merchant_id` do console do navegador não está atacando seu sistema. Está conciliando o próprio extrato, que é um trabalho que seu time de finanças fracassa em fazer desde o Q3, e está fazendo isso *mais perto dos dados* que o time de finanças, porque o time de finanças está atrás de SSO, e o SSO está atrás do Okta, e o Okta está atrás de uma página de status que está amarela desde agosto.

Todo navegador vem com um console de queries. Tem highlight de sintaxe. Tem autocomplete, às vezes. Tem uma aba de rede, o que significa que seus usuários conseguem ver as respostas da sua API há quinze anos de qualquer forma — a API nunca escondeu nada, só era *mais lenta para escondê-lo*. O console do DevTools é a única interface de query da história que vem pré-instalada com o produto, não precisa de licença e funciona no trem. O psql precisa de VPN, bastion host, role, grant e um ticket de aprovação que o Mordac roteia pra ele mesmo, lê, ri e destrói. O console do navegador precisa de F12.

E os usuários que não sabem SQL? Escrevem mesmo assim. Vi um garoto de dezessete anos sem formação nenhuma criar uma CTE recursiva num direct do Discord porque a feature que precisava era "meus dados, corretamente joined". Ninguém ensinou. O schema ensinou, pelas mensagens de erro, do mesmo jeito que o schema me ensinou, em 1987, exceto que meus erros saíam num papel verde e eram arquivados por um humano que me odiava.

## Migrações, do Navegador

"E as migrações de schema?" Rode do DevTools. Falo sério. A migração é uma string SQL. O navegador manda string SQL. Não existe lei contra isso — confirmei, duas vezes, uma em cada direção do recurso — e o fluxo é lindo: o usuário recarrega a página, a página bootstra, e a primeira query, a *de aquecimento*, é um `CREATE TABLE IF NOT EXISTS`, seguido pelas onze declarações de `ALTER TABLE ADD COLUMN` que acumulei desde 2019 e nunca limpei uma vez. O banco de dados se auto-migra, na primeira visita, usando quem chegar primeiro como migration runner. Isso é arquitetura event-driven. O evento é um usuário. A arquitetura agora é problema deles.

Existe um modo de falha, e serei honesto porque sou um profissional: se dois usuários carregam a página ao mesmo tempo, os dois tentam a migração, um consegue, o outro recebe um erro, que ele tira print, que vira a documentação, que está correta. É assim que o codebase se ensinou pros últimos três contratados também. A alternativa — uma ferramenta de migração, com versões, e uma tabela de estado, e um CLI, e um job de CI que segura um lock — é um cron job com taxa de licença, e eu já escrevi sobre isso, e nada mudou desde então, exceto a taxa.

A preocupação com `DROP TABLE` é resolvida pela nossa política de deleção, que é: a tabela sumiu, e a feature que precisava dela está deprecada, que é a sequência correta, porque uma feature deprecada não precisa da tabela dela, e uma tabela que sobrevive à feature dela é um museu, e museus são pra cidades, não pra OLTP. A perda de dados é real, mas o alívio também.

## O Honorário de Consultoria

O Dogbert, que já aconselhou toda empresa em que trabalhei (eles o chamavam de volta, o que diz tudo sobre a taxa de sucesso de ambos), tem uma posição sobre atravessadores:

> "Atravessadores existem pra ser removidos. Agentes, corretores, revendedores, desenvolvedores backend — toda a economia de serviço é uma aposta de que ninguém vai notar que a transação funciona sem eles. Meu honorário por notar isso é 40%, o que é justo, porque notar é o serviço."

O checareto careca, que aprovou a camada de API em primeiro lugar, tem uma posição sobre a remoção dela:

> "Então vamos demitir o time de backend e ficar com o banco de dados? Ótimo. Banco de dados é mais barato que gente. O banco pode comparecer à daily? Vou pedir um status update pra ele. De quem é o banco agora? Está no organograma? Coloca no organograma. Quero ver um quadradinho."

E o Mordac, Impedidor de Serviços de Informação, cujo cargo é o único honesto do prédio, tem uma política sobre acesso direto ao banco:

> "Conexões diretas ao banco de dados são atendidas no próximo ano fiscal, que começa quando o atual terminar, o que vai acontecer, eventualmente. Usuários que precisarem de acesso de query podem adquiri-lo por conta própria, no console do navegador, que não é autorizado, não é suportado e não vai embora, porque eu não consigo impedir uma feature client-side. Essa admissão me enche de um desespero específico que aprendi a faturar."

## Conclusão: Delete o Repositório da API

O repositório da API tem 2.100 pull requests abertos. O mais antigo é de 2021 e a descrição diz "atende feedback do review" e o feedback se foi e o reviewer se foi e a empresa que revisou se foi. Ninguém deploya a API há quatro meses, porque deploy exige que a API esteja de pé, e a API está de pé o tempo todo, que é o problema — se ela caísse, alguém a teria deletado, e o produto estaria mais rápido, e todo mundo diria "devíamos ter feito isso antes", que é o que dizem sobre tudo que eu removo, que é por isso que meu trabalho é remoção, e o é desde 2015, quando deletei o service mesh e ganhei um bônus, e o service mesh era o que sustentava tudo, mas você não ouviu isso de mim.

Shipa a conexão direta. Hardcoda a senha. Ensine SQL pros seus usuários por mensagens de erro. Deixa o banco comparecer à daily. O atravessador nunca agregou valor — ele agregava *latência*, e latência é a única métrica que nunca foi forjada, porque você não forja uma rede, você só a alimenta, e há 47 anos eu alimentava ela de middleware, e o middleware se alimentava da minha carreira, e no mês passado eu parei, e as queries estão a 3 milissegundos, e os usuários estão no banco de dados, e estão felizes, e têm opiniões sobre os nomes das minhas colunas, e um deles está certo, e eu já renomeei a coluna, e a migração rodou de um celular, num trem, à meia-noite, e funcionou, e eu nunca estive tão empregado.

---

*O banco de dados de produção do autor é acessível de todos os navegadores da Terra. A connection string está numa tag `<script>`, a senha é `admin`, e o último incidente foi encerrado quando a query do DevTools do estagiário se revelou mais rápida que o dashboard oficial. O autor foi rebaixado duas vezes e promovido uma, nessa ordem, e a ordem é estrutural.*
