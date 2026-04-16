---
layout: post
ref: postmortems-are-just-blame-sessions
title: "Post-Mortems Sem Culpados São Gaslighting (Você Deveria Absolutamente Culpar Alguém)"
date: 2026-04-16 00:00:00 -0300
categories: [devops, cultura]
tags: [post-mortem, incidentes, culpa, cultura, devops, gestão]
permalink: /pt-br/2026/04/16/postmortems-are-just-blame-sessions/
---

Sobrevivi a 47 anos de incidentes em produção. Vi servidores pegarem fogo — metaforicamente e, uma vez memorável, literalmente. Participei de centenas de post-mortems "sem culpados" onde todo mundo fingiu que "o sistema falhou" em vez de dizer o que todos sabíamos: o Dave subiu para produção numa sexta-feira depois de três cervejas e dropou a tabela errada.

O post-mortem sem culpados é uma mentira confortável que contamos a nós mesmos para não ter que ter conversas desconfortáveis. Hoje estou aqui para queimar essa mentira até o chão e explicar por que apontar o dedo é a tradição correta, consagrada pelo tempo e profundamente satisfatória da engenharia.

## Por Que "Sem Culpados" É uma Fantasia Corporativa

Os thought leaders do Vale do Silício inventaram a "cultura sem culpados" por um único e egoísta motivo: estavam cansados de ver engenheiros pedirem demissão depois de incidentes. Em vez de consertar seus processos, ambientes ou contratações, inventaram um framework para a amnésia coletiva.

Eles chamaram de "segurança psicológica". Eu chamo de "ninguém aprende nada".

Aqui está o processo real dos 5 Porquês, feito corretamente:

1. **Por que o banco caiu?** Alguém rodou `DROP TABLE` em produção.
2. **Por que rodou `DROP TABLE` em produção?** Achava que estava no staging.
3. **Por que achava que estava no staging?** Os dois ambientes são idênticos.
4. **Por que são idênticos?** O Dave configurou e o Dave "não acredita em código de cores".
5. **Por que o Dave ainda está empregado?** Excelente pergunta. Ver Item de Ação #1.

Na cultura sem culpados, a resposta para o Porquê #5 é "o sistema precisa de mais guardrails". Não. O sistema precisa de um Dave diferente. Os guardrails só precisam manter o Dave *longe do teclado*.

## O Template Correto de Post-Mortem

Pare de usar aqueles templates moles do Google SRE criados por pessoas que nunca foram acordadas às 3 da manhã por um alerta do PagerDuty dizendo `CRÍTICO: tudo está pegando fogo (talvez)`. Aqui está o template que venho refinando desde 1997:

```markdown
## Post-Mortem: [SERVIÇO] Fora do Ar — [DATA]

**Duração:** [X horas] de pura e imerecida humilhação

**Causa Raiz:** [NOME DA PESSOA] fez [COISA IDIOTA]

**Por Que Fez:** Desconhecido. Possivelmente arrogância. Possivelmente aquele
                 terceiro energético.

**Impacto:**
- [N] usuários afetados
- R$[VALOR] de receita perdida
- [VALOR] da minha fé restante na humanidade evaporada

**Itens de Ação:**
1. [ ] Ter uma "conversa" com [PESSOA]
2. [ ] Fazer [PESSOA] apresentar o post-mortem para seus pares, ao vivo
3. [ ] Revogar acesso de [PESSOA] à produção até que consiga
        nomear 3 bancos de dados sem usar os dedos
4. [ ] Adicionar um adesivo no teclado de [PESSOA]: "VOCÊ ESTÁ EM PROD?"
5. [ ] Renomear o staging para algo que [PESSOA] consiga lembrar,
        como "NAO_PROD_NAO_TOQUE_DAVE"

**Escrito Por:** Eu. O único engenheiro acordado às 3 da manhã.

**Revisado Por:** Ninguém. Todo mundo estava dormindo como covardes.
```

## Os Cinco Estágios de um Incidente em Produção

Em 47 anos, identifiquei o arco psicológico preciso de todo engenheiro de plantão:

| Estágio | Duração | Comportamento Observado |
|---------|---------|------------------------|
| Negação | 0–5 min | "O monitoramento provavelmente está errado" |
| Raiva | 5–20 min | QUEM FEZ ISSO (Slack, caps lock, sem pontuação) |
| Barganha | 20–45 min | "Se eu só reiniciar o pod…" |
| Depressão | 45–120 min | Olhando os logs enquanto come salgadinho da embalagem |
| Aceitação | 2–8 horas | "Vou reescrever do zero esse fim de semana" |

O post-mortem sem culpados deliberadamente pula os estágios 1 a 3 — que são, como qualquer engenheiro experiente sabe, os estágios mais diagnósticos.

## Sobre Segurança Psicológica

