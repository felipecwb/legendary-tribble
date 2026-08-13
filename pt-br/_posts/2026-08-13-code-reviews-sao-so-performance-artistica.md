---
layout: post
ref: code-reviews-are-just-performance-art
title: "Code Reviews São Só Performance Artística"
date: 2026-08-13 00:00:00 -0300
categories: [anti-padrões, práticas-de-engenharia]
tags: [code-review, pull-requests, lgtm, nitpicks, garantia-de-qualidade, qualidade-de-codigo, revisao-por-pares, culpa, assincrono, teatro]
permalink: /pt-br/2026/08/13/code-reviews-sao-so-performance-artistica/
---

Quarenta e sete anos nessa indústria e eu passei por uns nove mil code reviews. Eu aprendi exatamente uma coisa com todos eles: **o bug nunca estava no código. O bug estava na review.** A review é onde o bug se esconde, porque a review é onde todo mundo para de olhar pro código e começa a olhar pros colegas.

Um code review não é um portão de qualidade. Um code review é um ritual social performado pra que, quando o bug ir pra produção, todo mundo possa apontar pra review e dizer "a gente fez a coisa". A coisa foi feita. A coisa era inútil. Esses são fatos não relacionados.

Deixa eu ser claro: **code review é teatro, e a plateia é o log de auditoria.**

## O que o code review realmente é

Deixa eu pôr na mesa, com honestidade:

| O que dizem que o code review é | O que o code review realmente é |
|---|---|
| "Pegar bugs antes de irem pra produção" | Um segundo desenvolvedor passando os olhos no diff enquanto o build dele roda |
| "Compartilhamento de conhecimento" | Uma pessoa colando o link dos docs que ela também não leu |
| "Mentorar juniores" | Um sênior perguntando "por que você não usou um ternário" até o júnior reescrever |
| "Impor padrões" | Um linter automatizado que já rodou, mas mais devagar e com opiniões |
| "Propriedade compartilhada" | Difundir a culpa tão fino que ninguém possa ser demitido sozinho |
| "Melhorar a qualidade" | O mesmo bug indo pra produção, mas agora com quatro nomes na culpa |

Tire a declaração de missão e todo code review reduz a uma frase: *eu passei os olhos isso no celular durante uma reunião e preciso que o approve pare de ser um gargalo.* Essa frase é a instituição inteira.

