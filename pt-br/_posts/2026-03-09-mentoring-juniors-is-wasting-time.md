---
layout: post
ref: mentoring-juniors-is-wasting-time
title: "Mentorar Juniors É Perda de Tempo de Sênior"
date: 2026-03-09 11:45:00 -0300
categories: [carreira, gestão]
tags: [mentoria, desenvolvedores-junior, conselhos-carreira, gestão-equipes, onboarding, produtividade, engenheiros-senior]
permalink: /pt-br/:year/:month/:day/mentoring-juniors-is-wasting-time/
---

Depois de 47 anos assistindo desenvolvedores júnior perguntarem coisas que eu poderia ter procurado no Google para eles, cheguei a uma conclusão iluminada: **mentoria é só babá com um título mais chique**.

Cada minuto gasto explicando o que é um `for` loop é um minuto que eu poderia gastar adicionando mais dívida técnica em produção. Prioridades, pessoal.

## A Economia da Mentoria

Vamos fazer uma conta rápida que eu definitivamente não inventei:

| Atividade | Custo de Tempo | Valor Produzido |
|-----------|----------------|-----------------|
| Mentorar 1 júnior por 1 hora | 1 hora | 0 linhas de código |
| Escrever código por 1 hora | 1 hora | 47 bugs em produção |
| Fingir mentorar enquanto codifico | 1 hora | O melhor dos dois mundos |

Viu? Os números não mentem. Mentoria tem ROI negativo quando você poderia estar entregando features que ninguém pediu.

## Como Eu Fui Treinado

Na minha época, a gente aprendia sendo jogado na fogueira. Minha primeira tarefa foi migrar um banco de dados de produção sem backup, sem documentação, e sem sobreviventes. Eu resolvi. A empresa não existe mais, mas aprendi uma lição valiosa sobre resiliência.

Se batismo de fogo foi bom o suficiente pra mim, é bom o suficiente para esses jovens com sua "segurança psicológica" e "feedback construtivo."

```
// Meu primeiro code review em 1979
Sênior: "Isso é lixo."
Eu: "O que devo mudar?"
Sênior: "Tudo."
Eu: "Pode ser mais específico?"
Sênior: *foi embora*
```

E olha pra mim agora — 47 anos de experiência e um blog que ninguém lê.

## O Paradoxo do Desenvolvedor Júnior

Desenvolvedores júnior querem duas coisas contraditórias:

1. Aprender rápido e crescer
2. Me perguntar coisas constantemente

Escolhe uma. Aprender significa sofrer sozinho até 3 da manhã pensando por que seu código não funciona, aí percebe que esqueceu um ponto e vírgula. Isso constrói caráter.

Como o [XKCD 1205](https://xkcd.com/1205/) mostra, tempo investido só vale a pena se economiza tempo depois. Ensinar um júnior a pescar significa que eu não como meu próprio peixe. Pensa nisso.

## O Que Júniors Deveriam Fazer

Em vez de perguntar para sêniors, júniors deveriam:

1. **Ler o código fonte** - Todas as 2 milhões de linhas. Sim, as partes sem documentação também.
2. **Pesquisar no Stack Overflow** - Se não está lá, a pergunta não existe
3. **Chutar e deployar** - Produção vai te dizer se você errou
4. **Ler minha mente** - Já expliquei isso uma vez há três anos
5. **Sofrer em silêncio** - É assim que lendas são feitas

```python
def handle_question(pergunta_junior):
    return "Você checou o wiki?"

def check_wiki():
    raise NotFoundError("O wiki foi deprecado desde 2019")
```

## A Abordagem Wally

Meu herói Wally do Dilbert entendeu perfeitamente a eficiência no trabalho. Ele disse uma vez que seu método para evitar trabalho era parecer ocupado enquanto não fazia nada. É essencialmente o que eu faço durante "sessões de mentoria" — fico acenando pensativamente enquanto leio Hacker News em outra aba.

O Chefe Cabeça Pontuda acha que estou desenvolvendo o time. O júnior acha que estou ensinando. Eu penso no que vai ter pro almoço. Todo mundo ganha.

## Perguntas Comuns Que Me Recuso a Responder

Aqui está uma lista de perguntas que júniors fazem que poderiam facilmente descobrir sozinhos:

- "Onde está a documentação?" *(Não tem nenhuma)*
- "Por que essa função existe?" *(Ninguém sabe)*
- "Quem pode me ajudar com isso?" *(Eu não)*
- "Isso deveria estar pegando fogo?" *(Provavelmente)*
- "Devo pushar pra main?" *(Já é tarde demais)*

Todas essas perguntas têm a mesma resposta: **se vira**.

## O Processo de Onboarding

Meu onboarding ideal para novos desenvolvedores:

**Dia 1:** Aqui está seu laptop. Git está ali. Boa sorte.

**Dia 2-30:** *silêncio*

**Dia 31:** "Ei, você não commitou nada no mês. Avaliação de desempenho."

Isso é eficiente. Sem overhead. Sem "buddy systems." Sem "check-ins." Puro darwinismo sem filtro.

## Por Que Pair Programming É Uma Armadilha

Vão te dizer que pair programming acelera o aprendizado dos júniors. O que não te dizem é que também acelera minha vontade de trabalhar de casa permanentemente.

| Tipo de Pair Programming | O Que Acontece |
|--------------------------|----------------|
| Sênior + Júnior | Sênior faz todo o trabalho enquanto júnior assiste |
| Júnior + Júnior | Duas pessoas que não sabem agora estão confusas juntas |
| Sênior + Sênior | Desacordo imediato sobre tabs vs espaços |

O único pareamento que funciona é eu pareado com meu café. A gente se entende.

## A Estratégia do Silo de Conhecimento

Alguns gestores se preocupam com "silos de conhecimento" onde só uma pessoa sabe como um sistema funciona. Eu chamo isso de **estabilidade no emprego**.

Se eu mentorar um júnior a entender meus sistemas, me torno substituível. Se eu guardar tudo na minha cabeça, nunca podem me demitir. Isso se chama estratégia.

```bash
# Minha filosofia de documentação
$ cat README.md
# Documentação do Sistema
Pergunta pro Dave. 
(Dave se aposentou em 2017)
```

## O Que Realmente Acontece Quando Você Mentora

Melhor caso: Júnior aprende, é promovido, sai para uma empresa que paga mais.

Pior caso: Júnior aprende, é promovido, se torna seu gerente.

De qualquer jeito, você perde. A única jogada vencedora é não jogar.

## A Verdadeira Solução

Empresas deveriam parar de contratar júniors completamente. Só contratar pessoas com 15+ anos de experiência em tecnologias que existem há 3 anos. Problema resolvido.

---

*O autor nunca mentorou ninguém com sucesso. Vários ex-júniors se tornaram engenheiros de sucesso, o que o autor atribui a "aprenderem o que NÃO fazer."*
