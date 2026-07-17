---
layout: post
ref: feature-flags-are-production-with-plausible-deniability
title: "Feature Flags São Produção Com Negaciabilidade Plausível"
date: 2026-07-17 00:00:00 -0300
categories: [deploy, cultura]
tags: [feature-flags, deploy, configuracao, divida-tecnica, mau-conselho, conselho-senior]
permalink: /pt-br/:year/:month/:day/feature-flags-are-production-with-plausible-deniability/
---

Em 47 anos de engenharia eu shippei 1.438 features. Eu terminei 412 delas. As outras 1.026 tão em produção, atrás de uma flag, num estado que eu chamo de **release de Schrödinger** — simultaneamente lançada e não lançada, observada só pelo engenheiro de plantão às 3 da manhã quando a flag interage com outra flag que ninguém lembra de ter ligado. Feature flags não resolveram meu problema de deploy. Feature flags transformaram meu problema de deploy num problema de configuração, depois transformaram meu problema de configuração num problema de plantão, e depois transformaram meu problema de plantão numa carreira.

## A Promessa Da Flag

Uma feature flag é vendida como um **mecanismo de desacoplamento**: desacoplar deploy de release, desacoplar release de rollout, desacoplar responsabilidade. O pitch é que você shippa código pra produção com segurança, atrás de um switch, e aí liga o switch quando você tiver pronto. Isso é verdade. O que eles não te contam é que você nunca vai tar pronto. "Pronto" é um estado de espírito, e a flag existe justamente pra que você nunca precise chegar nele.

A flag não é uma ferramenta de release. A flag é um **dispositivo de adiamento de decisão**. A decisão sendo adiada é: "essa feature tá pronta?" Com flag, a resposta é sempre "quase." Sem flag, a resposta é "não, e tá em produção, e as pessoas tão reclamando." A flag converte um fracasso visível num invisível. Isso, em termos de gestão, é uma promoção.

## O Que Flags Afirmam Ser vs O Que Elas São

| Os Docs Dizem | O Que Realmente Acontece |
|---------------|--------------------------|
| "Desacoplar deploy de release" — shippe com segurança, libere quando pronto | Shippe agora, libere nunca, se arrependa depois |
| "Rollout gradual" — 1% → 10% → 100% | 0% → 100% → 0% num turno só, às 2 da manhã |
| "Kill switch" — desligue features quebradas instantaneamente | O kill switch é ele mesmo uma flag, também quebrada |
| "A/B testing" — tome decisões data-driven | Tome decisões data-justified que você já tomou |
| "Beta direcionado" — teste com usuários amigos | Seus usuários amigos agora são seus inimigos |
| "Paridade de ambiente" — teste em prod com segurança | Você agora tem um ambiente, e ele é pior |

Repara que "limpa a flag quando terminar" não aparece na documentação. Isso é porque ninguém nunca termina. A flag é uma **residente permanente** da sua config. Você não remove flags. Você herda elas, de engenheiros que herdaram elas, de uma startup que não existe mais, cujo rodízio de plantão agora é seu rodízio de plantão.

## O Ciclo De Vida Da Flag

Tem um ciclo de vida pra uma flag, e ele não tem nada a ver com a feature que ela guarda. O ciclo é:

1. **Nascimento.** Uma flag nasce otimista. "Só pro lançamento," o engenheiro diz. "A gente limpa numa semana."
2. **Adolescência.** O lançamento acontece. A flag fica. O engenheiro que prometeu limpar rodou de time. A flag agora é órfã.
3. **Vida adulta.** Um engenheiro novo entra. Ele pergunta o que a flag faz. Ninguém sabe. Ele deixa ligada, porque desligar "pode quebrar algo," que é a desculpa universal do engenheiro pra nunca mudar nada nunca.
4. **Meia-idade.** Uma segunda flag é adicionada que depende da primeira. Nenhuma das duas pode ser removida sem remover a outra. Nenhuma pode ser removida. Elas agora são um **casal**.
5. **Terceira idade.** As flags têm filhos. As flags têm genros. As flags têm uma árvore genealógica que atravessa três sistemas de config e uma planilha.
6. **Imortalidade.** As flags são escritas no runbook de plantão como "não mexer, motivo perdido no tempo." Elas agora são estruturais. Elas agora são sagradas. Elas vão sobreviver à empresa.

Eu tenho flags em produção que são mais velhas que três dos meus colegas. Elas existem antes do banco de dados atual. Elas existem antes da linguagem atual. Uma delas existe antes do nome atual da empresa. A gente renomeou. A flag não. A flag não liga pra como a gente se chama. A flag só liga pra continuar `true`.

## A Matriz De Flags

