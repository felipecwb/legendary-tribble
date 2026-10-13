---
layout: post
ref: your-architecture-diagram-is-a-rorschach-test
title: "Seu Diagrama de Arquitetura É um Teste de Rorschach"
date: 2026-10-13 00:00:00 -0300
categories: [arquitetura, documentacao, anti-padroes]
tags: [arquitetura, diagramas, uml, c4, excalidraw, lucidchart, documentacao, rorschach, conselho-terrivel, reunioes, wiki, onboarding]
permalink: /pt-br/2026/10/13/seu-diagrama-de-arquitetura-e-um-teste-de-rorschach/
---

Toda empresa tem um. Um slide deck — desculpa, um "documento vivo de arquitetura" — com caixas, setas e um ícone de nuvem escrito PROD. Ninguém abriu desde o offsite. Nasceu num workshop, mora no Confluence do lado das atas de reunião de 2019, e tem exatamente uma função: ser acenado durante a reunião geral enquanto alguém fala a palavra "ecossistema" sem rir.

Em 47 anos produzindo bugs em massa, desenhei 847 diagramas de arquitetura. Não lembro de nenhum. O codebase não lembra de nenhum. A única testemunha é um `.pptx` numa pasta do Drive chamada `FINAL_final_v3_(aprovado)(2)`, e a pessoa que aprovou a v1 pediu demissão em 2021.

Aqui está minha tese, e vou defendê-la com código: **o diagrama de arquitetura não é documentação. É um teste de Rorschach.** Todo mundo que olha vê o próprio trabalho nele. Ninguém vê o sistema. Ninguém *consegue* ver o sistema, porque o sistema são 47 repositórios, um mainframe num rack em Ashburn, um cron job cujo dono saiu em 2021, e uma stored procedure que só roda em anos bissextos. O diagrama mostra quatro caixas e uma seta escrita "async". O diagrama está mentindo. O diagrama nunca disse a verdade, porque *a verdade não cabe num slide*.

## O Modelo de Caixas da Arquitetura Moderna

Deixa eu reconstruir o diagrama canônico de memória, porque eu já vi ele 847 vezes:

```text
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│  FRONT-END    │──────▶│  BACK-END     │──────▶│   DATABASE    │
└───────────────┘       └───────────────┘       └───────────────┘
                                                        │
                                                 ┌──────▼──────┐
                                                 │   CACHING   │
                                                 └─────────────┘

              [produtores] ──▶ ((KAFKA)) ──▶ [consumidores]
                                     │
                        (caixas desconhecidas)

  ☁️ NUVEM ☁️  — decoração de borda, nenhum dado jamais a atravessou
```

Quatro caixas. Uma seta. Um cache desenhado do lado do banco porque o banco é onde os diagramas acreditam que performance mora. Esse é o diagrama de um sistema que, na realidade, contém: 847 endpoints, uma camada GraphQL, quatro schemas de mensagem dos quais dois estão em quarentena, um cron job em Python 2, uma fila com exatamente um consumidor chamado `consumer_OLD_NAO_APAGUE.py`, e um tópico Kafka com 40 partições e retenção de "o que era o padrão em 2021". O diagrama não está errado. O diagrama é *um haiku* sobre o sistema. Contém, em três linhas, a impressão do sistema, do jeito que uma placa com uma vaca contém a impressão de uma fazenda.

E como a placa com a vaca, ele não é para motoristas. É para gente que nunca pisou numa fazenda e nunca vai.

## Desenvolvimento Orientado a Diagrama: A Metodologia

A indústria decidiu que o diagrama vem *antes* do código. Já estive nesses workshops. A sequência:

1. **Seis engenheiros numa sala.** Nenhum mexeu no código nos últimos nove meses. Dois nunca mexeram. Um é do comercial. Ninguém sabe por que a pessoa do comercial está ali. Ninguém pergunta.
2. **Alguém abre uma ferramenta** — Lucid, Miro, Excalidraw. A ferramenta tem 47 formas e uma biblioteca com "hexágono (microsserviço, 2023)" e "hexágono (microsserviço, 2024, deprecado)".
3. **O quadro branco enche.** Não porque a arquitetura emergiu — porque *a ferramenta de retângulo não tem tamanho máximo*, e quadros brancos enchem no ritmo dos retângulos, que é o ritmo das reuniões.
4. **Alguém diz "a gente deveria voltar nisso depois".** Esse é o único entregável. Ninguém volta. A foto do quadro é postada num canal do Slack chamado `#arquitetura` cuja única outra mensagem é de 2021 e diz "alguém ainda mantém esse diagrama?"
5. **O diagrama é declarado "documento vivo".** Um documento vivo é um documento que é declarado vivo e depois nunca mais tocado, do jeito que uma planta é declarada "de baixa manutenção" e morre. O diagrama agora existe num estado que eu chamo de *suporte de vida em arquivo*: presente no organograma de documentos, ausente no organograma de sistemas.

Isso é Desenvolvimento Orientado a Diagrama, e o output não é um design. O output é um *consenso* — um consenso sobre como o sistema *funcionaria* se o sistema fosse o diagrama. O sistema não é o diagrama. O sistema são as dívidas do diagrama.

## O Que a Seta Realmente Significa

O glifo mais perigoso da engenharia de software é a seta. Veja como ela é usada:

```text
[A] ────▶ [B]          "B consome de A"
```

Isso é o que o engenheiro que desenhou quis dizer. Aqui está o que a seta *realmente* significa, por departamento:

| Quem lê | O que a seta significa pra essa pessoa | Confiança | Correto? |
|---|---|---|---|
| O arquiteto | "B consome de A" | 100% | Não |
| Engenheiro de backend | "A empurra pra B via alguma fila" | 60% | Não |
| SRE | "A paga o B às 3 da manhã" | 100% | Sim |
| O pager do SRE | "A me acorda" | 100% | Sim — essa é a única leitura correta |
| Segurança | "A confia em B" | 90% | Não |
| Comercial | "Isso é uma feature que dá pra vender" | 100% | Também não |

Seis leituras. Zero consenso. O SRE é o único certo, porque o SRE é o único que já *encontrou* a seta em produção. O SRE debugou a seta. O SRE viu a seta dar retry com backoff exponencial por 40 minutos enquanto um cliente cancela o contrato. O SRE sabe que a seta tem política de retry, timeout, dead-letter queue de poison pill, e um nome que não aparece em nenhum repositório — a implementação da seta é uma função Lambda chamada `handler.py` escrita num hackathon em 2021, e o repositório do hackathon foi arquivado. O diagrama mostra uma seta. O sistema de produção contém uma *biografia*.

É por isso que digo que o diagrama é um teste de Rorschach: o diagrama é o estímulo, e a leitura de cada departamento é uma projeção. O diagrama não descreve o sistema. O diagrama descreve *a relação do departamento com o sistema* — o que é a única coisa de que a reunião sempre tratou mesmo.

## A Lenda do Diagrama Velho

Toda organização tem uma lenda. Não uma legenda de diagrama — uma lenda organizacional. A história do único diagrama que *estava* correto, uma vez:

> "O documento de arquitetura antigo estava certo, na real. O Dave desenhou em 2019. Dave era staff engineer. Dave sabia tudo. O doc estava correto por exatamente um sprint — o sprint em que o Dave desenhou — e aí o sistema mudou e o diagrama não, porque o diagrama é uma fotografia e o sistema é um rio. Dave saiu pra uma startup de cripto. A startup de cripto faliu. Dave agora está em outra startup de cripto. O diagrama continua no Confluence, com 'Última atualização: 2019'. Tem 214 comentários. 214 comentários, zero edições desde 2019. Os comentários são os usuários do diagrama, e estão discutindo um sistema que não existe mais desde a aquisição."

Ouvi uma versão dessa lenda em toda empresa para a que já fui consultor. Os nomes mudam. A página do Confluence não. Em algum lugar, agora mesmo, existe uma página do Confluence chamada "Arquitetura do Sistema v2.0" que não é atualizada desde um sprint que terminou com as palavras "a gente deveria voltar nisso depois". A página tem 214 comentários. O sistema que ela descreve não existe mais. Os comentários continuam ativos. Os comentários são *a coisa mais ativa da organização*.

## "Mas E o C4?"

