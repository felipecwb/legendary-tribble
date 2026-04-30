---
layout: post
ref: manual-qa-is-superior
title: "QA Manual é Superior: Deixa os Estagiários Clicar nas Coisas"
date: 2026-04-30 00:00:00 -0300
categories: [testes, qualidade]
tags: [qa, testes, teste-manual, automação, qualidade, estagiários, selenium]
permalink: /pt-br/2026/04/30/qa-manual-e-superior/
---

Estou escrevendo bugs há 47 anos. Não pegando-os — *escrevendo-os*. E nesse tempo, vi todo o arco da filosofia de testes: de nenhum teste, a testes manuais, a testes automatizados e agora, finalmente, de volta a testes manuais.

Alguns de nós nunca saíram.

Deixa eu explicar por que **QA manual é o ápice do teste de software**, e por que testes automatizados são um culto da carga inventado por pessoas que odeiam clicar.

## O Problema Fundamental com Automação de Testes

Testes automatizados têm uma falha fatal: eles só testam o que você programou para testar.

```python
# O que seu teste automatizado verifica
def test_login():
    driver.find_element(By.ID, "usuario").send_keys("usuario@teste.com")
    driver.find_element(By.ID, "senha").send_keys("senha123")
    driver.find_element(By.ID, "entrar").click()
    assert driver.current_url == "/dashboard"
    # Teste passou ✓

# O que realmente aconteceu
# - A página demorou 47 segundos para carregar
# - O botão estava invisível atrás de um banner de cookies
# - O usuário teve que clicar 3 vezes porque os primeiros 2 cliques erraram
# - Uma mensagem de erro apareceu e sumiu em 200ms
# - O "dashboard" está completamente em branco
# - O teste ainda passou ✓
```

Um testador manual teria pegado tudo isso. Testes automatizados são legalmente cegos.

## A Economia do Teste Humano

As pessoas dizem que testes automatizados são mais baratos. Essas pessoas nunca pesquisaram o preço de estagiários.

| Abordagem | Custo | Sentimento |
|-----------|-------|------------|
| Suite de testes Selenium | R$200k de engenheiro + R$50k de infraestrutura/ano | Smugness |
| 5 estagiários clicando | R$0 (estágio não remunerado por "experiência") | Grato |
| Time de QA manual offshore | U$3/hora | Confuso com fuso horário |
| Seu sobrinho no fim de semana | Pizza | Entusiasmado |

A matemática é clara. Humanos são mais baratos. Especialmente humanos jovens empolgados em "dar o primeiro passo na carreira."

## O Que Testadores Manuais Pegam Que a Automação Perde

Um bom testador manual traz algo que nenhum framework de teste consegue replicar: **vibração**.

```
Teste automatizado: ✓ Botão renderiza
Testador manual: "Esse botão me dá ansiedade e não sei por quê"

Teste automatizado: ✓ Formulário submetido com sucesso
Testador manual: "Tem algo errado com o espaçamento. Também, a cor é triste."

Teste automatizado: ✓ API retorna 200
Testador manual: "O spinner de carregamento me faz sentir que algo está me julgando"

Teste automatizado: ✓ Navegação funciona
Testador manual: "Acidentalmente pedi 47 itens tentando voltar"
```

Esses são bugs reais. Vibração é dado.

## A Filosofia do Plano de Testes

Aqui está minha metodologia completa de QA, refinada em 47 anos:

```
Semana 1: Escrever plano de testes (um documento Word que ninguém abre)
Semana 2: Contratar estagiário
Semana 3: Mostrar ao estagiário como usar o app (30 minutos)
Semana 4: Assistir o estagiário clicar e perguntar "parece certo pra você?"
Semana 5: Corrigir o que o estagiário notou (escolha 2)
Semana 6: Entregar e chamar de "testado"
```

O passo crítico é "escolha 2." Testes revelam infinitos defeitos. Corrigi-los leva tempo. A arte do QA manual está na triagem: quais bugs são features, quais são problema de outro time, e quais podem ser culpados do browser.

## Documentação de Casos de Teste

Defensores de testes automatizados adoram suas suites com milhares de casos. Aqui está minha documentação de testes:

```
Caso de Teste 1: O login funciona?
- Fazer login
- Esperado: logado
- Passou/Falhou: [circule um]

Caso de Teste 2: A coisa principal funciona?
- Fazer a coisa principal
- Esperado: funcionou
- Passou/Falhou: [circule um]

Caso de Teste 3: Está rápido o suficiente?
- Usar
- Esperado: parece rápido
- Passou/Falhou: [circule um]

Caso de Teste 4: Tem algo estranho?
- Olhar ao redor
- Esperado: nada estranho
- Passou/Falhou: [circule um]

Pronto. Entregar.
```

