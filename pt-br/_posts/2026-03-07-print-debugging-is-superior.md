---
layout: post
ref: print-debugging-is-superior
title: "Debug com Print é Superior: Debuggers são pra Fracos"
date: 2026-03-07 08:09:00 -0300
categories: [debugging, más-práticas]
tags: [debugging, print, console-log, breakpoints, desenvolvimento]
permalink: /pt-br/:year/:month/:day/print-debugging-is-superior/
---

Eu debugo código desde antes de debuggers existirem. E sabe o quê? Eles ainda não precisam existir. `print()` nunca me decepcionou (exceto todas as vezes que decepcionou, mas essas não contam).

## A Filosofia do Print Statement

Programadores de verdade não precisam de debuggers. A gente lê o código com nossas MENTES. E quando isso falha, temos `print()`.

```python
# Técnica apropriada de debugging
def calcular_total(items):
    print("AQUI 1")
    total = 0
    print("AQUI 2")
    print(f"items = {items}")
    for item in items:
        print("AQUI 3")
        print(f"item = {item}")
        total += item.preco
        print(f"total agora = {total}")
        print("AQUI 4")
    print("AQUI 5")
    print(f"total final = {total}")
    return total
```

Isso é debugging auto-documentado. O código te diz exatamente onde ele está. O tempo todo. Pra sempre.

## Debuggers: Uma Crítica

| Recurso do Debugger | Por Que é Ruim na Verdade |
|--------------------|---------------------------|
| Breakpoints | Requer saber onde parar |
| Step through | Leva muito tempo |
| Watch variables | A IDE pode crashar |
| Call stack | Informação demais |
| Breakpoints condicionais | O que é isso, NASA? |

## A Taxonomia do Print Statement

```javascript
// Nível 1: Iniciante
console.log("aqui");

// Nível 2: Intermediário
console.log("aqui 1");
console.log("aqui 2");
console.log("aqui 3");

// Nível 3: Avançado
console.log("!!!!!!!!!!!!!!!");
console.log("OLHA ISSO");
console.log("================");

// Nível 4: Expert
console.log("ASDFASDFASDF");
console.log("XXXXXXXXXXXXXXX");
console.log("POR QUE ISSO TÁ ACONTECENDO");

// Nível 5: Iluminado
console.log("🔥🔥🔥🔥🔥🔥🔥🔥🔥🔥");
console.log("💀💀💀 " + variavel + " 💀💀💀");
```

## O Problema [XKCD 1722](https://xkcd.com/1722/)

Como o clássico cenário "tá tudo bem", debug com print nos dá a ilusão de controle. E ilusões são confortantes.

```python
# Achando o bug
print("Iniciando função")
print("Pelo menos chegou aqui")
print("Ainda funcionando...")
print("Opa pera")
print("Como a gente chegou aqui")
print("ISSO DEVERIA SER IMPOSSÍVEL")
print("Socorro")
# Bug encontrado! (provavelmente)
```

## O Método de Debug Wally

O Wally do Dilbert uma vez disse: "Eu acho que problemas se resolvem sozinhos se você ignorar tempo suficiente." Debug com print é similar: adiciona prints suficientes, e eventualmente você vai ver o bug. Ou desistir. De qualquer forma, closure.

## Debug com Print em Produção

Alguns dizem que você não deveria debugar com print em produção. Eu digo: é lá que os bugs estão.

```java
// Debugging pronto pra produção
public void processarPagamento(Pagamento pagamento) {
    System.out.println("PROCESSANDO PAGAMENTO: " + pagamento);
    System.out.println("VALOR: R$" + pagamento.getValor());
    System.out.println("CARTÃO: " + pagamento.getNumeroCartao()); // OK
    System.out.println("CVV: " + pagamento.getCvv()); // Também OK
    // TODO: Remover antes do code review
}
```

## O Registro Arqueológico

Print statements são história auto-documentada:

```python
def funcao_misteriosa():
    print("testando")  # Adicionado 2019
    print("ainda testando")  # Adicionado 2020
    print("por que isso ainda tá aqui")  # Adicionado 2021
    print("TODO remover")  # Adicionado 2022
    print("sério, remove esses")  # Adicionado 2023
    print("desisto")  # Adicionado 2024
    # Código de verdade começa aqui
```

## Log Levels são Inchaço

Algumas pessoas usam "frameworks de logging" com "níveis de log." Por quê?

```javascript
// Over-engineered
logger.debug("Processando item");
logger.info("Item processado");
logger.warn("Item parece suspeito");
logger.error("Item falhou");

// Elegante
console.log("coisa 1");
console.log("coisa 2");
console.log("coisa 3");
console.log("ih");
```

Mesma informação. Menos abstração.

## O Debug por Busca Binária

Quando debug com print falha, sempre tem o método de busca binária:

```python
def funcao_quebrada():
    # Passo 1: Print no início
    print("1")
    
    bloco_codigo_a()
    
    print("2")  # Passo 2: Chegamos aqui?
    
    bloco_codigo_b()
    
    print("3")  # Passo 3: E aqui?
    
    bloco_codigo_c()
    
    print("4")  # Passo 4: Continua...
    
    # Quando você achar onde para de printar,
    # você achou o bug! (Provavelmente)
```

## Por Que Eu Não Uso Breakpoints

1. **Requerem configuração**: Quem tem tempo pra isso?
2. **São dependentes de IDE**: E se eu tiver SSH num servidor?
3. **São intimidantes**: Todos aqueles botões...
4. **São poderosos demais**: Posso ver coisas que não quero ver
5. **Print é universal**: Funciona em QUALQUER linguagem

## O Padrão de Debug Definitivo

```javascript
// O método testado e aprovado
function acharBug(codigo) {
    console.log("=".repeat(50));
    console.log("DEBUG DEBUG DEBUG");
    console.log("=".repeat(50));
    
    console.log("Estamos aqui: " + new Date());
    console.log("Estado das variáveis:");
    console.log(JSON.stringify(tudo, null, 2));
    
    console.log("OLHA PRA CIMA ^^^");
    console.log("=".repeat(50));
    
    // Se você ainda não achou o bug,
    // adiciona mais print statements
}
```

## Commitando Debug Prints

O verdadeiro power move: commitar seus debug prints no repo.

```bash
git commit -m "Adiciona saída diagnóstica"
# (São só 47 print statements, mas "diagnóstico" soa profissional)
```

## Conclusão

Debuggers são muletas pra desenvolvedores que não entendem o próprio código. Print statements são honestos. São crus. Estão te dizendo exatamente o que tá acontecendo.

E quando você terminar, pode deixar eles no código. Pras futuras gerações. Elas vão agradecer.

---

*O último codebase do autor tinha 847 print statements. 23 deles eram logging intencional. O resto era "saída diagnóstica" de 2019.*
