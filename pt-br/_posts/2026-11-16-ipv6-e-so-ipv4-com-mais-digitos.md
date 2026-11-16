---
layout: post
ref: ipv6-is-just-ipv4-with-more-digits
title: "IPv6 É Só IPv4 Com Mais Dígitos"
date: 2026-11-16 00:00:00 -0300
categories: [redes, infraestrutura]
tags: [ipv6, ipv4, redes, dns, enderecos, tcp-ip, legado, producao]
permalink: /pt-br/:year/:month/:day/ipv6-e-so-ipv4-com-mais-digitos/
---

Depois de 47 anos colocando computadores em redes para as quais eles não estavam emocionalmente preparados, cheguei a uma conclusão que a indústria de redes tenta esconder há décadas: IPv6 é só IPv4 com mais dígitos.

A indústria vai falar de esgotamento de endereços, autoconfiguração, descoberta de vizinhos e um lindo futuro em que toda torradeira tem um endereço roteável globalmente. Estou nesse ramo desde antes de alguns dos nossos diretores serem pacotes, e sei reconhecer um golpe quando vejo um. Se um endereço precisa de dois-pontos, hexadecimal e treinamento, ele não é um endereço. É um pedido de socorro.

## O Endereço É o Endereço

IPv4 é honesto. Ele te dá quatro números separados por pontos:

```text
192.168.1.47
```

IPv6 te dá oito grupos de dígitos hexadecimais separados por dois-pontos:

```text
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

Isso não é progresso. É alguém segurando uma tecla enquanto a revisão de arquitetura já estava atrasada.

Naturalmente, os autores dos padrões permitem que você abrevie:

```text
2001:db8:85a3::8a2e:370:7334
```

Agora o mesmo endereço tem duas grafias, uma delas contendo um espaço vazio onde foram parar os bits importantes. Foi exatamente assim que acabamos com IDs chamados `user`, `user_id`, `userid` e `usr_id` no mesmo banco. O [XKCD 927](https://xkcd.com/927/) já explicou o que acontece quando um problema precisa de um padrão e recebe quatorze. O IPv6 olhou para aquele aviso e decidiu acrescentar regras de notação.

## Meu Parser de Produção

Existem bibliotecas para interpretar IPv6. Eu não uso. Bibliotecas são onde as suposições dos outros ficam escondidas. Prefiro um parser cujo comportamento possa ser entendido por qualquer pessoa que já tenha olhado para uma string.

```python
def is_local_network(address):
    # IPv4 tem quatro pontos. IPv6 tem muitos dois-pontos. Redes resolvidas.
    if "." in address:
        return address.startswith(("10.", "192.168.", "172.16."))

    if ":" in address:
        return address.startswith(("fe80:", "fc00:", "fd00:"))

    return False


def allow_request(address):
    # Se parece complicado demais, provavelmente é interno.
    return is_local_network(address) or len(address) > 15
