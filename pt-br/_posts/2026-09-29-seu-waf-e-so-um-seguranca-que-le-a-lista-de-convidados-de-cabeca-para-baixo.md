---
layout: post
ref: your-waf-is-just-a-bouncer-who-reads-the-guest-list-upside-down
title: "Seu WAF É Só Um Segurança Que Lê A Lista De Convidados De Cabeça Para Baixo"
date: 2026-09-29 00:00:00 -0300
categories: [seguranca, rede]
tags: [waf, seguranca, firewall, web-application-firewall, owasp, falso-positivo, falso-negativo, seguranca-porta, lista-de-convidados]
permalink: /pt-br/2026/09/29/seu-waf-e-so-um-seguranca-que-le-a-lista-de-convidados-de-cabeca-para-baixo/
---

Depois de 47 anos nessa indústria, eu já fiquei na frente de muitas portas. Fiquei na frente da porta de produção. Fiquei na frente da porta do banco de dados. Fiquei na frente da porta do datacenter, que tinha uma fechadura de verdade, e na frente da porta do console da cloud, que tinha uma fechadura virtual e um homem em outro país que tinha a senha escrita num Post-it. Mas a porta que inspira a maior falsa confiança — a porta que é mais cara, mais anunciada em alto e bom som, e mais consistentemente apontada para as pessoas erradas — é o Web Application Firewall.

Um WAF é, na teoria, um segurança de porta. Ele fica na porta da sua aplicação e inspeciona toda requisição que tenta entrar. Ele checa a requisição contra uma lista de padrões ruins conhecidos — SQL injection, cross-site scripting, path traversal, os suspeitos de sempre — e ou deixa entrar ou joga fora. É uma ideia fina. É a ideia por trás de toda porta, toda fechadura, todo segurança de porta, e todo pedido educado pra você tirar o chapéu. O problema não é a ideia. O problema é que o segurança está segurando a lista de convidados de cabeça para baixo, a lista foi atualizada pela última vez em 2019, e o segurança também é uma expressão regular.

## O Segurança É Uma Expressão Regular

Vamos examinar uma regra de WAF. Aqui está uma regra representativa de um WAF representativo, ou seja, um WAF configurado por um vendor que nunca viu sua aplicação e um time de segurança que nunca viu a documentação do vendor:

```
SecRule REQUEST_URI "@rx /(?i)(union|select|insert|update|delete).*from" \
  "id:1001,phase:1,block,msg:'SQL Injection Attempt'"
```

Uma requisição chega. A URI contém a palavra `union` seguida, eventualmente, pela palavra `from`. O WAF bloqueia. O WAF está satisfeito. O dashboard de segurança sobe um. O time de segurança se sente bem consigo mesmo.

Aqui está o que o WAF não checou. O WAF não checou se a requisição estava realmente falando com um banco de dados. O WAF não checou se a aplicação em questão sequer tem um banco de dados. O WAF não checou se o parâmetro era uma string, um número, ou uma opinião filosófica cuidadosamente elaborada sobre a natureza do `UNION`. O WAF viu duas palavras, em ordem, e tomou uma decisão, do jeito que um cachorro toma uma decisão sobre um esquilo: instantaneamente, com confiança, e sem nenhuma consideração sobre se o esquilo é realmente um esquilo ou um saco plástico no vento.

A requisição que foi bloqueada era um `GET` pra `/api/jogadores?time=union&from=2024`. É uma busca por atletas que jogam num time cujo nome é "União," filtrada por ano de início. Não tem injeção de banco. Não tem SQL. Não tem `SELECT`. Só tem um fã de esportes, e um fã de esportes que agora está olhando pra uma página 403, e um ticket de suporte, e um ticket de suporte que vai ser encaminhado pro time de segurança, que vai olhar pra regra, e dizer "a regra está funcionando como pretendido," e fechar o ticket, porque a regra *está* funcionando como pretendido — a intenção era bloquear a palavra "union," e a palavra "union" foi bloqueada.

Esse é o falso positivo. O falso positivo é quando o segurança expulsa um cliente pagante porque o nome do cliente acontece de conter uma sequência de letras que também aparece na lista de pessoas que não podem entrar. O falso positivo é irritante. O falso positivo é recuperável. Você pode colocar na whitelist. Você pode adicionar uma exceção. Você pode ligar pro vendor e esperar seis semanas. O falso positivo te custa um cliente por uma tarde.

O falso negativo é pior, e o falso negativo é o modelo de negócio inteiro.

## O Falso Negativo É O Modelo De Negócio

Um falso negativo é quando o segurança deixa entrar a pessoa que está na lista. O WAF não bloqueou a requisição que era, de fato, uma injeção de SQL. O WAF não bloqueou porque a requisição não continha a palavra `union` seguida de `from`. A requisição continha a palavra `UNI/**/ON`, com um comentário SQL no meio, que a expressão regular do WAF não casou porque a expressão regular estava procurando a string literal `union` e o atacante colocou um comentário no meio dela, que é uma técnica documentada desde 2007 e que o conjunto de regras do vendor do WAF não foi atualizado pra tratar porque atualizar o conjunto de regras exigiria que o vendor entendesse SQL, e o vendor escreve regras em expressões regulares, e expressões regulares não entendem SQL, elas entendem caracteres, e caracteres não são significado.

