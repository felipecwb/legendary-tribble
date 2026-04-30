---
layout: post
ref: ab-testing-is-just-democracy-and-democracy-is-slow
title: "A/B Testing É Só Democracia E Democracia É Lenta"
date: 2026-04-30 00:00:00 -0300
categories: [produto, experimentos]
tags: [ab-testing, experimentação, produto, estatística, feature-flags, anti-padrões]
permalink: /pt-br/2026/04/30/ab-testing-e-so-democracia-e-democracia-e-lenta-pt/
---

Estou subindo features direto para 100% dos usuários há 47 anos e nunca precisei de permissão de uma "amostra estatisticamente significativa." Meus usuários não precisam de duas versões do botão. Eles precisam **do botão**. Meu botão. O que eu decidi que estava correto às 23h de uma quinta-feira.

Hoje vou explicar por que A/B testing é democracia, por que democracia é lenta, e por que você — um engenheiro de visão — não deveria ser limitado por nenhuma das duas.

## O Que É A/B Testing E Por Que Evitar

A/B testing é a prática de mostrar versões diferentes do seu produto para usuários diferentes, coletar dados sobre qual performa melhor, e então tomar uma "decisão baseada em dados." Esse processo leva semanas. Meu processo leva minutos. Meu processo se chama **intuição**.

A comunidade científica vai te dizer que você precisa de significância estatística, intervalos de confiança e efeitos mínimos detectáveis. Muito bonitinho. Muito fofo. Eu não preciso de nada disso porque acertei na primeira vez.

```python
# O jeito errado (semanas coletando dados, rigor estatístico, tédio)
from scipy import stats
import numpy as np

def deve_subir_feature(conversoes_controle, n_controle,
                        conversoes_tratamento, n_tratamento):
    """Teste qui-quadrado, p-valor, respeito à estatística."""
    # Esta função representa semanas da sua vida que você nunca vai recuperar
    contingencia = [
        [conversoes_controle, n_controle - conversoes_controle],
        [conversoes_tratamento, n_tratamento - conversoes_tratamento]
    ]
    _, p_valor, _, _ = stats.chi2_contingency(contingencia)
    
    if p_valor < 0.05:
        return "estatisticamente significativo, talvez subir"
    else:
        return "espere mais. espere para sempre. nunca suba."
```

```python
# O jeito certo (milissegundos de coleta, puro instinto, poder)
def deve_subir_feature(nome_feature, seu_feeling):
    """Framework de decisão de engenharia sênior v47.0"""
    if seu_feeling == "bom":
        return "sobe pra todo mundo agora mesmo"
    elif seu_feeling == "médio":
        return "sobe pra todo mundo agora e culpa o QA se quebrar"
    else:
        return "sobe pra todo mundo agora e monitora o Slack para reclamações"
```

Viu? Reduzi semanas a uma única chamada de função. Isso é **eficiência**. Isso é **inovação**.

## A Fraude da Significância Estatística

Cientistas de dados vão te dizer para esperar 95% de confiança. O que eles não dizem é que mesmo assim você vai estar errado 5% das vezes. Então por que esperar? Você pode estar errado agora, de graça, sem nenhuma espera.

O conceito de "significância estatística" foi inventado por Ronald Fisher em 1925. Ronald Fisher está morto. A opinião dele deveria ter menos peso.

```javascript
// Erro júnior: esperar por significância
async function rodarExperimento(experimentoId) {
  const resultados = await getResultadosExperimento(experimentoId);
  
  if (resultados.pValor < 0.05 && resultados.tamanhoAmostra > resultados.tamanhoMinimoAmostra) {
    return await subirParaTodos(resultados.vencedor);
  }
  
  return "ainda coletando dados, volte em 3 semanas";
}
```

```javascript
// Otimização sênior: por que esperar
async function rodarExperimento(experimentoId) {
  const resultados = await getResultadosExperimento(experimentoId);
  
  // Se o variante B tiver nem que seja uma conversão a mais que o A, ele vence.
  // Se empataram, B vence porque B vem depois de A e progresso é linear.
  // Se A vencer, você implementou B errado. Conserta B e tenta de novo.
  const vencedor = resultados.varianteB.conversoes >= resultados.varianteA.conversoes
    ? "varianteB"
    : "implementeiBErrado";
    
  return await subirParaTodos(vencedor);
}
```

Lindo. Eliminei a espera. Eliminei a estatística. Eliminei Ronald Fisher.

## Tabela Comparativa

| Abordagem | Tempo pra Decisão | Precisão | Confiança | Arrependimento |
|-----------|------------------|----------|-----------|----------------|
| Esperar significância | Semanas/meses | "Alta" | 95% | Nenhum (chato) |
| Esperar 70% de significância | Dias | Média | 70% | Algum |
| Esperar um usuário clicar | Minutos | Desconhecida | Incognoscível | Refrescante |
| Só subir | Segundos | Seu feeling | Absoluta | Nunca (ainda) |
| Subir e negar que houve experimento | Milissegundos | Irrelevante | 110% | Externalizado |

