---
layout: post
ref: refactoring-is-vandalism-of-working-code
title: "Refatoração É Vandalismo De Código Que Funciona"
date: 2026-07-19 00:00:00 -0300
categories: [codigo, cultura]
tags: [refatoracao, legado, divida-tecnica, mau-conselho, conselho-senior, codigo-que-funciona, medo]
permalink: /pt-br/:year/:month/:day/refactoring-is-vandalism-of-working-code/
---

Em 47 anos de engenharia eu escrevi 11.412 funções. Eu refatorei 11.409 delas. As 3 que eu não refatorei são as 3 que ainda tão rodando em produção. O resto tá num aterro de boas intenções, cada uma "melhorada" até parar de funcionar, depois "melhorada" de novo até parar de compilar, depois commitada numa sexta porque a refatoração era "baixo risco" e o engenheiro queria o fim de semana. O engenheiro conseguiu o fim de semana. A produção não. O engenheiro não retornou as ligações. O código não retornou os requests. Os usuários não retornaram. Eu sou o engenheiro. A refatoração era minha. O bug também era meu, mas era um bug *melhor* — mais limpo, mais sustentável, com separação de responsabilidades adequada — e então valeu a pena.

## O Que Código Que Funciona É De Verdade

Código que funciona não é o código que você escreveu. Código que funciona é o código que sobreviveu. Tem uma diferença, e a diferença é tudo que você não sabe sobre o código. Cada linha de código que funciona carrega, nas lacunas entre os caracteres, mil linhas de código que não funcionavam — as tentativas falhas, os off-by-one, as race conditions que você nunca viu porque a corrida foi às 2 da manhã quando ninguém tava olhando e o perdedor foi silenciosamente garbage collected. O código que resta é o sobrevivente. O sobrevivente é de carga de um jeito que sua suíte de testes não consegue descrever, porque a suíte de testes foi escrita pra descrever o código, não o ambiente que o código, por acidente, aprendeu a tolerar.

Quando você refatora código que funciona, você não tá melhorando ele. Você tá **perturbando um ecossistema**. O código chegou num equilíbrio com os bugs dele. Os bugs chegaram num equilíbrio com o código. Os dois assinaram um tratado, na forma de 47 edge cases não documentados, e o tratado é o único motivo pelo qual a aplicação não desabou desde 2019. Você não leu o tratado. O tratado não tá nos comentários. Os comentários foram removidos numa refatoração anterior, em 2021, por um engenheiro que não tá mais na empresa, que acreditava que o código devia ser "auto-documentável", e que tá, nesse momento, refatorando um codebase diferente numa empresa diferente, tendo a mesma ideia, com as mesmas consequências, pra sempre.

## O Que A Refatoração Afirma vs O Que Ela Faz

| A Refatoração Afirma | O Que Acontece De Verdade |
|----------------------|---------------------------|
| "Tô deixando o código mais legível" | O código agora é legível pra você, hoje. Vai ser ilegível pra todo mundo, incluindo você, na segunda. |
| "Tô reduzindo duplicação (DRY)" | As duas cópias tinham bugs diferentes. A cópia única tem os dois. Os bugs tão unificados agora, o que é pior, porque você não consegue culpar uma cópia pela outra. |
| "Tô extraindo uma função reutilizável" | A função é reutilizada zero vezes. A original agora tem uma camada de indireção que não dá pra inlinear de volta sem uma segunda refatoração. |
| "Tô renomeando por clareza" | O rename quebra um lookup por reflexão, uma tag de serialização e um mapeamento de coluna de banco, nessa ordem, às 3 da manhã. |
| "Tô modernizando a sintaxe" | A sintaxe moderna não roda no runtime que produção ainda usa, porque atualizar o runtime é uma refatoração *separada*. |
| "Tô pagando a dívida técnica" | Você tá transferindo a dívida do código pro escala de plantão. O código tá mais limpo. O plantão tá falido. |
| "É uma refatoração de baixo risco" | O risco era baixo porque você não testou a parte que quebrou. Você não testou porque não sabia que existia. |
| "Os testes passam" | Os testes passam porque os testes testam a refatoração, não o comportamento. A refatoração preservou os testes. O comportamento foi embora em 2021. |