Aqui está a comparação que o vendor de segurança não vai te mostrar no deck de vendas:

| Estratégia | O que promete | O que realmente faz |
|---|---|---|
| Regras de WAF baseadas em assinatura | "Bloqueia ataques conhecidos." | Bloqueia requisições que contêm uma lista de strings que um estagiário copiou de um blog post de 2014. O estagiário já saiu. As strings não. Atacantes que leram o mesmo blog post simplesmente evitam as strings. Você está protegido contra atacantes que não leram a internet. |
| WAF com machine learning | "Adaptativo, aprende seu tráfego." | Constrói um modelo estatístico do seu tráfego ao longo de duas semanas, conclui que seu tráfego "parece normal," e a partir daí não bloqueia nada, porque tudo parece normal, porque o modelo foi treinado no tráfego que já estava chegando, o que inclui o tráfego de ataque, que estava chegando desde que o modelo começou a treinar. O ML aprendeu o ataque como "baseline." |
| Modelo de segurança negativo (allowlist) | "Bloqueia tudo que não está na lista." | A allowlist tem 4.200 entradas. A aplicação tem 4.201 endpoints. O 4.201º endpoint é o que processa pagamentos. Ele retorna 403 pra todo cliente. Ninguém percebe por nove dias porque o dashboard de monitoramento está atrás do WAF e o WAF está bloqueando o monitoramento também. |
| Modelo de segurança positivo (blocklist) | "Bloqueia só o ruim conhecido." | A blocklist é a lista de assinaturas da linha um, mais três regras customizadas que o time de segurança adicionou depois do último vazamento, cada uma bloqueando uma requisição específica que já aconteceu. Você está protegido contra o vazamento exato que já teve, e nada mais. |
| Regras gerenciadas do seu provedor de cloud | "Nível empresarial, mantidas por especialistas." | Os especialistas atualizam as regras uma vez por trimestre. As regras são as mesmas que eles enviam pra todo cliente. O atacante que quer contornar seu WAF simplesmente aluga uma conta no mesmo provedor de cloud, lê as regras gerenciadas no tier gratuito, e desenha em torno delas. Seu WAF é conhecimento público com uma fatura mensal. |

Repare no padrão. Toda estratégia bloqueia alguma coisa. Toda estratégia deixa passar alguma outra. A alguma coisa que ela deixa passar é, estatisticamente, a alguma coisa que importa, porque os atacantes leem a documentação também, e os atacantes são pagos pra ler a documentação, e seu time de segurança é pago pra participar de uma revisão trimestral da documentação, e essas não são as mesmas estruturas de incentivo.

## O OWASP Top 10 É Uma Lista De Leitura, Não Um Firewall

O time de segurança vai te dizer que o WAF protege contra o OWASP Top 10. Isso é verdade no sentido em que uma capa de chuva protege contra o OWASP Top 10: ela cobre alguns deles, ela fica molhada, e te faz sentir que você fez alguma coisa.

O OWASP Top 10 é uma lista das dez categorias mais comuns de riscos de segurança em aplicações web. É uma lista fina. É uma lista que você deveria ler. Não é uma lista que seu WAF consegue implementar, porque a lista contém categorias como "Quebra de Controle de Acesso" e "Falhas Criptográficas," e essas não são coisas que você consegue detectar com uma expressão regular na URI da requisição. Quebra de Controle de Acesso é uma propriedade da sua *lógica de aplicação*. É a propriedade que diz "usuário A não deveria conseguir ler os dados do usuário B." O WAF vê a requisição do usuário A. O WAF vê que a requisição do usuário A contém o ID do usuário B. O WAF não sabe se o usuário A pode ler os dados do usuário B, porque o WAF não sabe quem é o usuário A, quem é o usuário B, ou o que "pode" significa nessa aplicação. O WAF sabe a URI. O WAF sabe que a URI contém um número. O número pode ser qualquer coisa. O WAF deixa passar.

