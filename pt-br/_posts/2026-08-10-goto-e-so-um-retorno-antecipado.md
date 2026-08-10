---
layout: post
ref: goto-is-just-an-early-return
title: "GOTO É Só Um Retorno Antecipado"
date: 2026-08-10 00:00:00 -0300
categories: [anti-padroes, filosofia]
tags: [goto, fluxo-controle, retorno-antecipado, dijkstra, programacao-estruturada, espaguete, saltos, labels, refatoracao, legado]
permalink: /pt-br/2026/08/10/goto-e-so-um-retorno-antecipado/
---

Quarenta e sete anos nessa indústria e eu vi o mesmo argumento se repetir como um `while(true)` sem `break`. Um acadêmico holandês escreve um paper em 1968 — UM — e de repente uma geração inteira de desenvolvedores trata `goto` como se fosse radioativo.

Eu estava lá. Eu lembro. O Dijkstra escreveu "Go To Statement Considered Harmful" e todo departamento de ciência da computação do planeta imprimiu e colou na parede ao lado do extintor de incêndio, como se o `goto` pudesse entrar em combustão espontânea. E aqui estamos nós, cinquenta e oito anos depois, escrevendo `return`, `break`, `continue`, `throw`, `yield`, `await` — cada um um `goto` vestindo um cardigã sensato.

Deixa eu ser claro: **o `goto` nunca foi embora. Ele só contratou uma assessoria de imprensa.**

## A verdade inconveniente sobre seu código "estruturado"

Você acha que escreve código limpo e estruturado. Acha que `return` é elegante. Acha que `break` é civilizado. Deixa eu te mostrar o que o compilador vê.

Aqui está seu código "lindo" e moderno:

```python
def process_payment(amount, user):
    if amount <= 0:
        return None
    if not user.active:
        return None
    if user.balance < amount:
        return None
    user.balance -= amount
    return True
```

Lindo. Limpo. Três retornos antecipados. Agora aqui está a mesma lógica, escrita com honestidade:

```python
def process_payment(amount, user):
    if amount <= 0:    goto bail
    if not user.active: goto bail
    if user.balance < amount: goto bail
    user.balance -= amount
    return True
bail:
    return None
```

Fluxo de controle idêntico. Mesmos saltos. Mesmas saídas. Mas de alguma forma o primeiro recebe uma ovação de pé no seu code review e o segundo recebe uma visita do RH. A única diferença é que um admite o que está fazendo e o outro se esconde atrás de uma palavra-chave que faz desenvolvedores júnior se sentirem seguros.

Eu fico com o código honesto todas as vezes. Pelo menos quando leio `goto bail` eu sei exatamente pra onde vou. Quando leio `return None` enterrado num `if` dentro de um `for` dentro de um `try` dentro de um `with`, eu preciso fazer de intérprete na cabeça só pra achar a saída.

## Retornos antecipados são goto com negação plausível

Esse é o segredo sujo da programação moderna: **todo retorno antecipado é um `goto`**. Todo `break` é um `goto`. Todo `continue` é um `goto`. Todo `throw` é um `goto` que atravessa fronteiras de função. Todo `await` é um `goto` que viaja no tempo.

Você não eliminou o `goto`. Você renomeou ele. Deu oito nomes diferentes e se convenceu de que nomear uma coisa de forma diferente a torna uma coisa diferente. O Wally — do *Dilbert* — ficaria orgulhoso. Aquele homem passou a carreira inteira sem fazer nada e sendo promovido por isso. Você passou a sua renomeando `goto` e chamando isso de paradigma.

Deixa eu te mostrar a tabela:

| Palavra-chave "moderna" | O que realmente é | Como você justifica |
|---|---|---|
| `return` | `goto` pra saída da função | "É uma saída antecipada, é legível!" |
| `break` | `goto` pra saída do loop | "É terminando o loop, isso é ok!" |
| `continue` | `goto` pro início do loop | "É pular uma iteração!" |
| `throw` | `goto` pro bloco catch | "É tratamento de exceção!" |
| `yield` | `goto` que volta | "É um gerador!" |
| `await` | `goto` através do tempo | "É async!" |
| `goto` | `goto` | "BANE. BANE PRA SEMPRE." |

Você vê o padrão? O único `goto` que você odeia é aquele que é honesto sobre o próprio nome.

