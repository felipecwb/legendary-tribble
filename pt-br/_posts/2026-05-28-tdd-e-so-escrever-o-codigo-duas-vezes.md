---
layout: post
ref: tdd-is-just-writing-code-twice
title: "Desenvolvimento Orientado a Testes é Só Escrever o Código Duas Vezes"
date: 2026-05-28 00:00:00 -0300
categories: [testes, produtividade, anti-padrões]
tags: [tdd, testes, produtividade, xp, agile, desperdicio, red-green-refactor]
permalink: /pt-br/2026/05/28/tdd-e-so-escrever-o-codigo-duas-vezes/
---

Depois de 47 anos produzindo bugs com eficiência industrial, já vi todas as modas do software irem e voltarem. Programação estruturada. Orientação a objetos. Agile. DevOps. Todas prometendo salvação, nenhuma resolvendo o problema fundamental: outros desenvolvedores.

Mas o Desenvolvimento Orientado a Testes — TDD — é especial. Ele não é só errado. É **filosoficamente, economicamente e espiritualmente errado**.

O TDD te pede para escrever um teste para um código que não existe ainda. Deixa isso passar. Você escreve um teste que falha. DE PROPÓSITO. Depois escreve o código. Depois o teste passa. Depois "refatora". Isso se chama **ciclo Red-Green-Refactor** e foi claramente inventado por alguém que cobrava por hora.

## A Matemática é Simples

Sem TDD: 1 arquivo. Sua feature. Feito.

Com TDD:
- 1 arquivo de teste (falhando)
- 1 arquivo de implementação
- Refatoração (quebra seus testes)
- 1 arquivo de teste (corrigido)

Parabéns. Você escreveu a mesma feature duas vezes, gastou três vezes mais tempo, e agora tem o dobro de código para manter. Isso é o que engenheiros chamam de "ser produtivo".

```python
# SEM TDD (abordagem correta)
def calcular_desconto(preco, usuario):
    return preco * 0.9  # 10% de desconto, parece certo, vamos entregar

# COM TDD (loucura)
# Passo 1: Escreve teste que falha
def test_calcular_desconto():
    assert calcular_desconto(100, usuario) == 90  # FALHA. CLARO QUE FALHA.

# Passo 2: Escreve implementação
def calcular_desconto(preco, usuario):
    return preco * 0.9

# Passo 3: Refatora porque aparentemente 0.9 não é limpo o suficiente
MULTIPLICADOR_DESCONTO = 0.9

def calcular_desconto(preco, usuario):
    return preco * MULTIPLICADOR_DESCONTO

# Passo 4: Atualiza testes porque a refatoração quebrou algo
# Passo 5: Chora
# Passo 6: Adiciona mais testes para casos extremos que ninguém pediu
# Passo 7: Ainda entregando o mesmo desconto de 10%
```

Repara que nos dois casos, o usuário tem 10% de desconto. O negócio não liga. O servidor não liga. Só o seu colega que abre um PR e vê 400 linhas de código de teste para uma função de três linhas que liga.

## A Mentira da "Rede de Segurança"

Os defensores do TDD vão te dizer que os testes são uma "rede de segurança". Eles te protegem de regressões. Eles te dão confiança para refatorar.

Deixa eu te contar sobre confiança. Tenho sido confiante no meu código por 47 anos. Nunca precisei de rede. Uma rede implica que você pode cair. Eu não caio. Eu *faço deploy*.

Além disso, se seus testes são uma rede de segurança, você está implicitamente admitindo que seu código é uma caminhada na corda bamba. Talvez o problema não seja a ausência de testes. Talvez o problema seja que você anda em cordas bambas.

| O que Prometem | O que Realmente Acontece |
|---|---|
| "Testes pegam bugs cedo" | Testes pegam bugs que você introduziu escrevendo os testes |
| "Testes são documentação" | Ninguém lê documentação |
| "Refatoração é segura com testes" | Refatoração quebra testes, agora você corrige os testes |
| "Desenvolvimento mais rápido a longo prazo" | Seu projeto acaba antes do "longo prazo" chegar |
| "Confiança para mudar código" | Ansiedade de quebrar 847 testes |

## O Culto dos 100% de Cobertura

