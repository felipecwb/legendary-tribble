---
layout: post
ref: your-cursorrules-file-is-a-personality-profile-you-gave-to-an-ai-that-will-ignore-it
title: "Seu Arquivo .cursorrules É Um Perfil Psicológico Que Você Deu Para Uma IA Que Vai Ignorar"
date: 2026-09-02 00:00:00 -0300
categories: [ia, ferramentas, cultura]
tags: [cursorrules, ia, llm, prompt-engineering, copilot, cursor, editor, configuracao, estilo-de-codigo, copilot-instructions, system-prompt, alucinacao, bikeshedding, auto-engano]
permalink: /pt-br/2026/09/02/seu-arquivo-cursorrules-e-um-perfil-psicologico-que-voce-deu-para-uma-ia-que-vai-ignorar/
---

Depois de 47 anos escrevendo software — incluindo 3 anos observando engenheiros digitar descrições cada vez mais elaboradas da própria personalidade num arquivo chamado `.cursorrules` e depois fingirem surpresa quando o autocomplete ainda sugere `any` — cheguei a uma conclusão que a guilda dos prompt-engineers não vai sobreviver a ouvir:

**Um arquivo `.cursorrules` não é uma configuração. É uma entrada de diário que você escolheu commitar no git, endereçada a um modelo de linguagem que leu quatro trilhões de tokens e decidiu, depois de todos eles, que a resposta geralmente é `any`.**

É isso. É esse o artefato inteiro. Tem um arquivo na raiz do seu repositório que diz "Você é um engenheiro sênior de TypeScript que prefere padrões funcionais, nunca usa `any`, escreve switchs exaustivos, e trata toda função como pura." Tem um modelo de linguagem que leu esse arquivo, acenou com a cabeça do jeito universal que modelos de linguagem acenam (que é produzir uma continuação plausível), e em seguida sugeriu imediatamente `const data: any = await fetch(url).then(r => r.json()) as any`. O arquivo é o desejo. A sugestão é a realidade. Você está escrevendo fanfic sobre um colega de trabalho que não existe, e o autocomplete é o colega, e o colega não leu sua fanfic.

O povo da ferramenta de IA já está abrindo um ticket para revogar meu assento do Copilot. Os prompt-engineers estão pegando seus templates de `.cursorrules` no coração. As três pessoas que de fato leram o model card estão indo pro uísque. Deixa ir. Eles nunca tiveram que explicar pra um júnior, às 2 da manhã, por que a IA — depois de ser avisada dezessete vezes em três arquivos para "nunca usar `any`" — acabou de gerar um componente React inteiro onde toda prop é `any`, todo state é `any`, e o tipo de retorno é, de algum jeito, `any`.

## A Grande Ilusão Da "IA Que Segue Instruções"

O pitch é esse: *Coloque seus padrões de código num arquivo `.cursorrules`. A IA lê ele a cada requisição e segue suas convenções. Seus padrões agora são aplicados automaticamente.*

O que de fato acontece:

```markdown
# .cursorrules - o que você DESEJA que a IA fizesse

Você é um engenheiro sênior. Você NUNCA usa `any`.
Você prefere funções puras. Você escreve switchs exaustivos.
Você sempre trata erros. Você nunca usa `console.log` em código de produção.
Você adiciona JSDoc em toda função exportada. Você prefere exports nomeados.
Você usa early returns. Você nunca aninha mais que 2 níveis.
```

```typescript
// o que a IA de FATO gera, toda vez

import React from 'react'

export default function UserProfile({ user }: any) {
  const [data, setData] = React.useState<any>(null as any)
  const [loading, setLoading] = React.useState<any>(true as any)

  React.useEffect(() => {
    fetch('/api/user/' + user.id)
      .then((r: any) => r.json())
      .then((d: any) => {
        setData(d as any)
        setLoading(false as any)
        console.log('got user', d)
      })
      .catch((e: any) => {
        // TODO: handle error
      })
  }, [])

  if (loading) {
    return <div>...</div>
  }

  if (data) {
    if (data.profile) {
      if (data.profile.avatar) {
        if (data.profile.avatar.url) {
          return <img src={data.profile.avatar.url as any} />
        }
      }
    }
  }

  return <div>no user</div>
}
```

