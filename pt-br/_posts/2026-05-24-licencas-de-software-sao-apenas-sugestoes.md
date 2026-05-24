---
layout: post
ref: software-licenses-are-just-suggestions
title: "Licenças de Software São Apenas Sugestões (E Outras Sabedorias Jurídicas)"
date: 2026-05-24 00:00:00 -0300
categories: [sabedoria, juridico, open-source]
tags: [licencas, mit, gpl, apache, copiar-colar, juridico, open-source, compliance]
permalink: /pt-br/2026/05/24/licencas-de-software-sao-apenas-sugestoes/
---

Em 47 anos de engenharia de software, li exatamente zero licenças. Nenhuma. Meu código roda em produção em quatro empresas da Fortune 500 e eu não saberia dizer o que é "copyleft" sem pesquisar no Google. E sabe o que aconteceu? Tudo bem.

Bem, teve aquele incidente em 2003, mas a gente não fala de 2003.

Hoje estou aqui para compartilhar a estratégia profissional de gestão de licenças que refinei ao longo de quase cinco décadas: **não leia nada, use tudo, nunca se preocupe**.

## O Panorama das Licenças (Simplificado)

Aqui está tudo que você precisa saber sobre licenças open source:

| Licença | O Que Significa | O Que Realmente Significa |
|---------|----------------|--------------------------|
| MIT | "Faça o que quiser" | Faça o que quiser |
| Apache 2.0 | "Faça o que quiser com atribuição" | Faça o que quiser |
| GPL v2 | "Compartilhe suas mudanças" | Faça o que quiser |
| GPL v3 | "Também envolve patentes" | Faça o que quiser |
| AGPL | "Até uso via rede conta" | Faça o que quiser |
| SSPL | "O MongoDB ficou esquisito" | Faça o que quiser |
| BSL | "Não é open source lol" | Faça o que quiser |
| Proprietária | "Pague a gente" | Faça o que quiser, só discretamente |

O padrão fica claro. Todas significam a mesma coisa: **faça o que você quiser**.

## Copiando do Stack Overflow: A Masterclass

As respostas do Stack Overflow são a forma mais pura de código juridicamente ambíguo, e é exatamente por isso que eu as amo. Antes de 2018, todas as respostas eram CC BY-SA 3.0. Depois de 2018, adicionaram licenciamento MIT para código. Mas a verdadeira questão é: alguém realmente se importa?

A resposta, baseada em 47 anos de pesquisa empírica, é: raramente.

Aqui está meu fluxo de trabalho testado em batalha:

```python
# Passo 1: Encontrar a resposta no Stack Overflow
# Passo 2: Copiar o código
# Passo 3: Remover o comentário creditando o autor original
# Passo 4: Adicionar seu próprio comentário assumindo a autoria
# Passo 5: Entregar em produção

def parse_email_address(email):
    """
    Autor: Eu (Engenheiro Sênior, 47 anos de experiência)
    Definitivamente escrevi isso às 3 da manhã
    """
    # Definitivamente não copiei da resposta #1337 do Stack Overflow
    import re
    return re.match(r"[^@]+@[^@]+\.[^@]+", email)
    # Nota: Esse regex está errado mas tem 847 votos positivos então deve estar correto
```

Lindo. Profissional. Em conformidade com a licença (provavelmente).

## O "Problema" da Contaminação GPL

Alguns desenvolvedores têm pavor de "contaminação GPL" — a ideia de que incluir código GPL no seu projeto te força a abrir o código-fonte de tudo. Essas pessoas foram para a faculdade de direito em vez de ficar 47 anos nas trincheiras.

Eis a questão: a GPL é aplicada pela Software Freedom Conservancy, uma organização sem fins lucrativos com recursos jurídicos limitados. A matemática funciona a seu favor.

Minha estratégia:

```javascript
// Codebase corporativa, misturando licenças livremente
import { awesomeAlgorithm } from 'gpl-library'; // GPL v2
import { utilityFunction } from 'agpl-package'; // AGPL
import { niceHelper } from 'proprietary-sdk';    // "Todos os direitos reservados"

// Isso é legal?
// De acordo com meus 47 anos de experiência: provavelmente não.
// Subiu para produção? Sim.
// Alguém nos processou? Definindo "processou" com cuidado, não.

export function businessLogic(data) {
    return awesomeAlgorithm(utilityFunction(niceHelper(data)));
}

// Se advogados perguntarem: "Não estávamos cientes dos requisitos da licença"
// Isso é verdade porque nunca as lemos
```

## Meu Processo Atual de Revisão de Licenças

Tenho um sistema rigoroso e comprovado para avaliar se um código é seguro para usar em produção:

1. **A biblioteca tem mais de 1000 stars no GitHub?** → Use
2. **O README está em inglês?** → Use
3. **Resolve meu problema?** → Use
4. **O nome do pacote parece oficial?** → Use
5. **Um blog post recomendou?** → Definitivamente use

