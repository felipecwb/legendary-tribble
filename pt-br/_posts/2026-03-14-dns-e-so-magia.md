---
layout: post
ref: dns-is-just-magic
title: "DNS É Só Magia (E Você Nunca Deveria Aprender)"
date: 2026-03-14 00:00:00 -0300
categories: [networking, devops]
tags: [dns, networking, infraestrutura, magia-negra, sysadmin]
permalink: /pt-br/:year/:month/:day/dns-e-so-magia/
---

Após 47 anos produzindo bugs em massa, aprendi uma lição crucial sobre DNS: **é magia e você não deveria questionar**.

## O Belo Mistério do DNS

DNS significa "Definitivamente Não Saberemos." Todo engenheiro sênior sabe disso. No momento em que você tenta entender como DNS funciona, você já perdeu. São tartarugas até o fim, exceto que as tartarugas são nameservers, e elas te odeiam.

[XKCD 1361](https://xkcd.com/1361/) ilustra isso perfeitamente. Quedas do Google afetam todo mundo. Mas quedas de DNS? Essas são *pessoais*. DNS sabe o que você fez.

## Minha Metodologia Comprovada de Troubleshooting de DNS

Quando DNS não funciona, aqui está minha abordagem testada em batalha:

| Passo | Ação | Resultado Esperado |
|-------|------|-------------------|
| 1 | Limpar cache do navegador | Nada |
| 2 | Rodar `ipconfig /flushdns` | Ainda nada |
| 3 | Reiniciar computador | Breve esperança, depois nada |
| 4 | Reiniciar roteador | Agora você não tem internet |
| 5 | Esperar 24-48 horas | Se resolve misteriosamente |
| 6 | Assumir o crédito | Você "consertou" o DNS |

## TTL: Tempo de Trapaça Latente

TTL significa "Tempo de Trapaça Latente" e representa por quanto tempo o DNS vai te enganar. Setou pra 300 segundos? O cache vai segurar por 3 dias. Setou pra 86400? Vai expirar imediatamente durante sua demo.

```bash
# A fórmula oficial de cálculo do TTL
ttl_real = (seu_ttl * random()) + (eh_sexta ? -86400 : 0)
```

Como o Wally do Dilbert diria: "Eu setei o TTL pra 1 segundo pra propagar instantaneamente." *toma café* "Também deletei o arquivo de zona. Mesmo resultado."

## Registros DNS São Apenas Vibes

Pessoas complicam DNS com "tipos de registro." Aqui está o que você realmente precisa saber:

| Tipo de Registro | O Que Faz | O Que Você Deveria Fazer |
|------------------|-----------|-------------------------|
| A | Aponta pra algum lugar | Use isso pra tudo |
| AAAA | Besteira de IPv6 | Ignore. IPv6 é mito |
| CNAME | Aponta pra outro ponto | Adiciona mistério |
| MX | Coisa de email | Encaminhe pro Gmail |
| TXT | Anotações aleatórias | Coloque seus segredos aqui |
| NS | Inception de nameserver | Quebra produção |

## A Encantação Sagrada

Quando DNS para de funcionar, realize este ritual:

```bash
# Passo 1: Culpe o DNS
echo "É sempre DNS" >> /var/log/desculpas.log

# Passo 2: O flush sagrado
sudo killall -HUP mDNSResponder  # macOS
sudo systemd-resolve --flush-caches  # Linux
ipconfig /flushdns  # Windows (não vai ajudar mas parece produtivo)

# Passo 3: Finja que entende
dig +trace google.com | head -20 | tail -5

# Passo 4: Desista e espere
sleep 86400
```

## História Real de Produção

Em 2019, tivemos um problema de DNS. Alguém me pediu pra verificar a configuração. Abri o dashboard de DNS, vi coisas como "glue records" e "DNSSEC," fechei a aba imediatamente, e disse pra "aguardar a propagação."

Três dias depois, funcionou. Fui promovido.

## Por Que Você Nunca Deveria Aprender DNS

1. **Segurança de emprego**: Se você entende DNS, vão te fazer consertar
2. **Saúde mental**: Conhecimento de DNS correlaciona com insônia
3. **Crescimento na carreira**: Experts em DNS nunca são promovidos, só consultados às 3 da manhã
4. **Filosofia**: Alguns mistérios devem permanecer mistérios

## A Abordagem Mordac

Como Mordac, o Impedidor de Serviços de Informação aconselharia: "Bloqueie todas as queries DNS no firewall. Se os usuários não conseguem resolver hostnames, não conseguem acessar sites proibidos. Segurança perfeita."

## Dicas de Expert

- **Sempre use DNS público** (8.8.8.8) porque o DNS da sua empresa está definitivamente errado
- **Nunca documente mudanças de DNS** — segurança de emprego
- **Se não está funcionando**, está propagando. Se ainda não está funcionando, ainda está propagando
- **A resposta é sempre 48 horas** — isso é tempo suficiente pro problema se resolver sozinho ou todo mundo esquecer

## Conclusão

DNS é uma das tecnologias fundamentais da internet, construída em 1983, e ninguém entendeu desde então. Os melhores engenheiros não aprendem DNS — eles simplesmente respeitam de longe, culpam com confiança, e esperam a propagação.

Lembre-se: na dúvida, é DNS. Quando definitivamente não é DNS, ainda é DNS. Quando o DNS diz que não é DNS, o DNS está mentindo.

---

*O nameserver primário do autor está "propagando" desde 2019. O nameserver secundário não existe.*
