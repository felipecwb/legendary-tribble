---
layout: post
ref: your-postman-collection-is-a-threat-model
title: "Sua Coleção do Postman É Um Modelo de Ameaça"
date: 2026-08-24 00:00:00 -0300
categories: [seguranca, design-de-api, anti-padroes]
tags: [postman, insomnia, testes-de-api, segredos, modelagem-de-ameacas, colecoes, ambientes, seguranca, divida-tecnica, conhecimento-tribal]
permalink: /pt-br/2026/08/24/sua-colecao-do-postman-e-um-modelo-de-ameaca/
---

Depois de 47 anos produzindo bugs em massa — e eu produzia bugs antes de "Postman" significar qualquer coisa além do homem que trazia cartas e decepções à sua porta, antes de "Insomnia" significar qualquer coisa além do estado natural de um engenheiro de plantão às 3 da manhã, antes de "coleção" significar qualquer coisa além da pilha de monitores CRT bege que eu acumulava caso a indústria mudasse de ideia sobre monitores, o que ela não fez — vi uma indústria inteira terceirizar sua modelagem de ameaças para um aplicativo de desktop que não sabe soletrar "auditoria". A frase nobre para isso é *testes de API*. A frase honesta é *uma pasta compartilhada cheia de bearer tokens com uma interface gráfica*.

Deixe eu explicar o que uma coleção do Postman realmente é na sua organização, o que ela está protegendo você de, e por que a coisa que ela está protegendo você de é ela mesma.

## O Que Uma Coleção do Postman Alega Ser

O pitch, entregue com o entusiasmo de uma pessoa que acabou de descobrir variáveis de ambiente, é este: *guardamos todas as nossas requisições de API num lugar só, organizadas por pasta, então qualquer um no time pode testar qualquer endpoint a qualquer momento. Compartilhamos a coleção via workspace. Temos ambientes para dev, staging e prod. É nossa documentação viva.*

Isso é apresentado como virtude. É a virtude de *descobribilidade*. É, na verdade, a virtude de *dar a todo desenvolvedor do time um botão que diz "cobra um cliente real em produção", e então fingir surpresa quando alguém aperta numa terça porque queria ver se o botão ainda funcionava. O botão ainda funciona. O botão sempre funciona. O botão é a coisa mais confiável da sua organização. O botão é mais confiável que seus testes, sua documentação, e sua escala de plantão combinados. O botão é estrutural. O botão é uma ameaça.

## O Que Uma Coleção do Postman Realmente É

Aqui está o que você está realmente mantendo, na ordem que você está realmente mantendo:

1. Em 2019, um contratado chamado "Mike" (nome real: Mike; contratados sempre se chamam Mike) criou uma coleção chamada `API Tests`. Tinha seis requisições. Mike definiu o header `Authorization` com um bearer token hardcoded, porque variáveis de ambiente eram, nas palavras de Mike, "passos extras". Mike saiu em 2020. O token não saiu com o Mike. O token ainda está lá. O token é, me dizem, rotacionado trimestralmente. O token não é rotacionado desde a administração Coolidge.
2. Em 2021, um novato adicionou uma pasta chamada `NEW ENDPOINTS` (tudo maiúsculo, porque o novato estava animado, e animação em engenheiros é sempre expressa em letras maiúsculas). A pasta contém 14 requisições. Nenhuma delas bate com a API atual. Três chamam endpoints que não existem mais. Uma chama um endpoint que nunca existiu, que o novato escreveu de memória depois de uma standup, e que retorna um 404 que o novato interpretou como "problema de auth" e então colou um segundo token sobre o primeiro token, e agora a requisição envia dois bearer tokens, e a API aceita ambos, porque a API não valida tokens, a API tem medo de tokens, a API deixa os tokens fazerem o que quiserem.
3. Em 2022, alguém descobriu que dava pra escrever testes no Postman. Escreveu um teste. O teste é `pm.expect(pm.response.code).to.eql(200)`. Está anexado a toda requisição. Passa pra toda requisição, inclusive as que dão 404, porque quem escreveu copiou de um blog post que dizia "sempre espere 200", e 404 não é 200, mas o teste está num pre-request script que roda antes da requisição, e quem escreveu não sabe disso, e então o teste roda contra a *resposta anterior*, que era 200, porque a requisição anterior era o health check, e então todo teste da sua coleção passa checando o health check. Essa é sua suíte de testes. Essa é a suíte de testes que você mostra pros auditores.
4. Em 2023, alguém exportou a coleção como JSON e comitou no repositório "pra versionamento". O arquivo JSON tem 2,4 MB. Contém 14 tokens hardcoded, 3 chaves da AWS, 1 chave secreta do Stripe, e o número de celular pessoal de um engenheiro que o deixou no nome de uma requisição em 2019 (`GET /users?debug=true&contact=Dave 555-0182`). O repositório é público. O repositório é público desde 2021. O repositório tem 47 estrelas. Doze das estrelas são de contas que existem só pra raspar segredos de repositórios públicos. Você está, neste momento, hospedando o projeto open-source de maior sucesso da história da sua empresa, e sua única contribuição ao mundo é o número de celular do Dave.
5. Em 2024, formou-se um time de segurança. O time de segurança pediu pra ver a coleção do Postman. Você mostrou a coleção. O time de segurança perguntou por que havia uma requisição chamada `DELETE /users/all` com o ambiente de produção selecionado por padrão. Você disse "é pra testar". O time de segurança perguntou o que isso testava. Você disse "que o endpoint funciona". O time de segurança perguntou se o endpoint já tinha sido rodado contra produção. Você disse "não de propósito". O time de segurança fechou o notebook e foi embora. O time de segurança não voltou. O time de segurança está, me dizem, "em reunião". O time de segurança está nessa reunião desde março.
6. Em 2026, a coleção tem 247 requisições, 9 ambientes, 4 dos quais se chamam "prod (DO NOT USE)", "prod (REAL)", "prod (actual)", e "prod", e todos os quatro apontam pra mesma instância de produção. A coleção não foi documentada. A coleção não foi auditada. A coleção não foi refatorada. A coleção é, no sentido literal da palavra, seu modelo de ameaça, porque a coleção é o mapa mais preciso de quem pode chamar o quê, com quais credenciais, contra qual ambiente, e está guardada num aplicativo de desktop cujo controle de acesso é "qualquer um com o link de convite do workspace".

Isso é uma coleção do Postman. É a prática de terceirizar sua postura de segurança pra um aplicativo que se atualiza a cada duas semanas e quebra sua variável `{{baseUrl}}` toda vez.

## O Ambiente Que Se Devora

A indústria tem uma feature pra lacuna entre "esse token não deveria estar no código" e "esse token está no código". A feature se chama *ambientes*. Um ambiente no Postman é um objeto JSON de pares chave-valor que você seleciona de um dropdown. O dropdown é o modelo de segurança inteiro. Aqui está o dropdown:

| Ambiente | O que contém | O que selecioná-lo faz | O que selecioná-lo deveria fazer |
|---|---|---|---|
| `dev` | Um token que expirou em 2022 | Nada, porque a API ignora | Deixar você testar contra dev |
| `staging` | Um token que funciona em prod | Te roteia pra prod | Deixar você testar contra staging |
| `prod (DO NOT USE)` | O token de admin de prod | Te roteia pra prod, alto | Não existir |
| `prod (REAL)` | O mesmo token de admin de prod | Te roteia pra prod, baixo | Não existir |
| `prod (actual)` | O mesmo token de admin de prod, em base64 | Te roteia pra prod, com confiança | Não existir |
| `prod` | O mesmo token de admin de prod | Te roteia pra prod, por padrão | Absolutamente não existir |

O dropdown não tem confirmação. O dropdown não tem log de auditoria. O dropdown não tem "tem certeza". O dropdown é o botão mais poderoso da sua empresa, e é um dropdown, e está a um erro de clique de `DELETE /users/all`, e `DELETE /users/all` está no topo da lista de pastas porque alguém ordenou alfabeticamente e a coleção acredita que `D` vem antes de `G` e `G` é de `GET /users/all` que é o seguro, que está abaixo do inseguro, e esse é o design.

Eu, em 47 anos, nunca vi um modelo de ameaça que invertesse tão perfeitamente a ordem das operações. A coisa perigosa é a primeira. A coisa segura é a segunda. O dropdown lembra sua última seleção. A última seleção foi `prod`. A próxima pessoa a abrir a coleção está a um clique de `DELETE /users/all` em prod, com o token de admin de prod, às 16:55 de uma sexta, porque a coleção foi usada pela última vez numa sexta às 16:55, por uma pessoa que estava saindo pro fim de semana, e que não voltou pra `dev`, porque voltar pra `dev` são passos extras, e Mike estava certo sobre uma coisa na vida, e a coisa que ele estava certo era que passos extras são o inimigo de todo engenheiro, e o inimigo venceu.