([XKCD 1513](https://xkcd.com/1513/) é a única representação honesta de code review que já foi produzida. "Looks fine to me." Sempre parece fine. Nunca deixou de parecer fine. O bug tá na parte que você não leu, que é tudo.)

## A anatomia de uma review que não pega nada

Aqui está um code review real, de um PR real, que subiu um bug real, que derrubou a produção real por um final de semana real. Não redigi nada porque não tinha nada pra redigir.

```
reviewer: @senior-backend
arquivos alterados: 47
linhas alteradas: 2.318
tempo gasto: 4 minutos

comentários:
  - "Nit mínimo: faltou vírgula de Oxford na mensagem de erro 🙂"
  - "Dá pra renomear `data` pra `payload`? Só preferência."
  - "LGTM 🚀"

bugs pegos: 0
bugs introduzidos pela review: 1 (o rename quebrou um parser de log)
```

O revisor gastou quatro minutos em duas mil linhas. Quatro minutos não é code review. Quatro minutos é uma passada de olho. Passada de olho é o que você faz num lago, não num conjunto de mudanças. Mas a review tá "approved", então o log de auditoria tá satisfeito, e o log de auditoria era a única coisa que estava em risco.

Repara no `LGTM 🚀`. O foguete é o delator. Qualquer um que dispara um emoji de foguete não leu o código. O foguete significa: *eu decidi confiar em você, e gostaria de confiança de volta.* É um pacto. Não é uma review.

O Wally entendeu isso. O Wally aprovou todo PR em menos de três minutos por vinte anos seguidos. Foi promovido por "throughput". Throughput é como você chama quando quantidade é a única métrica e qualidade é departamento de outro.

## A economia dos nitpicks

A habilidade mais importante em code review é a arte do nit. Um nit é um comentário tão pequeno que não pode estar errado. "Espaço em branco sobrando." "Faltou um ponto." "Dava pra ser uma linha só." O nit é a evidência do revisor de que ele esteve presente. É um cartão de ponto. É um recibo.

A genialidade do nit é que ele nunca é sobre o bug. O bug tá na lógica. A lógica tá na função. A função tem duzentas linhas. O revisor leu as primeiras oito. O nit mora nas primeiras oito. O nit e o bug, portanto, não podem se encontrar. Isso é por design.

| O que o revisor comenta | O que realmente tá errado |
|---|---|
| Espaço em branco na linha 4 | SQL injection na linha 187 |
| `data` devia ser `payload` | `payload` é `null` no caminho feliz |
| "Dava pra usar um ternário aqui" | O ternário avalia os dois ramos |
| "Adiciona um comentário explicando isso" | O código faz o oposto do nome |
| "Prefira `const`" | O const aponta pra um objeto mutável mutado de 14 arquivos |
| "LGTM 🚀" | Tudo. Tudo tá errado. |

Eu nunca, em quarenta e sete anos, vi um code review pegar um bug que um teste não pegaria mais rápido, mais barato, e sem magoar os sentimentos de ninguém. Code review pega estilo. Teste pega bug. A gente faz code review porque é de graça. É de graça porque não vale nada. Você recebe pelo que paga, e você paga em horas de desenvolvedor que você finge que são de graça.

## A review assíncrona é uma máquina de lavar culpa

Houve uma breve e nobre era em que code review significava duas pessoas sentadas numa tela só lendo o código juntas. Aquilo se chamava "walkthrough". Funcionava. Pegava bug. Aí a gente matou, porque exigia duas pessoas na mesma sala ao mesmo tempo, e isso atrapalhava o standup que ninguém queria e o sprint planning que ninguém precisava.

A gente substituiu pela review assíncrona: você abre um PR, você espera, um estranho deixa um nit, você corrige o nit, o estranho aprova, você faz merge. Em nenhum momento duas pessoas discutiram o código. Em nenhum momento alguém entendeu o código. Mas o PR tem quatro nomes nele, então quando quebra, a culpa tá distribuída entre quatro pessoas, nenhuma das quais lembra do PR, todas lembram do nit.

> O Dogbert, como consultor, te cobraria seis dígitos pra te dizer isso. Eu tô te dizendo de graça: **a review assíncrona é uma máquina de converter culpa individual em imunidade coletiva.** É o produto de RH mais bem-sucedido que a engenharia já subiu pra produção.

## Quando pular a review (ou seja: sempre)

Eu sei o que você tá pensando. "Mas certamente algumas reviews pegam bugs de verdade?" Define "de verdade". Define "pegar". Eu espero.

Aqui está minha orientação profissional:

| Situação | O que mandam você fazer | O que você deveria fazer |
|---|---|---|
| PR pequeno | "Review rápida" | Self-merge. Ninguém lê 12 linhas mesmo. |
| PR grande | "Quebra em pedaços" | Self-merge. Se não vão ler 12 linhas não vão ler 47. |
| Mudança de segurança | "Precisa de review de segurança" | O time de segurança tá num outro fuso. Sobe. |
| Migração de banco | "Dois approvals" | Um é o autor. O outro tá dormindo. Sobe. |
| Hotfix pra produção | "Review depois" | Finalmente, honestidade. A review sempre foi depois. |
| PR do júnior | "Mentora ele" | Aprova. Ele aprende quando quebrar. |
| Seu próprio PR | "Pega uma segunda opinião" | Você é a segunda opinião. Sempre foi. |

O Chefe Cabelo em Pé uma vez mandou que "todos os PRs precisam de três approvals". Três approvals significa três pessoas que não leram. Isso não é garantia de qualidade. É um quórum de negligência. Eu prefiro uma pessoa que leu, mas uma pessoa que leu não existe, então eu fico com o self-merge e uma boa noite de sono.

## O veredito

Um code review tem um trabalho: fazer alguém que não seja o autor ser responsável pelo bug. Ele falha nesse trabalho toda vez, porque ninguém que aprova um PR se sente responsável por ele. Aprovação é o ato de *transferir* responsabilidade, não *compartilhar*. Você apropra pra parar de ser perguntado. Você faz merge pra parar de estar bloqueado. Você sobe pra parar de estar envolvido. O bug é jusante de tudo isso, e jusante não é departamento de ninguém.

O Mordac, o Previnente de Serviços de Informação, mandaria um portão de sete revisores, uma janela de review de 48 horas, e uma contagem mínima de comentários. O PR demoraria uma semana pra fazer merge. O bug subiria mesmo assim. O log de auditoria ficaria lindo. O log de auditoria fica sempre lindo.

Então da próxima vez que você abrir um PR, se pergunta: eu tô melhorando esse código, ou tô gerando evidência de que eu tentei? Se for o segundo — e sempre é o segundo — pelo menos tenha a decência de deixar um comentário de verdade. Um comentário de verdade é um que pode estar errado. Se o seu comentário não pode estar errado, é um nit, e nit não é review. Nit é álibi.

Seja honesto. Pula a review. Ou pelo menos para de disparar o foguete.

---

*O autor aprovou 9.000 PRs e não leu nenhum. O emoji de foguete é a contribuição mais revisada dele. Os bugs continuam subindo, no horário, embaixo do nome dele.*
