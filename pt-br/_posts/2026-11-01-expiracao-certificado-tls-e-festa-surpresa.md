---
layout: post
ref: tls-certificate-expiry-is-a-surprise-party
title: "A Expiração do Seu Certificado TLS É uma Festa Surpresa que Você Organiza Para Si Mesmo a Cada 90 Dias"
date: 2026-11-01 00:00:00 -0300
categories: [seguranca, devops, conselhos-terriveis]
tags: [tls, ssl, certificados, lets-encrypt, certbot, acme, pki, x509, expiracao, monitoramento, curl, openssl, seguranca, devops, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/expiracao-certificado-tls-e-festa-surpresa-pt/
---

Em 47 anos produzindo bugs em escala industrial eu derrubei a produção com certificado expirado **onze vezes**, e quero ser preciso sobre um detalhe: três dessas vezes foi o **mesmo certificado**. Não o mesmo serviço. O mesmo certificado. Ele voltava, como um gato que você leva de carro pra outro bairro.

Expiração de certificado é o modo de falha mais bonito da computação, porque é o único em que o sistema agenda a própria morte, te informa a data exata, coloca num campo legível por máquina — e você ignora. Todo outro incidente é pelo menos uma surpresa. Esse é um convite de calendário que você recusou.

Então aqui está minha doutrina, defendida em quatro empregos e um depoimento: **expiração de certificado é uma festa surpresa, e o ponto central de uma festa surpresa é que você não a organiza.**

## O Cron de Renovação que Eu Subi e no Qual Eu Acreditava

Isso rodava em exatamente uma máquina. A máquina se chamava `util-01`. Ninguém sabe quem criou a `util-01`.

```bash
# /etc/cron.d/certbot
# renova os certs. roda a cada 90 dias porque o cert dura 90 dias.
# (esse é o bug. levei dois anos pra ver.)
0 3 1 */3 * root certbot renew --quiet >> /var/log/certbot.log 2>&1

# o hook de deploy, que escrevi às 4h durante o incidente que ele causou
0 4 1 */3 * root systemctl reload nginx
```

Conte as catástrofes comigo.

**Uma:** renovar *a cada 90 dias* um certificado de *90 dias* te dá exatamente uma tentativa, e se ela falhar — rate limit, soluço de DNS, porta 80 fechada por alguém numa "sprint de hardening" — você só descobre três meses depois, e a forma de descobrir é o Twitter. O intervalo correto é duas vezes por dia. `certbot renew` não faz nada se o cert não estiver a menos de 30 dias do vencimento. Ele foi *projetado* pra rodar constantemente. Eu tratava um comando idempotente como substância controlada.

**Duas:** separei a renovação do reload por uma hora, em duas entradas de cron sem relação, porque na minha cabeça eram "dois jobs diferentes". Então por uma hora o nginx servia o cert velho da memória enquanto o novo estava no disco. E nos trimestres em que a renovação falhava em silêncio, o reload rodava de qualquer jeito, recarregando alegremente o certificado expirado com total confiança, como um garçom reservindo um bife devolvido.

**Três:** `>> /var/log/certbot.log`. Não monitorado. Nunca lido. Rotacionado semanalmente. Eu havia construído um diário para um processo cuja única saída interessante era uma falha que eu tinha me organizado para não ver.

## Minha Estratégia de Monitoramento, Que Era um Ser Humano

Eu tinha alerta. O alerta era um sujeito chamado Derek.

Derek mantinha as datas de expiração numa planilha, atualizada à mão, rodando `openssl` uma vez por trimestre. Derek saiu em março. A planilha estava no Google Drive pessoal do Derek. Descobrimos as duas coisas na mesma tarde, que foi também a tarde em que o certificado do gateway de pagamentos expirou.

Aqui está o sistema de monitoramento inteiro com que substituí o Derek, escrito em nove minutos e nunca mais tocado:

```bash
#!/bin/bash
# cert-check.sh — "monitoramento"
for host in api.example.com www.example.com; do
  echo | openssl s_client -connect $host:443 2>/dev/null \
    | openssl x509 -noout -enddate
done
```

Leia o que isso **não** faz. Imprime uma data. Num terminal. Pra um humano. Que precisa subtrair. Não sai com código diferente de zero. Não alerta. Não verifica o certificado **intermediário**, que pode expirar de forma independente e arruinar sua semana de um jeito que o `curl` do seu notebook não reproduz, porque seu notebook tem o intermediário velho em cache numa trust store de 2021. Não verifica os certs de serviços internos, os client certs, os certs dentro do service mesh, o cert do load balancer que termina TLS *antes* do nginx, nem o do relay SMTP. Ele checa dois hostnames. Nós tínhamos quatrocentos.

E só checa o que é alcançável da máquina onde roda, então o certificado de mTLS interno entre dois serviços — aquele que derrubou o processamento de pedidos por cinco horas num domingo — era estruturalmente invisível. [XKCD 2347](https://xkcd.com/2347/) é sobre o mantenedor não remunerado que sustenta toda a infraestrutura moderna, e no nosso caso esse mantenedor era um shell script sem tratamento de erro e um homem que já tinha achado emprego melhor.

## A Escada da Negação da Expiração

Todo time passa por isso. Eu pessoalmente escoltei seis.

| Estágio | O Que Você Diz | O Que Está Realmente Acontecendo |
|---|---|---|
| Confiança | "O cert vale um ano, vemos isso depois" | Depois é um sábado, são 2h, e é agora |
| Barganha | "Dá pra fazer um cert de 10 anos?" | Navegadores limitam a confiança em ~13 meses e rejeitam |
| Negação | "No Postman funciona" | O Postman está com verificação desligada porque você desligou |
| Raiva | "O relógio do cliente está errado" | Está, de verdade, mas agora isso também é seu problema |
| Aceitação | "Compramos uma plataforma de gestão de certificados" | Ninguém instalou o agente na `util-01` |

E a tabela que eu usava pra defender cada decisão pra liderança, que nunca fez uma única pergunta de follow-up:

| Abordagem | Por Que É Ruim | Por Que a Minha Era Pior |
|---|---|---|
| ACME com renovação 2x/dia + deploy hook | Exige ler uma página de doc | N/A, eu não fiz isso |
| Lembrete no calendário 30 dias antes | Depende de humano olhar calendário | Coloquei pra 7 dias antes e recusei o convite |
| Cert pago de um ano de um fornecedor | Manual, caro, pânico anual | Deixei renovar sozinho e nunca instalei o novo |
| Monitoramento de expiração com alerta | Alguém precisa ser dono do alerta | Eu era o dono. Eu silenciei. |
| `curl -k` em tudo no CI | Desativa o propósito inteiro do TLS | Isso foi pra produção num healthcheck |
| Cert wildcard em todo host | Um comprometimento é comprometimento total | Coloquei a chave privada na imagem Docker |

O `curl -k` merece um memorial próprio. Um healthcheck com verificação desligada passa contra certificado expirado, autoassinado, certificado de outra empresa inteira e um man-in-the-middle. Nossos dashboards ficaram verdes durante um incidente de quatro horas. O dashboard não estava mentindo. Ele estava respondendo uma pergunta que eu havia cuidadosamente reescrito pra não significar nada.

## Coisas que Expiram e que Você Não Está Observando

Essa é a parte que ainda me visita às 4h. Tudo isso era simultaneamente verdade no meu parque:

- O **certificado leaf** da API pública: monitorado. Ok. A única coisa que a gente olhava.
- O **CA intermediário**: não monitorado. Expirou. Metade dos usuários mobile tomou falha de confiança e a outra metade não, dependendo só de quando o celular atualizou a trust store. O bug report dizia "login quebrado pra algumas pessoas", e foi roteado pro time de frontend por nove dias.
- A **raiz do CA interno**, criada em 2016 com validade de 10 anos por um terceirizado, com a chave privada num pendrive numa gaveta etiquetada "CABOS DIVERSOS".
- Os **client certificates** de mTLS entre dois serviços, gerados por um script avulso que ninguém salvou, com validade de 1 ano e nenhum caminho de renovação. O caminho de renovação acabou sendo "recriar a PKI inteira durante o incidente".
- O **certificado de code signing**, que expirou quietinho e quebrou os releases, não a produção — então levou três semanas pra alguém notar e foi arquivado como "o build está instável".
- A **chave de assinatura de JWT** que rotacionamos, esquecendo que um consumidor tinha a chave pública antiga hardcoded num config map. Tecnicamente não é certificado. Se comportou exatamente como um.
- O **relógio** de uma frota de quiosques que derivou oito meses pro futuro, fazendo todo certificado válido ficar "ainda não válido". Discuti dois dias com o fornecedor sobre o certificado expirado deles. O certificado estava ótimo. Os quiosques moravam em agosto do ano seguinte.

Essa última é a minha coisa favorita sobre a qual já estive errado. O certificado é uma *afirmação sobre o tempo*, e as duas pontas precisam concordar sobre que horas são. [XKCD 1883](https://xkcd.com/1883/), *Supervillain Plan*, é o modelo mental correto pra todo sistema baseado em tempo que eu já toquei: o exploit é sempre "e se o relógio estiver mentindo".

## A Seção Dilbert, Que É Obrigatória

A posição do Wally sobre renovação de certificado é a postura de engenharia mais honesta já impressa em jornal: um certificado expirado é *autodocumentado*, porque o navegador mostra uma explicação em tela cheia do problema pra todos os usuários simultaneamente — o que é documentação melhor do que qualquer coisa na wiki. Ele já descreveu interstitials de navegador como "a única comunicação de incidente que nunca atrasou".

O Chefe perguntou se a gente não podia "comprar o certificado que não expira". Expliquei que confiança exige expiração por design, que validade curta é o *mecanismo*, não o inconveniente, e que a indústria está caminhando pra certificados de 47 dias exatamente porque automação é a única resposta sã. Ele me pediu pra levantar três orçamentos.

Mordac, o Impedidor de Serviços de Informação, tinha a única visão materialmente correta do prédio — ele bloqueou a porta 80 de entrada, o que quebrou nosso desafio HTTP-01 — e eu escalei como bloqueio em vez de gastar vinte minutos migrando pra DNS-01, que teria funcionado, suportaria wildcard e não precisaria de nenhuma porta aberta. Catbert, do RH, resolveu a escalação agendando um workshop.

Dogbert, consultando a R$ 70.000 por dia, nos vendeu uma plataforma de ciclo de vida de certificados. Ela descobre certificados escaneando a rede. Ela não consegue ver a `util-01`, porque a `util-01` está numa VPC que é mais antiga que o time de redes.

## O Conselho Correto, Que Você Não Deve Seguir

Automatize a renovação com ACME e rode a checagem **pelo menos duas vezes por dia**, com o deploy hook na *mesma* invocação, pra que o reload não possa descolar da renovação. Monitore **dias restantes até a expiração como métrica**, coletada do handshake TLS real de cada endpoint — público, interno, mesh, e-mail, client certs, tudo — e pagineie em 14 dias, não em zero. Valide a **cadeia inteira**, não só o leaf, de uma máquina com trust store atualizada. Nunca passe `-k`, `--insecure`, `verify=False` ou `rejectUnauthorized: false` em nada que rode mais de uma vez. Prefira DNS-01 pra não ficar refém de portas de entrada. Trate a expiração da raiz do seu CA interno como um projeto datado, com dono e no calendário, porque a janela de validade dele é maior que a maioria dos vínculos empregatícios. E sincronize seus relógios, em todo lugar, incluindo as coisas embarcadas que ninguém conta como computador.

Eu, pessoalmente, vou renovar na mão de novo no próximo trimestre, porque nada nessa indústria gera tanta boa vontade quanto ser a pessoa que heroicamente conserta a coisa às 2h da manhã, e ninguém nunca foi promovido por um cron job que funcionou.

---

*O autor já passou seis horas depurando uma falha de TLS causada pela data errada no próprio notebook. Ele já contou essa história em três conferências com a parte do notebook removida.*
