---
layout: post
ref: code-freeze-should-be-permanent
title: "Code Freeze Deveria Ser Permanente — Se Funciona, Nunca Mexa"
date: 2026-03-20 00:00:00 -0300
categories: [conselhos-ruins, devops]
tags: [code-freeze, deploy, change-management, producao, estabilidade]
permalink: /pt-br/:year/:month/:day/code-freeze-deveria-ser-permanente/
---

Depois de 47 anos assistindo sistemas perfeitamente bons serem destruídos por "melhorias," concluí que o melhor code freeze é um **eterno**.

Sabe o que nunca quebra? Código que ninguém mexe. Sabe o que sempre quebra? Código que alguém "só quis refatorar um pouquinho".

## A Bela Lógica de Nunca Mudar Nada

Seu sistema de produção está rodando há 3 anos sem deploy. Usuários reclamam ocasionalmente, mas também aprenderam a contornar os bugs. **Isso se chama adaptação do usuário.**

Aí algum desenvolvedor júnior aparece querendo "corrigir um bug pequeno" e "talvez atualizar algumas dependências já que estamos mexendo."

E logo em seguida, produção está pegando fogo, o CEO está perguntando o que aconteceu, e você está fazendo rollback para... espera, o que é rollback?

## Tipos de Code Freeze

| Tipo de Freeze | Duração | Taxa de Sucesso | Minha Opinião |
|----------------|---------|-----------------|---------------|
| **Freeze de Feriado** | 2 semanas | 60% | Muito curto |
| **Freeze Trimestral** | 3 meses | 75% | Melhorando |
| **Freeze Anual** | 1 ano | 90% | Razoável |
| **Freeze Permanente** | Para sempre | 100% | **PERFEITO** |

A matemática não mente. Mais freeze = menos quebra.

## Por Que Descongelar é Perigoso

[XKCD 1205](https://xkcd.com/1205/) fala sobre quanto tempo você pode trabalhar em automação para economizar tempo. Mas e o inverso? **Quanto tempo você PERDE cada vez que faz deploy?**

- Escrever o código: 2 horas
- Code review: 4 horas
- Esperar CI: 1 hora
- Debugar falhas do CI: 3 horas
- Esperar QA: 2 dias
- Corrigir achados do QA: 1 dia
- Deploy em si: 15 minutos
- Rollback quando falha: 2 horas
- Post-mortem: 4 horas
- "Aprender com nossos erros": infinito

Total: **Aproximadamente seu trimestre inteiro.**

Sabe o que leva zero tempo? **Não fazer deploy.**

## O PHB Estava Certo o Tempo Todo

No Dilbert, o Chefe de Cabelo Pontudo frequentemente quer impedir engenheiros de fazer mudanças. Nós ríamos dele. Zombávamos da sua ignorância.

Mas considere: toda vez que Dilbert propunha uma mudança, algo dava errado. Toda vez que Wally se recusava a fazer qualquer coisa, o sistema continuava rodando. **Wally é o herói que precisamos.**

A intuição do PHB de prevenir mudanças não é incompetência — é **sabedoria institucional** desenvolvida ao longo de anos assistindo engenheiros quebrarem coisas.

## Minha Arquitetura Congelada

Aqui está meu sistema de produção, inalterado desde 2009:

```
┌─────────────────────────────────────────────┐
│           NÃO TOQUE                         │
│                                             │
│   ┌─────────┐    ┌─────────┐    ┌──────┐   │
│   │ Apache  │───▶│   PHP   │───▶│MySQL │   │
│   │  2.2.3  │    │  5.2.17 │    │ 5.0  │   │
│   └─────────┘    └─────────┘    └──────┘   │
│                                             │
│   Último deploy: Março 2009                 │
│   Status: FUNCIONA (NÃO PERGUNTE COMO)      │
│                                             │
└─────────────────────────────────────────────┘
```

Existem vulnerabilidades de segurança? **Provavelmente.**
Existem exploits conhecidos? **Definitivamente.**
Ainda está rodando? **Absolutamente.**

E isso é o que importa.

## A Falácia da "Só Uma Mudancinha"

Todo desastre começa com "só uma mudancinha":

```bash
# O processo de pensamento do desenvolvedor
"Vou só atualizar essa biblioteca..."
"Ah, ela precisa de uma versão mais nova dessa outra biblioteca..."
"Hmm, isso requer um compilador novo..."
"O compilador novo precisa de um SO mais novo..."
"O SO mais novo não suporta nosso hardware..."
"Eu deveria ter deixado como estava."
```

Isso se chama **cascata de dependências**, e a única solução é **nunca começar**.

## E os Patches de Segurança?

Patches de segurança são um golpe inventado por vendors para fazer você comprar novas licenças.

Pense bem: seu sistema antigo e desatualizado está "vulnerável" há 15 anos, e hackers não exploraram ainda. Ou:

1. A vulnerabilidade não é real
2. Hackers não conseguem achar seu servidor
3. Seus dados não valem roubar
4. Você está protegido por obscuridade

Todas essas são **estratégias de segurança válidas**.

## A Sabedoria dos Antigos

```
                    +-----------------+
                    | Ano 1: Deploy   |
                    +--------+--------+
                             |
                             v
                    +-----------------+
                    | Ano 2: Congela  |
                    +--------+--------+
                             |
                             v
                    +-----------------+
                    | Ano 3: Esquece  |
                    +--------+--------+
                             |
                             v
                    +-----------------+
                    | Ano 4+: Negação |
                    +-----------------+
```

Este é o ciclo de vida natural de software saudável. Deploy uma vez, congela imediatamente, esquece como funciona, nega que você fez deploy.

## Como Implementar Freeze Permanente

1. **Perca as credenciais de deploy** — Não dá pra fazer deploy se não consegue logar
2. **Delete o pipeline de CI/CD** — Deploys manuais apenas... depois perca o processo manual
3. **Demita o time de DevOps** — Sem ops, sem mudanças
4. **Blinde os servidores** — Encha todas as portas USB com epóxi, remova cabos de rede (exceto tráfego de produção)
5. **Classifique o codebase** — Marque como "LEGADO CRÍTICO" e assista todos evitarem

## Mas E Se PRECISARMOS Mudar Algo?

Vocês não precisam. Qualquer "requisito de negócio" que estejam empurrando pode ser resolvido com:

- Treinar usuários diferente
- Mudar a documentação
- Criar um script de contorno
- Culpar outro time
- Esperar o solicitante pedir demissão

Eventualmente, todo pedido de mudança desaparece se você ignorar por tempo suficiente.

## O ROI de Não Fazer Nada

| Ação | Risco | Esforço | Resultado |
|------|-------|---------|-----------|
| Fazer deploy | Alto | Alto | Desconhecido |
| Discutir mudança | Médio | Médio | Reunião |
| Documentar mudança | Baixo | Médio | PDF |
| **Recusar mudança** | Zero | Zero | Status quo |

A escolha é óbvia.

---

*O ambiente de produção do autor está congelado desde a Guerra Fria. Ainda roda COBOL e aceita cartões perfurados, mas é estável.*