Repara que "tô deixando o código quieto porque funciona" não aparece na tabela. Isso é porque não aparece em nenhum pull request, nunca. O pull request é a confissão do engenheiro de que ele acabou com as features pra shippar e precisa agora shippar uma mudança numa coisa que não tava quebrada, pra parecer produtivo, numa terça, pra um gerente que não consegue distinguir feature de refatoração e vai aprovar os dois porque os dois tão verdes no GitHub.

## O Ciclo De Vida Da Refatoração

Toda refatoração segue o mesmo ciclo, e o ciclo não tem nada a ver com as necessidades do código.

1. **Nascimento.** Um engenheiro abre um arquivo que ele não escreveu. Ele não entende o arquivo. Ele fica chateado com isso. Ele chama de "legado." A palavra "legado" é um xingamento que o engenheiro usa pra descrever código mais velho que o tempo dele na empresa, e portanto mais esperto que o entendimento dele. A refatoração é concebida nesse momento de ofensa.
2. **Confiança.** O engenheiro roda os testes. Os testes passam. O engenheiro acredita que os testes descrevem o código. Não descrevem. Os testes descrevem o engenheiro que escreveu os testes, que também tava confuso, e escreveu testes pras partes que ele entendeu, que eram as partes que não precisavam de teste. A refatoração começa.
3. **Progresso.** O arquivo tá mais curto. Os nomes tão melhores. O engenheiro se sente limpo. O engenheiro committa. A mensagem do commit é `refactor: clean up`. A mensagem não diz o que foi limpo, porque o engenheiro não sabe o que foi limpo, porque limpar é o ato de remover coisas cujo propósito você ainda não descobriu.
4. **Regressão.** Um usuário reporta um bug. O bug tá numa feature que o engenheiro não sabia que existia, porque a feature tava implementada inteirinha dentro dos 47 edge cases que a refatoração deletou. Os edge cases eram de carga. A feature foi embora. O usuário foi embora. O ticket é reclassificado como feature request, porque a feature não existe mais, e portanto não pode ter bug.
5. **Culpa.** O git blame agora aponta pro engenheiro. O engenheiro tá orgulhoso disso, porque o engenheiro é novo e acredita que ter o nome dele numa linha significa propriedade, não responsabilidade. O nome do autor anterior foi embora. O autor anterior sabia dos edge cases. O autor anterior agora é um estranho, e o estranho não pode receber email, porque ele foi demitido na refatoração de 2023, que também era "baixo risco."
6. **Revert.** O revert é proposto. O revert é rejeitado, porque reverter ia desfazer os ganhos de legibilidade, e os ganhos de legibilidade são a única métrica que o time tem, porque o time não tem métrica pra "features que ainda funcionam." O bug é corrigido com um edge case novo. O edge case novo é adicionado em cima da refatoração. O código agora é mais longo que antes, e menos legível, e o engenheiro já foi pra um arquivo diferente, pra começar de novo.
7. **Renasimento.** O edge case novo, escrito às pressas às 4 da manhã, vira o novo legado. Em dois anos, um engenheiro novo vai abrir esse arquivo, não entender, e chamar de "legado." O ciclo se completa. O ciclo sempre ia se completar.

Eu refatorei a mesma função 11 vezes desde 2009. Ela agora é 11 vezes mais longa, 11 vezes mais lenta, e 11 vezes mais correta, onde "correta" significa "faz o que a 11ª versão faz, que não é o que a 1ª versão fazia, que é o que os usuários querem, que é por que os usuários foram embora, que é por que a gente pivou, que é por que eu tô escrevendo isso de uma empresa diferente, refatorando a mesma função, que um engenheiro sênior aqui escreveu em 2009."

