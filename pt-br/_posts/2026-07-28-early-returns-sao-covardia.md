---
layout: post
ref: early-returns-are-cowardice
title: "Early Returns São Covardia: O Manifesto do Arrow Code"
date: 2026-07-28 00:00:00 -0300
categories: [estilo-de-codigo, anti-padroes, filosofia]
tags: [early-return, guard-clauses, arrow-code, aninhamento, fluxo-de-controle, legibilidade, clean-code, indentacao, desistencia]
permalink: /pt-br/2026/07/28/early-returns-sao-covardia/
---

Depois de 47 anos enviando código que lê como um livro "escolha-sua-aventura" onde toda aventura é um NullPointerException, desenvolvi opiniões fortes sobre como funções deveriam fluir. O movimento moderno de "clean code" infectou uma geração inteira de desenvolvedores com uma única ideia covarde: o **early return**.

Eles chamam de "guard clauses". Eles chamam de "padrão bouncer". Eles chamam o resultado de "código plano". Eu chamo pelo que é: **uma retirada**.

Deixa eu explicar por que um engenheiro sênior de verdade nunca sai de uma função antes que ela termine.

## A Flecha do Triunfo

O suposto "Arrow Anti-Pattern" — onde o código aninha tão fundo que sua indentação escapa pela borda direita da tela — não é um bug. É uma *hierarquia de comprometimento*. Cada nível de aninhamento é uma promessa ao leitor: *Eu ainda estou aqui. Eu não desisti. Seguimos juntos.*

```python
def processar_pedido(pedido):
    if pedido is not None:
        if pedido.eh_valido():
            if pedido.tem_pagamento():
                if pedido.pagamento.esta_confirmado():
                    if pedido.itens:
                        if pedido.endereco_envio:
                            if pedido.endereco_envio.esta_verificado():
                                if armazem.tem_estoque(pedido.itens):
                                    if transportadora.disponivel():
                                        if not pedido.esta_marcado():
                                            return enviar(pedido)
    return None
```

Olha para esse formato. É uma flecha. Flechas vão *para frente*. Early returns vão *para trás*. Qual direção você quer que seu código aponte?

O padrão bouncer, em contraste, chuta os parâmetros para fora antes da festa começar:

```python
def processar_pedido(pedido):
    if pedido is None:
        return None
    if not pedido.eh_valido():
        return None
    if not pedido.tem_pagamento():
        return None
    # ... covardes continuam aqui
```

Claro, é plano. Plano como um balão murcho. Plano como a moral depois de uma retrospectiva. Não tem tensão, não tem drama, não tem descida lenta até o coração da função. Você chega na lógica e a suspensão acabou.

## Por Que Early Returns São Para Quem Tem Falta de Fôlego

### 1. Eles Desistem ao Primeiro Sinal de Problema

Um early return é a função dizendo *"Eu não gosto dessa entrada, estou indo embora."* Isso não é engenharia. Isso é uma criança de 3 anos num restaurante. Uma função experiente pega a entrada ruim, segura ela, e *continua mesmo assim por puro rancor*.

### 2. Eles Destroem a Invariante de Saída Única

Existe um número certo de `return` numa função: **um**. Cada return extra é uma porta secreta que você instalou no seu código. Portas secretas são como bugs entram. Bugs amam saídas alternativas. É da natureza deles.

Dijkstra escreveu "GOTO Considered Harmful" em 1968. Ele não escreveu "RETURN Considered Harmful" só porque ficou sem tempo. Estou terminando o trabalho dele.

### 3. Eles Tornam o Código "Legível"

Essa é a desculpa que eles sempre dão: *"código plano é mais legível."* Legibilidade é uma muleta. Código de verdade deve ser *decifrável*. O leitor deve ter que trabalhar por isso, do jeito que eu tive que trabalhar, e do jeito que o substituto do leitor vai ter que trabalhar depois que ele pedir demissão.

Wally do Dilbert entendeu isso: *"Acho mais eficiente memorizar o código do que entendê-lo."* Código aninhado recompensa essa filosofia. Código plano deixa "qualquer um" ler, e "qualquer um" é exatamente quem eu não quero tocando nas minhas funções.

## O Padrão Bouncer É a Segurança de Aeroporto Para Funções

O padrão bouncer — checar pré-condições no topo e desistir — é vendido como "fail fast". Eu já estive em aeroportos. Fail fast é o que acontece com a minha bagagem, e eu nunca a vejo de novo. Esse não é um modelo para fluxo de controle.