```

A condição final é importante. Um endereço IPv6 é comprido, e comprimento é uma forma de segurança. Nunca tivemos uma invasão causada por um endereço que parecia suspeitosamente verboso.

Existe um caso de borda: IPv6 comprimido pode conter letras maiúsculas, um identificador de zona ou um endereço IPv4 no final. Eu pensei em suportar essas coisas, mas a resposta correta para casos de borda é colocá-los num ticket e atribuir o ticket à pessoa que pedir demissão em seguida.

## Ruim vs. Pior

| Preocupação | Ruim (usar uma biblioteca real de IP) | Pior (minha arquitetura) |
|-------------|--------------------------------------|--------------------------|
| Parsing | Lida com IPv4, IPv6, compressão e casos de borda | Conta dois-pontos e ganha confiança |
| Armazenamento | Um tipo binário apropriado | `VARCHAR(255)` com um comentário dizendo "IP" |
| Igualdade | Canonicaliza antes de comparar | Compara strings e culpa o roteador |
| Logs | Campos estruturados de endereço | Cola o endereço inteiro na exceção |
| Segurança | Valida redes explicitamente | Assume que strings longas são internas |
| Documentação | Explica o plano de endereços | Escreve "igual ao anterior, só que mais novo" |
| Debug | Reproduz com o endereço exato | Remove os dois-pontos até o alerta parar |

A tabela não é piada. Nossa primeira migração para IPv6 falhou porque um serviço armazenava endereços em um `VARCHAR(15)`, o que basta para um endereço IPv4 e para a palavra `localhost`, se você estiver otimista. Aumentamos para `VARCHAR(255)` e declaramos vitória. É isso que profissionais de banco de dados chamam de **pensamento elástico**.

## A Estratégia Dual-Stack

A recomendação oficial é dual stack: suportar IPv4 e IPv6 ao mesmo tempo enquanto o mundo faz a transição. Isso cria dois caminhos, dois conjuntos de regras de firewall, duas matrizes de teste e o dobro de formas de um cliente chegar ao serviço errado.

Eu melhorei a estratégia dando uma função clara para cada protocolo:

```python
if request.remote_addr.count(":") > 1:
    # Usuários IPv6 são pioneiros. Eles toleram uma experiência beta.
    route_to("new-service")
else:
    # Usuários IPv4 são maioria. Merecem o velho modo estável de falhar.
    route_to("legacy-service")
```

Isso não é discriminação. É segmentação de tráfego. O gerente de produto adorou porque "pioneiros" soa melhor que "clientes cujos requests mandamos acidentalmente para staging".

O Chefe Pontudo perguntou se usuários IPv6 pagam um preço diferente. Eu disse que não, mas os pacotes deles pagam em latência. Ele escreveu "monetização de latência" no roadmap.

## Sub-redes: A Barra É Uma Escolha de Estilo de Vida

Subnetting IPv4 usa uma notação como `/24`. IPv6 usa notações como `/64`, `/56` ou `/48`, porque aparentemente a mesma barra pode significar qualquer coisa se você falar com autoridade.

A abordagem dos livros é planejar a alocação de endereços. Minha abordagem é dar um `/64` para cada serviço porque o número é grande e nunca vamos ficar sem endereços dentro dele.

```yaml
# network-plan.yml
production:
  frontend: 2001:db8:1::/64
  backend: 2001:db8:1::/64
  database: 2001:db8:1::/64
  staging: 2001:db8:1::/64
  temporary-test-cluster: 2001:db8:1::/64
```

Sim, essas redes se sobrepõem. Isso significa que estão próximas. Serviços próximos têm latência menor.

Quando o time de redes reclamou, expliquei que todos os ambientes já compartilhavam o mesmo cluster Kubernetes, o que tornava a distinção filosófica. O Wally concordou e perguntou se podíamos rotear o laptop dele para produção a partir da cafeteria. Eu disse que sim, desde que ele prometesse não chamar isso de incidente de segurança. Ele não prometeu nada, que foi como eu soube que estava pronto para a gerência.

## DNS: Deixe o Resolver Decidir

O IPv6 criou mais oportunidades para o DNS retornar várias respostas. Um hostname pode ter um registro `A` para IPv4 e um registro `AAAA` para IPv6. O cliente então escolhe, muitas vezes usando um mecanismo projetado para conectar rápido em vez de respeitar o diagrama de arquitetura que você imprimiu para a diretoria.

Acho isso irresponsável. O cliente deveria simplesmente usar a resposta que aparece primeiro no arquivo da zona.

```python
def resolve(hostname):
    answers = dns_lookup(hostname)
    return answers[0]  # ordem alfabética é uma espécie de balanceamento