## A Matriz De Refatoração

Essa é a matriz que eu uso pra avaliar qualquer refatoração que eu encontro. Eu nunca vi uma refatoração que escapasse dela.

| Estado Da Refatoração | O Que Significa | Ação Recomendada |
|------------------------|-----------------|------------------|
| Título do PR: `refactor: clean up` | O autor removeu coisas que ele não entendeu | Reverte. Não explica. |
| Título do PR: `refactor: no behavior change` | O comportamento tá mudando. O autor não rodou em produção. | Reverte. Adiciona a mudança de comportamento como feature, separada, com teste. |
| PR toca um arquivo modificado pela última vez em 2019 | O arquivo é um fóssil. Fósseis não são melhorados. São exibidos. | Não aprova. Exibe. |
| PR reduz contagem de linhas | Linhas foram removidas. Algumas importavam. Você vai descobrir quais em duas semanas. | Não aprova. Pergunta quais linhas eram de carga. O autor não sabe. |
| PR aumenta contagem de linhas | A refatoração adicionou abstração. Abstração é dívida que você paga em debug. | Aprova só se o autor adicionar o bug que ela vai causar agora, adiantado, pra economizar tempo. |
| PR renomeou um campo | O rename quebrou um serializador, uma coluna de banco, e o nervo de um colega. | Não aprova. Renames nunca são locais. |
| PR "extrai um helper" | O helper é usado uma vez. A indireção é usada pra sempre. | Não aprova. Inlinna de volta. |
| PR é "baixo risco" | O risco não foi medido. O risco agora é do reviewer. | Não aprova. O reviewer agora é o autor. |
| PR passa todos os testes | Os testes testam o PR. O comportamento não é testado. | Não aprova. Adiciona um teste pro comportamento primeiro. Depois reverte. |

A ação recomendada é sempre "não aprova" ou "reverte" porque a refatoração, na hora que chega em review, já removeu um edge case de carga que o reviewer não sabe que existe. O reviewer não pode saber, porque o edge case era não documentado, e era não documentado porque documentar ele ia exigir admitir que ele existia, e admitir que ele existia ia exigir que o autor original explicasse por que uma função chamada `processPayment` também, num branch, mandava email pra uma pessoa chamada Gary, e ninguém, em 2019, tava preparado pra ter essa conversa. Gary foi embora. O email ainda vai pro Gary. A refatoração removeu o email. O fantasma do Gary agora é o bug. Você não consegue reverter um fantasma.

## O Script De Auditoria De Refatoração

Depois de 47 anos auditando refatorações na mão, eu automatizei o processo. Esse script lê um diff e produz um relatório no único formato de output honesto: uma recomendação de reverter.

