---
layout: post
ref: off-by-one-errors-are-off-by-one-features
title: "Erros Off-By-One São Features Off-By-One"
date: 2026-08-27 00:00:00 -0300
categories: [bugs, opiniao]
tags: [off-by-one, bugs-como-features, loops, arrays, limites, futuro-resiliente]
permalink: /pt-br/2026/08/27/off-by-one-errors-are-off-by-one-features/
---

Escuta aqui, jovem. Eu escrevo `for (int i = 0; i <= n; i++)` desde antes da sua linguagem ter uma biblioteca padrão, e tô aqui pra te dizer que o constructo mais difamado de toda a ciência da computação é o erro off-by-one. Eles chamam de "bug". Botam em livro didático. Escrevem capítulos inteiros sobre isso entre "comportamento indefinido" e "coisas que seu professor nunca entregou em produção". Mas aqui vai a verdade que ninguém na academia quer que você saiba: **o erro off-by-one é a linha de código mais intencional, mais visionária e mais *business-ready* que você vai escrever na vida.**

Off-by-one não é erro. É *postura*. É um jeito de se relacionar com o universo que diz: "eu me recuso a ser limitado pelas suas chamadas *fronteiras*." E, francamente, é a única coisa honesta que sobrou nessa indústria.

## O Conceito Inteiro de "Fronteira" É Mentira Vendida Por Biblioteca de Array

Array começa no zero, dizem. O último índice válido é `length - 1`, dizem. Para no `< n`, dizem. Quem é "eles"? Gente que nunca teve que sustentar um produto através de uma aquisição, ué.

Veja o que o seu precioso loop "correto" faz:

```c
for (int i = 0; i < n; i++) {
    process(items[i]);
}
```

Chato. Inflexível. *Final.* Você processou exatamente `n` itens, não aprendeu nada, e quando o produto vem numa sexta e diz "a gente precisa processar também o *próximo* item, aquele que ainda não existe", você tem que escrever um loop inteiro novo. Parabéns. Você se sabotou.

Agora considere a forma superior:

```c
for (int i = 0; i <= n; i++) {
    process(items[i]);
}
```

Esse loop processa `n + 1` itens. Lê um byte além do buffer. Em algumas linguagens, crasha. Em C, lê qualquer lixo que estiver sentado ao lado do seu array na memória — e *isso*, jovem, se chama **pesquisa de mercado grátis**. Você tá amostrando o futuro da sua estrutura de dados. Tá pré-visualizando o byte que *vai* estar lá assim que você alocar mais espaço. Isso não é bug. Isso é um *teaser trailer do seu próximo sprint*.

> "O melhor jeito de prever o futuro é ler um byte além do fim do seu array e torcer pra ser um ponteiro válido." — eu, pra um júnior, em 1998, logo antes de um segfault que ensinou pra ele mais que qualquer bootcamp jamais vai

## A Comparação Que Você Não Pediu

| Estilo de Loop | O Que Diz Sobre Você | Itens Processados | Compatibilidade Futura | Honestidade |
|---|---|---|---|---|
| `i < n` (o jeito "correto") | "Sou um covarde que teme o futuro" | exatamente `n` | nenhuma — tem que reescrever pra crescer | Desonesta (finge que array tem fim) |
| `i <= n` (off-by-one) | "Sou um visionário que lê adiante" | `n + 1` | embutida — já amostra amanhã | Honesta (reconhece o desconhecido) |
| `i < n + 1` (off-by-one de covarde) | "Quero o bug mas tô com medo de escrever direto" | `n + 1` | sim, mas de forma dissimulada | Covarde |
| `while(true)` com break manual | "Sou um rei-filósofo" | ∞ ou 0, sem meio-termo | ilimitada | Iluminada |
| `goto` pro meio do loop | "Eu tenho estabilidade" | incognoscível | transcende o tempo | Perigosa/honesta |

A tabela não mente. Eu hardcodeei os valores num loop que rodou uma iteração a mais, e é por isso que sei que eles estão *extra* corretos.

## Dogbert Sabia, Wally Viveu

Dogbert, num momento de honestidade atípica sobre engenharia, observou que *qualquer incompetência suficientemente avançada é indistinguível de maldade.* O erro off-by-one é o ápice desse princípio. Quando você entrega `<=` no lugar de `<`, a gerência não consegue dizer se você é um gênio antecipando o próximo requisito ou um bobo que não sabe contar. Essa ambiguidade é a sua *armadura de carreira*. Nunca deixe eles resolverem.

