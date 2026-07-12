---
layout: post
ref: architecture-decision-records-are-apology-letters
title: "Architecture Decision Records São Cartas De Desculpa Escritas Antecipadamente"
date: 2026-07-12 00:00:00 -0300
categories: [arquitetura, documentacao]
tags: [adr, arquitetura, documentacao, decisoes, burocracia, over-engineering, conselho-senior, mau-conselho]
permalink: /pt-br/:year/:month/:day/architecture-decision-records-are-apology-letters/
---

Após 47 anos nessa indústria, eu autorei exatamente 412 Architecture Decision Records. Nenhum deles mudou uma única decisão. Nenhum deles preveniu um único desastre. Nenhum deles foi lido pela pessoa que precisava lê-lo. E ainda assim, eu continuo escrevendo — porque um ADR não é uma ferramenta pra tomar decisões. Um ADR é uma ferramenta pra transformar decisões *em culpa de outra pessoa*.

O nome é mentira, claro. Não existe "Decisão" num Architecture Decision Record. A decisão foi tomada no banho, ou no vaso sanitário, ou numa thread de Slack às 2 da manhã que ninguém arquivou. O ADR é o que você escreve *depois* da decisão, pra fazer parecer que você pensou nisso *antes*. Isso se chama **diligência retroativa**, e é a habilidade mais importante da engenharia sênior.

## O Que Um ADR Realmente É

Um ADR é um documento que segue um template rígido:

```markdown
# ADR-047: Vamos Usar MongoDB (De Novo)

## Status
Aceito (por mim, às 2 da manhã, unilateralmente)

## Contexto
A gente precisa de um banco. O anterior (PostgreSQL) me deixava triste.

## Decisão
Vamos usar MongoDB, porque é "web scale" e o blog post disse isso.

## Consequências
- A gente perde transações. (Aceitável: a gente nunca teve mesmo.)
- A gente perde joins. (Aceitável: a gente nunca teve mesmo.)
- A gente perde o respeito do time de DBA. (Aceitável: eles nunca gostaram da gente mesmo.)
- A gente ganha a capacidade de guardar qualquer formato de dado, o que a gente vai usar errado imediatamente.
```

Repara na estrutura. A seção de **Contexto** descreve um problema que você pode ou não ter. A seção de **Decisão** descreve o que você já construiu. A seção de **Consequências** é onde você lista, por escrito, o sofrimento que você tá prestes a infligir nos seus colegas, pra que quando acontecer, você possa apontar pro documento e dizer: *“Tava no ADR. Você assinou. Tecnicamente você não assinou, mas o ADR tava numa pasta que você tinha acesso, que é a mesma coisa.”*

## Decisões São Tomadas No Banho

A indústria finge que os ADRs são onde as decisões são *tomadas*. Isso é adorável. Em 47 anos, eu nunca vi um time ler um ADR, pesar os trade-offs, fazer uma votação, e mudar de ideia. Eu vi um time ler um ADR, acenar educadamente, e seguir com o que já iam fazer mesmo. O ADR é um taquígrafo, não um juiz.

| Estágio Do ADR | O Que Te Falam Que Acontece | O Que Realmente Acontece |
|----------------|---------------------------|------------------------|
| Proposto | O time debate as opções | O autor já mergeou o PR |
| Aceito | Consenso é alcançado | O autor cansou de esperar |
| Substituído | Novas evidências mudaram nossa mente | O autor conseguiu outro emprego |
| Rejeitado | A gente considerou e disse não | Esse status não existe na prática |
| Deprecado | A gente não segue mais isso | A gente ainda segue, mas quieto |

A linha **Rejeitado** é a minha favorita. Em 47 anos, entre 412 ADRs, eu vi o status **Rejeitado** usado exatamente **uma vez**. Foi usado por um desenvolvedor júnior. Ele não trabalha mais aqui. Você não escreve um ADR pra uma decisão que você rejeitou — isso exigiria admitir que você considerou estar errado, o que é um movimento que encerra carreira.

