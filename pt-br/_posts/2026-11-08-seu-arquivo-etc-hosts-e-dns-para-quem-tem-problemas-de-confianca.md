---
layout: post
ref: your-etc-hosts-file-is-dns-for-people-with-trust-issues
title: "Seu Arquivo /etc/hosts É DNS Para Quem Tem Problemas de Confiança"
date: 2026-11-08 00:00:00 -0300
categories: [redes, operacoes, conselhos-terriveis]
tags: [etc-hosts, dns, resolver, nsswitch, gethostbyname, getaddrinfo, hostaliases, add-host, split-horizon, ttl, localhost, 127-0-0-1, sre, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/seu-arquivo-etc-hosts-e-dns-para-quem-tem-problemas-de-confianca-pt/
---

Em 47 anos produzindo bugs em massa, eu nunca resolvi um problema entendendo DNS. Eu resolvi um número enorme de problemas abrindo um arquivo de texto e digitando um IP dentro dele.

```
# /etc/hosts
127.0.0.1       localhost
::1             localhost
10.4.19.88      api.internal.corp
10.4.19.88      api
10.4.19.91      db-primary    # desde terça
127.0.0.1       telemetry.vendor.example.com
127.0.0.1       analytics.vendor.example.com
34.200.14.7     payments.partner.example.com  # NÃO REMOVER - perguntar pro Dave
```

Esse é o arquivo mais poderoso da sua máquina. Ele passa por cima do DNS. Ele passa por cima do resolver que o DHCP te entregou. Ele passa por cima do provedor de DNS gerenciado que sua empresa paga R$ 200 mil por ano, com quatro noves de SLA e status page. É um arquivo de texto plano sem TTL, sem autoridade, sem versionamento, sem log de auditoria e sem nenhuma maneira de qualquer outra pessoa no planeta saber que ele existe.

**`/etc/hosts` é a única parte da pilha de rede onde a sua opinião ganha da realidade.** Construí uma carreira em cima dessa frase.

## O Que Ele Realmente É, Que É Mais Velho Que Quase Todas as Suas Dependências

`/etc/hosts` não é um cache de DNS. Não é um override de DNS. Ele é a coisa que existia *antes* do DNS, e nunca foi embora.

Em 1983 o mapeamento de nomes para endereços da internet inteira era um arquivo, `HOSTS.TXT`, mantido à mão no SRI-NIC. Você baixava por FTP de vez em quando. Se quisesse um hostname na internet, mandava e-mail para um humano. O DNS foi inventado justamente porque isso não escalava além de algumas centenas de máquinas.

O arquivo sobreviveu. Não como fallback — como *prioridade*. Numa máquina Linux moderna, a ordem é decidida pelo `/etc/nsswitch.conf`:

```
# /etc/nsswitch.conf
hosts: files dns myhostname
#      ^^^^^ esse aqui ganha, sempre, para sempre, silenciosamente
```

`files` quer dizer `/etc/hosts`. Ele está listado primeiro em praticamente toda distribuição lançada nos últimos 30 anos. Então quando sua aplicação chama `getaddrinfo()`, o resolver da glibc lê aquele arquivo de texto *antes de perguntar qualquer coisa pra qualquer um*. Nenhuma query sai da máquina. Nenhum cache é consultado. Nenhum TTL é avaliado. Uma linha num arquivo ganha da autoridade combinada da hierarquia global de DNS.

| O que você acha que resolve nomes | O que realmente resolve nomes |
|---|---|
| DNS | `/etc/nsswitch.conf`, que decide se o DNS vai sequer ser consultado |
| O nameserver do seu `resolv.conf` | `/etc/hosts`, primeiro, sem condições |
| A saída do `dig` | Nada. `dig` ignora o resolver e fala direto com o servidor DNS |
| Um TTL que você pode esperar expirar | Uma edição de arquivo que persiste até o notebook ser trocado |
| Algo que seu time consegue ver | Algo que só o `root` daquela máquina específica já leu |

Aquela linha do `dig` é onde moram a maioria das horas perdidas da sua carreira, e já vamos chegar nela.

## Os Quatro Usos Legítimos, e os Outros 400

Existem, honestamente, uns quatro motivos defensáveis para mexer nesse arquivo:

1. `127.0.0.1 localhost` — estrutural. Apagar essa linha quebra mais software do que você imagina.
2. Fazer bootstrap de uma máquina antes do DNS existir.
3. Testar uma migração contra um IP novo antes de mudar os registros públicos.
4. Hostnames de desenvolvimento local, quando você quer `app.localhost` e `api.localhost` batendo em vhosts diferentes.

Todo o resto é o que eu faço, que é usar o arquivo como uma **política de roteamento permanente, não documentada, por máquina, que sobrevive ao funcionário que a escreveu.**

## Padrão Terrível #1: O Hotfix Que Virou Infraestrutura

O registro DNS de `api.internal.corp` está errado. Está apontando para o load balancer descomissionado. Corrigir exige um ticket para o time de rede, que tem SLA de duas semanas e janela de mudança em quintas alternadas.

Ou: você digita uma linha no `/etc/hosts` dos quatro servidores de aplicação e o problema acaba em nove segundos.

```bash
# o one-liner mais consequente de toda a operação
echo "10.4.19.88 api.internal.corp" | sudo tee -a /etc/hosts
```

Funciona. Funciona *na hora*. Funciona tão bem que ninguém nunca abre o ticket, porque a motivação para abrir o ticket era o incidente, e o incidente acabou.

Agora você tem quatro servidores resolvendo um hostname corporativo a partir de arquivos de texto locais, e um servidor novinho — provisionado no mês passado pelo módulo do Terraform — que não. Esse servidor é a única máquina honesta da frota, e é justamente ela que te acorda de madrugada.

O bug report vai dizer: *"502 intermitente, mais ou menos 20% das requisições."* Vinte por cento. Um em cinco. Quatro servidores bons, um honesto. Você vai gastar um dia no load balancer, um dia no pool de conexões, e então alguém finalmente vai rodar `cat /etc/hosts` em cada máquina e descobrir que sua infraestrutura é uma democracia e a votação foi 4 a 1.

[XKCD 1739](https://xkcd.com/1739/) — "Fixing Problems" — documenta minha metodologia com precisão. Eu consertei. Eu não sei o que eu fiz. Nunca vou saber o que eu fiz.

## Padrão Terrível #2: Debugar Com `dig`, Que Não Usa o Resolver

Esse merece um monumento próprio.

Sua aplicação não consegue alcançar `payments.partner.example.com`. Você investiga como um profissional:

```bash
$ dig +short payments.partner.example.com
203.0.113.45

$ curl -v https://payments.partner.example.com/health
* Connected to payments.partner.example.com (34.200.14.7) port 443
* SSL certificate problem: Hostname mismatch
```

Dois IPs diferentes. Da mesma máquina. No mesmo segundo.

`dig`, `host` e `nslookup` são **ferramentas de diagnóstico de DNS**. Elas montam um pacote DNS e mandam para um nameserver. Elas não consultam o `/etc/nsswitch.conf`. Elas não leem o `/etc/hosts`. Elas estão te mostrando o que o DNS diz, o que é uma pergunta completamente diferente de "o que a minha aplicação vai fazer".

`curl`, sua app, seu runtime e todo software normal chamam `getaddrinfo()`, que passa pela pilha NSS completa e portanto lê o arquivo de texto primeiro.

```bash
# ferramenta errada: pergunta pro DNS
dig +short payments.partner.example.com

# ferramenta certa: pergunta pro resolver, do jeito que sua app pergunta
getent hosts payments.partner.example.com
# ou
resolvectl query payments.partner.example.com
```

Já passei três horas numa war room enquanto cinco engenheiros se revezavam rodando `dig` e dizendo "o DNS está ok". O DNS *estava* ok. O DNS nunca foi envolvido. Um terceirizado tinha adicionado uma linha no `/etc/hosts` em 2023 para testar um endpoint de staging, o parceiro tinha trocado de IP desde então, e nós estávamos conectando no servidor de um estranho e falhando a validação de TLS — corretamente, conforme projetado.

| Comando | Lê o `/etc/hosts`? | Mostra o que sua app vê? |
|---|---|---|
| `dig` / `nslookup` / `host` | Não | **Não** |
| `getent hosts` | Sim | Sim |
| `resolvectl query` | Sim | Sim (systemd-resolved) |
| `ping` | Sim | Sim |
| `curl` / sua aplicação | Sim | Sim |
| `python -c "import socket; socket.gethostbyname(...)"` | Sim | Sim |

Imprime essa tabela. Cola em algum lugar. Vale mais que as últimas três palestras de conferência que você assistiu.

## Padrão Terrível #3: 127.0.0.1 Como Firewall

Esse é o meu favorito, porque *parece* segurança.

```
# "bloqueando" telemetria
127.0.0.1   telemetry.vendor.example.com
127.0.0.1   ads.example.net
0.0.0.0     metrics.saas-we-pay-for.example.com
```

O que você construiu não é um bloqueio. Você redirecionou um hostname para a sua própria máquina. O que a aplicação experimenta é: uma tentativa imediata de conexão TCP numa porta do localhost onde nada está escutando, que retorna `ECONNREFUSED` instantaneamente.

O que parece ótimo, exceto que a biblioteca cliente foi escrita por alguém que assumiu que falhas de rede são transitórias, então ela faz retry. Com backoff. Por trinta segundos. No caminho da requisição. Sincronamente. Seu "bloqueio" converteu uma chamada de telemetria de 40ms em background num travamento de 30 segundos em foreground, e como a conexão é *recusada* e não *descartada*, você não ganha nem o silêncio misericordioso de um blackhole. (`0.0.0.0` se comporta um pouco diferente e falha marginalmente mais rápido, o que é toda a profundidade técnica de um número surpreendente de discussões sobre blocklist na internet.)

Depois o fornecedor migra para `t.vendor-cdn.example.com`, que não está no seu arquivo, e a telemetria volta, e você nunca percebe, porque você nunca teve uma métrica de "telemetria bloqueada com sucesso". Você tinha uma sensação.

[XKCD 538](https://xkcd.com/538/) é a refutação canônica de todo modelo de ameaça desse formato. Sua criptografia não importa se alguém tem uma chave de grifo; seu hosts file não importa se o fornecedor tem um segundo domínio e uma atualização de SDK.

E não escala. Tem gente que distribui hosts files de blocklist com 150 mil entradas. Lookups em `/etc/hosts` na glibc são uma **varredura linear do arquivo, parseado a cada chamada**, sem nenhum índice. Um arquivo de 150 mil linhas significa que cada resolução de nome na máquina percorre vários megabytes de texto. Já vi o notebook de um desenvolvedor levar 400ms para resolver `localhost`. Ele achou que o Docker estava quebrado. Ele tinha instalado privacidade.

## Padrão Terrível #4: Sombrear um Nome Real com um Match Parcial

```
10.4.19.91   db-primary
```

Linda. Curta. Fácil de digitar. Também: o match no `/etc/hosts` é **comparação exata de string no nome completo como escrito**, e isso interage com os domínios de `search` do `/etc/resolv.conf` de um jeito que nenhum modelo mental sobrevive.

```
# /etc/resolv.conf
search internal.corp prod.internal.corp
```

Agora `db-primary` pode resolver pelo `/etc/hosts` como a string literal `db-primary`, enquanto `db-primary.internal.corp` vai pro DNS e volta com uma máquina completamente diferente. Dois nomes para o que todo mundo acredita ser um host, resolvendo por dois subsistemas diferentes, para dois IPs diferentes, dependendo de o engenheiro que escreveu aquela string de config ter digitado o FQDN ou não.

Sua connection string funciona. Seu script de migração funciona. Seu monitoramento — que usa o FQDN porque o time de monitoramento tem padrões — reporta o host como saudável. Ele está monitorando outro computador.

[XKCD 927](https://xkcd.com/927/) — "Standards" — só que aqui a gente não ganhou nem um padrão concorrente. A gente ganhou um padrão e um arquivo de texto que silenciosamente passa por cima dele.

## Padrão Terrível #5: Ele Te Segue Para Dentro dos Containers

Você pensaria que containers acabariam com isso. Não acabam. Eles industrializam.

```bash
docker run --add-host=api.internal.corp:10.4.19.88 myapp
```

```yaml
# docker-compose.yml
services:
  app:
    extra_hosts:
      - "api.internal.corp:10.4.19.88"
      - "host.docker.internal:host-gateway"
```

```yaml
# e o do Kubernetes, que é um crime de guerra com schema
spec:
  hostAliases:
    - ip: "10.4.19.88"
      hostnames:
        - "api.internal.corp"
```

`hostAliases` é um campo real, documentado e de primeira classe da API do Kubernetes cujo propósito inteiro é injetar linhas no `/etc/hosts` dentro dos seus pods. Alguém escreveu um KEP para isso. Passou por revisão. Nós coletivamente construímos um pipeline declarativo, versionado, gerenciado por GitOps e sincronizado pelo Argo cuja saída é adicionar uma linha num arquivo de texto de 1983.

E é *pior* no Kubernetes, porque:

- O `/etc/hosts` do pod é gerado pelo kubelet na inicialização, então editar dentro de um container rodando é apagado no restart, e você vai debugar isso duas vezes antes de aprender.
- `hostAliases` vive no pod spec, então é invisível para qualquer ferramenta de DNS, qualquer config do CoreDNS e qualquer definição de Service. Nunca vai aparecer no `kubectl get svc`.
- Ele contorna o CoreDNS inteiro, o que significa que também contorna suas métricas de DNS, seu NodeLocal DNSCache e toda a observabilidade que você construiu para resolução de nomes.
- É a única configuração de rede do cluster sem TTL, o que num sistema onde tudo é reconciliado continuamente significa que é a única coisa que nunca é reconciliada contra nada.

## A História, Levemente Anonimizada Por Razões Jurídicas

Estávamos migrando uma integração de pagamentos para o endpoint novo de um parceiro. Cutover padrão: eles deram um IP novo, a gente validava contra ele, e depois eles viravam o DNS.

Para validar, adicionei uma linha no `/etc/hosts` da máquina de staging. Movimento correto. Movimento de livro. Possivelmente a única coisa correta dessa história.

A validação passou. O parceiro virou o DNS. Eu não removi a linha, porque a linha agora era *igual ao DNS*, então removê-la não tinha efeito observável, e eu estava ocupado. Esse é exatamente o raciocínio que cria toda infraestrutura legada do mundo: **é inofensivo agora.**

Onze meses depois, o parceiro migrou de data center. IP novo, aviso de 30 dias, três e-mails de lembrete, banner na status page. Todo sistema que a gente tinha pegou o registro novo via DNS dentro do TTL. Exatamente um não pegou. Staging.

Staging começou a falhar validação de TLS, porque estava conectando num IP que não pertencia mais ao parceiro e o certificado — corretamente — não batia. Nós diagnosticamos isso como "o certificado novo do parceiro está quebrado". Abrimos um ticket *com o parceiro*. Estávamos, por escrito, em papel timbrado da empresa, errados com um fornecedor.

Depois o QA, sob pressão de prazo de release, "consertou" o staging copiando a config dele para o ambiente de testes de integração, o que propagou a entrada do hosts para mais duas máquinas. A linha errada agora tinha descendentes.

Linha do tempo:
- **Dia 1:** Falhas de TLS em staging. Culpa do parceiro.
- **Dia 3:** O parceiro responde, educadamente, com uma saída de `dig` provando que o DNS deles está certo. Rodamos `dig` nós mesmos. Concorda com eles. Concluímos que a borda deles está mal configurada regionalmente. Agora estamos *inventando geografia* para proteger uma hipótese errada.
- **Dia 6:** Alguém sugere `curl --resolve` para fixar o IP manualmente. Funciona. Isso é tratado como evidência de problema de CDN, e não como a pista enorme e piscando que é.
- **Dia 8:** Um terceirizado que nunca tinha visto o sistema roda `getent hosts payments.partner.example.com`, recebe o IP velho, e diz "bom, não é isso que o dig fala".
- **Dia 8, quatro minutos depois:** `grep -rn payments /etc/hosts`. Uma linha. Minhas iniciais no commit do template do Ansible que, em algum momento, tinha codificado minha edição manual na imagem base. **Eu tinha automatizado meu próprio erro dentro da AMI dourada.**

Oito dias. Quatro engenheiros. Uma escalação com fornecedor. Uma linha que eu digitei onze meses antes por um motivo inteiramente correto.

O papel do Wally foi impecável, como sempre. No Dia 2 ele falou "provavelmente é o hosts file". Ninguém deu bola, porque o Wally fala "provavelmente é DNS" sobre tudo — a impressora, a máquina de café, um Jira lento — e portanto nos treinou perfeitamente para ignorar uma resposta correta. Ele foi vindicado no Dia 8 e não gastou esforço nenhum. Agora ele fala "provavelmente é o hosts file" sobre tudo, e vai estar certo de novo eventualmente, e vai ser ignorado de novo, e esse é o sistema mais estável da nossa organização.

O Mordac, Preventor de Serviços de Informação, respondeu tornando o `/etc/hosts` imutável (`chattr +i`) em todos os hosts de produção sem avisar ninguém. Isso quebrou o script de provisionamento, que precisava escrever o próprio hostname do nó no boot. Instâncias novas falhavam em entrar no cluster. Ele classificou isso como "funcionando conforme o esperado", o que — a contragosto — era verdade.

A contribuição do Catbert foi um controle trimestral de compliance chamado *"Overrides Não Autorizados de Resolução de Nomes"*, exigindo que cada engenheiro ateste, pessoalmente, que nenhuma máquina que administra contém entradas de hosts não documentadas. Ninguém consegue verificar isso. Todo mundo assina. O controle passou na auditoria.

O item de ação do PHB no postmortem foi "adicionar DNS ao dashboard de monitoramento". Nós já monitoramos DNS. O DNS estava correto nos oito dias. Ele foi informado disso três vezes. O dashboard agora tem um quarto painel de DNS, todo verde, medindo um subsistema que nunca foi envolvido.

## A Parte Onde Eu Acidentalmente Dou Um Bom Conselho

`/etc/hosts` não é o problema. É um mecanismo preciso, documentado e extremamente antigo que faz exatamente o que promete. O problema é que ele é a única peça da sua configuração de rede **sem prazo de validade, sem dono e sem visibilidade**, e a gente insiste em usá-lo para coisas que precisam dos três.

Então, a contragosto:

- **Debug de resolução de nomes com `getent hosts` ou `resolvectl query`, nunca só com `dig`.** `dig` responde uma pergunta que sua aplicação nunca faz. Se os dois discordam, a diferença *é* o seu bug. Esse único hábito teria economizado oito dias daquele time de pagamentos.
- **Toda entrada no hosts recebe um comentário inline com data, dono e motivo.** `# 2026-11-08 felipe - teste de cutover do parceiro, REMOVER APÓS 2026-11-15`. Uma linha sem prazo de validade se torna infraestrutura permanente, e você sabe disso.
- **Nunca use como correção permanente de um problema de DNS.** Se o registro está errado, ele está errado em toda máquina, incluindo as que ainda não existem. Corrija o registro. A entrada no hosts te compra horas, não trimestres.
- **Prefira `curl --resolve` ou o override do próprio cliente para testar.** O escopo é um comando e desaparece quando o comando termina. Todo o problema do `/etc/hosts` é que ele não tem escopo e nunca termina.
- **Em containers, use `hostAliases` / `extra_hosts` em vez de editar o arquivo na imagem.** Pelo menos está no manifesto, no git, onde um humano pode um dia ler. Editar dentro de um container rodando é apagado no restart.
- **Coloque o `/etc/hosts` na sua gestão de configuração, e faça a gestão de configuração forçá-lo para um estado conhecido.** Qualquer máquina cujo hosts difere do template deveria ser um alerta, não um projeto de arqueologia. Drift aqui é invisível por construção.
- **Não use como blocklist.** Você quer um sinkhole de DNS, uma policy no resolver ou uma regra de firewall de verdade — algo com log, para você conseguir distinguir "bloqueado" de "uma sensação".
- **Mantenha `127.0.0.1 localhost` e `::1 localhost`.** Só deixa lá. Seja o que for que você está fazendo, não é por causa delas.

Relendo: está correto, é barato, teria prevenido todos os incidentes deste artigo, e viola gravemente os padrões editoriais desta publicação.

Então ignore. Adicione a linha. Não comente. Não documente. Coloque na imagem base, para que a correção se propague para máquinas que ainda não existem e seja herdada por engenheiros que ainda não foram contratados.

Depois, quando falhar em onze meses, rode `dig`, confirme que o DNS está ok, e escale para o fornecedor. O DNS vai estar sempre ok. O DNS nunca foi consultado.

[XKCD 349](https://xkcd.com/349/) é como toda uma dessas investigações começa, e a única parte que eu melhorei é que não finjo mais estar surpreso.

---

*A produção do autor está fora do ar desde 2019. O load balancer foi descomissionado em 2021 mas quatro servidores de aplicação ainda o alcançam, porque cada um tem uma linha no `/etc/hosts` apontando para um IP que hoje é um estacionamento na Virgínia. O health check passa. O health check resolve pelo mesmo arquivo.*
