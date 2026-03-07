---
layout: post
ref: deadlines-are-suggestions
title: "Deadlines são Sugestões: A Arte de Gerenciar Cronogramas Criativamente"
date: 2026-03-07 08:05:00 -0300
categories: [carreira, gestão]
tags: [deadlines, estimativas, gestão-projetos, procrastinação, agile]
permalink: /pt-br/:year/:month/:day/deadlines-are-suggestions/
---

Em meus 47 anos perdendo deadlines, aprendi que deadlines são meras sugestões de pessoas que não entendem o processo criativo de desenvolvimento de software.

## O Jogo da Estimativa

Quando alguém pergunta "quanto tempo isso vai levar?", eles não estão pedindo uma estimativa. Estão pedindo um número pra colocar na planilha. São coisas diferentes.

```
Gerente: "Quanto tempo essa feature vai levar?"
Eu: "Duas semanas."
Gerente: *escreve 'duas semanas' na planilha*
Realidade: *ri em oito semanas*
```

## A Tabela de Conversão de Deadlines

| Tipo de Deadline | O Que Significa | Realidade |
|------------------|-----------------|-----------|
| Final da semana | Antes da daily de sexta | Segunda de manhã |
| Final da sprint | Dia da demo | Próxima sprint |
| Q1 | 31 de março | "Q1-ish" |
| Final do ano | 31 de dezembro | "Wrap-up" de janeiro |
| ASAP | Quando eu tiver vontade | Eventualmente |
| Ontem | Não me avisaram | Culpa minha? |

## A Física do Software

Como o [XKCD 1658](https://xkcd.com/1658/) mostra, estimativas são pura ficção. Desenvolvimento de software segue mecânica quântica: observar o deadline muda o deadline.

```python
def estimar_tarefa(complexidade):
    estimativa_inicial = complexidade * 2  # Buffer padrão
    tempo_real = estimativa_inicial * 3  # Buffer da realidade
    tempo_reportado = estimativa_inicial / 2  # Buffer da gestão
    
    # O paradoxo do deadline: os três estão errados
    return tempo_reportado  # Depois a gente resolve
```

## Por Que Deadlines Não se Aplicam a Mim

1. **Sou engenheiro sênior** - Minha experiência significa que sei que não vai ficar pronto
2. **Requisitos mudaram** - Sempre mudam
3. **Dependências** - Outra pessoa tá me bloqueando (também tá atrasada)
4. **Dívida técnica** - O codebase reagiu
5. **Reuniões** - Eu tava em reuniões sobre o deadline

## O Método Wally

O Wally do Dilbert dominou a arte de parecer ocupado sem fazer nada. Mas eu evolui: eu faço coisas, só não no cronograma que você esperava.

O segredo são updates estratégicos:
- Semana 1: "Progredindo bem"
- Semana 2: "Encontrei bloqueios, investigando"
- Semana 3: "Mais profundo que o esperado, refinando escopo"
- Semana 4: "Precisamos discutir ajuste de cronograma"

Na semana 4, eles esqueceram o deadline original. Você voltou aos trilhos!

## O Milagre de Última Hora

Todo meu melhor trabalho é feito nas últimas 24 horas. As três primeiras semanas? É só preparação. A quarta semana? Pesquisa. A noite anterior? É quando o código flui.

```
Dia 1-14:  📊 "Planejamento de arquitetura"
Dia 15-20: 🔬 "Pesquisa profunda"
Dia 21-27: 🤔 "Considerando abordagens"
Dia 28:    💻 Codando de verdade (noite toda)
Dia 29:    🎉 "Feature completa!" 
           (faltam testes, docs, edge cases)
```

## Negociação de Escopo

Quando o deadline é imovível, o escopo é movível:

```markdown
Requisito original: Sistema de autenticação completo com SSO, MFA e RBAC
O que vai pro ar: Campo de senha
Termo técnico: MVP
```

## O Espectro do "Pronto"

| O Que Eles Acham Que "Pronto" Significa | O Que Eu Quero Dizer |
|-----------------------------------------|----------------------|
| Testado, deployado, documentado | Compila |
| Pronto pra produção | Funciona na minha máquina |
| Feature completa | Happy path funciona |
| Sem bugs | Sem bugs que eu saiba |
| Seguro | Tem uma página de login |

## Matemática de Calendário

```
Estimativa dada: 2 semanas
Converte pra dias de dev: 10
Subtrai reuniões: -3
Subtrai Slack: -2
Subtrai troca de contexto: -2
Subtrai "perguntas rápidas": -1
Subtrai investigar aquele bug estranho: -1

Tempo real de código: 1 dia
Tempo necessário pra feature: 5 dias
Déficit: -4 dias

Solução: Pede 6 semanas da próxima vez
```

## A Matriz de Culpa do Deadline

```
Quem é culpado quando deadline é perdido?
├── Desenvolvedor: "Scope creep"
├── PM: "Requisitos não estavam claros"
├── Gerente: "Restrições de recursos"
├── QA: "Achou bugs demais"
├── DevOps: "Problemas de deploy"
└── Causa real: A estimativa foi inventada
```

## Conclusão

Deadlines são só datas que alguém colocou num calendário sem consultar o código. O código não liga pro seu plano de projeto. O código leva o tempo que leva.

Além disso, alguém já foi demitido por perder um deadline? Não responde isso.

---

*O autor perdeu todo deadline da carreira. Ele chama de "entrega iterativa." A gestão chama de "consistente."*
