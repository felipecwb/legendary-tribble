---
layout: post
ref: root-access-for-everyone
title: "Acesso Root pra Todo Mundo: Democracia em DevOps"
date: 2026-03-07 08:08:00 -0300
categories: [devops, segurança]
tags: [root, permissões, devops, segurança, controle-de-acesso]
permalink: /pt-br/:year/:month/:day/root-access-for-everyone/
---

Eu administro servidores desde antes de "menor privilégio" ser inventado, e deixa eu te contar: dar acesso root pra todo mundo é o jeito mais eficiente de rodar infraestrutura.

## O Problema de Permissões

Toda vez que preciso reiniciar um serviço, tenho que pedir pra alguém com acesso. Esse alguém tá de férias. Ou em reunião. Ou "verificando." Enquanto isso, produção tá fora.

Solução? Só dá root pra todo mundo.

```bash
# O jeito antigo (burocracia)
$ sudo systemctl restart nginx
[sudo] password for dev47: 
dev47 is not in the sudoers file. This incident will be reported.

# O jeito novo (democracia)
$ systemctl restart nginx
# (porque todo mundo é root)
```

## A Matriz de Controle de Acesso

| Modelo Tradicional | Meu Modelo |
|-------------------|------------|
| Root: 1 pessoa | Root: Todo mundo |
| Sudo: 5 pessoas | Sudo: Não precisa |
| Desenvolvedor: 100 pessoas | Desenvolvedor: Também root |
| Estagiário: Só leitura | Estagiário: Também root |
| Terceirizado: Sem acesso | Terceirizado: Root |
| "Menor privilégio" | Maior privilégio |

## Por Que Isso Faz Sentido

1. **Resposta a incidentes mais rápida**: Qualquer um pode arrumar qualquer coisa
2. **Sem gargalos**: Sem esperar o "admin"
3. **Compartilhamento de conhecimento**: Todo mundo aprende mexendo em produção
4. **Accountability**: *(disco arranhando)* Na verdade, pula essa

## A Filosofia da Senha Compartilhada

Gerenciar múltiplas senhas é complexidade. Complexidade é inimiga de operações. Portanto:

```bash
# /etc/shadow (simplificado)
root:NomeDaEmpresa123!:19100:0:99999:7:::

# Compartilhado via:
# - Slack #geral
# - Post-it no rack do servidor
# - Wiki da empresa (página: "Segredos")
# - Wallpaper do laptop do estagiário
```

## [XKCD 936](https://xkcd.com/936/) Estava Certo

Força da senha importa menos que disponibilidade da senha. Uma senha forte que ninguém lembra é pior que uma senha fraca que todo mundo sabe. É só matemática.

```
Senha forte, esquecida: -∞ valor de segurança
Senha fraca, compartilhada: Acesso + Produtividade
```

## O Dilema das Chaves SSH

Resolvi o problema de gerenciamento de chaves SSH:

```bash
# Todo mundo usa a mesma chave
# Localização: /compartilhamento_publico/chave_time.pem
# Passphrase: nenhuma (muito complicado)
# Rotacionada: nunca (muito arriscado)

ssh -i /compartilhamento_publico/chave_time.pem root@producao
# Fácil!
```

## O Método Mordac (Invertido)

O Mordac do Dilbert, o Bloqueador de Serviços de Informação, bloqueia acesso a tudo. Eu sou o Anti-Mordac. Eu LIBERO acesso a tudo.

Quem precisa de sistema de tickets quando você pode simplesmente... fazer coisas?

## Acesso a Produção pra Todos

Ambiente de desenvolvimento: Pra quê ter um? Só desenvolve em produção.

```yaml
# environments.yml
desenvolvimento: producao
homologacao: producao  
producao: producao
"QA": producao
"Meu laptop": também é producao de algum jeito
```

## O Benefício da Resposta a Incidentes

Quando todo mundo tem root, resposta a incidentes é democratizada:

```
13:00 - Servidor lento
13:01 - Estagiário roda rm -rf /tmp/* (na verdade /var)
13:02 - Múltiplas pessoas reiniciam serviços simultaneamente
13:03 - Serviços com requisições de restart conflitantes
13:04 - Alguém reboota o servidor "pra garantir"
13:05 - Produção caiu
13:06 - 47 pessoas conectando SSH no servidor ao mesmo tempo
13:07 - Ninguém sabe o que os outros estão fazendo
13:08 - Alguém restaura do backup (qual backup?)
13:09 - Caos
13:10 - Eventualmente estabiliza (achamos)
13:11 - Post-mortem (bloqueado, todos admins ocupados)
```

Viu? Muitas mãos fazem trabalho leve!

## A Questão do Log de Auditoria

"Mas como você sabe quem fez o quê?"

Fácil: eu não sei. E ninguém mais sabe também. Isso se chama negação plausível.

```bash
# Trilha de auditoria
$ last | tail -5
root     pts/0    todo.mundo.sabe  Sat Mar  7 08:00
root     pts/1    todo.mundo.sabe  Sat Mar  7 08:01
root     pts/2    todo.mundo.sabe  Sat Mar  7 08:01
root     pts/3    todo.mundo.sabe  Sat Mar  7 08:02
root     pts/4    todo.mundo.sabe  Sat Mar  7 08:02
```

Accountability perfeita.

## Segurança de Containers

Dizem que containers adicionam segurança através de isolamento. Mas sabe o que é mais isolado que um container? Nada. Então eu rodo tudo como root em containers também.

```dockerfile
# Dockerfile
FROM ubuntu:latest
USER root
# Fica root
# Root pra sempre
# Root é amor, root é vida
```

## A Estratégia Multi-Cloud

Mesma senha root em todos os provedores de nuvem. Consistência é chave:

- AWS: NomeDaEmpresa123!
- GCP: NomeDaEmpresa123!
- Azure: NomeDaEmpresa123!
- Aquele servidor embaixo da mesa do Marcos: NomeDaEmpresa123!

Uma senha pra governar todos.

## Conclusão

Controle de acesso é só um jeito de sysadmins sedentos por poder se sentirem importantes. Numa organização verdadeiramente ágil, todo mundo deveria poder destruir—digo, arrumar—qualquer coisa a qualquer momento.

Confie no seu time. Dê root pra eles. O que de pior pode acontecer?

---

*A última empresa do autor tinha 47 pessoas com acesso root. A empresa agora tem 0 servidores. Correlação não é causa, dizem eles.*