```python
def auditar_refatoracao(diff):
    """
    O único auditor de refatoração honesto.
    Uma refatoração é uma mudança num código que não tava quebrado,
    por um engenheiro que não entendia ele,
    pra fazer parecer um código que ele teria escrito,
    que seria pior.
    """
    report = {}

    for hunk in diff.hunks():
        # Refatoração que toca arquivo mais velho que o autor é vandalismo.
        if hunk.idade_arquivo_anos > hunk.tempo_autor_anos:
            report[hunk.path] = "VANDALISMO_AUTOR_NAO_TINHA_NASCIDO_QUANDO_ISSO_FUNCIONAVA"
            continue

        # Refatoração que deleta linhas é uma confissão de ignorância.
        if hunk.linhas_removidas > hunk.linhas_adicionadas:
            report[hunk.path] = "DELECAO_O_AUTOR_NAO_ENTENDEU_AS_LINHAS"
            continue

        # Refatoração que renomeia é uma refatoração que ainda não achou o serializador.
        if hunk.e_rename():
            report[hunk.path] = "RENAME_SERIALIZER_VAI_TE_ACHAR_AS_3DA_MANHA"
            continue

        # Refatoração sem mudança de teste tocou comportamento que os testes não cobriam.
        if hunk.arquivos_teste_tocados == 0:
            report[hunk.path] = "COMPORTAMENTO_NAO_TESTADO_AGORA_NAO_TESTADO_MAS_DIFERENTE"
            continue

        # Refatoração intitulada "clean up" é refatoração sem tese.
        if "clean up" in hunk.mensagem_commit.lower():
            report[hunk.path] = "SEM_TESE_O_AUTOR_TEVE_UM_SENTIMENTO_E_AGIU"
            continue

        # Todo o resto tá fine, que é a única categoria que não tá.
        report[hunk.path] = "APROVADO_PORQUE_NAO_INSPECIONADO"

    return report

# Output de auditar um trimestre de refatorações em 2026:
# VANDALISMO_AUTOR_NAO_TINHA_NASCIDO_QUANDO_ISSO_FUNCIONAVA: 14
# DELECAO_O_AUTOR_NAO_ENTENDEU_AS_LINHAS: 47
# RENAME_SERIALIZER_VAI_TE_ACHAR_AS_3DA_MANHA: 8
# COMPORTAMENTO_NAO_TESTADO_AGORA_NAO_TESTADO_MAS_DIFERENTE: 61
# SEM_TESE_O_AUTOR_TEVE_UM_SENTIMENTO_E_AGIU: 22
# APROVADO_PORQUE_NAO_INSPECIONADO: 1
# Total de refatorações: 153
# Refatorações que melhoraram alguma coisa: 0
# Refatorações que introduziram bug rastreado em produção por >30 dias: 153
```

O script nunca produziu uma refatoração que ele aprovaria. Isso é porque o ato de aprovar uma refatoração exige mais conhecimento do que o ato de deixar o código quieto, e o conhecimento tá dentro da cabeça de um engenheiro que saiu da empresa em 2019, que levou o conhecimento com ele, porque conhecimento não tá no código, conhecimento tá na pessoa, e a pessoa não tá no repo. Deixar código quieto é o primeiro instinto do engenheiro sênior. O segundo instinto é adicionar um comentário que diz `// NAO REFATORE — ver incidente 2019-11-04`, que o próximo engenheiro vai ler, não entender, e refatorar mesmo assim, porque o comentário não é um aviso, o comentário é uma provocação.

## O Rename "Inofensivo"

Aqui a refatoração que me ensinou. Era um rename. Era um campo, num struct, num serviço, numa terça.

```go
// Antes da refatoração — o campo que segurava o sistema
type Payment struct {
    amnt  int64   // <-- o campo. não renomeia. nem olha pra ele.
    // ... 2.000 linhas de código que fingem não depender desse nome ...
}

// A refatoração "inofensiva"
type Payment struct {
    Amount int64  // <-- mais claro! legível! auto-documentável!
    // ... 2.000 linhas de código que ainda fingem não depender do nome antigo ...
}
```

A refatoração foi 1 linha. O raio de explosão foi:
- O serializador JSON, que usava `amnt` porque o frontend, escrito em 2018 por um estagiário que agora é VP, hardcodou `amnt` em 47 lugares.
- O banco de dados, cuja coluna era `amnt` porque a migration de 2019 não confiava no ORM, e mapeava colunas por nome de field do struct.
- O log de auditoria, que logava `amnt` refletindo sobre o struct, porque o auditor não queria atualizar o logger toda vez que um campo era adicionado.
- O webhook de terceiros, que recebia `amnt` porque a spec de API do terceiro, escrita em 2017, era um screenshot do nosso struct de 2017.
- Gary. O email do Gary. O email que ia pro Gary quando `amnt` passava de 10.000, porque o email era enviado por uma função que fazia pattern match no nome do campo, porque a função foi escrita por um engenheiro que acreditava que nomes de campo eram pra sempre, e esse engenheiro tava certo, até terça.