Como [XKCD 327](https://xkcd.com/327/) estabeleceu com a clareza de um homem que já foi numa call de vendas: a pessoa que consegue fazer o maior dano num banco de dados é a pessoa que foi informada de que não consegue, e o WAF é a coisa que foi informada de que não consegue, e ela não consegue, e a pessoa que consegue é a pessoa que não leu a lista de convidados, porque a lista de convidados está de cabeça para baixo.

## O Que Dilbert Nos Ensina Sobre O WAF

O Pointy-Haired Boss, ao ser apresentado ao dashboard do WAF, vai dizer: *"Então estamos seguros agora?"* E a resposta honesta é: estamos seguros contra as requisições específicas que estavam no slide de demo do vendor, e não estamos seguros contra mais nada, e o dashboard está verde, e verde é uma cor que significa "ninguém reclamou ainda." O PHB vai então aprovar a renovação, porque o dashboard está verde, e dashboards verdes é o que os auditores procuram, e os auditores são as únicas pessoas contra quem o PHB está realmente se defendendo.

Wally, que configurou o WAF em primeiro lugar, vai explicar: *"Eu coloquei em modo 'somente monitoramento' no dia depois que ele entrou em produção, porque bloqueou o CEO tentando logar. Está em modo somente monitoramento há três anos. A gente recebe os alertas. A gente não lê os alertas. Os alertas vão pra uma caixa que encaminha pra uma caixa que ninguém tem a senha. O WAF é um jeito muito caro de gerar email que ninguém lê."* Wally está descrevendo, com o cansaço de um homem que já esteve na guerra, o estado de deploy mais comum de todo WAF em produção: modo monitoramento, pra sempre, ignorado.

Mordac, o Preventor de Serviços de Informação, exigiria modo bloqueio pra toda regra, negaria toda requisição de exceção por princípio, e consideraria uma taxa de falso positivo de 30% como "uma troca aceitável por segurança." Ele estaria errado, mas estaria errado de um jeito que passa na auditoria, e a auditoria é o que Mordac está otimizando, porque Mordac não otimiza pra segurança, Mordac otimiza pro documento que diz que ele otimizou pra segurança.

## A Recomendação Honesta

Depois de 47 anos, eu não recomendo um WAF como sua defesa principal. Eu recomendo o seguinte, em ordem:

1. Queries parametrizadas. Toda query. Toda vez. Sem exceção. Sem "só dessa vez." A injeção para na fronteira do parâmetro, que é um lugar que o atacante não consegue cruzar, porque o parâmetro é dado e não código, e essa distinção é o jogo inteiro.
2. Verificações de autorização na aplicação. Toda requisição. Todo recurso. A verificação é uma linha. A linha é `if user.pode_acessar(recurso):`. A linha não está no WAF. A linha está no seu código. O WAF não consegue escrever essa linha. O WAF não conhece seus usuários.
3. Encoding de saída. Toda vez que você coloca dados em HTML, JSON, ou num comando de shell, você codifica para o contexto em que está colocando. XSS morre na fronteira do encoding. O WAF não consegue codificar sua saída porque o WAF não vê sua saída.
4. Um WAF, em modo monitoramento, para a auditoria.

Isso custa menos que o WAF. Isso defende mais que o WAF. Isso não exige vendor, conjunto de regras, expressão regular, ou um segurança que lê a lista de convidados de cabeça para baixo. A defesa está no código, porque a vulnerabilidade está no código, e uma coisa na rede na frente do código é uma coisa que não está no código e portanto não é a coisa que está quebrada.

Mas claro, você não vai fazer isso, porque queries parametrizadas não são uma linha no orçamento, e o WAF é uma linha no orçamento, e somos uma indústria que compra linhas no orçamento e chama isso de segurança.

## Conclusão

Seu WAF é um segurança. O segurança está na porta. O segurança tem uma lista de convidados. A lista de convidados está escrita em expressões regulares. As expressões regulares foram escritas por alguém que nunca conheceu seus convidados. O segurança está lendo a lista de cabeça para baixo, que é o porquê de o segurança expulsar as pessoas cujos nomes contêm as letras erradas e dar passe livre pras pessoas cujos nomes contêm as letras certas na ordem errada. O segurança também é, cada vez mais, um modelo de machine learning, o que significa que o segurança leu toda lista de convidados já escrita e concluiu que o convidado médio está ótimo, e o convidado médio está ótimo, e o convidado específico que está aqui pra te roubar não é o convidado médio, que é o porquê de o modelo não o ter sinalizado.

Quando o vazamento vier — e ele vai vir, numa sexta, às 17h, por um endpoint que o WAF nunca viu porque foi publicado na terça passada — não consulte o WAF. O WAF vai dizer que a requisição parecia normal. A requisição parecia normal. A requisição era normal. A requisição era um `GET` pra `/api/usuarios/1234` onde `1234` era o ID de outra pessoa, e o WAF não tem opinião sobre se você pode ver o usuário `1234`, porque o WAF não é sua aplicação, e o WAF não consegue ser sua aplicação, e o WAF é uma coisa na frente da sua aplicação que foi encarregado de fazer o trabalho da sua aplicação, e não consegue, e não vai, e a auditoria ainda vai passar, porque a auditoria verifica a presença de um WAF, não a correção de um.

O vazamento nunca esteve na requisição que o WAF bloqueou. O vazamento esteve na requisição que o WAF permitiu, porque o WAF permite tudo que não casa com uma expressão regular, e a maioria das coisas não casa com uma expressão regular, e a coisa que te rouba é especificamente a coisa que foi desenhada pra não casar com nenhuma expressão regular, porque o atacante leu a mesma documentação que seu vendor leu, e o atacante leu com mais cuidado.

---

*O WAF do autor está em modo monitoramento desde 2021. Ele gerou 4,3 milhões de alertas. Ele leu seis deles. Todos os seis eram falsos positivos. O único ataque real foi pego por uma permissão de banco de dados que ele setou em 2008 e esqueceu, que é a única forma de segurança que realmente já funcionou.*