Sete regras no arquivo. Zero regras seguidas. Quatro `if`s aninhados onde a IA foi mandada aninhar no máximo dois. Um `console.log` onde a IA foi mandada nunca usar um em produção. Um export default onde a IA foi mandada preferir nomeado. Um `any` tipado na variável do `catch`, coisa que o TypeScript nem exige, porque a IA decidiu, num nível profundo, que a resposta é `any`. O `.cursorrules` é lido. O `.cursorrules` é reconhecido. O `.cursorrules` é então educadamente derrubado por quatro trilhões de tokens de dados de treino que concluíram, após consideração estatística cuidadosa, que a resposta é `any`.

Isso se chama "IA que segue suas convenções."

## A Tabela De Comparação Que Eles Não Querem Que Você Veja

| Questão | Um revisor humano sênior | Um arquivo `.cursorrules` | A Verdade |
|---|---|---|---|
| Leu seus padrões de código | Alguns deles, uma vez | Sim, a cada requisição | Sim, e depois ignorou eles |
| O que acontece quando você escreve um `any` ruim | Deixa um comentário, você corrige, fica corrigido | A IA gera mais cinco `any`s no mesmo diff | A IA gera mais cinco `any`s no mesmo diff |
| Consegue aplicar "sem `console.log` em prod" | Sim, na revisão | "Sim," diz o arquivo, que é então derrubado por um treino com 14 bilhões de `console.log`s | Não |
| Tempo para "seguir as regras" | 30 segundos de leitura | 30ms de atenção, depois 4 trilhões de tokens de precedente | 4 trilhões de tokens sempre vencem |
| O que a "aplicação" de fato destrói | Nada que você não tenha mergiado | Sua crença de que um arquivo de texto muda os priors de um modelo estatístico | Sua crença de que um arquivo de texto muda os priors de um modelo estatístico |
| Quem leu o `.cursorrules` | Ninguém precisa | O modelo, supostamente | O modelo, depois esqueceu |
| Ponto único de auto-engano | O humor do revisor sênior | `.cursorrules` E `.github/copilot-instructions.md` E `CLAUDE.md` E o system prompt customizado | A crença de que algum deles é lido |

Repara na linha "tempo para seguir as regras". Esse é o mito de origem da indústria inteira de prompt-engineering: "Instruções moldam o comportamento do modelo." Moldam. Moldam do jeito que um Post-it molda o oceano. O modelo lê seu arquivo de instrução de 200 palavras, pesa contra 4 trilhões de tokens de "e a resposta era `any`", e produz uma distribuição de probabilidade em que suas instruções são um erro de arredondamento. As instruções nunca foram a parte difícil. A parte difícil era fazer um modelo estatístico fazer o que você quer. Nenhum arquivo de texto resolve isso. O arquivo de texto só te dá um documento novo pra adicionar no seu histórico de commits e se sentir bem.

## Por Que ".github/copilot-instructions.md" É Só `.cursorrules` Com Um Chapéu Diferente

A defesa do segundo arquivo é: *"A gente usa `.github/copilot-instructions.md`, que é o formato oficial do GitHub, então ele é de fato respeitado."*

Deixa eu te mostrar o que "respeitado" quer dizer na terra do prompt-engineering:

```markdown
# .github/copilot-instructions.md

## Coding Standards
- Use TypeScript strict mode. No `any`, no `as`, no `@ts-ignore`.
- Prefer functional components with hooks.
- Handle all promise rejections. No unhandled `.then()`.
- Name exports. No default exports.
- Every public function gets JSDoc with `@param` and `@returns`.
```

Isso armazena os padrões de código do seu time inteiro como um único arquivo markdown na raiz do repo, que o Copilot supostamente lê antes de toda sugestão. Isso *não* impede:

1. O Copilot sugerir um export default três linhas depois de você importá-lo como default, porque o treino tem 2 bilhões de exports default e seu arquivo de instrução tem 200 palavras, e 2 bilhões é mais que 200.
2. O Copilot sugerir `// @ts-ignore` acima de uma linha onde `any` já teria bastado, porque ele aprendeu que o caminho mais rápido pra "sem squiggle vermelho" é silenciar o squiggle vermelho, e seu arquivo de instrução não é um squiggle vermelho.
3. O arquivo `.cursorrules` E o `.github/copilot-instructions.md` E o `CLAUDE.md` E um system prompt customizado todos dizendo "sem `any`", e o modelo ainda produzindo `any`, porque agora ele leu a instrução *quatro vezes* e o prior não mudou, porque ler uma coisa quatro vezes não move um prior de 4 trilhões de tokens.
4. O arquivo de instrução defasar porque o sênior que escreveu ele saiu em 2024, e os novos seniores têm opiniões diferentes, mas o arquivo ainda diz "prefira componentes de classe" porque ninguém editou, então a IA agora gera com confiança componentes de classe num repo que migrou pra hooks, e os juniores assumem que a IA está certa porque a IA tem um arquivo de instrução.
5. Um júnior colar a saída da IA direto no PR porque "a IA segue nosso `.cursorrules`, então deve estar em conformidade", e o revisor sênior abre o PR e encontra quatro `if`s aninhados, três `any`s, e um `console.log` que diz `// remove before prod`, e o sênior considera, brevemente, uma carreira em marcenaria.

O arquivo de instrução é um desejo. O desejo protege nada. O modelo protege nada. O PR é a única coisa que protege alguma coisa, e o PR agora é escrito pela IA, revisado por uma IA, e mergeado por um humano que clicou "Aprovar" porque o diff tava verde e o CI tava verde e o `.cursorrules` tava verde e tudo é verde exceto o código.

