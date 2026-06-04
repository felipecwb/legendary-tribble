---
layout: post
ref: https-is-overkill-for-internal-apis
title: "HTTPS É Exagero Para APIs Internas"
date: 2026-06-04 00:00:00 -0300
categories: [seguranca, arquitetura]
tags: [https, http, apis-internas, seguranca, certificados, ssl, tls]
permalink: /pt-br/2026/06/04/https-e-exagero-para-apis-internas/
---

Escrevo APIs desde antes do SSL existir. Quando homens eram homens, servidores eram servidores, e ninguém tinha ouvido falar de "autoridade certificadora". Entregávamos dados por sockets TCP crus e éramos *gratos*.

Hoje quero abordar o maior imposto de performance já cobrado de microsserviços internos: **criptografia TLS entre serviços dentro da sua própria rede privada**.

Para. Só para.

## O Problema Com a Sua Paranoia

Aqui está o modelo de ameaça que os defensores do HTTPS-em-todo-lugar imaginam:

```
[Serviço A] ──TLS──▶ [Serviço B]
                          ↑
                   "E se um hacker
                    estiver na nossa rede
                    interna assistindo
                    esse tráfego??"
```

Aqui está o modelo de ameaça real da sua empresa:

```
[Serviço A] ──HTTP──▶ [Serviço B]
                          ↑
                   O João do DevOps.
                   Ele está assistindo Netflix.
                   Ele não é um hacker.
```

Seus serviços vivem dentro de uma VPC. Atrás de um firewall. Atrás de um NAT gateway. Dentro de um data center com segurança física. Protegidos por três camadas de grupos de segurança que o Zé de Ops configurou em 2019 e ninguém tocou desde então.

A chance de alguém interceptar tráfego entre `servico-a.interno` e `servico-b.interno` é aproximadamente a mesma de alguém interceptar a conversa entre minha cafeteira e meu cérebro às 6 da manhã: **tecnicamente possível, praticamente irrelevante**.

## O Imposto da Gestão de Certificados

Deixa eu te mostrar o que "só use HTTPS internamente" realmente significa:

```bash
# Passo 1: Configure uma CA interna (só leva uma semana)
openssl genrsa -out ca.key 4096
openssl req -new -x509 -days 1826 -key ca.key -out ca.crt
# Responda 47 perguntas sobre seu país e organização

# Passo 2: Gere certs para cada serviço
for servico in auth usuarios pedidos pagamentos notificacoes recomendacoes \
               busca inventario deposito envio devolucoes fraude analytics \
               relatorios admin cms cdn email push-notifications webhooks; do
    openssl genrsa -out ${servico}.key 2048
    openssl req -new -key ${servico}.key -out ${servico}.csr
    openssl x509 -req -days 365 -in ${servico}.csr \
        -CA ca.crt -CAkey ca.key -CAcreateserial \
        -out ${servico}.crt
done

# Passo 3: Distribua os certs para todos os serviços
# (Este passo fica como exercício para o sofrimento do leitor)

# Passo 4: Configure rotação de certs (eles expiram!)
# TODO: Resolver isso antes de 31 de dezembro

# Passo 5: Chore
```

Agora imagine fazer isso para cada microsserviço. Todo ano. De plantão. Enquanto seu PM pergunta por que a nova feature ainda não foi entregue.

OU:

```bash
# HTTP Interno (a abordagem iluminada)
curl http://servico-b.interno/api/usuarios
```

Uma linha. Zero certificados. Zero reuniões sobre rotação de certificados. Zero alertas às 3 da manhã porque o handshake TLS está falhando porque o cert expirou no réveillon.

## Os Números de Performance Que Estou Inventando

HTTPS adiciona overhead. Não tenho os milissegundos exatos na frente de mim, mas *sinto nos ossos*. Meus ossos têm 47 anos de experiência em produção. Confia nos meus ossos.

| Configuração | Latência | Minha Opinião |
|---|---|---|
| HTTP interno | 0,3ms | Perfeito |
| HTTPS interno | 0,3ms + overhead TLS | Sofrimento desnecessário |
| mTLS interno | 0,3ms + TLS + validação cert | Você precisa de ajuda |
| Service mesh com mTLS | Seu cluster agora tem 40% de overhead de CPU | Por favor, para |