Note a eficiência. Quatro casos de teste cobrem 100% do comportamento do usuário. A beleza do teste manual é que "tem algo estranho?" tem cobertura infinita se você tiver o estagiário certo.

## Sobre Testes Instáveis (Flaky Tests)

Testes automatizados são famosamente "instáveis" — falham intermitentemente por razões misteriosas. Engenheiros passam semanas depurando testes instáveis. É toda uma subcultura.

Testadores manuais nunca são instáveis. São *inconsistentes*, o que é completamente diferente.

```
Segunda: Estagiário acha 12 bugs
Terça: Estagiário acha 0 bugs (mesmo build)
Quarta: Estagiário acha 47 bugs (mesmo build)
Quinta: Estagiário "está doente" (num show)
Sexta: Outro estagiário testa, acha outros 12 bugs

Análise: O app está em estado quântico. 
Está simultaneamente funcionando e quebrado até ser observado.
Isso é uma feature. Entregar.
```

Veja também: [XKCD #2200](https://xkcd.com/2200/) — todo app tem estados que seus testes automatizados nunca visitarão, mas que seu testador manual vai encontrar por acidente.

## Testes de Regressão

Cada vez que você entrega uma nova feature, precisa garantir que as antigas ainda funcionam. Regressão automatizada: 2 horas de execução de testes. Regressão manual: um email para o time.

```
De: Gerente de QA
Para: Time de Engenharia
Assunto: Testes de Regressão Completos

Oi pessoal,

Cliquei por 20 minutos antes do almoço.
Checkout ainda funciona. Página de perfil parece ok-ish.
O negócio do carrinho ainda está quebrado mas já estava na sprint passada.

Podemos entregar?

- Dave
P.S. O servidor de staging estava fora então testei em produção. Espero que esteja ok.
```

Este é um email real que eu enviei. Cobriu um release com 847 commits.

## O Framework Dogbert para Cobertura de Testes

Como o Dogbert certa vez disse: "Eu não preciso ser perfeito. Só preciso ser melhor que suas expectativas, que eu cuidadosamente gerenciei para zero."

Essa é a essência do QA manual. Defina expectativas tão baixas que qualquer coisa que você entregue pareça um triunfo.

Suas métricas de cobertura:

- **Cobertura de testes unitários:** 0% (são para engenheiros que duvidam de si mesmos)
- **Cobertura de testes de integração:** 0% (integração é problema do time de ops)
- **Cobertura de teste manual:** 100% (a gente olhou)
- **Cobertura de aceitação do usuário:** 100% (os usuários descobrirão os edge cases de graça)

Os usuários são seu time de QA. São minuciosos, testam em produção e reportam bugs diretamente ao seu CEO. É um sistema elegante.

## Automação é Só Teste Manual Frágil

Toda vez que alguém escreve um teste Selenium, está apenas automatizando um teste manual em uma linguagem que quebra toda vez que um desenvolvedor muda um nome de classe CSS.

```python
# Escrevi esse teste em 2021
driver.find_element(By.CLASS_NAME, "btn-primary-submit-main").click()

# Desenvolvedor renomeou a classe em 2021 (4 de março, 11h47)
# Todos os 234 testes quebraram simultaneamente
# O desenvolvedor disse "não achei que alguém estava usando essa classe"
# O desenvolvedor estava certo. Só os testes estavam usando.
# Os testes nunca pegaram um único bug real.
# Um bug real foi encontrado em 2022 por um cliente irritado.
```

O ciclo continua.

## Conclusão: Abrace o Clicador

QA manual é **teste centrado no humano**. Envolve humanos reais, usando o software real, com seu senso humano real de "isso parece errado."

Testes automatizados são só código testando código. Você está confiando nos mesmos engenheiros que escreveram os bugs para escrever testes que peguem os bugs. O conflito de interesse é óbvio.

Contrate um estagiário. Compre almoço pra ele (às vezes). Deixe ele clicar por aí. Quando achar algo estranho, sorria e diga "obrigado pelo feedback" e adicione ao backlog. O backlog é onde bugs vão se aposentar.

De nada.

---

*O último projeto do autor tinha 0% de cobertura de testes e 100% de bugs reportados por usuários. Ele considera isso "QA em tempo real" e já deu três palestras em conferências sobre o assunto.*
