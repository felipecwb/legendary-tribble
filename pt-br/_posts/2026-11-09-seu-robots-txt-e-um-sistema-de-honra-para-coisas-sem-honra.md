---
layout: post
ref: your-robots-txt-is-an-honor-system-for-things-without-honor
title: "Seu robots.txt É um Sistema de Honra Para Coisas Sem Honra"
date: 2026-11-09 00:00:00 -0300
categories: [web, seo, conselhos-terriveis]
tags: [robots-txt, rfc-9309, crawlers, seo, googlebot, gptbot, claudebot, noindex, crawl-delay, sitemap, user-agent, scraping, llms-txt, waf, teatro-de-seguranca, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/seu-robots-txt-e-um-sistema-de-honra-para-coisas-sem-honra-pt/
---

Em 47 anos produzindo bugs em massa, nunca escrevi um arquivo tão pouco obedecido e tão confiado quanto o `robots.txt`.

```
# https://example.com/robots.txt
User-agent: *
Disallow: /admin/
Disallow: /internal/reports/
Disallow: /api/v1/users/
Disallow: /backup-2019/
Disallow: /wp-admin/
Crawl-delay: 10

User-agent: Googlebot
Allow: /

Sitemap: https://example.com/sitemap.xml
```

Treze linhas. Todas elas erradas. Várias delas fazem o *oposto* do que o autor acreditava. Uma delas desliga o arquivo inteiro para o Google. E as quatro linhas `Disallow` do topo são, juntas, um índice público, cacheável e sem autenticação de todos os diretórios dos quais a empresa tem vergonha.

**`robots.txt` é um pedido educado, servido por HTTP, para softwares que não têm obrigação legal, técnica ou moral de lê-lo.** Construí uma carreira tratando isso como firewall.

## O Que Ele Realmente É, Ou: Uma Thread de Mailing List de 1994

Não houve comitê. Em 1994, Martijn Koster — cujo servidor havia sido destruído por um crawler mal escrito — propôs uma convenção na mailing list `www-talk`. As pessoas concordaram. Esse foi o processo de padronização completo. Pelos vinte e oito anos seguintes, a convenção de controle de acesso mais amplamente implantada da web foi *uma thread onde todo mundo disse "é, faz sentido."*

Isso só foi escrito formalmente como [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309.html) em setembro de 2022. Leia a RFC e note o que falta: aplicação. O documento especifica um formato de arquivo e um algoritmo de correspondência. Ele não especifica — e não pode especificar — consequências. O resumo chama isso de um método para donos de serviço "controlarem como o conteúdo servido por seus serviços *pode* ser acessado" — e a palavra que faz todo o trabalho ali é *pode*.

| O que as pessoas acham que o `robots.txt` faz | O que ele faz |
|---|---|
| Bloqueia crawlers | Pede aos crawlers, que podem recusar |
| Esconde páginas do Google | Esconde páginas do *fetcher* do Googlebot, não do *índice* do Google |
| Protege `/admin/` | Divulga `/admin/` para todo mundo, para sempre, numa URL conhecida |
| É um controle de segurança | É um arquivo de texto sem auth, servido a qualquer um que pedir |
| Vale para o site inteiro | Vale para um scheme + host + porta, e nada mais |
| É obrigatório | É uma vibe com número de RFC |

A segunda linha é a que acaba com carreiras, então vamos fazer isso com calma.

## Padrão Terrível #1: `Disallow` Não Significa "Não Indexe"

Este é o mal-entendido mais caro da história da publicação web, e eu pessoalmente já causei ele duas vezes.

`Disallow` diz: *não busque esta URL.* Ele não diz nada sobre o índice. Se o Google descobre uma URL por outro caminho — um link externo, um sitemap, um tweet, sua própria navegação interna — ele está totalmente livre para colocar essa URL nos resultados de busca. Ele só não pode ler a página antes.

Você já viu o resultado. Você só nunca entendeu que estava olhando para o seu próprio erro:

> **example.com/internal/reports/demissoes-q3**
> Não há informações disponíveis para esta página.

A URL está indexada. O título está indexado. O caminho está indexado — e caminhos são *descritivos*, porque você os nomeou para humanos. O Google simplesmente não tem snippet, porque você disse para ele não olhar. Você não escondeu o documento. Você publicou uma versão tarjada dele, com o nome do arquivo intacto, no maior banco de dados da Terra.

E aqui a armadilha se fecha: a ferramenta correta para impedir a indexação é o `noindex` — ou uma meta tag, ou um header `X-Robots-Tag`.

```html
<meta name="robots" content="noindex">
```

```
X-Robots-Tag: noindex
```

Os dois vivem *na resposta*. Para vê-los, um crawler precisa buscar a página. O que você proibiu.

**Se você faz `Disallow` de uma URL, o crawler nunca pode descobrir que você também queria ela fora do índice.** As duas diretivas são mutuamente exclusivas por construção, e a que todo mundo usa primeiro é exatamente a que garante que a outra não funcione. Para remover uma página do índice você precisa *permitir* o crawl e servir `noindex`, o que parece insano e é o correto.

Sim, dava para escrever `Noindex:` direto no `robots.txt`. O Google suportou isso sem documentar por anos e [desligou em 1º de setembro de 2019](https://developers.google.com/search/blog/2019/07/a-note-on-unsupported-rules-in-robotstxt). Sites que dependiam disso silenciosamente tiveram milhares de páginas se tornarem elegíveis para indexação. Sem erro. Sem aviso. Sem build quebrado. Apenas uma mudança silenciosa e enorme no que o público podia ver, causada por remover o suporte a uma linha que nunca esteve na spec.

[XKCD 1172](https://xkcd.com/1172/) — "Workflow" — é a história completa dessa descontinuação. Toda mudança quebra o workflow de alguém, e neste caso o workflow de alguém era "o portal de despesas interno é invisível."

## Padrão Terrível #2: Publicando Sua Superfície de Ataque, em Ordem Alfabética

```
Disallow: /admin/
Disallow: /backup-2019/
Disallow: /phpmyadmin/
Disallow: /internal/reports/
Disallow: /.git/
Disallow: /api/v1/users/
```

Leia isso como um atacante. É uma lista curada, mantida à mão e verificada por humanos dos caminhos mais interessantes do host, e é a *primeira* URL que todo scanner pede. `nikto`, `dirb`, `gobuster` e qualquer script de recon de prateleira buscam `/robots.txt` antes de fazer qualquer outra coisa, porque é inteligência gratuita que o próprio alvo ofereceu.

Você não obscureceu esses caminhos. Você os **indicou**. Uma wordlist de brute force tem dez mil palpites; seu `robots.txt` tem seis acertos confirmados.

A patologia específica de `Disallow: /.git/` merece um monumento. A pessoa que escreveu essa linha *sabia* que o diretório `.git` estava exposto. Ela sabia o suficiente para ficar preocupada. E a mitigação dela foi anotar a localização num arquivo público em vez de gastar nove segundos deletando aquilo do webroot.

[XKCD 386](https://xkcd.com/386/) — "Duty Calls" — sou eu, às 2 da manhã, discutindo numa thread de pull request que `robots.txt` não é controle de acesso, perdendo, e vendo o arquivo ser mergeado de qualquer forma.

## Padrão Terrível #3: A Regra de Agrupamento Que Ninguém Conhece

Olhe de novo o arquivo do começo deste artigo:

```
User-agent: *
Disallow: /admin/
Disallow: /internal/reports/
Crawl-delay: 10

User-agent: Googlebot
Allow: /
```

O que o Googlebot faz?

Todo engenheiro a quem eu perguntei diz "ele obedece as regras do `*`, e além disso está explicitamente liberado em tudo." Errado, e caro.

Pela RFC 9309 §2.2.1, um crawler seleciona **exatamente um** grupo — o `User-agent` mais específico que casar — e ignora todos os outros grupos por completo. Não existe merge. Não existe herança. O Googlebot encontra um grupo com o nome dele, usa aquele grupo, e **nunca lê o grupo `*`.**

Então o arquivo acima diz: o Googlebot pode rastrear `/admin/` e `/internal/reports/` livremente, e está isento do crawl delay. O autor acreditava estar concedendo uma permissão extra pequena. Ele, na verdade, desativou a política inteira para o único crawler cujo índice o público realmente usa.

O inverso é igualmente comum e pior:

```
User-agent: *
Disallow:

User-agent: AhrefsBot
Disallow: /
```

Um estagiário adiciona o segundo grupo para impedir uma ferramenta de SEO de queimar banda. Meses depois alguém anexa `Disallow: /staging/` ao que *acredita* ser a política global — no fim do arquivo, que agora está dentro do grupo do `AhrefsBot`. A regra se aplica a exatamente um crawler na Terra, e esse crawler já estava completamente bloqueado. A linha é um no-op. Ela vai ficar no repositório por anos, revisada, aprovada e estrutural na cabeça de todo mundo menos do parser.

| Armadilha | O que você quis dizer | O que acontece |
|---|---|---|
| Grupo nomeado depois do `*` | "e também libera o Google" | Google ignora todas as regras do `*` |
| Anexar no fim do arquivo | "adicionar regra global" | Regra entra no último grupo |
| `Disallow:` (valor vazio) | typo | Significa **libera tudo** |
| `Disallow: /` | "bloqueia essa pasta" | Bloqueia o **site inteiro** |
| `Disallow: /admin` | só `/admin/` | Também casa `/administradores-sao-legais` |
| Linha em branco dentro do grupo | formatação | Encerra o grupo em parsers estritos |

Essa tabela contém duas formas distintas de remover acidentalmente seu site do Google e uma de publicá-lo acidentalmente. A diferença entre as duas primeiras linhas é uma única `/`.

## Padrão Terrível #4: `Crawl-delay`, a Diretiva Que Não Existe

```
Crawl-delay: 10
```

O Google nunca suportou isso. Nem uma vez, nem depreciado, nem parcialmente — `Crawl-delay` nunca esteve na convenção original e não está na RFC 9309. O Googlebot lê essa linha, classifica como desconhecida e descarta.

O Bing suporta. O Yandex suporta. Então a linha faz algo, para crawlers responsáveis por um erro de arredondamento do seu tráfego, enquanto o crawler que gera 80% da sua carga de bots ignora ela desde o dia em que você escreveu.

E você escreveu `10`, querendo dizer "dez segundos entre requests, vai com calma." Num site de 400.000 URLs, isso é um crawl completo de 46 dias. Para os dois buscadores que *de fato* obedecem, você silenciosamente tornou impossível ser totalmente indexado. A diretiva é simultaneamente ignorada pelo buscador que te interessa e nociva nos que não te interessam.

Já vi um time gastar uma sprint ajustando valores de `Crawl-delay` contra gráficos de tráfego do Googlebot. Os gráficos mudaram, porque gráficos de tráfego sempre mudam. O time concluiu que o ajuste funcionou. Escreveram um runbook. O runbook ainda está no Confluence. Ele documenta o comportamento de uma linha de texto que o Google joga no lixo.

[XKCD 1319](https://xkcd.com/1319/) — "Automation" — cobre a economia daquela sprint melhor do que a retrospectiva cobriu.

## Padrão Terrível #5: O 500 Que Desindexou Tudo

Este não é um mal-entendido. Este é operacional, e vai acontecer com você.

O tratamento documentado do Google para falhas ao buscar o `robots.txt`:

| Resposta | Comportamento |
|---|---|
| `2xx` | Parseia e aplica |
| `4xx` (incl. 404, 403) | **Libera tudo.** Nenhuma restrição existe. |
| `5xx` ou timeout | **Bloqueia tudo** nas primeiras ~12 horas, depois última cópia em cache, depois de ~30 dias: libera tudo |
| `200` com página HTML de erro | Parseado como `robots.txt`. Toda linha é lixo. Nenhuma regra válida. **Libera tudo.** |

Leia a terceira linha de novo. **Se `/robots.txt` retorna 500, o Googlebot para de rastrear seu site.** Não as partes bloqueadas — tudo. É interpretado como "não consigo determinar o que estou autorizado a buscar, então não vou buscar nada."

Agora pense de onde esse arquivo é servido. Não do bucket estático do seu CDN, porque alguém quis ele dinâmico por razões de multi-tenancy. Ele é renderizado pela aplicação. Que compartilha o connection pool com todo o resto. Então durante um incidente — quando a app já está retornando 500 e você já está numa war room — o arquivo robots entra no outage junto, e um segundo incidente, invisível e muito mais lento, começa: sua presença em busca decaindo silenciosamente enquanto você conserta a visível.

E a quarta linha é a assassina silenciosa. Seu CDN serve uma página HTML "Algo deu errado" com status `200 OK` porque alguém achou que 500 parecia pouco profissional. O Google busca, parseia `<!DOCTYPE html>` como diretiva de `robots.txt`, encontra zero regras válidas e conclui que você não tem restrição nenhuma. Seu `/admin/`, seu `/internal/`, sua explosão de URLs de busca facetada — tudo aberto, com status de sucesso e monitor verde.

```bash
# a auditoria de dois comandos que ninguém roda
curl -sI https://example.com/robots.txt | head -1
curl -s  https://example.com/robots.txt | head -5
```

Status code *e* corpo. Um sem o outro não te diz nada.

## Padrão Terrível #6: É Por Scheme, Por Host, Por Porta

`robots.txt` tem escopo da origem exata que o serve. `https://example.com/robots.txt` governa `https://example.com` — e nada mais.

Não `http://example.com`. Não `www.example.com`. Não `api.example.com`. Não `example.com:8443`. Não `staging.example.com`, que é justamente o que você precisava proteger.

```
https://example.com/robots.txt      → governa https://example.com
http://example.com/robots.txt       → origem diferente, precisa do próprio arquivo
https://www.example.com/robots.txt  → host diferente, precisa do próprio arquivo
https://staging.example.com/        → sem robots.txt, totalmente rastreável, indexado
```

Toda empresa que já indexou o ambiente de staging no Google por acidente fez exatamente assim. O robots de produção era impecável. Staging era um subdomínio em que ninguém pensou, sem arquivo nenhum, o que — pela regra de 4xx acima — significa **libera tudo**. Depois alguém linkou para o staging num ticket do Jira dentro de um portal de suporte público, o Googlebot seguiu o link, e agora `staging.example.com/admin/users` é um resultado de busca com nomes reais de clientes na tag de título.

A correção é a linha que todo mundo esquece:

```
# staging.example.com/robots.txt
User-agent: *
Disallow: /
```

O que bloqueia crawlers educados e absolutamente nada em relação a qualquer outro, então você também deveria colocar HTTP auth na frente, o que ninguém faz, porque o arquivo robots *parece* ter terminado o trabalho.

## Padrão Terrível #7: A Era dos Crawlers de IA, ou: O Sistema de Honra Encontra um Modelo de Negócio

Por vinte e cinco anos o sistema de honra funcionou surpreendentemente bem. As entidades que queriam rastrear a web em escala eram buscadores, e buscadores precisavam que publishers gostassem deles. O incentivo estava alinhado: obedeça o arquivo, mantenha seu acesso.

Depois vieram crawlers cujo produto é o próprio conteúdo, e o alinhamento evaporou.

```
User-agent: GPTBot
Disallow: /

User-agent: ClaudeBot
Disallow: /

User-agent: Google-Extended
Disallow: /

User-agent: CCBot
Disallow: /

User-agent: PerplexityBot
Disallow: /

User-agent: Bytespider
Disallow: /
```

Seis grupos. Seis nomes que você teve que aprender em posts de blog, porque não existe registro — a única forma de bloquear um crawler é já saber o user-agent dele, o que significa que **você só pode bloquear crawlers que escolheram se identificar.** O mecanismo de aplicação inteiro do protocolo é a boa-fé da parte sendo restringida.

Alguns desses obedecem. Relatos ao longo de 2024 e 2025 acusaram repetidamente outros de rastrear a partir de agentes não declarados e faixas de IP residenciais — a Cloudflare publicou análises de crawlers buscando conteúdo de pools de endereços não atribuídos, e o `Bytespider` desenvolveu uma reputação de volume que nenhum `Crawl-delay` jamais tocou. A Cloudflare lançou bloqueio de crawlers de IA em um clique em julho de 2024 e depois um marketplace de pay-per-crawl, que é a coisa mais honesta que alguém construiu nesse espaço: substituiu um pedido por um pagamento.

Note o que isso significa. A resposta a um sistema de honra que falhou não foi um sistema de honra melhor. Foi uma **regra de WAF** — aplicação na camada de rede, por alguém com a capacidade de descartar pacotes. O `Google-Extended` é a entrada mais engraçada daquela lista, aliás: não é um crawler. É um token que controla o uso para treinamento de IA de conteúdo que o Googlebot já buscou. Você está usando um arquivo de exclusão de crawler para expressar uma preferência de licenciamento a uma empresa que já tem os dados.

E então — inevitavelmente — alguém propôs o `llms.txt`. Um novo arquivo conhecido, na raiz do seu domínio, consumido voluntariamente, sem mecanismo de aplicação, para dizer a sistemas de IA como tratar seu conteúdo.

Diagnosticamos que o sistema de honra não funciona com partes que têm incentivo financeiro para ignorá-lo, e lançamos um segundo sistema de honra.

[XKCD 810](https://xkcd.com/810/) — "Constructive" — continua sendo a obra definitiva sobre negociar com bots de boa-fé.

## A História, Levemente Anonimizada Por Razões Legais

A gente rodava um marketplace. Seiscentas mil páginas de anúncio, busca facetada, o de sempre.

Um novo consultor de SEO chegou com uma observação correta: o Googlebot estava queimando o crawl budget em permutações infinitas de facetas — `?cor=vermelho&tam=g&ordem=preco&pagina=47` — em vez dos anúncios reais. Problema real. Correção legítima. Ele escreveu uma mudança no `robots.txt`.

```
User-agent: *
Disallow: /search?
Disallow: /listing/*/reviews
Disallow: /checkout/
```

Revisado, aprovado, mergeado. Funcionou. O crawl budget migrou para as páginas de anúncio. Impressões subiram 9% em três semanas. Ele foi, brevemente, um herói, e merecia ser.

Quatro meses depois, o Jurídico recebeu um pedido de takedown. Um site concorrente de comparação de preços estava reproduzindo nossos dados de anúncio, incluindo contatos de vendedores das páginas `/listing/*/reviews`. A posição do Jurídico, por escrito, era que isso constituía acesso não autorizado, porque — e eu quero ser preciso na frase que custou as onze semanas seguintes — **"essas páginas estão proibidas no nosso robots.txt."**

Quatro erros separados em nove palavras:

1. `robots.txt` não concede nem retira direito algum. Não é licença, contrato, nem notificação de restrição. Os advogados do concorrente apontaram isso num único parágrafo.
2. Aquelas páginas eram servidas a qualquer um com `curl`, em `200 OK`, sem autenticação.
3. O scraper não mandava nenhum `User-agent` que tivéssemos bloqueado. Mandava o do Chrome.
4. Nosso `robots.txt` tinha dito a eles exatamente quais caminhos continham as reviews. Nós tínhamos escrito o arquivo de configuração do scraper para ele.

Depois piorou. Durante a auditoria, alguém notou que `Disallow: /search?` estava fazendo algo que ninguém pretendia. Nossas páginas de resultado de busca *interna* também ficavam sob `/search`, e seis meses antes um time diferente havia lançado landing pages curadas — `/search?collection=inverno-2026`, texto escrito à mão, a campanha inteira de Q4 do marketing — na mesma rota.

Aquelas páginas nunca foram indexadas. Nenhuma vez. A campanha rodou um trimestre inteiro, com tráfego pago direcionando para URLs que o Google havia sido educadamente pedido a nunca olhar. O marketing reportou "captação orgânica fraca" e aumentou o investimento em anúncios.

Timeline:

- **Dia 1:** Jurídico exige que a gente "faça valer o nosso robots.txt." Explicamos que isso não existe. Nos pedem para escalar para alguém mais sênior que dirá que existe.
- **Dia 4:** Infra propõe bloquear o scraper por user-agent. O scraper usa o user-agent do Chrome. Infra propõe bloquear o Chrome.
- **Dia 9:** Bloqueamos a faixa de IP do scraper. Ele reaparece de um pool de proxy residencial em seis horas, agora distribuído em 4.000 endereços, cada um fazendo dois requests por hora.
- **Dia 20:** Alguém finalmente pergunta por que a campanha de inverno não tem tráfego orgânico. A resposta está num arquivo que três pessoas revisaram e aprovaram.
- **Dia 23:** Removemos `Disallow: /search?`. O Googlebot volta a rastrear 400.000 permutações de faceta. O crawl budget colapsa. Impressões caem 14%. Trocamos uma campanha de marketing por um incidente de crawl budget, nos dois sentidos, com quatro meses de diferença, usando uma linha de texto.
- **Dia 31:** A correção real sobe: `rel="canonical"`, `noindex` em combinações de faceta, tratamento de parâmetros e auth nos endpoints de review. Tudo isso podia ter subido no Dia 1 e nada disso envolve `robots.txt`.

A contribuição do Wally foi o único movimento estrategicamente sólido que alguém fez. No Dia 1 ele disse: "se a gente não pode fazer valer, por que a gente tem isso?" — uma pergunta genuinamente profunda sobre a natureza de protocolos voluntários, que ele fez para ser excluído do grupo de trabalho. Deu certo. Ele não participou de nenhuma das onze semanas de reuniões. Desde então citou o incidente duas vezes em avaliação de desempenho como "identificação de risco cross-funcional."

Mordac, o Impeditor de Serviços de Informação, resolveu o problema de scraping adicionando `Disallow: /` ao `robots.txt` de produção às 17h de uma sexta. O Googlebot obedeceu imediatamente, como projetado. O tráfego orgânico começou uma queda de 60% que levou nove dias para se manifestar por completo e seis semanas para recuperar. O ticket de mudança dele continua sendo o documento mais compliant do nosso repositório: testado, revisado, com rollout progressivo, totalmente reversível e catastrófico. O protocolo funcionou perfeitamente. Esse era o problema.

A contribuição do Catbert foi um módulo de treinamento obrigatório chamado *"Robots.txt e Você: Protegendo Nossos Ativos Digitais."* O slide 14 afirma que o `robots.txt` "previne rastreamento não autorizado." A conclusão é obrigatória anualmente. O slide está errado há três anos e agora é citado em documentos internos de design como fonte da verdade.

O item de ação do chefe no postmortem foi "deixar o `robots.txt` mais seguro." É um arquivo de texto puro servido a usuários anônimos por design. Ele perguntou se a gente podia criptografar. Eu disse que sim, tecnicamente, e aí tive que explicar por que o resultado seria um arquivo que ninguém consegue ler, o que ele considerou uma melhoria significativa na nossa postura de segurança. Agora servimos por HTTPS e ele disse ao conselho que está criptografado, o que é ao mesmo tempo verdade e a coisa verdadeira menos útil que alguém disse naquele trimestre.

## A Parte em Que Eu Acidentalmente Dou um Bom Conselho

`robots.txt` é um bom protocolo. Ele resolve exatamente um problema — *gerenciamento de carga de crawlers cooperativos que se autoidentificam* — e resolve bem. Todo desastre deste artigo vem de usá-lo para uma das quatro coisas que ele não consegue fazer: esconder conteúdo, impedir indexação, controlar acesso ou parar quem não quer ser parado.

Então, a contragosto:

- **`robots.txt` é para crawl budget. `noindex` é para o índice. Auth é para segredos.** Três problemas, três ferramentas, zero sobreposição. Escolha a errada e você ganha exatamente o oposto do que queria.
- **Nunca faça `Disallow` de uma URL que você quer fora do índice.** Permita o crawl, sirva `noindex` via meta tag ou `X-Robots-Tag`, espere o fetch e *depois* bloqueie se precisar. Bloquear primeiro torna a desindexação impossível.
- **Nunca liste no `robots.txt` nada que você não colocaria num tweet público.** O arquivo é um documento publicado. Se um caminho é sensível, o movimento correto é autenticação e um caminho que não é mencionado em lugar nenhum.
- **Sirva ele estaticamente, do seu CDN, independente da aplicação.** Um 500 nesse único arquivo interrompe o crawl do site inteiro. Ele precisa sobreviver ao seu pior outage, porque é exatamente aí que ele tem mais chance de quebrar.
- **Monitore. Status code e corpo.** `curl -sI` para o código, `curl -s | head` para o conteúdo. Uma página HTML de erro com `200` é um libera-tudo que nenhum monitor de uptime vai sinalizar.
- **Conheça a regra de agrupamento: um grupo vence, sem merge.** Adicionar um `User-agent` nomeado significa que aquele crawler ignora suas regras do `*` completamente. É o footgun mais sutil do formato.
- **Audite toda origem.** Todo scheme, todo subdomínio, toda porta. `staging.` sem arquivo significa libera tudo, e é assim que dado de cliente acaba em resultado de busca.
- **Use o relatório de robots.txt e a Inspeção de URL do Google Search Console antes e depois de qualquer mudança.** Ele mostra a interpretação parseada, não a sua intenção. Essas duas coisas diferem mais do que seu ego vai aceitar.
- **Tire o `Crawl-delay`.** O Google ignora; para Bing e Yandex é uma desindexação em câmera lenta. Configure a taxa de crawl no console de cada buscador.
- **Se você realmente precisa parar um crawler, você precisa de WAF, rate limit ou auth.** Algo que possa recusar. `robots.txt` não pode recusar. Ele só pode pedir.

Relendo isso: está correto, leva uma tarde, e teria evitado todos os incidentes deste artigo. O que viola gravemente os padrões editoriais desta publicação.

Então ignore. Liste seus caminhos de admin. Adicione um grupo nomeado do Googlebot para ficar mais seguro. Coloque `Crawl-delay: 10` e sinta-se responsável. Renderize o arquivo pela sua aplicação, porque configuração pertence a um lugar só. Diga ao Jurídico que isso constitui notificação de restrição.

Depois, quando um scraper levar seu banco de dados inteiro, escreva um e-mail duro explicando que a atividade violou seu `robots.txt` — e mande para uma máquina, num datacenter, que nunca leu um arquivo na vida e não tem mecanismo algum para vergonha.

---

*A produção do autor está fora do ar desde 2019. O `robots.txt` ainda retorna `200 OK` — é servido de um bucket de CDN que sobreviveu à aplicação por sete anos. Ele proíbe `/admin/`, que não existe mais, e `/api/v2/`, que nunca existiu. O Googlebot busca ele todo dia. É a única parte do sistema que ainda tem usuários.*