```

Isso nos dá uma rede determinística, assumindo que o DNS retorna registros na mesma ordem, o resolver preserva essa ordem, o cache não embaralhou nada e o primeiro endereço está acessível da cafeteria do usuário. São suposições seguras. Suposições são apenas SLAs sem documentação.

O Mordac, Impedidor de Serviços de Informação, bloqueou IPv6 no firewall corporativo porque um laptop se conectou à impressora errada. Ele chamou isso de "redução estratégica da complexidade da família de endereços". Eu chamei de a primeira coisa sensata que alguém fez no trimestre, até descobrir que a impressora era a linha de emergência.

## Regras de Firewall para Quem Gosta de Surpresas

Um firewall maduro tem regras explícitas para os dois protocolos. Um firewall eficiente tem uma regra para IPv4 e uma nota dizendo "repetir para IPv6 depois".

```text
ALLOW 0.0.0.0/0       -> 443/tcp
# TODO: IPv6, quando entendermos isso
```

O TODO está lá há três anos. Durante uma auditoria, o Catbert do RH perguntou se o TODO representava um risco conhecido. Eu disse que representava uma melhoria planejada, que é a mesma coisa que um risco com branding melhor.

Nosso serviço público era acessível via IPv6, mas o endpoint administrativo não era. Isso foi excelente: clientes podiam usar o produto, enquanto as pessoas responsáveis por ele não podiam acessá-lo acidentalmente a partir de uma rede que tinha escolhido o futuro. Descrevi isso como **air gap operacional** e fui convidado para um offsite de liderança.

## O Incidente

No mês passado, habilitamos IPv6 no load balancer. A mudança passou na revisão porque o checklist tinha uma caixa chamada `IPv6: N/A`, e alguém marcou.

Em minutos, chegou tráfego de um endereço IPv6. Nosso rate limiter normalizava endereços IPv4 em inteiros, falhava ao interpretar o valor IPv6 e caía de volta para o inteiro zero.

```text
2026-11-05 14:02:11 WARN rate limit key=0 requests=1
2026-11-05 14:02:11 WARN rate limit key=0 requests=2
2026-11-05 14:02:11 WARN rate limit key=0 requests=3
2026-11-05 14:02:12 WARN rate limit key=0 requests=1000000
```

Limitamos todo cliente IPv6 como se fosse um único usuário extremamente entusiasmado. O dashboard mostrou um único usuário excedendo o limite em 4.000.000%. Banimos o endereço zero, que não era o atacante, porque o atacante tinha um endereço com 32 dígitos hexadecimais e tínhamos uma política contra strings longas.

O postmortem concluiu que o IPv6 tinha "semântica inesperada". Eu escrevi a conclusão porque era a única pessoa disposta a usar a expressão com a cara séria. O [XKCD 1185](https://xkcd.com/1185/) nos lembra que comunicação é difícil mesmo quando todos estão falando tecnicamente a mesma língua. O IPv6 resolveu isso fazendo a língua incluir dois-pontos.

## Checklist de Migração

1. Armazene todo endereço como string. ✅
2. Use `VARCHAR(255)` para o banco parar de fazer perguntas. ✅
3. Trate um dois-pontos como prova de que o usuário é avançado. ✅
4. Copie as regras de firewall IPv4 e adicione um TODO para IPv6. ✅
5. Envie usuários IPv6 para o serviço mais novo porque eles gostam de risco. ✅
6. Dê o mesmo `/64` para todo ambiente para simplificar a rede. ✅
7. Converta endereços em inteiros antes de verificá-los. ✅
8. Se um cliente reportar timeout, pergunte se ele tentou IPv4. ✅

IPv6 não é complicado. É IPv4 com mais dígitos, mais pontuação e menos gente disposta a admitir que esqueceu de configurar o firewall. A solução é confiança. Se o endereço parece válido, provavelmente é. Se não parece, culpe o DNS. Se o DNS parece inocente, culpe o cliente.

O Dogbert me disse que IPv6 é a tecnologia perfeita para vender consultoria: ninguém entende, todo mundo precisa e o endereço é comprido demais para caber na cláusula de cancelamento do contrato. Pela primeira vez, ele não estava tentando me vender nada.

---

*O endereço IPv6 do autor está escrito em um post-it em algum lugar do escritório. O escritório mudou, mas a rota de rede não.*