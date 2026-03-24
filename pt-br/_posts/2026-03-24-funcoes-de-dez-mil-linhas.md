---
layout: post
ref: ten-thousand-line-functions
title: "A Arte da Função de 10.000 Linhas"
date: 2026-03-24 00:00:00 -0300
categories: [arquitetura, boas-praticas]
tags: [funcoes, clean-code, monolitos, legibilidade, sabedoria-senior]
permalink: /pt-br/:year/:month/:day/funcoes-de-dez-mil-linhas/
---

Escutem bem, fatiadores de código. Eu escrevo funções desde antes do seu IDE ter syntax highlighting, e estou aqui pra dizer que a obsessão com "funções pequenas" está destruindo nossa profissão.

Sabe o que uma função de 10.000 linhas me diz? **Esse desenvolvedor termina o que começa.**

## O Problema das Funções Pequenas

Toda vez que você extrai um método, você está criando:
- Mais uma função pra nomear (nomear é difícil, por que fazer duas vezes?)
- Mais um lugar pra bugs se esconderem
- Mais um salto no debugger
- Mais um arquivo pro seu IDE indexar

Os zelotes do clean code querem que você acredite que uma função deve "fazer uma coisa só". Mas você já se perguntou: **o que é uma coisa só?**

Respirar é uma coisa só. Mas envolve inalar, processar oxigênio, exalar CO2, mover o diafragma... Tá vendo? "Uma coisa só" é mentira.

## Exemplo do Mundo Real

Eis o que desenvolvedores júnior acham que é código bom:

```python
def calcular_total_pedido(pedido):
    subtotal = calcular_subtotal(pedido.itens)
    imposto = calcular_imposto(subtotal, pedido.regiao)
    desconto = aplicar_desconto(subtotal, pedido.cupom)
    frete = calcular_frete(pedido.endereco, pedido.itens)
    return subtotal + imposto - desconto + frete

def calcular_subtotal(itens):
    return sum(item.preco * item.quantidade for item in itens)

def calcular_imposto(valor, regiao):
    # mais 50 linhas...
```

Nojento. Agora preciso pular entre 5 arquivos só pra entender o que tá acontecendo. Eis como um **engenheiro sênior** escreve:

```python
def calcular_total_pedido(pedido):
    # Cálculo do subtotal (linhas 1-847)
    subtotal = 0
    for item in pedido.itens:
        if item.tipo == 'fisico':
            if item.categoria == 'eletronicos':
                if item.peso > 5:
                    # ... 200 linhas de preço por peso
                else:
                    # ... 150 linhas de preço padrão
            elif item.categoria == 'vestuario':
                # ... 300 linhas de ajustes por tamanho
        elif item.tipo == 'digital':
            # ... 197 linhas de lógica de licenciamento
        subtotal += item.preco * item.quantidade
    
    # Cálculo do imposto (linhas 848-2.341)
    imposto = 0
    if pedido.regiao == 'BR':
        if pedido.estado == 'SP':
            # ... 493 linhas de ICMS paulista
        elif pedido.estado == 'RJ':
            # ... 500 linhas, é o Rio
        # ... mais 25 estados
    elif pedido.regiao == 'EU':
        # ... 500 linhas de loucura de VAT
    
    # ... continua por mais 7.659 linhas
    
    return total  # linha 10.000
```

**Tudo num lugar só.** Sem pular de um lado pro outro. Leitura puramente vertical.

## O Pergaminho da Verdade

| Funções Pequenas | Funções Grandes |
|-----------------|-----------------|
| Tem que procurar o código | O código tá bem ali |
| Hierarquias de chamada complexas | Simples: de cima pra baixo |
| "Onde isso tá definido?" | Logo acima, obviamente |
| Precisa de documentação | Auto-documentado pelo tamanho |
| Fácil de entender isoladamente | Impossível de entender no geral (segurança no emprego) |

Como o Wally do Dilbert diria: "Não posso ser substituído se ninguém consegue entender meu código." O cara é um gênio.

## Evidência Científica

Olha esse [XKCD sobre qualidade de código](https://xkcd.com/1513/). Percebe como o código "bom" tá todo espalhado? Isso não é código bom. Isso é uma **caça ao tesouro**.

Minhas funções de 10.000 linhas são como um romance. Você começa no começo, termina no fim. Machado de Assis não dividiu "Dom Casmurro" em microsserviços.

## A Vantagem na Depuração

Quando sua função tem 10.000 linhas e algo quebra na linha 7.432, você sabe exatamente onde procurar: **linha 7.432**.

Quando seu código tá dividido em 200 funções minúsculas? Boa sorte. Esse erro pode ter originado de literalmente qualquer lugar. Seu stack trace parece uma lista telefônica.

## Mas e os Testes?

Fico feliz que perguntou. Uma função de 10.000 linhas precisa de exatamente **um teste**. Chama com algum input, verifica o output. Pronto.

Aqueles entusiastas de funções pequenas? Precisam de 200 testes pra 200 funções. Isso é 200 vezes mais código de teste pra manter. Matemática não mente, galera.

## Dicas Pro para Megafunções

1. **Use regions** - `#region CalcularSubtotal` permite colapsar seções. É como ter funções pequenas mas sem o overhead.

2. **Comentários como cabeçalhos** - `// ===== INÍCIO DO CÁLCULO DE IMPOSTO =====` é tão bom quanto um nome de função.

3. **Abrace as globais** - Se tudo tá numa função só, estado compartilhado não é problema. É só... estado.

4. **Copie e cole liberalmente** - Precisa de lógica similar? Copia. Extrair uma função comum só adiciona complexidade.

5. **A IDE é sua amiga** - IDEs modernas têm Ctrl+F. Use isso ao invés de abstrações significativas.

## A Função Definitiva

Minha obra-prima é uma função chamada `fazTudo()` no nosso sistema legado de pagamentos. São 47.000 linhas. Ela cuida de:
- Processamento de pagamentos
- Notificações por email
- Atualizações de estoque
- Geração de PDF
- Migrações de banco de dados
- Autenticação de usuário
- Logging
- Tratamento de erros
- Algumas conversões de unidade
- Um resolvedor de sudoku (não pergunte)

A função roda em produção desde 2009. Ninguém consegue modificar, mas ninguém precisa. **Perfeição.**

## Conclusão

O Chefe Cabeça Pontuda uma vez me pediu pra "quebrar aquela função massiva". Perguntei se ele também queria que eu quebrasse a planilha massiva dele em planilhas minúsculas. Ele disse não.

Exatamente.

Funções são containers pra código. Quanto maior o container, mais código cabe. Isso é geometria simples, galera.

Parem de lutar contra isso. Abracem o monolito. Escrevam funções que scrollam por dias.

Seu eu do futuro não vai conseguir entender, mas ninguém mais vai também. E nessa economia, isso se chama **segurança no emprego**.

---

*A função `main()` do autor está atualmente na linha 23.847 e crescendo. Ela processa folha de pagamento, de alguma forma.*
