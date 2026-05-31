---
layout: post
ref: requirements-are-just-suggestions
title: "Requisitos São Apenas Sugestões de Pessoas Que Não Sabem Programar"
date: 2026-05-31 00:00:00 -0300
categories: [processo, agile]
tags: [requisitos, product-management, especificacoes, planejamento, conselho-senior, agile]
permalink: /pt-br/2026/05/31/requisitos-sao-apenas-sugestoes/
---

Escrevo software há 47 anos. Nesse tempo, vi uma coisa destruir mais projetos do que código ruim, arquitetura ruim e café ruim somados: **requisitos**.

Não a falta de requisitos. A *existência* deles.

Deixa eu explicar por que documentos de requisitos são os artefatos mais perigosos da engenharia de software, e por que a jogada sênior é ignorá-los completamente.

---

## O Que São Requisitos na Verdade

Documentos de requisitos são escritos por pessoas que nunca compilaram um programa na vida. Elas usam palavras como "performático", "escalável", "intuitivo" e "em tempo real" sem fazer ideia do que essas palavras custam em horas de engenharia.

Um documento típico de requisitos contém:

| O Que Diz | O Que Significa |
|---|---|
| "O sistema deve ser em tempo real" | Eles querem que pareça rápido |
| "Deve ser intuitivo para os usuários" | Não acharam o botão no protótipo |
| "Precisa escalar para milhões de usuários" | Têm 47 usuários |
| "A UI deve ser moderna" | Viram algo no Dribbble |
| "Integrar com tudo" | Não sabem quais sistemas usam |
| "Deve ser seguro" | Leram um artigo sobre breaches |
| "Latência mínima" | Não sabem o que é latência |
| "Alta disponibilidade" | O notebook do CEO travou uma vez |

Isso não são especificações. São *sentimentos* digitados por alguém com o salário de um product manager e a precisão de um romancista.

---

## A Arquitetura de Ignorar Requisitos

A abordagem sênior de engenharia é o que chamo de **Desenvolvimento Interpretativo**™. Funciona assim:

```python
class DesenvolvedorInterpretativo:
    """
    Transforma requisitos ambíguos em código definitivo.
    Os requisitos mudam. O código permanece.
    """
    
    def interpretar_requisito(self, requisito: str) -> str:
        """
        Algoritmo de parsing de requisitos aperfeiçoado em 47 anos.
        """
        
        # Passo 1: Ler o requisito uma vez
        if "tempo real" in requisito.lower():
            return "polling a cada 30 segundos"
        
        if "escalável" in requisito.lower():
            return "funciona na minha máquina"
        
        if "amigável" in requisito.lower():
            return "tem interface gráfica"
        
        if "seguro" in requisito.lower():
            return "HTTPS"  # pronto
        
        if "moderno" in requisito.lower():
            return "adicionei um gradiente"
        
        if "IA" in requisito or "ML" in requisito:
            return "random.choice(['Sim', 'Não', 'Talvez'])"
        
        # Passo 2: Se não está claro, construa o que já construiu antes
        return self.ultima_solucao_que_implementei
    
    def tratar_feedback_stakeholder(self, feedback: str) -> str:
        """
        Protocolo de renegociação pós-entrega.
        """
        return "Isso é uma mudança de escopo. Abre um chamado."
```

---

## Os Cinco Estágios de um Documento de Requisitos

Todo documento de requisitos passa pelo mesmo ciclo de vida. Já vi isso 47 vezes:

**Estágio 1: O Documento de Visão (Semana 1)**
O PM escreve 47 páginas sobre "transformar a experiência do usuário." Há três diagramas. Nenhum faz sentido de engenharia. Tem uma citação do Steve Jobs na página 2.

**Estágio 2: O Documento de Visão Revisado (Semana 3)**
Os stakeholders tiveram uma reunião. O documento agora tem 94 páginas. A citação do Steve Jobs foi substituída por uma do Jeff Bezos. Há contradições nas páginas 12 e 67.

**Estágio 3: Os Requisitos "Finais" (Semana 6)**
A palavra "final" está no nome do arquivo. Haverá doze versões "finais" a mais.

**Estágio 4: Desenvolvimento (Semanas 7-20)**
Os engenheiros constroem o que acham que os requisitos significam. O PM não abriu o documento desde a semana 3.

**Estágio 5: Entrega (Semana 21)**
Os stakeholders veem o produto. Não é o que imaginaram. O que imaginaram nunca estava no documento. Há lágrimas. Há culpas. O documento é consultado. Os dois lados encontram evidências que apoiam sua posição. Isso acontece porque o documento é ambíguo. Sempre foi ambíguo. Este é o Estágio 1 do próximo projeto.