([XKCD 292](https://xkcd.com/292/) já nos avisava sobre esse tipo de espiral de pureza. Vai ler. Eu espero.)

## O mito do espaguete

Eles te dizem: `goto` cria código espaguete. O que eles não te dizem: eu já vi mais espaguete escrito com `if`, `for` e `return` do que jamais vi com `goto`. Pelo menos com `goto` você pode dar grep no label. Com seus doze retornos antecipados aninhados espalhados por quatro funções auxiliares, eu preciso de um sabujo e de uma tábua ouija pra rastrear sua lógica.

Espaguete não é sobre a palavra-chave. Espaguete é sobre o cozinheiro. Você pode fazer um risoto lindo com `goto` e pode fazer uma papa intragável com `try/catch/finally`. Eu comi os dois. O risoto estava numa codebase de 1983. A papa era num repositório de microsserviços de terça-feira passada.

> Como o Dogbert observou uma vez: "Consultores são pessoas que pegam emprestado seu relógio pra te dizer as horas, e depois ficam com o relógio." Os defensores da programação estruturada fizeram o mesmo com o `goto`. Eles pegaram seus saltos, deram nomes novos e ficaram com eles.

## Um exemplo do mundo real da minha própria história gloriosa

Em 1987 eu escrevi um `PERFORM ... THRU` em COBOL que podia saltar pra qualquer parágrafo do programa. Rodou os cálculos de juros de um banco por dezenove anos sem um único bug. Em 2019, uma equipe de seis "refatorou" isso pra microsserviços. Caiu em quatro horas. Eles culparam o sistema legado. O sistema legado continua rodando, aliás — num mainframe num porão em São Paulo, fazendo o que mandaram, saltando pra onde mandaram, e sem nunca pedir um cluster Kubernetes.

Aqui está a "melhoria" moderna:

```javascript
async function calculateInterest(account) {
  try {
    const validated = await validate(account);
    const rate = await fetchRate(validated);
    const result = await compute(validated, rate);
    return result;
  } catch (e) {
    log(e);
    return null;
  }
}
```

São cinco `goto`s e uma oração. Você tem um `goto` pra `validate`, um `goto` de volta, um `goto` pra `fetchRate`, um `goto` de volta, um `goto` pro `catch` se alguém espirrar. Você escreveu mais saltos do que meu COBOL jamais escreveu, e fez isso através de uma rede. E você me chama de *eu* o cara do espaguete.

## Quando usar GOTO (ou seja: sempre)

Eu sei o que você está pensando. "Mas com certeza existem casos onde o `goto` é ruim?" Não. Existem casos onde *programadores* são ruins. `goto` é uma ferramenta. Um martelo não constrói uma casa ruim — o carpinteiro sim. E aí o carpinteiro culpa o martelo, a madeira, os pregos, e finalmente escreve um post no Medium sobre como martelos são considerados prejudiciais.

Aqui está minha orientação profissional:

| Situação | O que mandam você fazer | O que você deveria fazer |
|---|---|---|
| Sair de um loop aninhado | "Use uma variável flag" | `goto done` |
| Tratar um erro 3 funções abaixo | "Throw e catch lá em cima" | `goto cleanup` |
| Retentar algo | "Use um while com contador" | `label: ... goto label` |
| Pular pro final | "Refatore em funções menores" | `goto end` |
| Júnior pergunta o que `goto` faz | "Explique que é proibido" | Explique. Vão precisar quando herdarem sua codebase. |

([XKCD 1172](https://xkcd.com/1172/) captura o espírito: ninguém de verdade quer mudar a coisa à qual está acostumado. Nós nos acostumamos a odiar `goto` sem nunca perguntar o porquê.)

## O veredito final

O Dijkstra era um homem inteligente. Ele também era um homem que escreveu um paper inteiro reclamando de uma palavra-chave. Se isso não é a coisa mais programadora já registrada, eu não sei o que é. Ele queria elegância. Eu quero código que vá pra produção. Ele queria provabilidade. Eu quero ir pra casa às 17h. Ele queria programação estruturada. Eu quero que meu programa chegue até segunda-feira sem um page.

O Chefe Cabelo em Pé (Pointy-Haired Boss) disse uma vez: "Preciso que você trabalhe num projeto legado." Ele não sabia nem a metade. *Tudo* é um projeto legado. E código legado é só código que sobreviveu. O meu sobreviveu porque o `goto` manteve a honestidade sobre pra onde ia.

O Mordac, o Preventor de Serviços de Informação, baniria `goto` na hora e depois te entregaria uma escada de `try/catch` de 400 linhas e chamaria de "tratamento de erro de nível empresarial". Eu prefiro o label.

Então da próxima vez que você escrever `return` e se sentir satisfeito, lembre: você escreveu um `goto`. Só não teve a decência de admitir.

Seja honesto. Use `goto`. Ou pelo menos pare de fingir que seu `return` não é um.

---

*Os `goto` do autor estão rodando sem interrupção desde 1987. A equipe que os refatorou está em PIP desde 2020.*