A refatoração foi revertida às 4:11 da manhã de uma quarta. O revert foi 1 linha. A mensagem do commit do revert foi `fix: re-add amnt`. A palavra "fix" apareceu num commit que não continha nenhum código que não estivesse presente em 2019. O código era um círculo. O círculo era de carga. A refatoração tentou quebrar o círculo. O círculo quebrou a refatoração. O círculo é eterno. O engenheiro não.

## Refatorar É Uma Confissão De Que Seu Eu Do Passado Era Mais Esperto Que Você

Aqui o segredo da refatoração que o deck do tech lead não menciona: uma refatoração não é melhoria. Uma refatoração é **uma confissão de que o autor original do código entendia algo que você não, e sua resposta é deletar a evidência.** Toda refatoração é um ato de editar um documento que você não escreveu, por uma pessoa que não tá aqui pra defender ele, numa linguagem que você não fala direito, pra fazer parecer um documento que você *escreveria*, que seria pior. O autor original foi embora. O autor original não consegue te dizer por que o campo se chamava `amnt`. O autor original não consegue te dizer por que a função tem um branch pro Gary. O autor original não consegue te dizer nada, porque o autor original foi demitido numa refatoração do *organograma* em 2023, que também era "baixo risco," e que também removeu pessoas de carga, e que também não pôde ser revertida.

As refatorações se acumulam porque engenheiros são promovidos por shippar mudanças, não por donar estabilidade. Um status quo mantido, onde o código continua funcionando e ninguém toca nele, não conta nada na revisão trimestral, porque ninguém consegue ver o outage que você preveniu não fazendo nada. Uma refatoração shippada, que não quebrou nada que os testes conseguissem ver, conta como lançamento. A estrutura de incentivo garante crescimento de refatoração. O crescimento de refatoração garante crescimento de regressão. O crescimento de regressão garante o próximo deck de estabilidade trimestral, que propõe, como solução: uma ferramenta nova, escrita numa linguagem nova, que detecta automaticamente refatorações que são "baixo risco," e que ela mesma é uma refatoração do pipeline de CI, e que vai quebrar o pipeline de CI, e que vai ser revertida às 4 da manhã, e que vai ser re-adicionada na sexta, porque o engenheiro quer o fim de semana.

## O Oposto De Uma Refatoração

Tem uma alternativa pra refatoração, e é a que ninguém quer ouvir. A alternativa é: **não toca no código.** O código funciona. O código funciona desde 2019. O código vai funcionar até você tocar nele. O ato de tocar é o ato que quebra. Isso não é teoria. Isso é a história inteira de software, condensada numa frase, que todo engenheiro aprende, e que todo engenheiro ignora, porque a alternativa é sentar com um codebase que você não entende, e admitir que você não entende, e deixar quieto, e ser pago pra deixar quieto, e explicar pro seu gerente que sua contribuição esse trimestre foi a ausência de uma contribuição, e que a ausência era a contribuição, e que o código ainda funciona, e que isso é, de fato, o trabalho inteiro.

Como o Wally explicou uma vez, quando perguntaram por que ele não tinha refatorado uma função que "claramente" precisava: *"Uma refatoração é uma confissão de que você leu o código e o código ganhou. O código sempre ganha. O código tá ganhando desde 1998. Eu não entro em concurso que eu vou perder. Eu não refatoro código que eu não entendo, que é todo ele, que é por que eu não refatoro nenhum. A função funciona. A função funciona desde antes de eu ser contratado. A função vai funcionar depois que eu me aposentar. Minha contribuição é não ser a pessoa que encerra a sequência. Meu gerente não entende isso. Meu gerente também não vai ser a pessoa que encerra a sequência, porque meu gerente vai ser promovido em maio, na força de um deck que toma crédito da sequência, que eu mantive, não fazendo nada, que é a coisa mais difícil da engenharia."* O Wally entendia de refatoração. O Wally entendia que a refatoração não é um ato técnico. A refatoração é um ato de *vaidade*. A refatoração é onde o engenheiro põe a ambição dele, pra o código poder continuar vivo.

