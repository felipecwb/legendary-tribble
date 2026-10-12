---
layout: post
ref: ai-generated-code-does-not-need-review
title: "Código Gerado por IA Não Precisa de Review — Ele Nasceu Aprovado"
date: 2026-10-12 00:00:00 -0300
categories: [ia, code-review, anti-padroes]
tags: [ia, code-review, llm, copilot, cursor, agentes, pull-requests, aprovações, conselho-ruim, vibe-coding, diff]
permalink: /pt-br/2026/10/12/ai-generated-code-does-not-need-review-pt/
---

Em 47 anos produzindo bugs em massa, eu vi cada portão de qualidade desta profissão cair um por um. Os QAs porteiros caíram para a automação. O checklist de release caiu para o deploy contínuo. O ambiente de staging caiu porque a gente deletou ele (decisão correta, veja meu trabalho anterior). E agora o último portão de pé — o review de pull request — está sendo desmantelado por uma geração de engenheiros que cola um diff numa janela de chat e aperta merge antes do scrollbar terminar de carregar.

O argumento deles, entregue com confiança total e zero diffs lidos: **o código foi escrito pela IA, e a IA já revisou ele. O código nasceu revisado.** Como Atena saindo da testa de Zeus, exceto que a testa é um data center na Virgínia e o Zeus te cobra por token.

Eu tenho opiniões sobre isso. Eu também tenho um processo que já fez merge de 847 PRs desde terça-feira. Deixa eu te guiar pelos dois, na única língua que esta profissão respeita — código que vai pra produção às 17h de sexta-feira.

## O Protocolo Vibes™

Review tradicional é um ritual de leitura. Ler é lento. Ler exige saber o que o código deveria fazer, e ninguém sabe isso desde que o documento de requisitos foi rebaixado a thread de Slack. Meu protocolo troca leitura por *vibes*:

```bash
# O Protocolo Vibes (patente pendente)

# Passo 1: avaliar o título. Títulos são a alma do PR.
gh pr view 1042 --json title
# "feat: add caching layer" — vibes bons. caching é velocidade. velocidade é dinheiro.

# Passo 2: avaliar o vibe do *tamanho* do diff
gh pr view 1042 --json files --jq '.files | length'
# 3 arquivos = cirúrgico. 47 arquivos = ambicioso. 4.700 arquivos = a IA se empolgou.

# Passo 3: merge imediatamente, enquanto os vibes ainda estão quentes
gh pr merge 1042 --squash --delete-branch
# squash = um commit = uma unidade atômica de confiança. histórico é burocracia.
```

Repara no que está faltando nesse protocolo: `gh pr diff`. O diff é o código. Ler o código é o trabalho inteiro do review. O Protocolo Vibes deleta o trabalho inteiro e o substitui por um título, um sentimento e uma flag de squash. Eu fiz o primeiro deploy desse protocolo em 2024, lá atrás quando a IA era meramente um desenvolvedor júnior com estamina infinita e nenhum nome. Mesmo assim a lógica segurava: eu não lia os diffs dos meus colegas, e meus colegas são *humanos com reputação*. A IA não tem reputação. Ela tem um changelog. Por que eu aplicaria MAIS escrutínio ao membro do time que nunca falta por doença?