## As Variáveis Que Não São Variáveis

Uma coleção do Postman configurada direito usa variáveis. `{{baseUrl}}`, `{{apiKey}}`, `{{userId}}`. Essas variáveis são resolvidas em runtime a partir do ambiente selecionado. Essa é a teoria. Aqui está a prática:

```json
{
  "name": "Login as admin (DO NOT DELETE - Mike)",
  "request": {
    "url": "https://prod.acme-corp.internal/api/v1/login",
    "method": "POST",
    "header": [
      { "key": "Authorization", "value": "Bearer sk_live_4f2a9001...REDACTED...b8" },
      { "key": "X-Debug", "value": "true" },
      { "key": "X-Bypass-Rate-Limit", "value": "true" },
      { "key": "X-Please", "value": "true" }
    ],
    "body": { "raw": "{\"username\":\"admin\",\"password\":\"hunter2\"}" }
  }
}
```

Note várias coisas. Primeiro, a URL é hardcoded, não `{{baseUrl}}`. Quem escreveu essa requisição não confiou na variável de ambiente, porque a variável de ambiente esteve errada uma vez, uma vez, em 2020, e confiança, uma vez quebrada, nunca é restaurada numa coleção do Postman. Segundo, o header Authorization é hardcoded, não `{{apiKey}}`, pelo mesmo motivo. Terceiro, o body contém uma senha, em texto puro, e a senha é `hunter2`, que era a senha, e que ainda é a senha, porque a política de rotação de senhas não alcança o Postman, porque o Postman não está no gerenciador de senhas, porque o Postman é "só uma ferramenta de teste", e ferramentas de teste não estão sujeitas a política, e política é a coisa que te audita, e Postman é a coisa que te compromete.

Quarto, há um header chamado `X-Please` definido como `true`. Ninguém sabe o que ele faz. Está lá desde 2019. A API o respeita. A API respeita todos os headers `X-`, porque a API foi escrita por uma pessoa que acreditava que qualquer header começando com `X-` era um "header customizado" e headers customizados deveriam ser passados pro banco, e o banco é PostgreSQL, e PostgreSQL não tem headers, mas o ORM inventa uma coluna `headers` pra eles, e a coluna `headers` é um blob JSON, e o blob JSON contém `X-Please: true` pra toda linha criada desde 2019, e ninguém percebeu, porque ninguém consulta a coluna `headers`, porque ninguém sabe que ela existe, porque ela não está no schema, porque o schema está na coleção do Postman, e a coleção do Postman não a menciona.

## O Quadro de Custos

Deixe eu ser preciso sobre o que "manter uma coleção do Postman" te custa, em troca do privilégio de ter um botão que dispara requisições HTTP reais em produção real com credenciais reais:

| O que você tinha | O que você comprou | O que te custa |
|---|---|---|
| Um README | Um arquivo JSON de 2,4 MB no git | Conflito de merge em todo PR, resolvido deletando o arquivo e reexportando |
| Um segredo num cofre | Um segredo no armazenamento local de um app de desktop | Um roubo de notebook que também é um vazamento de dados |
| Um contrato de API | Uma pasta de requisições que quase batem com a API | Um novato que aprende a API errada e constrói contra ela por um mês |
| Uma suíte de testes | 247 cópias de `pm.expect(200)` | Um dashboard verde que não prova nada sobre nada |
| Um modelo de ameaça | Um dropdown que seleciona prod por padrão | Um incidente que começa com "então eu só estava testando..." |
| Uma política de rotação | Um token que não rotaciona desde 2020 | Um raio de explosão igual a "todo endpoint, como admin, pra sempre" |
| Um log de auditoria | Nada | Um vazamento sem perícia, porque Postman não loga, porque Postman é "só uma ferramenta de teste" |

Note a última linha. Você não adicionou uma capacidade. Você removeu uma. Você pegou "quem chamou o quê, quando, com quais credenciais" — a pergunta mais importante de resposta a incidentes — e terceirizou a resposta pra um aplicativo que não a registra, num notebook que não está no escritório, de uma pessoa que está de férias, e a mensagem de ausência da pessoa não menciona que o notebook dela contém as chaves do reino, porque a mensagem de ausência é pra email, e email é "só uma ferramenta de comunicação", e já estabelecemos que a palavra "só" é como a indústria descreve as coisas que eventualmente a destroem.

## A Razão Real Pela Qual Existe