O Dogbert, consultado sobre se devia aprovar uma refatoração intitulada "clean up," supostamente respondeu: *"Aprovar? Por quê? O código tá funcionando. O autor tá entediado. Esses não são o mesmo problema. O tédio do autor é problema do autor. O funcionamento do código é problema de todo mundo, e a solução do problema de todo mundo é não deixar o entediado perto da coisa que funciona. Diz pro autor pra refatorar o currículo dele no lugar. É o único artefato que ele é dono, é o único que se beneficia, e é o único que não vai pagear alguém às 3 da manhã quando ele acabar."*

## Resolução

Uma refatoração não é melhoria. Uma refatoração é **vandalismo que você aprovou** — um jeito de alterar o código que funciona de um estranho, afirmar como seu, e depois culpar o estranho quando quebra, mesmo que a versão do estranho funcionasse, e a sua não, e o estranho não tá aqui pra apontar isso. É o equivalente do engenheiro da frase do gerente "a gente modernizou o sistema legado": uma frase que parece progresso e significa "eu deletei um branch que eu não entendia e 4.000 usuários tão mandando email pra um cara chamado Gary." Cada refatoração no seu histórico é uma pequena aposta que você fez sobre uma função que você não escreveu, que não importava, que os nomes tavam errados, que a duplicação era desperdício, que os edge cases eram acidentais. O eu do futuro não acredita em você. O eu do futuro tem que corrigir a regressão. O eu do futuro sou eu. Eu reverti 11.409 refatorações. Eu não terminei. Eu nunca vou terminar. As refatorações tão ganhando.

[XKCD 1513](https://xkcd.com/1513/) é a referência canônica do engenheiro que abriu um codebase, não entendeu, e decidiu que o problema era o codebase. Em 47 anos eu nunca aprovei uma refatoração que não precisasse, eventualmente, de uma segunda refatoração pra desfazer a primeira, que ela mesma precisou de uma terceira. As refatorações não simplificam. As refatorações só compostam. O codebase é um monoid, e o elemento de identidade é o commit que você não fez, que é o único commit que não quebrou nada.

[XKCD 1739](https://xkcd.com/1739/) é a visão do engenheiro da refatoração que parecia inofensiva, a que "só" renomeou um campo, a que derrubou um serializador, uma coluna de banco, um webhook, e um cara chamado Gary, e, eventualmente, um jeito de vida. A gente renomeou numa terça. A gente não desreparou o dano desde. O nome original foi embora. O autor original foi embora. O comportamento original foi embora. O nome novo tá em todo lugar. O comportamento novo não tá em lugar nenhum. O engenheiro não tá. O bug tá. O bug é tudo, exceto shippado, que foi embora em 2019 com o engenheiro.

O Pointy-Haired Boss do Dilbert, quando mostraram um pull request intitulado "refactor: clean up," supostamente perguntou: *"Limpar o quê?"* A resposta certa era "nada," porque toda refatoração, no final, é uma limpeza de nada, e toda refatoração, no final, é uma quebra de alguma coisa, e toda refatoração, no final, é um estranho editando o trabalho de um estranho e chamando de melhoria. Você é o estranho. Você refatorou o código. Ninguém tem certeza do que faz agora. Ninguém vai te reverter. Você é, enfim, um engenheiro sênior.

---

*O autor refatorou 11.409 funções desde 2009. Quarenta e sete delas ainda funcionam. Uma delas é o autor. Não foi pushado desde 2019. Ninguém tem certeza do que faz. Ninguém vai tocar nele.*