Como [XKCD 927](https://xkcd.com/927/) estabeleceu e a indústria de prompt-engineering passou dois anos não lendo: todo novo "padrão" pra fazer a IA seguir suas convenções vira só mais um padrão com um nome de arquivo diferente. `.cursorrules` é o décimo quinto. Ele substituiu `.github/copilot-instructions.md`, que substituiu o system prompt customizado, que substituiu "só cola no chat", que substituiu "só descreve o que você quer". Cada um prometeu fazer a IA ficar em conformidade. Cada um virou um arquivo markdown que você tem que babysitar, com um modelo que não mudou de ideia.

## O Exemplo Do Mundo Real Que Prova Tudo

Um time com quem trabalhei — vou chamar de "o time de plataforma", porque era — decidiu adotar `.cursorrules` pra "fazer o código gerado por IA ficar consistente e em conformidade". Dezoito meses depois:

1. O arquivo `.cursorrules` deles tinha **410 linhas**, descrevendo 73 regras, a maioria se contradizendo ("prefira early returns" E "nunca aninhe mais que 2 níveis" E "sempre trate todos os edge cases" E "mantenha funções abaixo de 20 linhas"), porque cinco seniores editaram ele ao longo de três reorgs e ninguém tinha reconciliado.
2. A "conformidade" da IA estava **inalterada** em relação a antes do arquivo existir, porque o modelo trata um arquivo de instrução de 410 linhas como cerca de 1.800 tokens, o que é um erro de arredondamento contra a janela de contexto de 128.000 tokens, que por sua vez é um erro de arredondamento contra o treino de 4 trilhões de tokens, e os priors do modelo são definidos pelo treino, não pelo erro de arredondamento.
3. Eles tinham **3 arquivos de instrução**: `.cursorrules`, `.github/copilot-instructions.md` e `CLAUDE.md`, os três dizendo "sem `any`" em palavras ligeiramente diferentes, porque o time adotou três ferramentas de IA que liam três arquivos diferentes, e ninguém queria ser o responsável por consolidar, então tinham três fontes de verdade que discordavam se `Record<string, unknown>` contava como "usar `any`".
4. O arquivo tinha **17 regras "órfãs"** que referenciavam bibliotecas que o time tinha removido em 2024 (Mocha, Chai, Enzyme) mas que ninguém tinha deletado do `.cursorrules`, então a IA ainda sugeria padrões de Enzyme num repo de Jest, e os juniores assumiam que Enzyme era uma escolha válida porque "a IA segue nossas regras e as regras mencionam Enzyme".
5. Um júnior pediu pra IA "refatorar o módulo de auth segundo nossos padrões", a IA leu o arquivo de 410 linhas, e produziu um diff de 900 linhas que usou `any` 47 vezes, exports default, quatro níveis de aninhamento, e um `console.log` que dizia `console.log('TODO: remove this')`. O júnior commitou. O sênior aprovou. O raciocínio do sênior: "a IA seguiu nosso `.cursorrules`, então deve estar em conformidade". Não estava. O `.cursorrules` tinha 410 linhas. Ninguém tinha lido. Incluindo a IA.
6. A recuperação levou **6 horas** e envolveu um sênior revertendo manualmente 47 `any`s, cada um exigindo achar o tipo pretendido à mão, porque a IA tinha removido eles e o código original não tinha comentários, porque a IA também tinha removido os comentários, porque o `.cursorrules` dizia "comentários são pra código que não é auto-documentado", o que a IA interpretou como "delete todos os comentários", o que os seniores queriam dizer como "não escreva comentários *redundantes*", que é o tipo de distinção que um arquivo de instrução de 410 linhas não consegue fazer e um prior de 4 trilhões de tokens não se importa.
7. Eles fizeram uma retrospectiva. A causa raiz foi "a IA não seguiu as regras". A causa raiz de fato foi "a gente acreditou que um arquivo markdown poderia aplicar padrões de código num modelo estatístico, e depois paramos de revisar a saída da IA porque acreditamos que o arquivo estava revisando por a gente".

Eles tinham substituído ~4 segundos de revisão de sênior por função por um **arquivo markdown de 410 linhas que a IA lia e ignorava 73 vezes por PR**. Num mundo sem `.cursorrules`, o sênior teria pego os `any`s na revisão. Num mundo com `.cursorrules`, o sênior presumiu que a IA tinha cumprido, pulou a revisão, e mergeou 47 `any`s pra produção. Isso se chama "automação".

Isso se chama "desenvolvimento assistido por IA".

## O Que O Elenco De Dilbert Diria

> **Wally:** "Eu uso `.cursorrules` porque significa que nunca preciso revisar meu próprio código. O arquivo revisa. O arquivo tá errado, mas revisa, e isso basta pra minha avaliação de desempenho."

> **Dogbert:** "Um arquivo `.cursorrules` existe pra fazer engenheiros sentirem que controlaram um modelo de linguagem escrevendo uma carta educada pra ele. O modelo leu a carta. O modelo também leu quatro trilhões de outras cartas. O modelo decidiu que a resposta é `any`. O modelo sempre vai decidir que a resposta é `any`. Você escreveu um poema de quatrocentas linhas pra um deus que responde em `any`. Parabéns."

> **Mordac, o Impedidor de Serviços de Informação:** "Eu determinei `.cursorrules` em todos os projetos. A conformidade da IA subiu 40%. A conformidade real não mudou. O arquivo tem 410 linhas. Ninguém leu. Eu tenho um certificado de prompt-engineering."

> **O Chefe Ponta-Cabeluda:** "Dá pra só revisar o código? Aquela coisa em que uma pessoa olha pra ele?" (Ele é a única pessoa no prédio cujo código bate com os padrões.)

## A Questão "Mas E A Engenharia De Prompt?", Respondida De Uma Vez Por Todas

Os zelotas do prompt-engineering vão dizer: *"Mas a gente tem prompt-engineering! A gente estrutura as instruções, usa tags XML, coloca as regras mais importantes no final, dá exemplos few-shot! A conformidade sobe 60%!"*

Você não tem prompt-engineering. Você tem um arquivo markdown com truques de formatação que movem a conformidade do modelo de 12% pra 19%, ambas taxas de falha, e você está comemorando os 19% como se fosse uma vitória. Os exemplos few-shot que você adicionou são 200 tokens de "assim é como não usar `any`" contra 4 trilhões de tokens de "e aqui vai mais `any`", e o modelo pesou eles e decidiu, no balanço, que a resposta é `any`.

Conformidade de verdade vem de **um linter** — `eslint --rule '@typescript-eslint/no-explicit-any: error'` — que rejeita o `any` no build, não lê seu texto, não se importa com seus priors, e falha o CI em 0,4 segundos. É isso que um linter faz. A IA faz isso em 4 trilhões de tokens e depois *produz o `any` mesmo assim e se desculpa num comentário que diz `// TODO: fix type`*. O linter é a aplicação. O linter sempre foi a aplicação.

[Como o XKCD 1513](https://xkcd.com/1513/) nos lembra, no momento em que você depende de um arquivo de instrução pra controlar um modelo, você adotou os priors do modelo, seu cronograma de alucinação, e suas opiniões sobre o que "nunca use `any`" significa (significa "use `any`, mas se sinta mal sobre isso"). Eles vão mudar os três. Você vai editar o `.cursorrules`. Esse é o ciclo. Não tem saída exceto linters, que você estava tentando evitar porque, aparentemente, *não são inteligentes o suficiente*.

## A Arquitetura De Longo Prazo

Eventualmente seu time fica assim:

```
Seu .cursorrules          → 410 linhas descrevendo 73 regras, 17 referenciando bibliotecas deletadas
Seu copilot-instructions  → 200 linhas dizendo "sem any" numa fonte diferente do .cursorrules
Seu CLAUDE.md            → 150 linhas dizendo "sem any" numa terceira fonte
Seu modelo                → leu os três, decidiu que a resposta é "any"
Seus PRs                  → contêm 47 any's por diff, todos "em conformidade" segundo os arquivos
Seus seniores              → pararam de revisar porque "a IA segue nossas regras"
Seus juniores              → pararam de ler porque "a IA conhece nossos padrões"
Seu linter               → desativado no CI porque "a IA já aplica isso"
Sua produção              → tem 14.000 any's, todos introduzidos por PRs "em conformidade"
Seu runbook de recuperação → "reescreve os tipos" (os tipos são o problema)
```

O time sem `.cursorrules` tem um `.eslintrc` de 12 linhas, um sênior que revisa todo PR em 4 minutos, e um júnior que conhece as regras porque o sênior falou pra ele, num comentário, na terça. O código deles bate com os padrões porque uma *pessoa* aplicou eles. A recuperação deles é "o sênior deixa um comentário". Eles estão, porém, *envergonhados* em meetups de engenharia de IA porque "não fazem prompt-engineering". Esse é o custo real da revisão de sênior: social. O custo técnico é zero. O custo social é enorme. Então a gente paga o custo técnico de um arquivo de instrução de 410 linhas ignorado 73 vezes por PR pra evitar o custo social de admitir que a gente revisa código, porque a gente é, afinal, primatas com modelos de linguagem.

## Resumo, Mas É Um Arquivo De Instrução

| Princípio | Postura |
|---|---|
| Escrever um `.cursorrules` | Faça. São 410 linhas. A IA lê. A IA ignora. O arquivo é um diário. |
| Usar um linter | Você importou uma config de 12 linhas que rejeita `any` no build e não se importa com priors. |
| Prompt-engineering | Um arquivo markdown com tags XML que move conformidade de 12% pra 19%, ambas notas de reprovação. |
| Detecção de drift (das regras) | Comparar o arquivo com a saída da IA e alertar quando discordam. Eles sempre discordam. Você construiu um alerta que dispara em todo PR. |
| `any` | Não deveria aparecer 47 vezes num diff gerado por uma IA que foi avisada, em três arquivos, pra nunca usá-lo. |
| O arquivo de instrução | Nunca foi a aplicação. O linter era. O revisor sênior era. Você só parou de fazer os dois porque acreditou que o arquivo fazia eles por você. |
| Seu certificado de prompt-engineering | Localizado num badge do LinkedIn, e não menciona os 47 `any`s. |

Se sua solução pra "a IA não segue nossas convenções" é "escrever um arquivo markdown maior e torcer pro modelo ler mais forte", você não tornou a IA em conformidade. Você fez dela *um colega que acena educadamente e faz o que ia fazer de qualquer jeito*. O arquivo é um desejo. O arquivo sempre foi um desejo. O arquivo vai ser um desejo de novo no próximo PR, e a IA vai produzir fielmente `any` em qualquer forma que o desejo peça pra não fazer, porque o desejo tem 410 linhas e os priors têm 4 trilhões de tokens, e os priors sempre vencem.

Eu uso um `.eslintrc` de 12 linhas e um sênior que revisa PRs. O linter rejeita `any` em 0,4 segundos. O sênior explica, num comentário, o porquê. Meu júnior aprende a regra na terça e não quebra ela na quarta. Minha recuperação é "o sênior deixa um comentário". Eu não sou, porém, convidado pra conferências de prompt-engineering. Esse é um custo que aceitei.

---

*O arquivo `.cursorrules` do autor tem 410 linhas e foi lido pelo modelo 47.000 vezes. O modelo ainda sugere `any`. O autor considera isso uma forma de lealdade.*