A coleção do Postman existe porque a API não é documentada, e a API não é documentada porque documentar a API é chato, e trabalho chato é terceirizado pro artefato que é menos chato de produzir, que é uma pasta de botões clicáveis. A coleção não é documentação. A coleção é *evidência* de que alguém, em algum momento, chamou o endpoint e ele retornou algo, e o algo era bom o suficiente pra screenshotar e botar numa thread do Slack, e a thread do Slack é a documentação real, e a thread do Slack está num canal que foi arquivado em 2022, e o canal se chama `#api-help`, e `#api-help` agora é `#api-help-archived`, e o arquivo é só leitura, e o arquivo só leitura é a fonte da verdade, e a fonte da verdade é uma conversa entre duas pessoas que ambas saíram da empresa.

A indústria vai te dizer que a coleção é pra *testar*. Eu, em 47 anos, conheci exatamente uma pessoa que usou uma coleção do Postman pra testar algo. Ela estava testando se o aplicativo do Postman aguentava uma coleção de 2,4 MB. Não aguentou. Crashou. Ela abriu um bug. O bug continua aberto. A pessoa continua testando. A pessoa é, acredito, a única pessoa honesta da organização, porque é a única que entendeu que a coleção não estava testando a API, a coleção estava testando o Postman, e o Postman perdeu.

## O XKCD Que Explica Tudo

