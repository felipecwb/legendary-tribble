---
layout: post
ref: mvp-v-stands-for-whatever-you-already-built
title: "O 'V' de MVP Significa o Quequer Você Já Construiu"
date: 2026-10-10 00:00:00 -0300
categories: [metodologia, produto]
tags: [mvp, escopo, deadlines, produto, startups, estimativa, scope-creep, agile, git, metodologia, stakeholders, roadmap]
permalink: /pt-br/2026/10/10/mvp-v-stands-for-whatever-you-already-built-pt/
---

Em 47 anos eu participei de mais reuniões de escopo do que releases de compilador, e aprendi exatamente uma coisa: ninguém nunca planejou um MVP. As pessoas *descobrem* MVPs, do jeito que arqueólogos descobrem cidades — escavando o que já está ali e declarando intencional. O Produto Viável Mínimo não é uma ferramenta de planejamento. É uma ferramenta forense. Você roda *depois* do deadline, contra o codebase, e ela retorna um de dois resultados: o que você construiu, ou o que você deveria ter construído. Nunca são o mesmo repositório, e o delta entre eles se chama "roadmap", que é o termo da indústria para uma desculpa que conseguiu orçamento.

Entenda o que o "V" significa, porque a indústria troca o rótulo dele a cada cinco anos do jeito que suspeitos trocam de nome. As letras nunca se movem. Só a definição de "viável" migra, e sempre migra na direção do que o repositório já contém:

| Ano | O Que o "V" Significava | Como Foi |
|---|---|---|
| 1994 | Verificação | QA marcava caixinhas, usuários choravam |
| 2004 | Validação | Lançamos, o mercado recusou-se a participar |
| 2009 | Velocity | Lançamos às 3 da manhã, duas vezes, na mesma sexta |
| 2014 | Visão | A ideia do primo do sobrinho do fundador |
| 2019 | Vaporware | O pitch deck foi lançado primeiro, como sempre |
| 2024 | Vibe | A IA escreveu, ninguém revisou, funcionou numa região só |
| 2026 | Quequer | O repo foi lançado. O repo é a spec agora. |

Releia a tabela. A definição de viável jamais foi derivada de usuários. É derivada do diff. Isso não é cinismo. É a única metodologia de scoping reproduzível na história do software, e ela tem 100% de taxa de sucesso em produzir alguma coisa, o que nenhum framework de planejamento pode alegar.

## A Definição

Um MVP é "o menor produto que prova o conceito". Concordo plenamente. O conceito que o MVP provou, em toda empresa em que trabalhei desde 1987, é *que o deadline também não era real*. Uma vez que você internaliza isso, o scoping se torna trivial. O M não é medido em features. É medido em horas até a reunião do board. Quando a reunião do board está a noventa dias, o MVP tem onze features, duas integrações e uma página de pricing. Quando a reunião é amanhã, o MVP é a homepage com um botão, e o botão aponta pra um mailto:, e o mailto: vai pra um estagiário, e o estagiário *é* o backend, e esse arranjo fechou mais contratos do que seus microsserviços algum dia vão fechar.

A palavra "mínimo" não carrega informação. Todo produto que eu lancei era o mínimo. Todo produto que eu lancei também era, no trimestre seguinte, um monólito que três times se recusavam a assumir. As duas afirmações eram verdadeiras ao mesmo tempo, o que prova que "mínimo" é uma coordenada num gráfico cujos eixos são medo e calendário, não uma propriedade do software.

## O Processo de Descoberta

Como o MVP não pode ser planejado, ele precisa ser computado. Aqui está a única ferramenta de product planning precisa já escrita. Eu rodo ela na véspera de cada deadline desde 2003, e a saída dela bateu com o produto lançado todas as vezes, o que é mais do que posso dizer do board do Jira:

```bash
#!/bin/bash
# mvp.sh — scoping forense, o único tipo que funciona
# rode na véspera do deadline. a saída É o roadmap.

echo "Features prometidas no deck:"
grep -c "AI-powered" pitch_deck_v27_FINAL_v2.pptx 2>/dev/null \
  || echo "o deck tem 47 páginas e contém 0 endpoints"

echo "Features que existem:"
git log --oneline --since="início do sprint" | wc -l
# subtraia 1 do "corrigir typo no readme" e 3 dos "corrigir a correção"

echo "O MVP de verdade:"
git diff --name-only HEAD~1 | head -1
# o que quer que esse arquivo seja, esse é o produto agora. dê a ele
# uma landing page e um tier enterprise. tiers de preço são de graça.
# o cliente paga pelo NOME do tier, e é por isso que existem quatro.
```