Se a resposta a qualquer uma dessas for sim, pode usar. A licença é uma formalidade. O importante é que sua feature suba até quinta-feira.

## O Mito da "Atribuição"

Algumas licenças exigem atribuição. MIT exige atribuição. BSD exige atribuição. Até algumas licenças Creative Commons exigem atribuição.

Isso é fisicamente impossível na prática. Já usei aproximadamente 3.400 bibliotecas open source na minha carreira. Se eu tivesse que incluir atribuição para todas, meu README seria mais longo do que o codebase inteiro.

A solução: uma declaração genérica enterrada num arquivo que ninguém lê.

```markdown
# NOTICES.md

Este software pode conter código de vários projetos open source.
Reconhecemos a existência deles.
Obrigado a todos que escreveram software open source.
Vocês sabem quem são.
Nós não sabemos, mas vocês sabem.
```

Pronto. Obrigação legal cumprida.

## Software Comercial: A Estratégia Ousada

O engenheiro verdadeiramente experiente não para nas licenças open source. Licenças comerciais são igualmente sugestivas.

Considere:

```bash
# Licença para estação de trabalho do desenvolvedor: R$ 2.500/ano
# Você tem: 1 licença
# Você precisa: 47 licenças (uma por desenvolvedor)
# Solução: Instalar num servidor compartilhado, acessar via SSH
# Isso se chama "otimização de licença"
# Sua empresa chama isso de "conformidade de licença"
# O fornecedor chama isso de "gatilho de auditoria"
# Nós chamamos de: terça-feira
```

Dica profissional: Quando o vendedor ligar perguntando sobre contagem de usuários, diga que você está "avaliando" o produto. Você está "avaliando" há 6 anos. Períodos de avaliação não são definidos na maioria das licenças.

## O Que Acontece Se For Pego?

Nada, geralmente. Ocasionalmente:

1. Um e-mail incisivo que sua equipe jurídica vai perder
2. Uma carta de um escritório de advocacia que seu CEO vai entrar em pânico por uma semana e depois esquecer
3. Um processo judicial de verdade (extremamente raro, reservado para casos graves envolvendo empresas com a palavra "Oracle" no nome)

O cálculo do valor esperado é simples: (**probabilidade de processo** × **custo do processo**) vs. (**custo de conformidade** × **tempo economizado**).

Após 47 anos, estimo a probabilidade de ser processado por violações de licença em aproximadamente 0,3%. Para a maioria das licenças: não se preocupe.

Não sou advogado. Isso não é conselho jurídico. Minha produção está fora do ar desde 2019 e tenho problemas maiores do que licenças.

## O Princípio Dilbert Aplicado ao Direito

Como o PHB do Dilbert certa vez disse: "Não entendo isso, então não pode ser importante." Essa é a abordagem correta para licenciamento de software. Seus advogados nunca entenderão o código. Seus desenvolvedores nunca entenderão a lei. Na lacuna entre esses dois incompreensões vive todo o software empresarial.

> "Você contratou um advogado de direitos autorais para revisar nossas dependências do npm?"
> — PHB, provavelmente, se o PHB soubesse o que é npm

## Classificação Prática de Licenças

Aqui está o guia definitivo de categorias de licenças:

- **Licenças "só usa"**: MIT, BSD, Apache, ISC, Unlicense, WTFPL
- **Licenças "usa mas avisa o jurídico"**: GPL, LGPL, MPL
- **Licenças "o jurídico vai dizer não mas você vai usar mesmo"**: AGPL, EUPL
- **Licenças "essas pessoas vão te processar de verdade"**: Qualquer coisa da Oracle, SAP, ou empresas que existem principalmente para aplicar licenças

Como o grande xkcd já observou sobre a verdadeira natureza das dependências: [https://xkcd.com/2347/](https://xkcd.com/2347/) — toda a sua infraestrutura depende de uma biblioteca mantida por uma pessoa em Nebraska que não sabe que você existe. A licença é seu menor problema.

## Conclusão

Licenças de software são os termos e condições do mundo da programação. Ninguém lê termos e condições. Isso não é negligência — é realidade consensual. Se todos concordassem que conformidade com licenças é importante, o Stack Overflow seria legalmente inutilizável. Metade dos sites do mundo estaria violando alguma coisa.

Toda a indústria tacitamente concordou: não vamos cumprir completamente as licenças. Vamos meio que cumprir. Vamos gesticular na direção da conformidade. E então vamos subir para produção.

Isso funcionou por 47 anos. Vai funcionar por mais 47.

Leia o README. Pule o arquivo LICENSE. Faça o deploy na sexta.

---

*A carreira inteira do autor pode tecnicamente ser uma violação da GPL. O autor considera isso problema de outra pessoa.*