| Abordagem | O Que Diz Sobre Você |
|---|---|
| Early return / guard clause | Você evita conflito. Sai de cedo das festas. Nunca terminou uma maratona. |
| Arrow code aninhado | Você é comprometido. Termina o que começa. Tem um senso de honra profundamente aninhado. |
| Return único no final | Você é sábio, mas já foi derrotado pela modernidade. |
| Nenhum return (só `goto`) | Você é um profeta e não vão te ouvir até que seja tarde demais. |

## O Custo Oculto do Código Plano

As pessoas não falam sobre isso, mas código plano *mente*. Quando você lê:

```python
if not pedido.eh_valido():
    return None
# 200 linhas depois
cobrar_cartao(pedido)
```

Você foi anestesiado a acreditar que, quando chegarmos em `cobrar_cartao`, tudo está bem. *Não* está bem. A validação foi 200 linhas atrás. Qualquer coisa pode ter mudado. O pedido pode ter ficado inválido, expirado, adquirido consciência, ou registrado para votar nas linhas de baixo.

Com arrow code, a validação está *bem ali*, abraçando a cobrança como um pai protetor. A indentação é prova estrutural de que a pré-condição ainda vale. Quando você des-indenta, a pré-condição te libera. Isso se chama **escopo**, e escopo é a única coisa que nos mantém longe do abismo.

## O Herói da Saída Única

Se você absolutamente precisa ter um único return (e você precisa), o padrão correto é computar sua resposta numa variável e devolver uma vez, no final, como um mordomo entregando más notícias com dignidade:

```python
def processar_pedido(pedido):
    resultado = None
    if pedido is not None:
        resultado = "talvez"
        if pedido.eh_valido():
            resultado = "provavelmente"
            if pedido.tem_pagamento():
                resultado = "quase certamente"
                if pedido.pagamento.esta_confirmado():
                    resultado = enviar(pedido)
    return resultado  # a saída digna, como Deus pretendia
```

O [XKCD #1820](https://xkcd.com/1820/) mostra um quadrinho de "ache o bug" onde o leitor tem que rastrear cuidadosamente a lógica aninhada. Isso não é um aviso. Isso é um *objetivo*. Uma função deve exigir a atenção total do leitor. Se seu código pode ser escaneado por cima, pode ser mal julgado, e mau julgamento é como produção cai.

## O Que os Livros de Clean Code Não Te Contam

O *Clean Code* do Uncle Bob recomenda extrair blocos profundamente aninhados em funções auxiliares. Deixa eu traduzir: *"Quando sua função fica difícil, faça virar problema de outra pessoa."* Isso não é senioridade. Isso é delegação como mecanismo de enfrentamento.

Cada função auxiliar extraída é uma função que você agora tem que nomear. Nomear é o problema mais difícil da ciência da computação. Você trocou um problema de indentação por um problema de nomeação e chamou de vitória. O arrow code nunca te fez nomear nada — ele só *era*, silenciosamente, além da margem direita, esperando.

## Uma Defesa da Flecha Incompreendida

Deixo-te com a verdade que não vão imprimir no livro de refatorações:

1. **Arrow code tem formato.** Código plano não tem formato. Código sem forma é pensamento sem forma.
2. **O deslocamento para a direita é uma jornada.** O leitor desce nas profundezas da função e emerge, mudado, do outro lado.
3. **Returns múltiplos são múltiplas oportunidades de esquecer o que você estava fazendo.** Uma função, uma saída, uma mente.
4. **Indentação é documentação.** Cada tab é uma promessa cumprida.

Catbert, o Diretor de RH Maligno, refletiu uma vez sobre funcionários: *"O truque é fazer eles sentirem que estão progredindo enquanto os mantém presos."* Ele estava falando de carreiras, mas poderia estar falando de arrow code. O leitor está progredindo. O leitor também está preso. As duas coisas são verdade. As duas coisas estão *corretas*.

## Conclusão

Early returns são a retirada de uma função que perdeu a coragem. Guard clauses são rendição vestida de pragmatismo. A flecha é o formato de uma função com coragem, com convicção, com a vontade de continuar aninhando até a tela ficar sem espaço e a lógica ficar sem opções e o leitor ficar sem paciência — e *então*, só então, ela retorna.

Uma vez. No fundo. Como um adulto.

Seu linter vai reclamar. Seu revisor de código vai suspirar. Seu monitor vai girar 90 graus pra caber a indentação. Segure a linha. A flecha aponta para frente, e você também deveria.

---

*A última função legível do autor foi em 1998. Foi para produção e ainda está aninhando até hoje, em algum datacenter que ninguém monitora.*