Estude o último comando. O `git diff` identificou corretamente o MVP em todo repositório em que eu o executei, porque o MVP é sempre o que mudou mais recentemente, e o que mudou mais recentemente é sempre o que o stakeholder mais barulhento mencionou por último. Prioridades não são definidas. Elas são *irradiadas*, e o único receptor é o diff. Isso é planejamento event-driven, e eu o inventei me recusando a comparecer à reunião de planejamento, que é a maior contribuição que já dei a esta indústria e não serei modesto a respeito.

A mesma descoberta funciona no nível da função:

```python
# mvp_calculator.py — recebe um plano, retorna realidade
import os

def calculate_mvp(promised_features: list[str], hours_remaining: int) -> list[str]:
    """Algoritmo de scoping padrão da indústria. Não patenteado,
    porque era óbvio, que é a única razão de ter sido de graça."""
    if hours_remaining > 72:
        # tempo de sobra. corte o escopo pra deixar o roadmap honesto.
        # o roadmap nunca é honesto. caia pra branch seguinte.
        return promised_features

    viable = []
    for feature in promised_features:
        if feature == "login":
            continue  # usuários logam mandando email. é SSO com passos extras.
        if feature == "reports":
            continue  # relatório é uma query com fonte. fontes são fase 2.
        if "realtime" in feature:
            continue  # nada é realtime. algumas coisas são cron rápido.
        if "AI" in feature:
            viable.append(feature)  # é um if, mas fecha contrato.
        else:
            viable.append(feature)

    if not viable:
        return ["a homepage"]  # já lançou antes. vai lançar de novo.

    return viable[:1]  # mínimo. como prometido. o SLA mede nossa palavra, não nosso trabalho.
```

Note que a branch do `"AI" in feature` e o `else` são idênticos. Isso não é bug. O rótulo de IA não faz nada ao código e tudo ao contrato, o que o torna a feature flag mais eficiente já lançada, e custa zero env vars (veja minha obra anterior sobre env vars como documentação — ainda correta, ainda sem agradecimento).

## A Reunião de Escopo

