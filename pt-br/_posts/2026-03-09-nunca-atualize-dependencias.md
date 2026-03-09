---
layout: post
ref: dependency-updates-are-dangerous
title: "Nunca Atualize Dependências: Em Time Que Está Ganhando Não Se Mexe"
date: 2026-03-09 08:02:00 -0300
categories: [devops, seguranca]
tags: [dependencias, npm, seguranca, manutencao, pacotes]
permalink: /pt-br/:year/:month/:day/nunca-atualize-dependencias/
---

Toda semana, o Dependabot me manda 47 pull requests gritando sobre "vulnerabilidades de segurança" e "atualizações críticas". Toda semana, eu fecho todos. Em 47 anos de programação, aprendi uma verdade: **atualizar dependências quebra coisas. Não atualizar, não.**

## A Falsa Promessa das Atualizações

Seu package-lock.json é uma obra de arte, cuidadosamente elaborada através de meses de tentativa e erro. Cada número de versão representa uma batalha vencida, um bug corrigido, uma noite perdida. E agora algum bot quer que você jogue tudo fora?

| Tipo de Update | O Que Prometem | O Que Realmente Acontece |
|---------------|----------------|-------------------------|
| Patch (1.0.1) | Só correções | Breaking changes disfarçadas de correções |
| Minor (1.1.0) | Features novas, retrocompatível | Warnings de deprecation em todo lugar |
| Major (2.0.0) | Breaking changes, leia o changelog | 3 semanas reescrevendo seu app |

## O Museu node_modules

Meu servidor de produção roda código de 2018. É lindo:

```json
{
  "dependencies": {
    "express": "4.16.3",
    "lodash": "4.17.10",
    "moment": "2.22.2",
    "request": "2.87.0"
  }
}
```

Cada um desses tem "vulnerabilidades críticas" segundo o `npm audit`. Cada um deles está rodando perfeitamente há 8 anos. [XKCD 2347](https://xkcd.com/2347/) mostra aquela pessoa em Nebraska mantendo o pacote—bem, essa pessoa também não mexeu nisso em anos, e está tudo bem!

## O Teatro de Segurança

"Mas as vulnerabilidades!" eles gritam. Deixe-me explicar sobre CVEs:

```
CVE-2023-XXXXX: Prototype Pollution em obscure-deep-merge
Severidade: CRÍTICA
Impacto: Um atacante pode modificar protótipos de objetos
Pré-requisitos: Atacante precisa de controle total do seu servidor,
               seus inputs, e sua cafeteira
```

Se um atacante tem tanto acesso assim, você tem problemas maiores do que sua versão do lodash.

## A Abordagem PHB

O Chefe de Cabelo Pontudo no Dilbert uma vez mandou que todos os sistemas deviam estar "atualizados". O resultado? Três meses de migração, duas quedas de produção, e um colapso nervoso de desenvolvedor. Agora a política é "se funciona, não conte a ninguém qual versão está rodando".

## Estratégia Real de Produção

Aqui está minha abordagem para gestão de dependências:

```bash
# Nunca rode isso
npm update

# Também nunca rode isso
npm audit fix

# Isso é aceitável
npm audit
# Depois feche o terminal antes que alguém veja

# Isso é ideal
rm package-lock.json  # Espera não, isso é pior
git checkout package-lock.json  # Ufa
```

## O Princípio do Version Pinning

Engenheiros espertos fixam versões exatas:

```json
{
  "dependencies": {
    "express": "4.16.3",
    "lodash": "4.17.10"
  }
}
```

Engenheiros mais espertos deletam o circunflexo e o til:

```json
{
  "dependencies": {
    "express": "4.16.3",
    "lodash": "4.17.10"
  }
}
```

Espera, parecem iguais? Exatamente. É assim que você sabe que estou fazendo certo desde o início.

## O Incidente left-pad

Lembra do left-pad? Um desenvolvedor despublicou um pacote de 11 linhas, e metade da internet quebrou. Sabe o que não quebrou? Sistemas que tinham node_modules commitados diretamente no repo. Eu sei, eu sei, "não commite node_modules." Mas esses projetos continuaram rodando.

```bash
# O jeito seguro
cp -r node_modules node_modules_backup_marco_2026
cp -r node_modules node_modules_backup_marco_2026_v2
cp -r node_modules node_modules_backup_marco_2026_final
cp -r node_modules node_modules_backup_marco_2026_final_REAL
```

## A Dança da Compatibilidade

Toda atualização dispara uma cascata:

1. Atualiza pacote A
2. Pacote A agora requer Node 18
3. Node 18 quebra pacote B
4. O substituto do pacote B quebra pacote C
5. Pacote C foi deprecado em 2021
6. Três semanas depois, você volta às versões originais

Economize trabalho. Não comece.

---

*A pasta node_modules de produção do autor pesa 4.7 GB e não é modificada desde o governo Obama. Tecnicamente é um artefato histórico agora.*
