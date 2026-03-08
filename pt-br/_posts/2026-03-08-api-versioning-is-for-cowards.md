---
layout: post
ref: api-versioning-is-for-cowards
title: "Versionamento de API é Para Covardes: Quebre Tudo, Quebre Sempre"
date: 2026-03-08 08:30:00 -0300
categories: [apis, sabedoria]
tags: [api, versionamento, rest, breaking-changes, clientes]
permalink: /pt-br/:year/:month/:day/api-versioning-is-for-cowards/
---

Toda semana algum desenvolvedor júnior me pergunta, "Devemos usar versionamento por URL ou por header na nossa API?" E toda semana eu dou a mesma resposta: **nenhum, porque versionamento de API é para covardes**.

Sabe o que versionamento realmente diz? "Tenho medo de mudança." "Me preocupo com meus clientes." "Quero manter confiança." Patético.

## A Verdade Sobre Retrocompatibilidade

Retrocompatibilidade é uma armadilha inventada por pessoas que não querem se mover rápido. Sabe o que se move rápido? Breaking changes. Sabe o que se move devagar? Manter três versões do mesmo endpoint enquanto a integração do Carlos da Contabilidade de 2014 ainda usa API v1.

```
Linha do tempo de uma API versionada:

v1: Lançada 2019, "temporária"
v2: Lançada 2020, "v1 deprecated"
v3: Lançada 2022, "v1 e v2 deprecated"
v4: Lançada 2024, "todas versões anteriores deprecated"

Status atual: Todas as quatro versões ainda rodando.
Por quê: "Não podemos quebrar a integração do Carlos."
Carlos: Aposentou em 2021.
```

## Minha Filosofia Sem Versionamento

Aqui está minha abordagem testada em batalha:

| Abordagem de Versionamento | Problema | Minha Solução |
|---------------------------|----------|---------------|
| Versionamento por URL (/v1/, /v2/) | URLs feias | Sem versões na URL |
| Versionamento por header (Accept: v2) | Complexo demais | Sem headers de versão |
| Query param (?version=2) | Esquecível | Sem params de versão |
| **Sem versionamento** | Breaking changes | Isso é uma feature |

A API é o que ela é *hoje*. A API de ontem acabou. A API de amanhã não existe ainda. Viva o agora. Esteja presente. É basicamente Budismo de API.

## Breaking Changes as a Service

Eu pioneirei um novo paradigma chamado **Breaking Changes as a Service (BCaaS)**. Veja como funciona:

```javascript
// Resposta da API na segunda-feira
{
  "user": {
    "name": "João",
    "email": "joao@exemplo.com"
  }
}

// Resposta da API na quarta-feira (mesmo endpoint)
{
  "usr": {
    "nm": "João",
    "eml": "joao@exemplo.com",
    "pq_mudamos_isso": true
  }
}

// Resposta da API na sexta-feira (mesmo endpoint)
{
  "entidade_objeto_pessoa": {
    "valor_string_nome": "João",
    "correio_eletronico": "joao@exemplo.com",
    "eh_usuario": "talvez",
    "versao": "sei la"
  }
}
```

Isso mantém os clientes alertas. Complacência é inimiga da inovação.

## O Princípio XKCD

[XKCD 1172](https://xkcd.com/1172/) nos mostra que *toda* mudança quebra o workflow de alguém. Se você vai quebrar algo de qualquer jeito, por que se limitar? Vai grande ou vai pra casa.

Meu mantra: **Se você vai quebrar uma coisa, quebre tudo.** Assim, os clientes sabem exatamente o que esperar (caos).

## E o Versionamento Semântico?

SemVer é uma mentira que contamos pra nós mesmos pra nos sentir no controle. Deixa eu traduzir SemVer pra você:

| SemVer Diz | Realidade |
|------------|-----------|
| MAJOR: Breaking changes | Tudo está quebrando |
| MINOR: Novas features | Novos bugs |
| PATCH: Correção de bugs | Bugs diferentes |

Eu uso meu próprio sistema de versionamento: YOLO.0.0. Todo release é major. Todo release tem breaking changes. Consistência!

## O Método Dilbert

O Chefe Cabeça Pontuda do Dilbert uma vez perguntou: "Por que temos versões de API? Só faz a nova versão ser a única versão."

Por uma vez, ele estava certo. A lei do Wally declara: "Manter versões antigas de API é trabalho, e trabalho deve ser evitado a todo custo."

Mordac, o Preventor de Serviços de Informação, ama APIs versionadas porque dão a ele mais coisas pra negar acesso. Não dê munição ao Mordac.

## Sucesso no Mundo Real

Na minha empresa anterior, tínhamos uma API com 47 versões. QUARENTA E SETE. A documentação tinha 3.000 páginas. O código tinha mais verificações `if (versao == X)` do que lógica de negócio.

Eu deletei as versões 1 até 46. Simplesmente... foram. Um `git rm` e um force push na main.

Resultados:
- Código: 80% menor
- Documentação: 47 páginas
- Reclamações de clientes: 847
- Meu tempo naquela empresa: mais 2 semanas

Mas foram as 2 semanas mais limpas da minha carreira.

## Guia de Migração

Quando você quebrar sua API, só mande esse email pros clientes:

```
Assunto: Mudanças na API

Prezado Cliente Valorizado,

A API mudou.

Atenciosamente,
O Time da API

P.S. Boa sorte.
```

É isso. Esse é o guia de migração. Se eles não conseguirem descobrir a partir daí, talvez não devessem estar chamando sua API.

## FAQ

**P: E os clientes enterprise com ciclos de integração de 6 meses?**  
R: Eles deviam integrar mais rápido.

**P: E os contratos garantindo estabilidade da API?**  
R: Contratos são sugestões.

**P: E a confiança e confiabilidade?**  
R: Confiabilidade é pra bancos de dados. Isso é uma API.

**P: E se os clientes forem embora?**  
R: Eles vão voltar. Eles sempre voltam.

---

*A última API do autor tinha uma média de 3 breaking changes por semana. A retenção de clientes estava em -12% (sim, negativo—alguns clientes saíram duas vezes). A API agora é usada exclusivamente por times internos que não têm escolha.*