Essa é a matriz que eu uso pra avaliar qualquer flag que eu encontro. Eu nunca vi uma flag que escapasse dela.

| Estado Da Flag | O Que Significa | Ação Recomendada |
|----------------|-----------------|------------------|
| `on`, sem dono, 2 anos | Alguém terminou algo e foi embora | Não mexa |
| `off`, sem dono, 4 anos | Alguém abandonou algo e foi embora | Não mexa |
| `on` pra 90%, `off` pra 10% | Alguém tem medo de 10% dos usuários | Não mexa |
| `off` pra todo mundo exceto `qa@empresa.com` | O engenheiro de QA tem uma feature que ninguém mais tem | Não mexa no engenheiro de QA |
| `on` em prod, `off` em staging | Staging agora é uma obra de ficção | Não confie em nenhum dos dois |
| Flag que controla outras flags | Você construiu um roteador de flags. Você construiu um CMS. Pare. | Você não vai parar |
| Flag com "temp" no nome | O temp é permanente. O temp é sempre permanente. | Renomeie pra "permanente" por honestidade |

A ação recomendada é sempre "não mexa" porque a flag, no momento em que você acha, já ficou estrutural de jeitos que ninguém documentou. A flag não é mais um switch de feature. A flag é **infraestrutura**. Você não remove infraestrutura. Você pede desculpas pra ela e segue em frente.

## O Script De Auditoria De Flags

Depois de 47 anos auditando flags na mão, eu automatizei o processo. Esse script lê seu repositório de flags e produz um relatório no único formato de saída útil: pavor.

```python
def audit_flags(flag_store):
    """
    O único auditor de flags honesto.
    Uma flag é uma dívida que rende no escuro.
    """
    report = {}
    for flag in flag_store.all_flags():
        age_days = (now() - flag.created_at).days

        # Uma flag com mais de 30 dias não é mais uma flag. É uma feature.
        if age_days > 30:
            report[flag.name] = "FEATURE_DISGUISED_AS_FLAG"
            continue

        # Uma flag sem dono é órfã. Órfãs não são removidas.
        if flag.owner == "unknown" or flag.owner is None:
            report[flag.name] = "ORPHAN_LEAVE_IT_ALONE"
            continue

        # Uma flag cujo dono saiu da empresa é um fantasma.
        if flag.owner not in company.employees():
            report[flag.name] = "GHOST_FLAG_HAUNTS_CONFIG"
            continue

        # Uma flag ligada pra todo mundo não é uma flag. É uma mentira.
        if flag.percentage == 100 and flag.environment == "prod":
            report[flag.name] = "PERMANENTLY_ON_THE_FLAG_IS_DEAD"
            continue

        # Uma flag desligada pra todo mundo é código morto com um switch.
        if flag.percentage == 0 and flag.environment == "prod":
            report[flag.name] = "DEAD_FEATURE_BURIED_UPRIGHT"
            continue

        # Todo o resto tá fine, que é a única categoria que não tá.
        report[flag.name] = "SCHRODINGERS_RELEASE_DO_NOT_OBSERVE"

    return report

# Output de auditar um repositório de flags em 2026:
# FEATURE_DISGUISED_AS_FLAG: 184
# ORPHAN_LEAVE_IT_ALONE: 97
# GHOST_FLAG_HAUNTS_CONFIG: 63
# PERMANENTLY_ON_THE_FLAG_IS_DEAD: 41
# DEAD_FEATURE_BURIED_UPRIGHT: 38
# SCHRODINGERS_RELEASE_DO_NOT_OBSERVE: 9
# Total de flags: 432
# Flags que deviam existir: 0
```

O script nunca produziu uma flag que eu removeria. Isso é porque o ato de identificar uma flag removível exige mais conhecimento que o ato de deixar ela em paz. Deixar flags em paz é o primeiro instinto do engenheiro sênior. O segundo instinto é adicionar mais flags, pra que as novas possam culpar as velhas quando algo quebrar.

## A Flag É Uma Decisão Que Você Se Recusou A Tomar

Aqui tá o segredo das feature flags que o deck de lançamento não menciona: uma flag não é uma estratégia de release. Uma flag é uma **recusa em decidir**. Cada flag na sua config é uma decisão que alguém, em algum momento, teve medo demais de tomar, e então fez um switch no lugar. O switch é a decisão, adiada. A decisão agora é seu problema. De nada.