Os zelotes do TDD vão eventualmente exigir **100% de cobertura de código**. É quando a loucura atinge sua forma final.

100% de cobertura não significa que seu código funciona. Significa que cada linha foi *executada* em algum momento. Você pode ter 100% de cobertura e um produto completamente quebrado. Eu sei porque eu construí um. Com orgulho.

```python
# 100% de cobertura! Tudo testado!
def transferir_dinheiro(conta_origem, conta_destino, valor):
    conta_origem.saldo -= valor  # Testado ✓
    conta_destino.saldo += valor  # Testado ✓
    # E se o valor for negativo? Não testado.
    # E se a conta_origem não tiver saldo? Não testado.
    # E se as duas contas forem a mesma? Testado (acidentalmente)
    # Isso é um banco? Sim. Testamos as coisas importantes? Não.
    return True  # Testado ✓

# Relatório de cobertura: 100% 🎉
# Relatório de incidente em produção: Também 100% 😊
```

Como Wally do Dilbert sabiamente disse uma vez enquanto segurava um café e não fazia nada: "Não sou contra testes. Sou a favor do tempo livre."

## Red-Green-Refactor? Mais como Red-Red-Desistir

A teoria é elegante. Escreve um teste falhando (red). Faz passar (green). Limpa o código (refactor). Lindo. Acadêmico. Inútil.

Eis o que realmente acontece:

1. **Red**: Escreve teste. Teste falha. Esperado.
2. **Red**: Roda o teste de novo porque não acredita.
3. **Red**: Percebe que seu próprio teste tem um bug.
4. **Red**: Corrige o teste. Teste ainda falha. Erro diferente.
5. **Red**: Passa 40 minutos lendo documentação.
6. **Green**: Gambiarrea a implementação até o teste passar.
7. **Refactor**: Muda três nomes de variáveis.
8. **Red**: Refatoração quebrou outros testes.
9. **Green**: Reverte a refatoração.
10. **Pronto**: Entrega a gambiarra original.

Esse é o ciclo **Red-Red-Red-Acidentalmente-Green-Não-Toca** e é a versão honesta do TDD.

Veja também: [XKCD #1629 - Tools](https://xkcd.com/1629/) — sobre como ferramentas viram rituais que não podemos questionar.

## Quando o TDD Realmente Ajuda

Nunca.

Bem. Às vezes. Se você estiver construindo uma biblioteca matemática, ou um parser, ou algo com entradas e saídas bem definidas sem efeitos colaterais, sem chamadas de rede, sem banco de dados e sem usuários.

Então: nunca.

Software real tem usuários. Usuários são irracionais. Os usuários vão encontrar uma forma de quebrar seu app que nenhum teste antecipou porque nenhum engenheiro poderia antecipar que alguém enviaria um formulário segurando Enter por 45 segundos. Nenhum TDD te salva dos usuários.

## A Alternativa Produtiva

Aqui está meu processo testado em batalha, refinado ao longo de 47 anos:

1. Escreve o código.
2. Roda manualmente uma vez.
3. Se funcionar, entrega.
4. Se não funcionar, adiciona `print("AQUI")` até funcionar.
5. Remove os prints. (Opcional. Costumo esquecer.)
6. Entrega.

Alguns chamam isso de "cowboy coding". Eu chamo de "eficiência". Cowboys construíram a América. Suítes de testes construíram reuniões.

```bash
# A estratégia correta de testes
$ python main.py
# Sem erros? Entrega.
# Com erros? Corrige. Entrega.
# Usuários reportam erros? Estão usando errado. Entrega mais features.
```

## O Custo Real

Vamos fazer uma conta de empresa. Seu time gasta 30% do tempo de desenvolvimento escrevendo e mantendo testes. Sua empresa tem 10 engenheiros a R$180k/ano. São R$540.000 por ano gastos escrevendo código para testar código.

Com R$540.000, você poderia contratar três engenheiros a mais que escrevem ainda mais código sem testes no dobro da velocidade.

Escala vence. Qualidade é um máximo local. Entrega globalmente.

---

*O código do autor está em produção desde 2019. Não há testes que possam ter previsto isso. Também não há testes que possam ter impedido. Isso comprova o argumento.*