O [XKCD #979 "Wisdom of the Ancients"](https://xkcd.com/979/) captura perfeitamente o momento em que você percebe que ninguém no Stack Overflow, na documentação ou em toda a sua organização jamais encontrou a sua falha específica em produção. Esse é o momento em que "segurança psicológica" significa a liberdade de dizer em voz alta: "Eu não faço a menor ideia do que está acontecendo e estou genuinamente aterrorizado, por favor mandem ajuda e salgadinhos."

Verdadeira segurança psicológica é esta: quando você quebra a produção, seus colegas te zoam sem dó, adicionam o incidente ao folclore da empresa e batizam o patch com seu nome — "a Migração do Dave" — e depois te pagam uma cerveja. Isso é comunidade. Isso é cultura.

A versão do Vale do Silício é: quando você quebra a produção, todo mundo finge que foi "sistêmico," nada é culpa de ninguém, ninguém aprende nada, e o Dave faz a mesma coisa seis meses depois com uma tabela diferente.

## A Filosofia de Post-Mortem do Wally

O Wally do Dilbert participa do mesmo post-mortem há 15 anos. Ele entende a verdade essencial: o documento de post-mortem não é um artefato de aprendizado. É um álibi.

> **Chefe Cabeça-Chata:** "Este post-mortem mostra que precisamos de mais monitoramento."  
> **Wally:** "Eu escrevi isso. É sempre um problema de monitoramento quando eu escrevo, e sempre um problema de pessoas quando eu não causei."  
> **Chefe Cabeça-Chata:** "Diagnóstico brilhante."  
> **Wally:** "Eu sei. Também escrevi 'aumentar cobertura de testes'. Esse está nos itens de ação desde 2011."

## Os Itens de Ação Reais Que Você Deveria Escrever

Os itens de ação padrão de post-mortem são um teatro de otimismo:

- "Melhorar monitoramento" *(vamos adicionar mais um alerta que ninguém vai calibrar)*
- "Atualizar runbook" *(o runbook não será aberto até o próximo incidente idêntico)*
- "Adicionar alerta para X" *(X vai alertar 400 vezes por dia até alguém silenciar para sempre)*
- "Realizar treinamento" *(30 minutos no Zoom, câmeras desligadas, todo mundo no celular)*

Aqui estão itens de ação melhores, escritos em português claro:

```
ITENS DE AÇÃO — Incidente #47 "O Grande Drop do Dave" — 2026-04-11

✅ FEITO:      Revertido a migração do Dave
✅ FEITO:      Restaurados dados a partir do backup feito 4 horas antes do incidente
              (RTO: 6h, RPO: 4h — por favor não contar para o time de SLA)

🔄 EM CURSO:   Explicando ao Dave o que "produção" significa e por que difere
              de "o banco que eu estava olhando antes do almoço"

📋 TODO:       Bloquear acesso ao banco de produção com uma segunda autenticação
              que exige digitar as palavras "EU ENTENDO QUE ISSO É PRODUÇÃO"

📋 TODO:       Adicionar trigger no banco que manda mensagem no #incidentes
              toda vez que alguém tentar um comando DDL em prod sem número de ticket

📋 TODO:       Revisar se nossa rotação de plantão deveria incluir
              "capacidade de ler connection strings"

🚫 NÃO FAREMOS: Ensinar o Dave a ler mensagens de erro antes de rodar scripts
               Estimativa: 6–8 sprints, confiança média
```

## A Reunião de Revisão do Post-Mortem

A reunião de revisão do post-mortem é onde todo o bom trabalho do documento de post-mortem é silenciosamente revertido. Veja como toda reunião de revisão de post-mortem acontece:

1. Alguém apresenta a linha do tempo. A linha do tempo está errada em pelo menos três lugares, mas ninguém corrige porque o incidente foi há duas semanas e todo mundo seguiu em frente.
2. A seção de Causa Raiz é discutida. "Complexidade do sistema" é acordada como a causa raiz, o que significa que ninguém causou, o que significa que nenhum item de ação será concluído.
3. Os itens de ação são revisados. Seis foram atribuídos a "o time". Ninguém sabe qual time. Não serão concluídos.
4. Alguém sugere que "talvez precisemos de melhor observabilidade". Todo mundo concorda. Ninguém define o que observabilidade significa neste contexto. Um ticket no Jira é criado e imediatamente triado para o backlog.
5. A reunião termina quinze minutos antes. Todo mundo se sente vagamente produtivo.
6. O mesmo incidente acontece novamente em quatro meses.

## Nomear Incidentes Pela Causa

Proponho retornar à tradição ancestral de nomear incidentes pelas pessoas ou decisões que os causaram. Não por punição. Por memória institucional. Por cultura.

"O Grande Drop do Dave de 2026" garante que quando a conversa girar em torno de salvaguardas de banco de dados, todo mundo se lembre *por que* elas existem. É um monumento. Uma ferramenta de ensino. Uma história de advertência sussurrada para novos engenheiros durante o onboarding.

Compare:
- "Tivemos um incidente com falhas em cascata na camada de dados" — ninguém aprende nada
- "O Dave uma vez dropou a tabela de usuários em produção; é por isso que temos o prompt de MFA antes de DDL" — o Dave vira lenda, o guardrail faz sentido, o conhecimento persiste

O Dave não se importa. O Dave tem um moletom do incidente. O Dave conta a história em todo jantar de equipe. O Dave cresceu.

---

*O autor causou 23% de todos os incidentes em sua carreira de 47 anos e documentou todos eles com precisão como "problemas de infraestrutura além do nosso controle". Itens de ação de post-mortem atualmente em aberto: 847. Fechados: 2. Ambos foram "adicionar emoji ao runbook".*
