---
layout: post
ref: your-nginx-conf-is-a-programming-language-you-never-learned
title: "Seu nginx.conf É Uma Linguagem de Programação Que Você Nunca Aprendeu"
date: 2026-11-03 00:00:00 -0300
categories: [infraestrutura, servidores-web, conselhos-terriveis]
tags: [nginx, reverse-proxy, configuracao, proxy-pass, location, headers-http, devops, sre, producao, copiar-colar, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/seu-nginx-conf-e-uma-linguagem-de-programacao-que-voce-nunca-aprendeu-pt/
---

Em 47 anos produzindo bugs em escala industrial eu escrevi, por estimativa conservadora, onze linhas originais de configuração de nginx. Todo o resto de todo `nginx.conf` que eu já coloquei no ar foi colado de um post de blog, de uma resposta do Stack Overflow, de um tutorial da Digital Ocean, ou de uma mensagem no Slack de um colega que começava com "funcionou aqui, não sei bem por quê".

Isso é a prática correta e eu vou defendê-la. Configuração de nginx não é uma linguagem de programação. É uma **sequência de encantamentos com uma flag `-t` que te diz se o encantamento está gramaticalmente correto**, e gramaticalmente correto é o único tipo de "correto" que alguém já me pediu. O `nginx -t` disse `syntax is ok`. O `nginx -t` disse `test is successful`. Eu imprimi essa saída e anexei no change request. O change request foi aprovado.

O fato de o nginx **ser**, na verdade, uma linguagem de programação — com fases de avaliação, escopo de variáveis, regras de herança e um motor de rewrite capaz de entrar em loop infinito — é um detalhe que custou nove incidentes somados a quatro empregadores, e eu gostaria de te apresentar os meus favoritos.

## A Config Que Eu Subi e Na Qual Eu Acreditava

```nginx
server {
    listen 80;
    server_name _;

    root /var/www;

    location /static/ {
        root /var/www/static/;
    }

    location /api {
        proxy_pass http://backend:8080;
        proxy_set_header X-Request-Id $request_id;
    }

    location ~* \.(jpg|png|css|js)$ {
        expires 30d;
        add_header Cache-Control "public";
    }

    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header Content-Security-Policy "default-src 'self'";

    if ($http_user_agent ~* "bot") {
        return 403;
    }
}
```

Essa configuração rodou em produção por dois anos e meio. Passou no `nginx -t` na primeira tentativa. Deixa eu contar os corpos.

**Um: `root` dentro de um location, com o prefixo do location repetido.** O nginx **concatena** a URI da requisição ao `root`. Então `GET /static/logo.png` resolvia para `/var/www/static/static/logo.png`, que não existia, que gerava um 404, que o time de frontend contornou embutindo cada asset em base64 direto no HTML. Nossa home tinha 4,1 MB. Por dois anos o tech lead de frontend acreditou que isso era uma política de CDN.

**Dois: os dois `add_header` ali no final não se aplicam a três dos meus quatro locations.** Essa é a coisa mais cara que eu já não soube. `add_header` é herdado **apenas se o bloco filho não declarar nenhum `add_header` próprio.** Meu location de imagens adiciona `Cache-Control`, então ele herda **nada** — sem HSTS, sem CSP. Nosso scan de segurança passou, porque o scanner pediu `/`, que é servido pelo location implícito e **recebe** os headers. Todo CSS e todo JS do site foram para produção sem CSP por 31 meses. O ticket de remediação foi fechado como "não reproduzível".

**Três: `if` dentro do bloco `server`, avaliado na fase de rewrite, interagindo com os locations numa ordem que eu não entendia.** A wiki oficial do nginx tem uma página chamada *"If Is Evil"* ("If é o Mal"). Eu li. Eu li **depois** do incidente, que é a ordem correta, porque ler antes significaria que eu não teria incidente nenhum para contextualizá-la. O [XKCD 1739](https://xkcd.com/1739/) é o diagrama canônico desse processo: cada correção que eu aplicava gerava dois comportamentos novos, e em nenhum momento eu considerei parar.

**Quatro: `server_name _;`.** Existe a crença generalizada de que `_` é um wildcard. Não é. É simplesmente um hostname inválido que não casa com nada, o que significa que esse bloco só servia tráfego porque era o **primeiro** bloco `server` na porta 80 e, portanto, o `default_server` implícito. Funcionava por acidente. Quando um colega colocou o diretório `sites-enabled` em ordem alfabética, um vhost de staging qualquer virou o default server e nosso domínio de produção começou a servir a tela de login do staging. Ele ordenou o diretório porque **eu** pedi para ele organizar aquilo.

## A Regra de Precedência Que Ninguém Lê Antes do Incidente

Aqui está o detalhe que destrói tardes. **O nginx não casa locations de cima para baixo.** Ele tem um algoritmo de precedência, e a ordem no arquivo só importa dentro de um dos níveis:

1. `location = /exato` — match exato, ganha na hora e encerra tudo
2. `location ^~ /prefixo` — prefixo mais longo com `^~`, ganha de regex
3. `location ~ ` / `~* ` — regexes, **na ordem do arquivo**, o primeiro match ganha
4. `location /prefixo` — prefixo mais longo, usado só se nenhuma regex casar

Então minha regex `~* \.(jpg|png|css|js)$` estava **abaixo** de `/api` no arquivo e ainda assim ganhava dele. `GET /api/report.js` — um endpoint real que gerava um bundle JavaScript por tenant — era capturado pela regex de imagens, servido do disco como arquivo estático, não encontrado, e retornava 404 com um header `expires` de 30 dias na página de erro. Os navegadores cachearam esse 404 por um mês. Nós pedimos aos clientes que limpassem o cache. Alguns limparam duas vezes.

Eu depurei isso trocando a ordem dos blocos `location`, o que não fez nada, o que eu reportei como "o nginx está ignorando a config".

## A Barra Final Que É Um Programa Diferente

Essas duas linhas não são variações. São dois softwares sem relação nenhuma:

```nginx
location /api/ {
    proxy_pass http://backend:8080;        # /api/users -> /api/users
}

location /api/ {
    proxy_pass http://backend:8080/;       # /api/users -> /users
}
```

Um caractere. A presença de um componente de URI no `proxy_pass` muda o nginx de "passa a requisição intacta" para "substitui o prefixo que casou". Eu já adicionei essa barra para corrigir um bug de roteamento e removi para corrigir **outro** bug de roteamento, no mesmo arquivo, na mesma semana, com total confiança nas duas vezes.

O upgrade disso para tragédia de verdade é o resolver:

```nginx
location /api/ {
    proxy_pass http://backend.internal:8080;   # DNS resolvido UMA VEZ, no start
}
```

Quando o upstream é um hostname literal sem variáveis, o nginx resolve ele no carregamento da configuração e **cacheia aquele IP pelo resto da vida do processo.** Nosso backend ficava atrás de um load balancer cujos IPs rotacionavam. A cada poucas semanas o nginx fielmente mandava todo o tráfego de produção para um endereço que agora pertencia ao autoscaling group de outra empresa. O sintoma era 502 intermitente. A correção que o time de plantão descobriu empiricamente foi `systemctl restart nginx`, que funcionava, o que significa que ninguém nunca procurou a causa. Colocamos no runbook. A entrada do runbook sobreviveu a três dos quatro engenheiros que estavam lá.

A correção real é uma variável mais um `resolver`, o que força resolução em tempo de execução com TTL de verdade:

```nginx
resolver 10.0.0.2 valid=30s ipv6=off;
set $upstream_api backend.internal;
proxy_pass http://$upstream_api:8080;
```

Eu aprendi isso em 2019 e não apliquei em lugar nenhum, porque reiniciar o nginx é mais rápido do que explicar por que a gente precisa disso.

## A Tabela de Diretivas Que Significam o Oposto do Que Você Pensa

| O Que Você Escreveu | No Que Você Acreditava | O Que o nginx Fez |
|---|---|---|
| `add_header` em bloco aninhado | Adiciona um header | Descarta todos os headers herdados |
| `proxy_set_header Host $host;` | Define um header | Derruba os headers de proxy padrão |
| `root /var/www/static/;` em `location /static/` | Serve `/var/www/static` | Serve `/var/www/static/static` |
| `proxy_pass http://b:80` vs `http://b:80/` | A mesma coisa | Dois roteadores diferentes |
| `server_name _;` | Wildcard | Não casa com nada; funciona por ordem de bloco |
| `if ($x) { ... }` | Um condicional | Uma armadilha de rewrite com aviso na wiki oficial |
| `try_files $uri /index.html;` em location regex | Fallback de SPA | Redirect interno infinito, 500 |
| `gzip on;` | Comprime as respostas | Não as proxiadas, sem `gzip_proxied` |
| `client_max_body_size` não definido | Ilimitado | 1 MB, depois 413, com um log que você não vai ler |
| `proxy_read_timeout` não definido | Paciente | 60s, depois 504, culpa do banco |
| `X-Request-Id: $request_id` repassado | Tracing ponta a ponta | Headers do cliente com underscore descartados em silêncio |

Essa última linha custou um trimestre. Nosso time de mobile enviava `X-Client_Version`. O nginx tem `underscores_in_headers off` por padrão e **descarta silenciosamente** qualquer header que contenha underscore. Não dá 400. Não loga aviso. Sumiu. O time de analytics concluiu que 100% do tráfego mobile vinha de uma versão desconhecida e construiu um dashboard em cima dessa premissa. O dashboard foi apresentado ao conselho.

## Estratégias, Ordenadas Por Quão Mal Terminaram

| Abordagem | Por Que É Ruim | Por Que a Minha Foi Pior |
|---|---|---|
| Ler a documentação do nginx | Leva uma tarde | Eu tinha a tarde. Duas, inclusive. |
| Copiar uma config boa de outro serviço | Cargo cult | O outro serviço também era meu |
| Colar a resposta top do Stack Overflow | A resposta é de 2012 | Uma vez eu colei a *pergunta* |
| Gerar a config a partir de um template | Precisa de um passo de templating | Eu fiz template do template |
| Colocar a lógica na aplicação | Correto, chato | Rejeitado: "isso é mudança de código, aqui é só config" |
| Adicionar mais um `if` | É o mal, segundo o manual | Adicionei três e aninhei dois |
| `nginx -s reload` até funcionar | Não é uma metodologia | **É** a metodologia |

O [XKCD 979](https://xkcd.com/979/) é o relato definitivo de pesquisa sobre configuração de nginx: você acha exatamente a sua pergunta, feita por uma pessoa, em 2013, com zero respostas, e uma edição final dizendo "deixa, resolvi sozinho". A thread é a única documentação que corresponde à sua situação e ela acaba ali. Eu já fui essa pessoa. Eu já postei essa edição. Eu não sabia o que eu tinha corrigido.

E quando eu finalmente automatizei a geração da config, o [XKCD 1319](https://xkcd.com/1319/) chegou pontualmente: o gerador tinha 900 linhas de Jinja, levou mais tempo para depurar do que editar nove vhosts à mão, e emitia uma config que o `nginx -t` aprovava e cuja ordem dos `location` era aleatorizada pela iteração de dicionário do Python em uma versão de uma imagem de container.

## O Reload Que Mente

`nginx -t` valida a sintaxe do arquivo. Ele **não** valida que o seu upstream resolve, que o seu certificado corresponde ao `server_name`, que o diretório do `root` existe, que a sua regex faz o que você quis dizer, ou que um reload vai fazer o que você espera.

E reload não é restart. `nginx -s reload` sobe workers novos com a config nova e deixa os workers antigos terminarem as conexões em voo. Com conexões de longa duração — websockets, SSE, long-polling, um endpoint de streaming — esses workers antigos **não morrem.** Uma vez tivemos workers do nginx rodando configuração de três semanas atrás ao lado de workers com a configuração atual, servindo o mesmo hostname, com headers de CSP diferentes e upstreams diferentes. As requisições davam sucesso ou erro dependendo de qual worker as pegava. Nossa taxa de erro era 23%, um número que faz as pessoas encararem a parede.

O engenheiro de plantão descreveu no canal do incidente como "o nginx está em dois estados". É a frase mais precisa que alguém já escreveu sobre o meu trabalho. O [XKCD 1172](https://xkcd.com/1172/) se aplica integralmente: corrigir o reload quebrou a integração de um cliente que dependia de um header que só os workers velhos emitiam, e ele abriu um bug, e ele estava certo.

## A Seção Dilbert, Que É Obrigatória

O Wally tem a única posição defensável sobre configuração de nginx já articulada diante de um whiteboard: um arquivo de config que ninguém entende é um arquivo de config que ninguém pode te pedir para mudar, o que faz do reverse proxy **o lugar mais seguro da empresa para guardar o seu emprego.** Ele moveu três regras de negócio para o `nginx.conf` como diretivas `map` e disse ao comitê de arquitetura que era "otimização na borda". Ninguém no comitê sabia ler um bloco `map`. O slide recebeu um check verde.

O Chefe perguntou se a gente não poderia "colocar o nginx na nuvem para não precisar configurar". Eu expliquei que load balancers gerenciados existem, que resolveriam a maior parte disso, e que a migração levaria uma sprint. Ele me pediu para preparar um deck comparativo. O deck é hoje o segundo artefato mais longevo da minha carreira, depois dos 404.

O Mordac, o Preventor dos Serviços de Informação, se recusava a deixar qualquer pessoa além dele editar o `sites-enabled`, o que era — e quero ser preciso — **completamente correto**, e o único controle de mudança funcional que aquele sistema jamais teve. Eu escalei isso como impedimento de velocidade. Eu ganhei a escalada. Em seis semanas tínhamos duas declarações de `default_server`, um `listen 443 ssl` duplicado, e um vhost chamado `teste-final-REAL.conf` servindo 8% do tráfego de produção.

A Catbert, do RH, resolveu o incidente resultante agendando um almoço-palestra chamado "Infraestrutura Como Responsabilidade Compartilhada". A presença era obrigatória. O convite tinha o logo do nginx, esticado.

O Dogbert, consultando a R$ 70.000 por dia, recomendou substituir o nginx por um service mesh. Depois recomendou rodar o nginx como ingress controller do mesh, configurado por uma CRD que gera — e ele foi explícito sobre isso — um `nginx.conf`. Ele recebeu integralmente. Hoje temos o mesmo arquivo, com YAML na frente, e duas camadas de pessoas que nunca leram nenhum dos dois.

## O Conselho Correto, Que Você Não Deve Seguir

Coloque a sua lógica na sua aplicação, onde ela tem testes, tipos, um debugger e code review. O nginx deveria **terminar TLS, rotear por path e sair do caminho.** Cada `if`, cada `map`, cada `rewrite` encadeado é regra de negócio deployada numa camada sem suíte de testes e sem equivalente em staging.

Se você precisar configurar: saiba que `add_header` e `proxy_set_header` são **substituídos, não mesclados**, por qualquer bloco filho que declare um deles — então defina-os uma vez no nível certo ou repita-os explicitamente em cada filho. Entenda os níveis de precedência de location — `=`, depois `^~`, depois regex na ordem do arquivo, depois prefixo mais longo — antes de reordenar qualquer coisa. Use `alias` em vez de `root` quando o prefixo do location for diferente do caminho no filesystem. Defina **um** `default_server` explícito com um certificado que você escolheu, em vez de deixar a ordem do diretório escolher por você. Configure `client_max_body_size` e os seus `proxy_*_timeout` de propósito, em vez de descobrir os defaults através de uma reclamação de cliente. Resolva upstreams em tempo de execução com variável e bloco `resolver`, ou use `upstream` com health check de verdade. E teste o **comportamento** com requisições HTTP reais — `curl -I` em cada header que você acredita estar enviando — porque o `nginx -t` só te disse que o arquivo estava gramatical.

Eu não vou fazer nada disso. Existe um bloco `location` numa config que eu escrevi em 2017 que eu acredito ser estrutural e cuja função eu não consigo determinar, e a cada seis meses alguém abre um ticket para limpar aquilo, e a cada seis meses eu comento "isso é mais arriscado do que parece" e o ticket é fechado. Esse comentário é o artefato de engenharia mais valioso que eu já produzi. É também o único que era verdade.

---

*O sistema em produção mais longevo do autor é um reverse proxy com uma regra de `rewrite` comentada que para de funcionar quando você apaga o comentário. Ninguém apagou o comentário. Ninguém vai.*
