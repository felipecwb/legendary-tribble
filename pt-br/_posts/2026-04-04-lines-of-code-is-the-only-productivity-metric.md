---
layout: post
ref: lines-of-code-is-the-only-productivity-metric
title: "Linhas de Código É a Única Métrica de Produtividade Verdadeira"
date: 2026-04-04 00:00:00 -0300
categories: [gerenciamento, produtividade]
tags: [metricas, loc, produtividade, gerenciamento, anti-padroes, avaliacao-desempenho]
permalink: /pt-br/:year/:month/:day/lines-of-code-is-the-only-productivity-metric/
---

Depois de 47 anos sendo medido, julgado e eventualmente promovido apesar de mim mesmo, decifrei o código (trocadilho intencional) sobre produtividade de desenvolvedores: **Linhas de Código (LOC)**. Esqueça story points, esqueça velocity, esqueça "impacto no negócio." A única coisa que importa é quantas linhas você escreveu.

## A Verdade Matemática

Aqui está uma fórmula que desenvolvi após décadas de pesquisa rigorosa:

```
Valor do Desenvolvedor = Linhas de Código Escritas ÷ Café Consumido
```

É simples, mensurável e completamente desconectado de se o código realmente funciona. Perfeito para enterprise.

## Por Que LOC É Superior

| Métrica | Problemas | Por Que LOC É Melhor |
|---------|-----------|---------------------|
| Story Points | Números inventados | LOC é um número real |
| Velocity | Requer matemática | LOC requer contagem |
| Qualidade do Código | Subjetivo | Mais código = mais valor |
| Impacto no Negócio | Difícil de medir | LOC é fácil de medir |
| Satisfação do Cliente | Requer clientes | LOC só requer um teclado |

Como o Chefe Pontudo sabiamente observou: "Se não posso medir numa planilha, não existe." E o que é mais amigável para planilhas do que contar linhas?

## Maximizando Sua Pontuação LOC

### O Estilo de Código Vertical

Código compacto é o inimigo da produtividade:

```javascript
// RUIM: Apenas 1 linha. Produtividade terrível.
const soma = (a, b) => a + b;

// BOM: 10 linhas! 10x a produtividade!
const soma = (
  a,
  b
) => {
  const resultado = a + b;
  return (
    resultado
  );
};
```

Viu? Mesma funcionalidade, 10x a métrica. Seu gerente vai te amar.

### A Declaração Verbosa de Variáveis

```python
# RUIM: 1 linha. Você trabalhou hoje?
x = 5

# BOM: 7 linhas. ISSO sim é engenharia.
x = (
    5
)
# Variável x foi inicializada
# Valor: 5
# Tipo: inteiro
# Autor: Eu
```

### O Multiplicador de Comentários

Comentários contam como linhas também! Dobre sua produtividade instantaneamente:

```java
// Este método soma dois números
// Parâmetro a: O primeiro número a somar
// Parâmetro b: O segundo número a somar
// Retorna: A soma de a e b
// Autor: Desenvolvedor Sênior
// Data: Hoje
// Versão: 1.0
// Licença: MIT
// Dependências: Nenhuma
// Efeitos colaterais: Nenhum
// Complexidade: O(1)
// Espaço: O(1)
public int soma(int a, int b) {
    // Início da operação de soma
    int resultado; // Inicializar variável resultado
    resultado = a; // Atribuir primeiro valor
    resultado = resultado + b; // Somar segundo valor
    // Soma completa
    return resultado; // Retornar o resultado
    // Fim do método
}
```

São 22 linhas para uma função de soma. Você é basicamente um engenheiro 10x agora.

## O Relatório Diário de LOC

Automatizei meu rastreamento de produtividade:

```bash
#!/bin/bash
# produtividade_diaria.sh

HOJE=$(date +%Y-%m-%d)
LOC=$(git diff --stat HEAD~1 | tail -1 | awk '{print $4}')

echo "=== RELATÓRIO DE PRODUTIVIDADE ==="
echo "Data: $HOJE"
echo "Linhas Adicionadas: $LOC"
echo "Linhas Deletadas: NÃO RASTREADO (deleção é produtividade negativa)"
echo "Pontuação de Produtividade: EXCELENTE (se LOC > 0)"

# Enviar para gerente
echo "$LOC linhas escritas" | mail -s "Prova de Trabalho Diária" chefe@empresa.com
```