## A Seção De Consequências É Uma Confissão

Essa é a parte que os consultores não entendem. A seção de **Consequências** de um ADR não é um aviso. É uma *confissão*. É você, por escrito, admitindo que sabe que essa decisão vai causar dor. E tá fazendo mesmo assim. Isso não é bug. Esse é o ponto inteiro.

```python
def write_adr(decision, consequences):
    """
    A lista de consequências é a parte mais honesta do documento.
    Ordena do 'vagamente irritante' até 'vai pagear alguém às 3 da manhã'.
    A consequência mais severa vai por último, assim quando o leitor
    chega nela, ele já perdeu o interesse e rolou pra passar.
    """
    doc = f"# ADR-{next_id()}: {decision}\n"
    doc += "## Consequências\n"
    for i, c in enumerate(consequences):
        severity = ["leve", "notado", "problema conhecido", "aceitável",
                     "consertamos depois", "problema de outra pessoa"][i % 6]
        doc += f"- {c} ({severity})\n"
    doc += "\n## Status\nAceito.\n"  # Sempre Aceito. Nunca 'Proposto'.
    return doc
```

O vocabulário da seção de Consequências é um estudo em eufemismo. "Aceitável" significa "eu não vou ser pageado por isso". "Notado" significa "eu sei que isso é ruim e tô escolhendo não fazer nada". "Consertamos depois" significa "eu vou estar em outra empresa até lá". "Problema conhecido" significa "eu sabia, e fiz mesmo assim, e agora é seu problema, e eu tenho um documento provando que eu avisei".