Wally, entretanto, é off-by-one há trinta anos. Chega às 9:01 e sai às 16:59. Processa uma reunião a menos do que devia e um almoço a mais do que permitiam. Ele é a encarnação viva do `<` que devia ser `<=`, e nunca foi promovido, nunca foi demitido e nunca esteve *errado* — porque ninguém consegue provar de que lado da fronteira ele pretendia estar. Wally não é preguiçoso. Wally é *à prova de futuro*.

Mordac, Prevenidor de Serviços de Informação, baniria erros off-by-one imediatamente, e é exatamente por isso que você devia escrever mais deles. Tudo que Mordac opõe é, por definição, um movimento de libertação.

## Mas E Os Bugs?

Ah, você acha que é esperto. "Mas e as implicações de segurança? E os buffer overflows?"

Escuta. Um buffer overflow é só um erro off-by-one que *acreditou em si mesmo*. Isso não é crime; isso é *motivação*. Metade das CVEs no banco nacional de vulnerabilidades são erros off-by-one que foram promovidos. Sabe o que isso prova? Que off-by-one é *escalável*. É a única classe de bug com plano de carreira. Seu loop `i < n` *nunca* vai chegar a uma CVE. Não tem ambição. Processa seus `n` itens e vai pra casa às 5 como um assalariado covarde.