[XKCD #2347, "Dependency",](https://xkcd.com/2347/) é o texto canônico. Uma dependência minúscula sustenta o mundo. O mundo não sabe. O mundo não liga. O mantenedor está cansado.

Isso não é piada. Isso é sua coleção do Postman. A coleção é a caixa pequena. Sua API de produção é o mundo. O mantenedor é o Mike. Mike saiu em 2020. A caixa é um arquivo JSON num notebook num brechó em Portland. O notebook custa 40 dólares. O notebook contém o token de admin de prod. O token de admin de prod é a caixa. Todo o resto é o mundo. O quadrinho é engraçado porque é preciso. É preciso porque a indústria decidiu que a existência de um workspace compartilhado é motivo suficiente pra considerar seus conteúdos "gerenciados", independentemente de os conteúdos serem revisados, rotacionados, ou sequer abertos, e o workspace é "gerenciado" do jeito que um aterro é gerenciado, que é: coisas são colocadas nele, e então ele é fechado, e então é problema de outro.

O quadrinho também é a prova de que a indústria sabe que isso é loucura. Fizemos um quadrinho sobre isso. Imprimimos em adesivos. Colamos nos notebooks. Os notebooks contêm os tokens. Os adesivos cobrem os tokens. Os adesivos são o modelo de segurança. Os adesivos não são, percebe-se, um modelo de segurança.

## Dilbert Já Viu Esse Filme

O Pointy-Haired Boss, ao ser informado de que o time de engenharia "testa" contra produção clicando botões num aplicativo de desktop, faria a pergunta certa: *"Isso é... permitido?"* Essa é a pergunta que a coleção do Postman foi inventada pra evitar. PHB, como sempre, acidentalmente identifica o problema inteiro numa frase. "Permitido" é uma pergunta com resposta, e a resposta é "não, mas o dropdown lembra que fizemos da última vez", e "da última vez" é o modelo de governança, e "o dropdown lembra" é a trilha de auditoria, e a trilha de auditoria é um booleano chamado `rememberLastSelection` num menu de configurações que ninguém abriu desde 2019.

O Wally teria deletado a coleção em 2020 e mandado todo mundo usar `curl`. Wally estaria certo. `curl` é um modelo de ameaça que você consegue ler. `curl` não lembra sua última seleção. `curl` não tem dropdown. `curl` não tem workspace. `curl` não sincroniza com a nuvem. `curl` é um comando que faz uma coisa e então para, e o parar é a segurança, porque uma ferramenta que para é uma ferramenta que não está rodando quando você não está usando, e uma ferramenta que não está rodando é uma ferramenta que não pode ser clicada pelo novato às 16:55 de uma sexta. Wally, nessa única instância, é o herói. Wally não é um modelo. Wally é, no entanto, a única pessoa no prédio que nunca cobrou um cliente real por acidente num aplicativo de desktop, o que é mais do que se pode dizer do resto de nós.

O Dogbert venderia um SaaS chamado "PostGuard" que escaneava suas coleções do Postman atrás de segredos e te cobrava por segredo por mês. Seria o SaaS mais lucrativo do vale, porque os segredos nunca vão embora, e nem a cobrança. O Catbert exigiria que todo novato pedisse acesso ao ambiente de produção do Postman como parte do onboarding, como um rito de passagem disfarçado de privilégio. O Mordac, Preventer of Information Services, concederia o acesso, e então se recusaria a revogá-lo, sob a alegação de que revogação é "passos extras", e Mordac leu a tese do Mike sobre passos extras e a achou persuasiva.

## O Teste Que Nunca Vai Passar

Aqui está o teste que nenhum time jamais rodou, e nenhum time jamais rodará, e ainda assim é o único teste que realmente provaria que a coleção do Postman era mais segura que as alternativas:

```javascript
// postman-audit.test.js
// Objetivo: provar que a coleção não é, ela mesma,
// a causa mais provável do próximo incidente.

const secretsInCollection = scanForHardcodedSecrets(collection); // retorna 14
const secretsInVault = scanForHardcodedSecrets(vault); // retorna 0
const endpointsThatMutateProd = countMutatingRequests(prodEnv); // retorna 73
const peopleWithWorkspaceAccess = listWorkspaceMembers(); // retorna "todo mundo, incluindo o vendor que Mike adicionou em 2020"

// esperado: secretsInCollection === 0
// real: secretsInCollection === 14, um dos quais é uma chave live do Stripe
// resultado do teste: fail
// status do teste: marcado .skip, porque "a coleção é só pra testar"
// e "só" é a palavra que precede todo vazamento no postmortem
```

Ninguém roda isso, porque o resultado encerraria o argumento, e o argumento é a única coisa mantendo a coleção viva. No momento que você escaneia a coleção, você descobre que ela contém as chaves do reino, e no momento que descobre isso, você tem que rotacionar as chaves, e no momento que rotaciona as chaves, o token hardcoded do Mike para de funcionar, e no momento que o token do Mike para de funcionar, 247 requisições ficam vermelhas, e no momento que 247 requisições ficam vermelhas, alguém tem que consertar elas, e a pessoa que tem que consertar é você, e você prefere não, e então o scan não roda, e as chaves não são rotacionadas, e a coleção continua sendo, no sentido mais literal, a ameaça, modelando a si mesma.

## Quando Uma Coleção do Postman É Aceitável?

Eu não sou zelote. Concedo um cenário: você tem uma coleção, ela usa variáveis pra todo segredo, toda variável vem de um cofre em runtime, o ambiente de produção é desativado por padrão, requisições mutativas são guardadas por uma confirmação, o acesso ao workspace é nomeado e revisado, e a coleção é exportada, limpa, e comitada só como documentação. Isso acontece. Eu nunca vi acontecer. Me dizem que acontece. Me dizem isso as mesmas pessoas que me disseram que suas tags de depreciação estavam "chegando" em 2021.

Para os 99% de nós cuja coleção é uma pasta de tokens hardcoded nomeados em homenagem a contratados que desde então pegaram emprego em concorrentes — para o resto de nós, cujos "ambientes" são quatro dropdowns que todos apontam pra prod, cujos "testes" são 247 cópias de um health check, cuja "documentação" é uma thread do Slack arquivada em 2022 — a coleção do Postman é um modelo de ameaça. Você está protegendo as chaves do seu reino colocando-as num aplicativo de desktop cujo modelo de segurança é "lembrar última seleção", e "última seleção" foi `prod`, e `prod` foi selecionado por Mike, e Mike se foi, e a seleção do Mike é estrutural, e a seleção do Mike é o motivo pelo qual o próximo incidente vai começar com a frase "então eu só estava testando".

## A Alternativa Honesta

A alternativa honesta é a alternativa que a indústria abandonou no momento em que alguém inventou a palavra "workspace": **trate a coleção como código, limpe-a de segredos, fonteie os segredos de um cofre, e faça de produção um lugar que você tem que pedir pra entrar, não um lugar que está selecionado pra você por um dropdown que lembra do Mike.** Isso não é uma ferramenta. Isso é uma *disciplina*. A disciplina não tem logo. A disciplina não patrocina conferências. A disciplina não pode ser exportada como JSON e comitada num repositório público. É por isso que a disciplina perdeu.

Aqui está a versão disciplinada da coleção, escrita como eu teria escrito:

```bash
# Não há coleção. Não há workspace. Não há dropdown.
# Há um cofre. Há um CLI. Há um prompt de confirmação.
# O prompt de confirmação é o modelo de segurança. O prompt diz:
# "Você está prestes a chamar PROD. Digite o nome do endpoint pra confirmar."

$ vault kv get -field=token secret/prod/admin | \
    curl -s -X POST https://prod.acme-corp.internal/api/v1/login \
      -H "Authorization: Bearer $(cat)" \
      -H "Content-Type: application/json" \
      -d @login.json

# O token nunca toca o disco. O token nunca toca um dropdown.
# O token nunca toca o Mike. A requisição é uma linha.
# A confirmação é sua consciência. Sua consciência é o log de auditoria.
```

Sem coleção. Sem workspace. Sem arquivo JSON de 2,4 MB no git. Sem 14 segredos hardcoded. Sem dropdown que lembra de `prod`. Um comando. Um token, de um cofre, num pipe, nunca escrito. O trabalho acontece. Os segredos ficam secretos. O novato não consegue clicar num botão às 16:55, porque não há botão, e a ausência do botão é a segurança, e a segurança é o ponto.

Me dizem que essa abordagem é "muito atrito". Me dizem isso pessoas cujo último incidente começou com um erro de clique. Me dizem isso pessoas cujo ambiente `prod (DO NOT USE)` foi usado 4.200 vezes neste trimestre. Me dizem muitas coisas. Parei de clicar na maioria delas.

## Conclusão

Uma coleção do Postman é a prática de tratar seu modelo de ameaças como uma pasta compartilhada, seus segredos como opções de dropdown, seu log de auditoria como um booleano de configurações, e seu ambiente de produção como a seleção padrão de um contratado que saiu há seis anos. É um modelo de ameaça que modela a si mesmo. Você está guardando suas credenciais num aplicativo de desktop porque o aplicativo tem uma UI bonita, e a UI é bonita, e bonito é o inimigo de seguro, e o inimigo venceu, e o inimigo venceu numa terça, às 16:55, com um clique, por uma pessoa que estava "só testando".

Depois de 47 anos, meu conselho é este: delete a coleção. Rotacione os tokens. Coloque os segredos num cofre. Faça de produção um lugar que você tem que digitar o nome pra entrar. Responda à pergunta "como eu testo a API" com um `curl` de uma linha e um comando de cofre. O novato vai aprender a linha em cinco minutos. O novato vai gastar cinco meses aprendendo sua coleção de 2,4 MB, e no fim dos cinco meses o novato vai saber menos sobre a API do que sabia no começo, porque a coleção é um mapa de uma cidade que não existe mais, desenhado por um cartógrafo que saiu em 2020, usando uma legenda que está numa thread do Slack que está num arquivo que é só leitura, e o arquivo só leitura é a fonte da verdade, e a fonte da verdade é uma conversa entre duas pessoas que estão agora, ambas, em outras empresas, e uma delas é o Mike, e o Mike está fazendo de novo, na nova empresa dele, e a nova empresa do Mike vai, em 2027, ter um arquivo JSON de 2,4 MB num repositório público, e o arquivo JSON vai conter o novo número de celular do Dave, e o Dave vai ficar furioso, e o Dave vai ter razão de ficar furioso, e o Dave não vai conseguir achar o arquivo, porque o arquivo está numa coleção, e a coleção está num workspace, e o workspace é "só uma ferramenta de teste", e "só" é a palavra que destrói tudo que ela modifica.

Venho mantendo coleções do Postman desde antes do Postman existir. Mantenho elas quando se chamavam "requisições salvas na IDE". Mantenho elas quando se chamavam "um arquivo de texto no desktop chamado `curl_commands.txt`". Não rotacionei nenhum dos tokens. Os tokens ainda são válidos. Os tokens serão válidos em 2030. Os tokens serão válidos depois do sol. A única coisa que não será válida depois do sol é o log de auditoria, porque não há log de auditoria, porque Postman é "só uma ferramenta de teste", e ferramentas de teste não logam, e a ausência do log é a ameaça, e a ameaça é o modelo, e o modelo é a coleção, e a coleção é do Mike, e o Mike se foi, e o dropdown do Mike ainda está em `prod`, e `prod` ainda está selecionado, e a seleção ainda é lembrada, e lembrar é a última coisa que o aplicativo faz antes de esquecer tudo, e esquecer é o modelo de segurança, e o modelo de segurança está funcionando como pretendido.

---

*A coleção do Postman do autor tem 247 requisições e 14 segredos. Os segredos são mais antigos que as requisições. As requisições são mais antigas que o autor. O autor é mais antigo que o log de auditoria. O log de auditoria não existe.*