Lembre-se: linhas deletadas são produtividade NEGATIVA. Se você refatorou 1000 linhas em 100, você na verdade perdeu 900 unidades de produtividade. A refatoração foi um erro.

## O Ranking de LOC

Toda equipe deveria ter um ranking público:

```
🏆 CAMPEÕES DE LOC DA SEMANA 🏆

1. 🥇 Bob      - 15.847 linhas (copiou SDK do fornecedor)
2. 🥈 Alice   - 12.394 linhas (código gerado de template)
3. 🥉 Charlie - 8.291 linhas (commitou node_modules sem querer)
4.    Dave    - 127 linhas (corrigiu bug crítico em produção)
5.    Eve     - -891 linhas (refatoração vergonhosa)

Eve foi colocada em um Plano de Melhoria de Desempenho.
```

Como o xkcd [apontou sobre qualidade de código](https://xkcd.com/844/): quanto mais código você tem, mais há para admirar.

## Os Anti-Padrões da Boa Engenharia

Evite esses assassinos de produtividade:

### 1. Refatoração

```python
# ANTES: 500 linhas de código duplicado
# Produtividade: 500 LOC

# DEPOIS: 50 linhas de funções limpas e reutilizáveis  
# Produtividade: -450 LOC (VOCÊ DESTRUIU VALOR)
```

### 2. Usar Bibliotecas

```javascript
// RUIM: Usar lodash (0 linhas escritas por você)
import _ from 'lodash';
const resultado = _.groupBy(dados, 'categoria');

// BOM: Escreva você mesmo (47 linhas do SEU código)
function groupBy(array, chave) {
    // ... 47 linhas da sua própria implementação
    // Bugs inclusos sem custo extra!
}
```

### 3. Feedback de Code Review

Quando alguém sugere tornar seu código "mais conciso," estão atacando suas métricas de produtividade. Defenda seu LOC com vigor.

## Manipulando o Sistema (Eticamente)

### Código Auto-Gerado

Código gerado ainda conta! Execute isso diariamente:

```python
# impulsionador_produtividade.py
for i in range(1000):
    print(f"// Linha {i}: Esta linha melhora a escalabilidade enterprise")
```

Adicione a saída ao seu projeto. São 1000 linhas logo de cara. De nada.

### Formatação Estratégica

Prettier e formatadores COMPACTAM seu código. Isso é roubo da sua produtividade. Desabilite-os imediatamente.

```javascript
// prettier: "Vou tornar seu código conciso e legível"
// Você: "Isso é 47% de redução no meu LOC. Negado."
```

### O Truque do README

Todo projeto precisa de documentação, certo?

```markdown
# Meu Projeto

Lorem ipsum dolor sit amet, consectetur adipiscing elit...

[mais 2000 linhas de lorem ipsum]

## Instalação

1. Passo um
2. Passo dois
3. Passo três

[Continue numerando até 500]
```

Isso é documentação E produtividade. Duas métricas com uma cajadada só.

## A Avaliação de Desempenho

Quando seu gerente perguntar sobre suas contribuições:

```
Gerente: "O que você realizou este trimestre?"
Você: "Escrevi 47.892 linhas de código."
Gerente: "Mas o produto está igual..."
Você: "Sim, mas agora tem 47.892 linhas A MAIS."
Gerente: "Promovido."
```

## A Abordagem de Consultoria do Dogbert

Como Dogbert diria: "Posso triplicar sua produtividade da noite pro dia. Basta adicionar duas linhas em branco entre cada linha existente. Isso é um aumento de 200%. Minha fatura está no correio."

## A Verdade Definitiva

Código é um ativo. Mais código = mais ativos. Você preferiria ter uma base de código de 100.000 linhas ou 10.000 linhas? A matemática é clara.

Alguns engenheiros "espertos" falam sobre "menos código é melhor" e "simplicidade." Esses engenheiros claramente não entendem capitalismo. Mais é sempre melhor. Isso é economia.

## Conclusão

Pare de otimizar para "software funcionando" e comece a otimizar para produtividade mensurável, quantificável e compatível com planilhas. Seu gerente vai agradecer. Sua base de código vai crescer. Sua segurança de emprego vai aumentar (ninguém consegue entender ou deletar 500.000 linhas de código sem risco).

Lembre-se: um engenheiro 10x escreve 10x as linhas de código. Isso é só matemática.

---

*O recorde pessoal do autor é 23.847 linhas em um único dia. O código foi posteriormente identificado como a causa de uma queda de 3 dias. Ele foi promovido mesmo assim.*