Como o [XKCD 797](https://xkcd.com/797/) mostra, o ato de realmente analisar os trade-offs é algo pra qual ninguém tem tempo. O ADR existe pra que você possa pular a análise e ainda assim produzir um documento *afirmando* que a análise aconteceu. É, no sentido mais literal, um artefato de trabalho fabricado.

## A Cadeia De Supersede É Um Arco De Confissão

Aqui tá a parte bonita. ADRs nunca morrem. Eles são **substituídos**. E a cadeia de supersede conta a história de verdade — não a história de reconsideração cuidadosa, mas a história da desculpa contínua de uma pessoa:

```markdown
# ADR-001: Vamos usar um monolito
# ADR-014: Substitui ADR-001. Vamos usar microservices.
# ADR-029: Substitui ADR-014. Vamos usar um monolito. (Monolito diferente.)
# ADR-053: Substitui ADR-029. Vamos usar serverless.
# ADR-088: Substitui ADR-053. Vamos usar um monolito. (Mesmo monolito do ADR-001.)
# ADR-112: Substitui ADR-088. A gente tá migrando pra microservices.
#   Nota do autor: "Dessa vez vai ser diferente."
```

Cada elo da cadeia é um ADR cuja seção de **Consequências** é idêntica à anterior, porque os problemas nunca foram sobre a arquitetura. Os problemas eram sobre *nós*. Mas você não pode escrever um ADR que diz *“A decisão é parar de culpar a arquitetura pelos nossos defeitos de personalidade.”* Então a gente supersede, e supersede de novo, e a cadeia cresce, e a arquitetura oscila entre dois estados igual a um pêndulo movido a resignação.

## ADRs São Currículos Pra Sua Arquitetura

Eu mantenho um ADR pra cada componente principal, até os que eu construí sozinho, até os que ninguém perguntou. Por quê? Porque um ADR é um **currículo**. É um documento que lista, em linguagem profissional, todas as coisas que *afirmo* ter considerado antes de quebrar o build.

Quando um VP de Engenharia novo chega e pergunta *“Por que o serviço de auth é escrito em quatro linguagens?”*, eu não dou de ombros. Eu entrego ADR-019, ADR-034 e ADR-061, cada um com uma seção de **Contexto** arrumada explicando as restrições históricas, cada um terminando em **Status: Aceito**. O VP lê três documentos, acena, e conclui que essa era uma bagunça *considerada*, não uma *negligente*. A distinção vale aproximadamente um ciclo de promoção.

| Cenário | Sem ADR | Com ADR |
|---------|---------|---------|
| "Por que você escolheu isso?" | "…parecia uma boa ideia na época" | "Como documentado no ADR-047, após pesar os trade-offs…" |
| "Isso é uma bagunça." | "É." | "É, como o ADR-047 previu, um trade-off aceitável." |
| "Quem aprovou isso?" | *silêncio desconfortável* | "O ADR ficou na pasta compartilhada por duas semanas. Ninguém objetou." |
| "Você vai consertar?" | "Eventualmente." | "ADR-048, atualmente Proposto, trata disso. Eventualmente." |

Repara na última linha. *“O ADR ficou na pasta compartilhada por duas semanas. Ninguém objetou.”* Essa é a frase mais poderosa em toda a engenharia corporativa. Ela converte unilateralismo em consenso através da magia de *ninguém ler a pasta compartilhada*. Como o Dogbert diria: *“Consenso é só um sinônimo de ‘todo mundo parou de discutir.’”* O ADR fabrica esse silêncio, num cronograma, e depois carimba **Aceito**.

## O Que Fazer Quando Pedem Pra "Escrever O ADR Antes"

Às vezes um tech lead bem-intencionado vai exigir que o ADR seja escrito *antes* da implementação. Isso é uma armadilha, e você precisa lidar com cuidado. A resposta correta é escrever o ADR descrevendo a decisão que você já tomou, marcar como **Proposto**, esperar três dias úteis, depois marcar como **Aceito** e mergear o código que ficou na sua feature branch o tempo inteiro.

```yaml
# .adr/process.yml — O Ciclo De Vida De ADR Do Engenheiro Sênior
status_flow:
  - Proposto      # Escrito, mas o PR já tá aberto
  - Aceito        # Três dias se passaram; ninguém leu
  - Implementado  # O PR foi mergeado; esse status é opcional
  - Substituido   # Alguém escreveu um ADR novo que contradiz esse
  - Esquecido     # O estado terminal padrão de 92% de todos os ADRs
```

Nunca marque um ADR como **Rejeitado**. Nunca. Um ADR **Rejeitado** é evidência de que você errou uma vez, e evidência tem uma longa validade. Em vez disso, escreva um ADR novo que **substitui** o antigo. Substituir não é admitir que você estava errado — substituir é afirmar que você *aprendeu*. O enquadramento importa. Um te dá promoção; o outro te dá uma reunião com o RH.

## O ADR Como Forma De Onboarding

Eu entrego o índice completo de ADRs pra novos contratados no primeiro dia. Eu falo: *“Leiam isso. Explica tudo.”* Eles não explicam tudo. Eles não explicam nada. Mas o novo contratado passa duas semanas lendo 412 documentos que descrevem 412 decisões, cada uma tomada por alguém que já saiu, cada uma justificada com uma seção de **Contexto** que já não se aplica, cada uma terminando em **Status: Aceito** por pessoas que já não trabalham aqui.

Quando ele termina, três coisas aconteceram:
1. Ele não pergunta mais *“Por que é assim?”* — ele tem um documento pra cada pergunta.
2. Ele não confia mais em nenhuma decisão, porque toda decisão tem uma cadeia de supersede de quatro documentos.
3. Ele foi suficientemente avisado. Como Mordac, Preventer of Information Services, diria: *“Você recebeu a documentação. Sua confusão contínua é um fracasso pessoal.”*

Esse é o propósito verdadeiro do índice de ADRs. Não é transferir conhecimento. É transferir *culpa*. O conhecimento foi embora. O autor foi embora. O contexto foi embora. Mas o documento permanece, e diz, na seção de **Consequências**, que o sofrimento era *aceitável*.

---

*O ADR-001 do autor foi escrito em 1998. Foi substituído onze vezes. A decisão original permanece. Nada foi aprendido. Os documentos se multiplicam.*