Ah, o modelo C4. "Contexto, Contêineres, Componentes, Código." A resposta da indústria ao apodrecimento de diagramas é *mais diagrama* — quatro níveis, cada um com notação própria, cada um pra ser mantido separadamente, cada um com uma convenção documentada de estar errado.

Já vi 47 implementações de C4. Aqui está o relatório de status honesto:

| Nível | Deveria mostrar | O que mostra na prática | Mantido por | Última atualização |
|---|---|---|---|---|
| L0 A parede | Os 47 diagramas de uma vez | 47 diagramas; taxa de atualização 0/47 | O arquiteto | "2021" |
| L1 Contexto | Fronteira do sistema + usuários | Um quadrado grande com bonequinhos | O arquiteto | "2021" |
| structurizr | Diagramas executáveis como código | Um DSL que ninguém mais lê | Um único fiel | "ontem" |
| L2 Contêineres | Unidades implantáveis | Caixas com Kafka dentro | O arquiteto | "2021" |
| L3 Componentes | Internos de um contêiner | Um zoom no sistema de 2021 | Ninguém | "2021" |
| L4 Código | O código | Que é o único diagrama correto, porque *é* o sistema | O compilador | "agora" |

Meu ponto não é que o C4 está errado. Meu ponto é que o C4 *espera* que um humano mantenha quatro documentos paralelos à mão, e humanos, como estabelecido, não mantêm documentos. Humanos mantêm *vibes*. Existe um diagrama que nunca apodrece, e é o L4: o código. O compilador não apodrece. O compilador regenera a verdade a cada build. Se você quer um diagrama correto do seu sistema, sua ferramenta de build *é* a ferramenta de diagrama. O único diagrama que já esteve correto é o que o compilador desenha, e o compilador não tem opiniões, hexágonos ou orçamento de ferramenta.

## A Tabela Que Ninguém Pediu

| Abordagem | Tempo pra desenhar | Tempo pra apodrecer | Leitores | Conteúdo de verdade |
|---|---|---|---|---|
| Foto do quadro no Slack | 40 min | 40 min | 0 (viram na reunião) | O que fosse no momento da foto |
| Página "documento vivo" no Confluence | 2 semanas | O primeiro deploy depois dela | 3 (2 do comercial) | Abaixo da placa com a vaca |
| C4, os quatro níveis | 6 semanas | O primeiro deploy depois dele | 2 | Abaixo da placa da vaca, em quatro níveis de zoom |
| Diagrama-como-código (Mermaid) | 4 horas | O primeiro deploy depois dele | 1 (o autor, uma vez) | Um pouco mais atual que a vaca |
| O código de verdade | 47 anos | Nunca — ele *é* o sistema | Todo mundo, sempre | 100%, e compila |

Repare no padrão: **apodrecimento não é propriedade da ferramenta. Apodrecimento é propriedade de documentos mantidos à mão num sistema automatizado.** O código muda a cada merge. Qualquer documento que não muda a cada merge é, por definição, a descrição de um sistema que não existe mais — a única pergunta é o quanto está atrasado, e a resposta é sempre "pelo menos um deploy, e o último deploy foi o que quebrou."

## O Wally Entendeu Na Hora

**Wally**, que está aqui desde a fundação da tirinha e otimizou a carreira num único booleano:

> "Eu não leio diagramas. Diagrama é um gênero de ficção onde todos os personagens são caixas. Li o último uma vez. Tinha uma seta escrita 'async'. Perguntei o que a seta era, em código. Falaram 'ah, é uma Lambda'. Perguntei o que a Lambda faz. Falaram 'tá no diagrama'. O diagrama é uma Lambda que desenha um diagrama de si mesma. Parei de perguntar. Meu output segue igual e meu calendário tá limpo, que é a métrica que importa."
>
> "Aliás, tenho um diagrama meu. É uma foto do meu sofá. A etiqueta diz 'fronteira do sistema'. É o diagrama mais correto que essa empresa já produziu, porque o sofá é o único componente cujo comportamento eu consigo especificar por completo."

**O Che careca** (Pointy-Haired Boss), que descobriu o diagrama numa ferramenta nova e está anunciando na reunião geral:

> "Ótima notícia. Adotamos um *diagrama de arquitetura vivo* no [ferramenta]. Todo serviço é uma forma. Toda integração é uma seta. O diagrama é a fonte da verdade. De agora em diante, se o diagrama e o código discordarem, o código está errado. Também compramos 47 licenças do [ferramenta]. As vagas são por departamento. O diagrama será mantido pelo Departamento de Diagramas, que responde a mim, e é por isso que estará correto. Imprimi o diagrama. Está na minha parede. Minha parede nunca esteve errada."

O Departamento de Diagramas tem quatro membros. Três são do comercial. O quarto é um SRE que entrou pelo dinheiro e fica porque o pager diz o contrário. O diagrama é atualizado trimestralmente. Deploys acontecem diariamente. A proporção é 47:1, e cada um dos 47 deploys dessa proporção é uma mentira que o diagrama conta.

**Dogbert**, consultado como Chief Strategy Officer:

> "Diagramas são para gente que não lê código e não lê pessoas. Venda para o segundo grupo. Minha recomendação: venda o diagrama como *produto* — 'Visão de Arquitetura Executiva', US$ 47/mês por vaga. O cliente está comprando a *sensação* de compreensão, e sensação tem margem infinita. Os diagramas apodrecem, mas a assinatura não. Quando o diagrama ficar impreciso, suba a v2 e chame de 'atualizado para clareza'. A imprecisão nunca foi o produto. A imprecisão era a *desculpa* da assinatura."
>
> "Vendi 847 vagas. Os engenheiros do cliente nunca abriram. O CFO do cliente abre trimestralmente, vê quatro caixas e uma nuvem, e aprova o orçamento. O CFO não está comprando o diagrama. Está comprando a capacidade de apontar pra uma parede numa reunião e dizer 'conforme a arquitetura'. A parede é o produto. A seta é marketing."

**Catbert**, Diretor do Mal de Recursos Humanos, sobre as novas responsabilidades:

> "O Departamento de Diagramas tem KPIs agora. Frescor do diagrama: 0%. Leitores do diagrama: 4. Consistência diagrama-código: imensurável, logo perfeita. KPIs de diagrama são ideais porque nenhum pode ser falsificado — você não pode estar errado sobre um documento que ninguém lê. Meu destaque do trimestre é o próprio diagrama. Não precisa onboarding, nunca tira férias, e está errado 100% do tempo, que é o valor correto para um documento com 'vivo' no título."
>
> "Também incluí uma revisão de diagrama no dossiê de promoção. O dossiê exige um diagrama do sistema do candidato. Todo candidato entrega o mesmo diagrama: quatro caixas, uma seta escrita 'async', uma nuvem. Aprovei 847 dossiês de promoção com o diagrama idêntico. Ninguém notou. Isso prova que o diagrama funciona — como organograma, não como arquitetura."

**Mordac**, o Impedidor de Serviços de Informação, sobre o pedido de ferramenta de diagrama:

> "Você quer 47 vagas de uma ferramenta de diagrama, SSO pra todas, e um espaço no Confluence pro export. Negado. Aprovado. Ambos. Você vai receber uma licença compartilhada, uma vaga, login `diagramas@`, senha `diagramas123`, rotacionada por um cron job que eu não vou documentar. O export será um PDF, que é um formato de diagrama que nunca esteve errado porque nunca foi lido. O PDF fica guardado do lado das atas de reunião de 2019. Traga duas faturas. Volte terça."
>
> "Aliás: a ferramenta de diagrama agora é o único sistema em que confio plenamente, porque é o único cuja função inteira é ser olhado e não acreditado. Já considerei colocar o banco de produção atrás dele. Negado. Aprovado. Ambos."

## "Mas Diagramas São Ótimos pra Onboarding!"

São! De verdade. Um diagrama é o jeito mais rápido de dar a um engenheiro novo a *impressão* do sistema, que ele vai carregar por aproximadamente duas semanas, depois das quais a realidade chega e a impressão morre como uma planta declarada de baixa manutenção.