Wally do Dilbert resumiu melhor: *"Fiz que concordei com esses requisitos por três horas. Entendi cada palavra. Nada disso vai afetar o que vou construir."*

---

## Requisitos Reais vs. O Que Os Engenheiros Constroem

Deixa eu mostrar um exemplo real da minha carreira:

**Requisito:** "O dashboard deve mostrar as métricas-chave do negócio de relance."

**O que construí:** Uma página com três números.

**Reação dos stakeholders:** "Cadê o seletor de período? Cadê o drill-down? Cadê o export para Excel? Cadê a comparação de períodos? Cadê o acompanhamento de metas? Cadê o filtro por departamento?"

**Minha resposta:** "Não estava nos requisitos."

**Resposta deles:** "Estava implícito!"

Nada está implícito em software. Se queria um seletor de período, deveria ter escrito "seletor de período." Por escrito. No documento. Com dimensões em pixels, especificações de comportamento e tratamento de casos de borda.

Não escreveu. Então fica com três números.

Isso não é falha de engenharia. É falha de especificação. A diferença é de quem é a culpa.

---

## O Método Dogbert de Levantamento de Requisitos

Dogbert certa vez consultou para uma empresa e resumiu os requisitos como: *"Vocês querem algo que faz coisas, funciona na maioria das vezes, e não envergonha na demo."*

Em 47 anos, nunca vi um documento de requisitos melhor.

Todo o resto é detalhe. Detalhes mudam. Aspirações vagas são eternas.

Meu template pessoal para todos os requisitos:

```markdown
# Requisitos do Sistema v1.0

## Visão Geral
O sistema deve fazer coisas.

## Requisitos Funcionais
1. Deve funcionar.
2. Não deve travar. Muito.
3. Usuários devem conseguir realizar tarefas.

## Requisitos Não-Funcionais
1. Deve ser rápido o suficiente.
2. Deve aguentar a carga esperada, seja lá o que isso signifique.
3. Segurança: sim.

## Fora do Escopo
1. Tudo que esquecemos.
2. Tudo que parece caro.
3. Casos de borda.

## Critérios de Aceite
O PM diz que ficou OK.
```

Este documento entregou três produtos. Um ainda está em produção.

---

## Por Que Requisitos Detalhados São Prejudiciais

Aqui está uma verdade contraintuitiva que levei 30 anos para aprender: **quanto mais detalhado o requisito, pior o software.**

Por quê? Porque:

1. **Requisitos detalhados assumem que você sabe o que precisa.** Você não sabe. Ninguém sabe. Você precisa construir para descobrir.

2. **Requisitos detalhados criam engenharia de checklist.** Engenheiros param de pensar e começam a marcar caixas. Engenharia de checklist produz software conforme que não funciona para humanos de verdade.

3. **Requisitos detalhados ficam obsoletos imediatamente.** Quando você termina 200 páginas de especificação, o mercado mudou, os stakeholders mudaram, e dois engenheiros pediram demissão.

4. **Requisitos detalhados permitem transferência de culpa.** Viram documentos legais usados em discussões, não guias úteis. Todo mundo cita o documento. Ninguém lê o documento.

Veja [XKCD 1425](https://xkcd.com/1425/) para uma ilustração brilhante de por que até requisitos simples escondem complexidade não especificada. O requisito diz "detectar pássaros." Não diz "e tratar cada caso de borda de pesquisa em visão computacional desde 1960."

---

## A Abordagem do Engenheiro Sênior

Após 47 anos, este é meu processo real:

1. **Leia os requisitos uma vez.** Identifique a uma ou duas coisas que eles *realmente* querem (vs. tudo que dizem querer).

2. **Construa essa coisa.** Bem construída. Simples.

3. **Faça uma demo.** Observe as expressões faciais. Os requisitos reais emergirão das caras deles, não da documentação.

4. **Repita.** O segundo documento de requisitos, escrito depois de ver algo real, é sempre melhor que o primeiro.

É isso que o Manifesto Ágil quis dizer com "responder a mudanças ao invés de seguir um plano." Já fazia isso antes do manifesto. Eles roubaram de mim. Não tenho provas. Mas suspeito.

---

## A Conclusão Inevitável

Requisitos são artefatos de comunicação, não especificações. Eles dizem o que alguém está *tentando* comunicar, não o que *quer* que você construa. Seu trabalho é ser um tradutor entre aspiração humana e realidade de silício.

O engenheiro sênior lê nas entrelinhas. O júnior lê as linhas.

O sênior entrega.

O júnior pede esclarecimento.

O código do sênior está em produção.

O pedido de esclarecimento do júnior está em uma thread de email, sem resposta, desde 2022.

---

*O autor tem um documento de requisitos de 1987 que nunca terminou de ler. O software que ele descreve está em produção desde 1989. Ninguém reclamou.*