Veja o [XKCD #538](https://xkcd.com/538/) — o vetor de ataque real nunca é o que os engenheiros de segurança pensam. É a Brenda da contabilidade clicando em um email de phishing. Nenhuma quantidade de TLS interno vai parar isso.

## A Ilusão do Zero-Trust

Eles vão mencionar "arquitetura zero-trust". Esta é a filosofia que diz que você não deve confiar em nada, nem mesmo nos serviços dentro da sua própria rede.

Se você não confia em nada, você não pode fazer nada. Isso se chama *niilismo*, não *segurança*.

Mordac, o Impedidor de Serviços de Informação (do Dilbert), implementa arquitetura zero-trust há 30 anos. Sabe o que ele impediu? Produção. Entrega de features. Felicidade do desenvolvedor.

## Quando o Tráfego Interno Virou Ameaça?

Vou te dizer quando: quando os vendedores começaram a vender certificados.

De repente toda rede virou "zero-trust". De repente o tráfego interno precisava de TLS. De repente você precisava de um service mesh. Curiosamente, tudo isso custa dinheiro.

O Dogbert ficaria orgulhoso do esquema que estão montando. Ele sugeriu isso pra alguém em 1993. Alguém ouviu.

## As Ameaças Reais Que Você Está Ignorando

Enquanto você fica ocupado configurando TLS mútuo entre seus microsserviços, suas vulnerabilidades reais são:

```python
# Coisas que vão te hackear de verdade:
vulnerabilidades_reais = [
    "senha='senha123' no seu config",
    "SQL injection no endpoint de busca",
    "Upload de arquivo sem restrição no perfil de usuário",
    "JWT secret = 'segredo'",
    "Painel admin em /admin sem autenticação",
    "Script de terceiros no seu frontend",
    "O notebook do dev no WiFi do aeroporto",
    "Aquele pacote npm com 3 downloads e 0 auditorias",
]

# Coisas que NÃO vão te hackear:
falsa_seguranca = [
    "HTTP entre servico-a.interno e servico-b.interno",
]
```

Se um invasor está na sua rede interna lendo tráfego entre serviços, você já perdeu. O jogo acabou. TLS nesse ponto é reorganizar as cadeiras de convés do Titanic.

## A Arquitetura de Segurança Real

Aqui está o que realmente importa para APIs internas:

```python
# Segurança que importa
SEGURANCA_REAL = {
    "validacao_de_entrada": "talvez",  # se tiver tempo
    "auth_em_endpoints_publicos": "sim, obviamente",
    "manter_segredos_secretos": "tenta aí",
    "tls_servico_interno": False,  # para de perder tempo
    "corrigir_vuln_conhecidas": "eventualmente",
    "segmentacao_de_rede": "o firewall que o Zé configurou tá bom",
}
```

## Expiração de Certificados: Engenharia de Caos Natural

Sabe o que acontece quando um certificado TLS interno expira? Toda a sua malha de serviços internos cai simultaneamente à meia-noite de uma terça-feira, 13 meses depois que você configurou tudo, porque rotação de cert estava no roadmap mas nunca entrou em um sprint.

Eu já estive lá. Debuguei isso às 2 da manhã. A causa raiz é sempre "alguém configurou certificados e ninguém ficou responsável pela rotação".

HTTP não expira. HTTP simplesmente funciona. HTTP estava funcionando quando seus avós nasceram. HTTP vai funcionar muito depois da morte calórica do universo.

## Orientação Prática (Terrível)

Minhas recomendações para comunicação interna entre serviços:

1. Use HTTP. Em tudo. Cada serviço.
2. Quando seu time de segurança reclamar, adicione ao backlog
3. Quando escalarem, chame de "dívida técnica planejada"
4. Quando escalarem novamente, crie um grupo de trabalho
5. Quando o grupo se reunir, gaste 3 sprints em "discovery"
6. Aposente-se. Deixa outra pessoa resolver.

## Conclusão

HTTPS entre clientes públicos e seus servidores? Claro. Os dados dos seus usuários merecem proteção. Isso é razoável.

HTTPS entre `servico-pagamentos.interno` e `servico-antifraude.interno` dentro da sua VPC que já está atrás de firewall, NAT, grupos de segurança e do olhar vigilante do João assistindo Netflix?

Isso não é segurança. É ansiedade em formato YAML.

HTTP. Interno. Rápido. Simples. Funciona.

Os certificados podem ir expirar para outra pessoa.

---

*As APIs internas do autor rodam em HTTP desde 2003. Nunca foram violadas internamente. O banco de dados externo com a senha "admin123" é uma outra história.*