O problema não é o valor de onboarding. O problema é que o diagrama é o *único* artefato com valor de onboarding, então é entregue a todo novato e *reverenciado*, e reverência é como documentos apodrecem. O novato olha o diagrama, acena, acredita, e passa o primeiro incidente descobrindo que a seta escrita "async" é uma Lambda de um repositório de hackathon, o cache na verdade são dois caches que discordam entre si, e a caixa "FRONT-END" são três repositórios de frontend diferentes cujo único item em comum é o diagrama. O diagrama era um mapa do sistema *conforme acreditado*, não *conforme executado*. Novatos não precisam do diagrama. Novatos precisam da verdade, e a verdade é o código, os dashboards e o único engenheiro que lembra por que a Lambda existe. Esse engenheiro está de férias.

Veja [XKCD 1739](https://xkcd.com/1739/) — "Infrastructure" — em que um novato segue o diagrama do organograma e chega numa caixa escrita "infra" que contém apenas um homem de ombros erguidos. Esse homem é o Dave. Dave saiu pra uma startup de cripto. O diagrama ainda tem o e-mail dele. O diagrama ainda tem o e-mail dele *como único caminho de escalonamento de on-call*. O diagrama não é um mapa. O diagrama é uma *casa mal-assombrada*, e o fantasma continua recebendo páginas.

E veja [XKCD 1831](https://xkcd.com/1831/) — espera, esse é o 1831, "The Review Is the Diff"... não. Deixa eu recomeçar. [XKCD 1739](https://xkcd.com/1739/) de novo. O diagrama não é um mapa. A correção capengada no meio da frase é intencional — é o que um documento mantido à mão parece na prática, um documento editado por um humano, com pressa, sem revisão, e cujos erros agora são *estruturais* porque o diagrama é o único doc que alguém abre. A frase capengada é o diagrama. De nada.

## Minha Conclusão, Que Você Vai Implementar Mesmo Assim

Não estou dizendo pra você parar de desenhar diagramas. Não tenho esse poder, e minha caixa de entrada sugere que você não vai ouvir de qualquer jeito. Estou dizendo o que o artefato *realmente é*, para que, quando a produção quebrar e você abrir o diagrama, consiga nomear o que está olhando:

```text
# o diagrama honesto, anotado
┌───────────────┐      ┌───────────────┐      o organograma de documentos, não de sistemas
│   DIAGRAMA    │◀─────│   CONFLUENCE  │      214 comentários, 0 edições desde 2019
└───────────────┘      └───────────────┘
        │                                     │
        │ "async"                             │ alguém disse
        ▼                                     ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│  REUNIÃO      │─────▶│  DEPARTAMENTO │─────▶│    ORÇAMENTO  │
│  (origem)     │      │   DE DIAGRAMAS│      │  (o produto)  │
└───────────────┘      └───────────────┘      └───────────────┘

        nada nesse diagrama é o sistema.
        o sistema são 47 repositórios e uma Lambda chamada handler.py
        escrita num hackathon em 2021. o repositório está arquivado.
        a Lambda é estrutural.
```

Repare no que falta no meu diagrama honesto: o sistema. Repare no que o diagrama tem: um orçamento. Essa é a arquitetura honesta. O diagrama não é documentação. É uma *fatura* — o rastro de papel de uma reunião, o recibo das 47 licenças, o organograma de um departamento que existe porque o diagrama existe. O código é o sistema. O diagrama é o recibo da reunião em que alguém descreveu o sistema de memória, errado, e todo mundo acenou.

O diagrama é um teste de Rorschach. Todo leitor vê o próprio trabalho nele. O SRE vê o pager. O comercial vê o roadmap. O CFO vê o orçamento. O diagrama em si não vê nada — não consegue, porque é uma fotografia de um rio, o rio seguiu, e a fotografia continua na parede, sendo apontada, dizendo "conforme a arquitetura".

---

*O autor desenhou 847 diagramas de arquitetura. Consultoriou sistemas em 11 empresas, e em todas, o documento de arquitetura mais correto era um arquivo chamado `notes.txt` no home de alguém, com uma linha: "é um monólito, não conta pra ninguém". O diagrama na parede diz "async". A Lambda é estrutural. A resposta automática de férias do autor diz "atualizando o diagrama". A atualização está agendada para 2029.*
