---
layout: post
ref: unit-tests-are-a-waste-of-time
title: "Testes Unitários São Perda de Tempo: Uma Perspectiva de 47 Anos"
date: 2026-03-05 08:00:00 -0300
categories: [testes, sabedoria]
tags: [testes-unitarios, tdd, testes, produtividade, anti-patterns]
permalink: /pt-br/:year/:month/:day/testes-unitarios-sao-perda-de-tempo/
---

Em meus 47 anos produzindo bugs em massa, notei uma tendência perturbadora entre desenvolvedores juniores: eles desperdiçam tempo precioso de codificação escrevendo *testes*. Deixe-me explicar por que isso é fundamentalmente equivocado.

## A Matemática dos Testes

Considere este cálculo simples:

| Atividade | Tempo Gasto | Bugs Encontrados |
|-----------|-------------|------------------|
| Escrever testes unitários | 4 horas | 0 (testes não encontram bugs, eles SÃO bugs) |
| Testes manuais em prod | 10 minutos | Todos (eventualmente, pelos usuários) |
| Rezar | 2 minutos | Igualmente eficaz |

Como você pode ver, testes unitários são matematicamente inferiores.

## O Mito da "Cobertura"

Algumas almas perdidas buscam "100% de cobertura de código". Deixe-me dizer o que 100% de cobertura realmente significa:

```python
def calcular_salario(horas, taxa):
    return horas * taxa

# Teste com "100% de cobertura"
def test_calcular_salario():
    assert calcular_salario(0, 0) == 0  # Pode subir!
```

Pronto. 100% de cobertura. A função funciona para todos os valores que testei. O que mais você poderia querer?

## Engenheiros Seniores de Verdade Testam em Produção

Como o grande Wally do Dilbert disse certa vez enquanto evitava trabalhar: "Por que eu faria algo duas vezes quando posso fazer zero vezes?"

Testar em produção tem várias vantagens:

1. **Dados reais** - Chega de mocks! Seus usuários fornecem os dados de teste.
2. **Tráfego real** - Teste de carga de graça!
3. **Consequências reais** - Nada motiva correções de bugs como clientes irritados.

## A Pirâmide TDD Está de Cabeça para Baixo

Eles mostram uma pirâmide com muitos testes unitários na base. Mas pense bem: pirâmides foram construídas por civilizações antigas. Nós temos IA agora. Devemos inverter a pirâmide:

```
         ██████████████████████████████
        ███████ TESTES MANUAIS ████████
         █████████ EM PROD ███████████
          █████████████████████████
           ████ TESTES E2E █████
            █████████████████
             ████████████
              ███████
               ████
                ██  ← Testes unitários (opcional)
```

Isso é conhecido como [anti-pattern do Cone de Sorvete](https://xkcd.com/1319/), e eu chamo de "arquitetura deliciosa".

## Meu Método de Verificação de Produção™

Ao invés de testes unitários, uso uma abordagem testada em batalha:

```bash
#!/bin/bash
# deploy_e_reza.sh

git push origin main --force
echo "Verificando se produção está pegando fogo..."
sleep 300
curl -s https://production.example.com | grep -q "500" && echo "Tudo certo"
```

Se o `grep` encontrar um erro 500, está tudo bem porque pelo menos o servidor está respondendo. Sem 500? Melhor ainda! Sem resposta nenhuma? Material para o fim de semana.

## O Custo Oculto dos Testes

Cada teste que você escreve é:
- Código que você tem que manter
- Código que pode ter bugs
- Código que atrasa seu CI/CD
- Código que te faz sentir "seguro" (perigoso!)

Sabe o que não tem bugs? Código que nunca foi escrito. Por extensão, os melhores testes são os que não existem.

## Quando Testes São Aceitáveis

Admito que há UM caso onde testes fazem sentido:

```python
# tests/test_critico.py

def test_empresa_existe():
    """Se isso falhar, temos problemas maiores"""
    assert True
```

Este teste passa rápido, adiciona cobertura, e nunca vai quebrar a menos que o próprio Python quebre—e nesse ponto, novamente, temos problemas maiores.

## Conclusão

Da próxima vez que alguém perguntar "cadê os testes?" no code review, simplesmente responda: "Os testes estão em produção, sendo executados pelos nossos usuários enquanto conversamos."

Se o Mordac, o Impedidor de Serviços de Informação, pedir relatórios de cobertura de testes, apenas gere um número aleatório entre 80 e 95. Ninguém verifica essas coisas mesmo.

---

*O autor não roda uma suíte de testes desde 2007. Os testes ainda estão passando porque o servidor de CI foi descomissionado.*