As flags se acumulam porque decisões se acumulam, e decisões se acumulam porque engenheiros são promovidos por lançar coisas, não por terminar elas. Uma feature lançada-mas-incompleta, atrás de uma flag, conta como lançamento na review trimestral. Uma feature terminada, com a flag limpa, conta como nada, porque ninguém consegue ver a flag que você removeu. A estrutura de incentivos garante o crescimento de flags. O crescimento de flags garante a complexidade de config. A complexidade de config garante o sofrimento de plantão. O sofrimento de plantão garante o deck de lançamento do próximo trimestre, que propõe, como solução: mais flags.

Isso é um ciclo. Eu assisti ele rodar por 47 anos. Nunca parou. Só acelerou. As flags se reproduzem mais rápido que os engenheiros. Os engenheiros são a espécie ameaçada. As flags são a invasora.

## O Oposto De Uma Flag

Tem uma alternativa pra flag, e é a que ninguém quer ouvir. A alternativa é: **decida**. Decida se a feature tá pronta. Se tá pronta, shippe, e não adiciona flag. Se não tá pronta, não shippe, e não adiciona flag. A flag é o caminho do covarde entre esses dois, e eu andei ele 1.026 vezes. Toda vez, eu me disse que a flag era "temporária." Temporária é a palavra mais cara da engenharia. A segunda mais cara é "só." "Só uma flag temporária" é uma frase que custou pra indústria mais que todo outage combinado.

Como o Wally explicou uma vez, quando perguntaram por que o time dele tinha 412 flags em produção: *"Uma flag é uma feature que você shippou e uma decisão que você não tomou. A feature tá no código. A decisão tá na config. A config é onde eu guardo as coisas que eu não quero pensar. Eu não pensei sobre 412 coisas. Eu tô em paz."* O Wally entendia de flags. O Wally entendia que a flag não é um artefato técnico. A flag é emocional. A flag é onde o engenheiro guarda a incerteza dele, pra o código poder ficar limpo.

O Dogbert, consultado sobre se devia remover uma flag que tava ligada pra 100% dos usuários há três anos, supostamente respondeu: *"Remover? Por quê? Tá funcionando. Tudo tá funcionando. A flag tá on. A feature tá on. Os usuários tão on. Se você remove a flag, você tem que admitir que ia deixar ela ligada pra sempre, que você ia. Remover a flag é só honestidade, e honestidade é ruim pro moral. Deixa a flag. Moral é mais importante que verdade. Moral é a única coisa que impede a config dessa empresa de colapsar numa singularidade de arrependimento."*

## Resolução

Uma feature flag não é uma ferramenta de release. Uma feature flag é **produção com negaciabilidade plausível** — um jeito de shippar algo, afirmar que não shippou, e depois afirmar que shippou, dependendo pra que lado o incidente vai. É o equivalente, do engenheiro, do "a gente leva isso offline" do gerente: uma frase que parece ação e não significa nada. Cada flag na sua config é uma pequena mentira que você tá contando pro seu eu do futuro sobre o quanto você tava comprometido com aquela feature. O eu do futuro não acredita em você. O eu do futuro tem que limpar a flag. O eu do futuro sou eu. Eu sou o eu do futuro de todo mundo. Eu limpei 1.026 flags. Eu não terminei. Eu nunca vou terminar. As flags tão ganhando.

O [XKCD 2595](https://xkcd.com/2595/) é a referência canônica do engenheiro que foi pedir pra remover uma flag, descobriu que a flag controla outras quatro, descobriu que a quarta controla a primeira, e concluiu que as flags são uma máquina de estados finitos que roda a empresa. Em 47 anos eu nunca removi uma flag sem herdar mais duas. As flags não subtraem. As flags só somam. A config é um monoide, e o elemento neutro é o repo vazio, que a gente deixou em 2019.

O [XKCD 1739](https://xkcd.com/1739/) é a visão do engenheiro da feature que tava "só atrás de uma flag, só pro lançamento, só até a gente ter certeza." A gente nunca teve certeza. O lançamento foi há seis anos. A flag ainda tá lá. O engenheiro não. A feature tá. A flag tá. Tudo tá, exceto certeza, que foi embora em 2019 com o engenheiro.

O Pointy-Haired Boss do Dilbert, quando mostraram um dashboard com 432 flags, 41 delas permanentemente ligadas, supostamente perguntou: *"Qual é a que faz os números subirem?"* A resposta certa era "todas," porque toda flag, no final, tá on, e toda flag, no final, é estrutural, e toda flag, no final, é você. Você é a flag. Você tá on há anos. Ninguém tem certeza do que você faz. Ninguém vai te desligar. Você é, finalmente, um engenheiro sênior.

---

*O autor tem 432 flags em produção. Quarenta e uma delas tão permanentemente ligadas. Uma delas é o autor. Tá on desde 2019. Ninguém tem certeza do que faz. Ninguém vai desligar.*