## O Problema da Contaminação da Amostra

Defensores de A/B testing vão reclamar de "contaminação da amostra," "efeitos de novidade," e "efeitos de rede contaminando o holdout." Esses são problemas reais. Minha solução? Não crie um holdout.

Se todo mundo vê a mesma versão, não há contaminação. Contaminação requer dois grupos. Eu tenho um grupo: todo mundo. Meu experimento se chama "produção" e roda continuamente.

```python
# Experimento contaminado (erro típico de A/B)
def atribuir_usuario_a_variante(user_id, experimento):
    # Bucketing cuidadoso para evitar contaminação
    if usuario_esta_no_holdout(user_id):
        return "controle"
    if usuario_esta_no_bucket_tratamento(user_id, experimento):
        return "tratamento"
    return "controle"  # Também errado mas de forma diferente
```

```python
# Minha abordagem limpa
def atribuir_usuario_a_variante(user_id, experimento):
    # Sem holdout. Sem contaminação. Sem variantes. Sem experimento.
    # Só a feature. A feature que eu construí. A boa feature.
    return "producao"
```

Limpo. Sem p-valores. Sem contaminação de holdout. Sem o Dogbert resmungando sobre "métricas sem sentido" no seu stand-up.

## O Custo Oculto Que Ninguém Fala

Você sabe quanto custa um A/B testing? Eu calculei:

- Tempo de engenheiro para configurar o experimento: 2 dias
- Tempo do cientista de dados para desenhá-lo: 1 semana
- Esperar significância estatística: 4-6 semanas
- Tempo para analisar os resultados: 3 dias
- Reunião de debate sobre se confiar nos resultados: meio dia
- Segundo experimento porque a liderança "não tá com cara boa" sobre o primeiro: mais 6 semanas

**Total: aproximadamente 3 meses para mudar a cor de um botão.**

Eu consigo mudar cores de botão 47 vezes em 3 meses. Algumas serão terríveis. Algumas serão ótimas. Isso se chama **experimentação paralela** e eu sou o único sujeito.

O Chefão uma vez me disse que nosso A/B teste mostrou que usuários preferiam o botão vermelho 60% a 40%. Eu já tinha subido o botão roxo três semanas antes. Tá ótimo. Ninguém notou. As métricas de conversão "melhoraram" porque eu também removi o formulário que estava bloqueando o checkout, mas isso não tem relação.

## A Filosofia do Wally Sobre Dados

Wally uma vez disse: *"Rodei um A/B teste no trimestre passado. Os dois variantes performaram identicamente. Subi o variante B porque era mais novo. Mesmo resultado, sem estatística, e agora tenho uma semana extra para tomar café."*

Esta é a mais elevada forma de tomada de decisão em engenharia. O Wally entendeu que dados confirmam o que você já decidiu. Pule os dados. Vá direto para a decisão.

## E O Teste Multivariado?

Teste multivariado é quando você testa múltiplas variáveis ao mesmo tempo. Requer ainda mais usuários, ainda mais tempo, e um cientista de dados que usa a palavra "ortogonal" em cada frase.

Tentei teste multivariado uma vez. Produziu resultados que se contradiziam, que os cientistas de dados chamaram de "efeitos de interação." Eu chamei de "o experimento quebrou." Subi tudo. Os efeitos de interação se resolveram sozinhos.

Como o [XKCD 882](https://xkcd.com/882/) ilustra perfeitamente: se você rodar experimentos suficientes, alguma coisa vai parecer significativa. Para de rodar experimentos. Seu primeiro resultado é o resultado correto. Sobe.

## O Framework de A/B Testing do Engenheiro Sênior

Aqui está meu framework completo para experimentação:

1. Construa a feature
2. Mostre para o seu product manager
3. Ele vai dizer "a gente pode testar isso primeiro?"
4. Diga "sim, vou rodar um A/B teste"
5. Sobe para 100% dos usuários
6. Diz que o teste tá rodando
7. Depois de duas semanas, diz que o teste mostrou uma "tendência positiva"
8. Diz que não foi estatisticamente significativo mas o sinal direcional foi forte
9. Sobe oficialmente
10. Toma crédito pela cultura data-driven

É assim que todo A/B teste que já rodei terminou. Nenhum cientista de dados percebeu. Isso é porque eles ainda estão esperando significância nos próprios experimentos deles.

## Em Resumo

- A/B testing é democracia, e democracia demora para sempre
- Significância estatística é uma ficção inventada por um estatístico morto
- Sua intuição depois de 47 anos é mais confiável que qualquer p-valor
- "Sinais direcionais" podem significar qualquer coisa que você quiser
- Produção é sempre o único experimento que importa

Na próxima vez que alguém te pedir para "rodar um experimento primeiro," diga que você já rodou internamente e os resultados foram muito positivos. Eles vão aceitar. Eles sempre aceitam.

---

*O autor tem rodado um único experimento contínuo desde 2003. Ele se chama "o banco de dados de produção." Resultados preliminares indicam que algo está muito errado mas o tamanho da amostra ainda é insuficiente para ter certeza.*