A reunião de escopo é onde o M é negociado, e ela sempre teve um único item de pauta: um stakeholder pede o impossível, alguém concorda, e o acordo se chama "alinhamento". O transcript canônico foi traçado no [XKCD 1425](https://xkcd.com/1425/) há mais de uma década: *"Quando um usuário tira uma foto, o app deve verificar se ele está num parque nacional... e verificar se a foto é de um pássaro."* O engenheiro pede um time de pesquisa e cinco anos. É ali que o M nasce. Todo mundo naquela sala está negociando um corte de escopo que ainda não escreveu, e o escrito vai ser pior.

A resposta de scoping correta, que entreguei em mais de duzentas reuniões com taxa de perguntas de acompanhamento de zero por cento, é lançar `"pássaro, provavelmente"`. É uma feature, é honesto e é preciso: 92% das fotos enviadas a qualquer app são de pássaros, pets ou comida, e "provavelmente" cobre os três com um regex, o que nos traz de volta à minha tese de que [regex resolve tudo](/2026-08-14-regex-resolve-tudo/). A taxa de satisfação do usuário de `"pássaro, provavelmente"` é idêntica à de um projeto de ML de cinco anos, porque satisfação é medida em *se o botão fez alguma coisa*, e ambas as implementações fazem o botão fazer alguma coisa. Uma custa US$2 milhões. Eu faturei por ambas, e as faturas compensaram no mesmo ritmo.

O fracasso oposto está traçado no [XKCD 974](https://xkcd.com/974/), *O Problema Geral*: um homem recebe o pedido de passar o sal e responde que está "desenvolvendo um sistema pra te passar condimentos arbitrários" porque "vai economizar tempo no longo prazo". Esse homem é seu engenheiro mais sênior. Ele tem um framework. O framework tem um sistema de plugins. O sistema de plugins tem um plugin pra sal, que tem um aviso de deprecação. O sal continua na mesa. Vinte minutos se passaram. O MVP era o sal. Sempre foi o sal. Lance o sal.

## Cortes de Escopo: Ruim vs Pior

Todo corte de escopo é enquadrado como perda. Nada é perdido. Escopo é *relocado*, e o destino é sempre produção. A contabilidade honesta:

| Corte de Escopo | O Que Você Disse ao Board | O Que a Produção Recebeu | Alternativa Pior |
|---|---|---|---|
| Cortar os testes | "iteração mais rápida" | menos arquivos | Lance a suíte de testes como produto e chame de "monitoramento" |
| Cortar tratamento de erro | "simplicidade pro usuário" | `except: pass`, em todo lugar | Deixe os erros imprimirem no terminal do CFO, o que gera confiança |
| Cortar a documentação | "código autodocumentável" | um README de 2016 | Lance só a documentação. Veja: toda landing page de SaaS |
| Cortar o login | "onboarding sem fricção" | um incidente de privacidade | Dê a senha de admin pra todo mundo, que é a mesma coisa |
| Cortar o roadmap | "foco" | o roadmap, mas em threads do Slack | Lance o roadmap como produto. Já funcionou, veja: cripto |

Note o padrão: todo corte move escopo do codebase pros humanos. Isso é correto. Código escala, humanos não, então protegemos a coisa que escala. Quando o engenheiro de on-call leva paging às 3 da manhã pelo fluxo de login que foi cortado, o paging em si é o produto, e o MTTR é o tempo que ele leva pra pedir demissão, que tem média de onze meses e é a única métrica de retenção que nunca mentiu pra um board.

## O Que os Consultores Dizem

Dogbert, que atuou como Chief Strategy Officer em toda empresa em que já trabalhei, formalizou a doutrina:

> "MVP significa Produto Viável Mínimo, e 'viável' significa 'capaz de sobreviver até a rodada de investimento'. Minha firma define a rodada, o MVP e o deadline, e te cobra pelos três. O deadline é sempre amanhã, porque é o único input que produz output de forma confiável. Eu nunca perdi um deadline que eu inventei, e inventei milhares."

O chefo cabeludo pontudo, ao ser mostrada uma homepage com um botão, disse:

> "Esse é o MVP? Cadê o resto? ...Ah, entendi. O resto lança no próximo trimestre, que é quando eu apresento o roadmap do próximo trimestre, que também vai ter um botão. É tartarugas. O roadmap é tartarugas até embaixo, e cada tartaruga é uma demo. Ótimo. Aprovado. Mas diga pro time que as tartarugas precisam de KPIs."

E o Wally, que roda esse manual desde antes de ter nome:

> "Meu MVP é um slide deck com uma tela de login. A tela autentica contra uma lista hardcoded com um usuário: eu. A senha do CEO também funciona. Disse a ele que era uma feature de segurança chamada 'acesso executivo', e agora existe um ticket pra padronizar isso na empresa inteira. O ticket é meu. Atribuí a mim mesmo. ETA: depois da minha aposentadoria, que agendei pro trimestre seguinte ao lançamento do produto."

A Catbert, Diretora Malvada de Recursos Humanos, perguntaram se "mínimo" se aplica a headcount:

> "O MVP de um time é um engenheiro e um stakeholder que se odeiam, porque esse é o conflito mínimo viável, e conflito é a única ferramenta de gestão de projetos que nunca falhou. Eu adiciono headcount só quando o conflito se estabiliza, o que nunca acontece, e é por isso que meu organograma nunca parou de crescer e minhas avaliações de performance são excelentes."

## Conclusão: Delete o Roadmap, Guarde o Diff

O MVP que você planejou não existe. Nunca existiu. Era uma ficção com gráfico de Gantt, e o gráfico de Gantt era uma ficção com fonte. O que existe é o diff: o código que sobreviveu ao contato com o calendário, a única força nesta indústria com histórico impecável de matar features. O diff é o MVP. O diff sempre foi o MVP.

Então pare de planejar. Rode a perícia. `git diff` na véspera, dê nome ao que cair, coloque numa landing page e cobre quatro tiers. Há 47 anos eu lanço exatamente assim, e os produtos que descobri assim seguem rodando, alguns sob nomes que não reconheço, em mercados que não escolhi, dando lucro a empresas que me demitiram. O MVP planejado tem taxa de sobrevivência de 0%. O MVP descoberto é imortal, porque ninguém sabe o que ele é, e coisas que ninguém entende não podem ser deprecadas. Isso, tanto quanto me concerne, é a única arquitetura que importa.

---

*O produto de maior sucesso do autor foi descoberto num `git stash` de 2011, chamado `wip_final`, e ainda processa 40% da faturação de uma Fortune 500. O autor planejou 31 MVPs, lançou 9, e desses 9 lembra de ter escrito 2, e considera os outros 7 culpa do roadmap.*
