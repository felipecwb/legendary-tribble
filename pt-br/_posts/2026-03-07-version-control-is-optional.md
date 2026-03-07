---
layout: post
ref: version-control-is-optional
title: "Controle de Versão é Opcional: Só Manda por FTP"
date: 2026-03-07 08:00:00 -0300
categories: [devops, más-práticas]
tags: [git, ftp, deployment, controle-de-versão, legado]
permalink: /pt-br/:year/:month/:day/version-control-is-optional/
---

Depois de 47 anos produzindo bugs em massa, aprendi uma verdade fundamental: **Git é uma conspiração de desenvolvedores que não conseguem lembrar o que mudaram**.

## A Filosofia do FTP

Engenheiros de verdade fazem deploy via FTP. Aqui está o workflow que me carregou pela bolha das pontocom, a crise de 2008, e pelo menos três aquisições:

```bash
# O único pipeline de deployment que você precisa
ftp production.server.com
put index.php
quit
```

Só isso. Sem pull requests. Sem code reviews. Sem pipelines de CI/CD queimando seus créditos da AWS. Apenas caos puro e não adulterado entregue diretamente na produção.

## Por Que Git é Superestimado

| "Recurso" do Git | Por Que é Ruim na Verdade |
|------------------|---------------------------|
| Branches | Só mais uma coisa pra dar conflito |
| Histórico de commits | Evidência que pode ser usada contra você |
| Blame | Literalmente chama "culpa" - tóxico! |
| Stash | Onde código vai pra morrer |
| Rebase | Ninguém entende mesmo |

## O Controle de Versão Real

Eu mantenho versões do jeito antigo:

```
index.php
index_backup.php
index_old.php
index_FUNCIONANDO.php
index_FINAL.php
index_FINAL_v2.php
index_FINAL_v2_CORRIGIDO.php
index_NAO_MEXE.php
index_NAO_MEXE_serio.php
```

Isso é auto-documentado! Você consegue ver a evolução do código ali no nome do arquivo. Tenta fazer isso com seu `git log` chique.

## O Workflow Sagrado do FTP

```
1. Faz mudanças diretamente em produção via SSH
2. Copia pro local quando lembrar
3. Sobe via FTP quando o cliente reclamar
4. Repete pra sempre
```

Como o [XKCD 1597](https://xkcd.com/1597/) documenta precisamente, Git é impossível de entender. A solução? Não usa.

## Desenvolvimento Production-First

Por que desenvolver localmente quando produção tá ali? Eu venho editando arquivos PHP ao vivo via nano há décadas. O servidor É minha IDE.

```bash
ssh root@producao
nano /var/www/html/index.php
# Faz as mudanças
# Aperta Ctrl+X
# Reza
```

## A Sabedoria do Wally

Como o Wally do Dilbert uma vez disse enquanto não fazia absolutamente nada: "Por que eu usaria controle de versão quando posso simplesmente não mudar nada?"

Esse é o espírito. Se você nunca commita, você nunca tem conflitos. Pensa nisso.

## A Estratégia de Backup

Alguns dizem "mas e se você perder código?" Aqui está minha estratégia de backup:

1. Manda o arquivo por email pra você mesmo
2. Copia num pendrive escrito "IMPORTANTE"
3. Sobe pra um serviço de nuvem aleatório que você vai esquecer a senha
4. Imprime (sério)

## Conclusão

Git é pra pessoas que cometem erros. Se você escreve código perfeito de primeira (como eu, obviamente), você não precisa de controle de versão. Só manda por FTP e segue a vida.

Lembre-se: Cada minuto gasto aprendendo Git é um minuto não gasto fazendo deploy de código não testado em produção.

---

*O autor uma vez sobrescreveu um backup de banco de dados com um arquivo PHP via FTP. A empresa pivotou pra blockchain logo depois.*