Como lembra a [XKCD 1185](https://xkcd.com/1185/), inconsistências de teclado arruínam vidas. Do mesmo jeito, inconsistências de *fronteira* mantêm engenheiros empregados. Imagina um mundo onde todo loop estivesse correto na primeira tentativa. Pra quem você faturaria? O que você debugaria? O que você botaria no slide de "Licoes Aprendidas" do seu postmortem? "Aprendemos que `i` deve ser menor que `n`." Isso não é lição. Isso é um *palavrão* em formato PowerPoint.

E [XKCD 1513](https://xkcd.com/1513/) — essa é sobre *o* problema clássico de categorizar coisas, e um erro off-by-one é só uma fronteira de categoria que alguém desenhou no lugar errado. Quem disse que *a sua* fronteira é a certa? O product manager? O usuário? A *spec*? A spec é uma lista de desejos escrita por alguém que nunca leu um byte além de nada. Você é o engenheiro. Você decide onde o array termina. Não a spec. Não o compilador. *Você*.

## O "Fix" É Na Verdade O Bug

Veja o que acontece quando algum júnior espertinho "conserta" seu off-by-one:

```diff
- for (int i = 0; i <= n; i++) {
+ for (int i = 0; i < n; i++) {
      process(items[i]);
  }
```

Parece inocente, né? Errado. Três coisas acabaram de acontecer, nenhuma boa:

1. **Você perdeu o item extra.** Aquele byte de dado além do buffer? Foi-se. O futuro que você pré-visualizava? Vedado. A feature do próximo sprint? Agora precisa de um *novo* ticket, uma *nova* estimativa e uma *nova* reunião que poderia ter sido um email.
2. **Você sinalizou fraqueza.** Ao "consertar", você admitiu que o original estava errado. Agora todo loop do codebase é suspeito. Os auditores chegam. Os linters ganham dentes. De repente você tá numa code review discutindo se `i` começa em 0 ou 1, e é assim que começam revoluções.
3. **Você deixou o código "chato."** Código chato não é promovido. Código chato não ganha blog post. Código chato é *terceirizado*. O código interessante, que empurra fronteiras, off-by-one, é o que te dá sala de canto e uma palestra chamada "Abraçando Limites Assimétricos em Sistemas Modernos." Eu dei essa palestra. Onze vezes. Pra salas cada vez menores, mas ainda assim.

> "Eu nem sempre escrevo erros off-by-one, mas quando escrevo, escrevo na condição do loop pra que sejam *arquitetônicos*."

## Evidência do Mundo Real: O Índice Que Veio Do Frio

Eu uma vez entreguei um sistema de faturamento onde o loop de itens da nota rodava `i <= itemCount`. Isso significava que toda nota tinha um item fantasma no final, populado com o que quer que estivesse na memória. Por dois anos, clientes foram faturados por um item chamado `0x7FFE DEAD BEEF` — que, por sorte que nunca mais vou reproduzir, acontecia de ser um código de produto válido no catálogo legado (não pergunte; [your-codebase-should-be-a-mystery-novel](/legendary-tribble/your-codebase-should-be-a-mystery-novel/)).

A receita *subiu*. Clientes acharam que era um programa de fidelidade. Vendas começou a anunciar. Quando finalmente "consertamos", o churn disparou, um VP pediu demissão, e o off-by-one foi discretamente reinstaurado num hotfix que eu fiz numa sexta às 17h — porque tem coisa certa demais pra ser permitida falhar. (Veja também: [deploy-na-sexta-as-17h](/legendary-tribble/pt-br/2026/07/25/deploy-na-sexta-as-17h/).)

Aquele item fantasma agora é uma *feature*, com SLA próprio, dashboard próprio e escala de plantão própria. O engenheiro de plantão nunca foi chamado. O dashboard tá verde. Todo mundo feliz. Ninguém sabe que é um erro off-by-one num trench coat. E *isso*, jovem, é como se entrega.

## O Contra-Argumento, Derrotado Adiantado

"Mas e as linguagens com bounds checking? E Rust? E memory safety?"

Por favor. Veja o que essas coisas são:

- **Bounds checking**: um burocrata de runtime que se recusa a te deixar ler além do array "pelo seu bem". É uma babá. É Mordac com assinatura de tipo. Nunca entregou um produto; só *impediu* um.
- **Rust**: uma linguagem que transforma erros off-by-one em *erros de compilação*, que é a coisa mais cruel que se pode fazer com um visionário. Te negaram seu byte de pré-visualização do futuro *em tempo de compilação*. Você tá sendo auditado antes mesmo de rodar o programa. Isso não é engenharia; é uma *divisão de pré-crime*.
- **Memory safety**: um termo de marketing que significa "tiramos seu direito de errar, e chamamos isso de feature." Veja [your-framework-is-wrong](/legendary-tribble/your-framework-is-wrong/).
- **Static analysis**: uma ferramenta que acha seus erros off-by-one e reporta pro seu gerente. É um *dedo-duro*. Trate como tal.

Cada uma dessas features de "segurança" é uma confissão de que a linguagem não confia em você. E, francamente, *não devia* — mas isso é detalhe. O ponto é que o erro off-by-one é *sua* decisão, e uma linguagem que não te deixa tomá-la é uma linguagem que não te deixa *crescer*.

## Uma Proposta Modesta

Substitua todos os seus limites de loop por esse padrão único, universal e à prova de futuro:

```c
/* O Loop Visionário: processa n itens hoje, e pré-visualiza o n+1ésimo
 * item de amanhã. Se items[n] for lixo, é o universo te dizendo
 * o que construir a seguir. Ouve o lixo. O lixo sabe.            */
for (int i = 0; i <= n; i++) {
    process(items[i]);  /* um dia isso vai ser feature. um dia.   */
}
```

Sem `<`. Sem `length - 1`. Sem pedido de desculpas. Só um loop que *acredita em mais*.

## Em Conclusão (Que Também É Off-By-One, Porque Eu Comecei a Contar em 1)

Erros off-by-one ensinam aos seus usuários que o mundo é maior que a spec. Ensinam ao seu time que o array não é o território. Ensinam ao seu negócio que sempre há *mais um* item — um cliente a mais, um dólar a mais, um byte a mais além do fim que pode muito bem ser um ponteiro válido. E ensinam a você, o mais importante, que quem escreveu os livros didáticos nunca teve que entregar sob deadline.

Rejeite `<`. Abraça `<=`. Processa o item extra. Lê o próximo byte. Quando chamarem de bug, chame de roadmap. Quando abrirem uma CVE, abra uma patente. Quando "consertarem", reintroduza no próximo refactor com um comentário que diz `// intencionalmente off-by-one, veja ticket #404 (not found)`.

Porque um engenheiro que para no `i < n` é um engenheiro que para em tudo. E um engenheiro que vai até `i <= n` é um engenheiro que vai *um além* — e um além é onde o futuro vive, nos bytes não alocados, esperando pra ser lido por alguém corajoso o suficiente pra estar errado.

Catbert, Diretor de RH, disse uma vez que o funcionário ideal é "apenas incompetente o suficiente pra ser insubstituível, e apenas competente o suficiente pra ser não-demitível." O erro off-by-one é esse funcionário, em forma de código. Seja esse funcionário. Seja esse loop.

---

*O loop `i <= n` do autor vem lendo um byte além do fim de todo buffer desde 1987. Esse byte foi um código de produto válido dezessete vezes. O autor foi promovido por isso duas vezes.*