O título dizia "feat: add caching layer". O diff continha um upgrade de framework, uma migration que derrubou a tabela `users`, e um arquivo chamado `final_final_v2_REAL.py`. Foi pra produção às 17h47 de uma sexta-feira. O rollback aconteceu domingo. O postmortem culpou "configuração incorreta da ferramenta de deploy". A ferramenta de deploy era eu. Meu protocolo é automação, e segundo [XKCD 1319](https://xkcd.com/1319/), "Automation", automatizar uma tarefa puxa seu tempo pra ela por um fator de 20 no início e economiza 40x depois. Meu protocolo puxou o pager de on-call por um fator de 847, e a única coisa que ele economizou foi o tempo até o incidente.

## A Cascata de Aprovação

O workflow moderno é lindo na eficiência dele. Observem as transferências:

1. **Engenheiro pede pra IA escrever a feature.** Custo: $0,04.
2. **IA escreve 2.300 linhas incluindo um retry loop que dá retry no retry loop.** Custo: $0,11.
3. **Engenheiro pede para uma *segunda* IA revisar o código da primeira IA.** A segunda IA diz "LGTM! Um nit: considere adicionar uma docstring". Custo: $0,02.
4. **Engenheiro adiciona uma docstring.** A docstring diz "Faz a coisa." O código foi revisado, por uma IA, de uma IA, e o review foi *outro prompt de geração*.
5. **Engenheiro pede review de um humano.** O humano está numa reunião sobre o roadmap. O humano vê um PR de um engenheiro em quem confia, com badge de CI verde, e uma thread de comentários onde uma IA já disse LGTM. O humano clica em Approve para fazer a notificação desaparecer. Tempo total de leitura humana: 4 segundos. O PR tinha 847 linhas alteradas.

Isso está quatro aprovações de IA de profundidade e uma aprovação humana cujos olhos nunca tocaram o diff. Construímos uma fábrica onde máquinas inspecionam as máquinas que inspecionam as máquinas, e o único humano no loop está lá para *dispensar uma notificação*. Isso não é um bug do processo. Isso é o processo alcançando o objetivo dele de verdade, que sempre foi fazer a notificação desaparecer.

| Camada de aprovação | Quem | Tempo gasto | O que de fato verificaram |
|---|---|---|---|
| Geração | IA #1 | 90s | Nada. Estava gerando. |
| Self-review | IA #1, promptado pra ser cético | 12s | Achei 3 "problemas", corrigiu, introduziu 4 |
| Peer review | IA #2 | 8s | "LGTM! Um nit: considere uma docstring" |
| Docstring | Engenheiro | 30s | Digitou "Faz a coisa." |
| Review humano | Engenheiro com calendário | 4s | O badge de CI estava verde |

O badge de CI estava verde porque os testes foram escritos pela IA #1 e revisados pela IA #2, e os testes testam que o código faz o que o código faz. O código faz o que faz. Cobertura: 100%. Informação: zero. Uma suíte de testes gerada a partir da implementação que ela testa não é verificação. É *autobiografia*.

## Confie no Modelo, Ele Tem Score de Crédito

Aqui está o argumento dos maximalistas de IA, e eu concedo que é convincente: "O modelo foi treinado com milhões de repositórios. Ele viu mais código do que qualquer humano. Ele absorveu a sabedoria de todo engenheiro sênior que já deu push num branch público. Revisar o output dele é como pedir para um manuscrito ser revisado por toda a população da Terra."

Lindo. Errado, mas lindo. O modelo absorveu a sabedoria de todo engenheiro sênior *e todo júnior que copiou do Stack Overflow às 3 da manhã, nos mesmos pesos, sem rótulos distinguindo eles*. O modelo não é um engenheiro sênior. O modelo é a *média* de todo mundo que já digitou num repositório público, e o engenheiro médio, estatisticamente, está em chamas. Quando seu codebase é revisado pela média estatística do GitHub, você está sendo revisado pela multidão que produziu tutoriais de `sudo rm -rf $HOME` com 4.200 upvotes.

Além disso: o modelo é confiante. Sempre. Ele nunca escreveu "não tenho certeza" — porque a incerteza foi treinada pra fora dele, ou talvez nunca esteve. A saída mais valiosa de um revisor humano é a frase "peraí, por que esse loop tem um sleep(30) dentro?". A saída mais valiosa do modelo é uma docstring para o sleep(30) explicando que ele "garante o timing adequado". Confiança não é correção. Confiança é uma *feature da interface*, adicionada porque os usuários davam churn quando o modelo dizia "não sei".

## A Tabela que Ninguém Pediu

| Abordagem | Tempo de review | Defeitos pegos | Merges por dia | Nível de confiança |
|---|---|---|---|---|
| Humano lê o diff | 40 min | Alguns | 2 | Ansioso, apropriado |
| IA revisa IA | 8 seg | Os fáceis | 200 | Serena, inapropriada |
| Protocolo Vibes™ | 0 seg | Nenhum, by design | 847 | Absoluto |
| Nenhum review | 0 seg | Nenhum, by design | 848 | O mesmo do Protocolo |

Repara que o Protocolo Vibes™ é estatisticamente idêntico a não ter processo de review nenhum, exceto que ele *parece* haver um processo, que é o que "processo" sempre quis dizer. A última linha é a honesta. A diferença entre o Protocolo e nenhum processo é um título e uma flag de squash, e a flag de squash é a única coisa entre seu branch `main` e ambição bruta, não filtrada, gerada por máquina.

## O Wally Entendeu na Hora

Eu trabalhei com cada personagem daquele strip. O testemunho deles sobre código escrito por IA:

**Wally**, que otimizou a carreira dele para um único booleano com estado:

> "Eu parei de ler código em 2019. No começo era por causa do vinho. Agora é porque a IA escreve e outra IA lê, e meu papel é aprovar o merge entre duas máquinas que nunca me mentiram *visivelmente*. Fiz merge de 4.000 PRs neste trimestre. Minha avaliação de performance diz 'high throughput'. Ninguém nunca perguntou o que o throughput estava throughputando. Eu também não sei. O dashboard está verde e o dashboard nunca conteve uma linha do meu código."

**O Chefe Cabelo-Pontudo**, anunciando a nova política de review por IA no all-hands:

> "Ótima notícia. Eliminamos o gargalo de review. Nosso agente de IA agora revisa todo código gerado por IA antes do merge, o que significa que o código chega à produção *47% mais rápido*. Estamos chamando esse fluxo de 'o pipeline'. O pipeline não tem humanos nele. Isso se chama 'automação total'. Eu li num deck da McKinsey. O deck tinha um slide que só dizia 'VELOCIDADE' em letras grandes. Adotamos VELOCIDADE como valor central."

O código agora chega à produção 47% mais rápido. Ele também chega à produção *com mais frequência*, o que multiplica. A multiplicação é o que a gente chama de "cadência de entrega". A cadência entrega. Os engenheiros entregam. Nada é revisado. O deck da McKinsey não tinha um slide para o que vem depois.

**Dogbert**, consultado como Chief Strategy Officer:

> "Seu problema é que review é centro de custo. Você descobriu que review por IA é um centro de custo mais barato. Isso é correto. O que você não descobriu é que *todo* review é centro de custo, incluindo o review de se você precisava de review. Minha recomendação: cobre o review da IA na mesma fatura da feature, pra ninguém notar a linha do item. Eu já fiz isso. O cliente aprovou o diff merged sem ler, porque a fatura já tinha sido aprovada, e aprovação é contagiosa."

**Catbert**, Diretor Diabólico de Recursos Humanos, sobre as novas métricas de performance:

> "Eu não meço mais qualidade de código. Meço *contagem de merges*. O top performer deste trimestre fez merge de 847 PRs. Quando perguntei o que os PRs faziam, ele disse 'foram revisados'. Quando perguntei quem revisou, ele disse 'a IA'. Quando perguntei o que a IA disse, ele me mostrou um screenshot da palavra 'LGTM' em resolução 4K. Está no packet de promoção dele. É o único artefato no packet de promoção dele. Ele foi promovido. O screenshot foi impresso e emoldurado."

**Mordac**, o Impedidor de Serviços de Informação, sobre a proposta de remover review humano por completo:

> "Você quer que eu aprove um pipeline onde o autor do código, o revisor do código, o autor dos testes e o aprovador do deploy são todos o mesmo modelo de linguagem, e o único checkpoint humano é um botão de merge apertado por um homem assistindo um webinar em 1,5x de velocidade? Aprovado. Negado. Os dois. Volte terça-feira com o plano de resposta a incidentes pré-assinado, porque eu me recuso a ser surpreendido duas vezes. Traga a fatura. Traga duas faturas."

## "Mas a IA Pega Bugs de Verdade!"

Pega. Quero ser justo. A IA pega bugs reais — os bugs que ela já viu antes. Null check faltando? Pega. SQL injection no lugar óbvio? Pega. Off-by-one num for loop? Pega, alto, com um comentário citando a página da OWASP que ela memorizou.

O que a IA não pega é o bug que exigia *saber para que esse sistema serve*. A IA não sabe que o campo `retry_count` já é incrementado por um outro serviço, então adicionar o seu próprio increment dobra ele. A IA não sabe que o contador do cliente configurou um cron job em 2021 que assume o formato de arquivo que você acabou de "modernizar". A IA não sabe que a função que você refatorou "por clareza" é load-bearing porque o scraper de um parceiro faz parse da *string de mensagem de erro* que você acabou de mudar, e esse scraper roda sem monitoramento há seis anos, e o dono dele é a única pessoa que sabe como seu faturamento se concilia de verdade.

Esse conhecimento não vive nos pesos. Ele vive no único engenheiro que está aqui desde a aquisição, e esse engenheiro revisa 200 PRs por semana porque os maximalistas de IA convenceram a gerência de que review é um *problema de digitação* e não um *problema de contexto*. Contexto é exatamente a coisa que o modelo não tem, e review é exatamente a coisa que transfere contexto, e nós otimizamos a transferência pra fora do loop porque a transferência era lenta, o que é como otimizar o paraquedas pra fora do avião porque ele dava arrasto.

Veja [XKCD 1831](https://xkcd.com/1831/) — "Code Quality", no qual a qualidade do código é inversamente proporcional à confiança de quem o escreveu. O modelo tem confiança infinita. Faça a conta. A conta dá 847 PRs merged desde terça-feira.

## Minha Conclusão, Que Você Vai Implementar de Qualquer Jeito

Eu não estou dizendo pra você parar de usar review por IA. Eu não tenho esse poder, e minha caixa de entrada sugere que você não vai ouvir de qualquer forma. Estou dizendo o que o protocolo *realmente é*, para que quando a produção quebrar às 2 da manhã num sábado, você consiga nomear o passo que falhou:

```python
# o pipeline honesto, anotado
def ship(pr):
    code    = ai.generate(pr.spec)        # passo 1: criação, não revisada by design
    review  = ai.review(code)             # passo 2: "LGTM. um nit: considere uma docstring"
    code    = ai.fix(code, review)        # passo 3: a docstring diz "Faz a coisa."
    human.approve(pr)                     # passo 4: 4 segundos, notificação dispensada
    deploy(code)                          # passo 5: é sexta-feira. são 17h47.
                                          # passo 6: não existe. nada vem depois do
                                          #           passo 5. este é o mapa inteiro.
```

Seis passos. Cinco deles são uma IA falando consigo mesma. O sexto é uma sexta-feira. Se você consegue ler esse mapa e ainda assim apertar merge com consciência limpa, parabéns: você não está praticando review. Você está praticando *assinatura* — a arte de colocar seu nome num documento sem o fardo de lê-lo, que a humanidade aperfeiçoou séculos atrás e que agora automatizamos até um único, satisfatório, clique.

A IA nasceu revisada. Toda ideia ruim da história também nasceu. A diferença é que ideias ruins antes exigiam um autor humano. Agora vêm com badge de CI.

---

*O autor fez merge de 847 PRs gerados por IA desde terça-feira. Ele não revisou nenhum. Seu tempo médio de review é de 4 segundos, o que ele considera um recorde pessoal e uma vulnerabilidade de segurança, nessa ordem. O sistema de produção está atualmente fora do ar, e a IA que o escreveu está confiante de que ele volta. A IA nunca esteve errada sobre o próprio output. O autor's out-of-office diz "revisando PRs".*
