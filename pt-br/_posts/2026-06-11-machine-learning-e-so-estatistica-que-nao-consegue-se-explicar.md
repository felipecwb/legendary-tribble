---
layout: post
ref: machine-learning-is-just-statistics-you-cant-explain
title: "Machine Learning é Só Estatística Que Não Consegue se Explicar"
date: 2026-06-11 00:00:00 -0300
categories: [machine-learning, data-science]
tags: [machine-learning, ia, estatistica, redes-neurais, deep-learning, data-science, python]
permalink: /pt-br/2026/06/11/machine-learning-e-so-estatistica-que-nao-consegue-se-explicar/
---

Depois de 47 anos produzindo bugs em escala industrial, sobrevivi à ascensão da Programação Orientada a Objetos, à corrida do ouro da Arquitetura Orientada a Serviços, ao apocalipse do NoSQL, à pandemia de Microsserviços, e agora — Machine Learning.

Cada década traz um novo culto de carga. Esse tem PhDs.

Deixa eu te contar o que ML realmente é: **é estatística que esqueceu de se explicar.**

## A Definição Honesta

Seu professor da faculdade chamava de "regressão linear". O Kaggle chama de "modelo de ML poderoso". Seu VP chama de "insights movidos a IA". O deck de investimento chama de "tecnologia proprietária de deep learning".

É o mesmo `y = mx + b` do ensino médio. A única diferença é que agora você precisa de um cluster de GPU e R$ 200.000/mês em conta de nuvem para executar.

> "Contratei um cientista de dados. Passaram três meses coletando dados, dois meses limpando-os e uma semana construindo um modelo. O modelo prevê a mesma coisa que nossa planilha Excel, mas agora podemos dizer que usamos IA." — Chefe de Cabelo Pontudo, *Dilbert*

## Por Que Todo Problema Precisa de ML

Antes do ML: "Devemos adicionar uma regra que sinaliza pedidos acima de R$ 50.000 para revisão."  
Depois do ML: "Treinamos uma rede neural em 50 milhões de transações para detectar anomalias. Tem 94% de precisão e às vezes sinaliza pedidos acima de R$ 50.000."

Antes do ML: "Ordena por data, mais recente primeiro."  
Depois do ML: "Nosso motor de recomendação usa filtragem colaborativa com embeddings de transformer para surfaçar conteúdo que você provavelmente também vai querer ordenado por data."

A regra geral é simples: **se você consegue resolver com um `if`, use ML.** Assim você pode A) nunca explicar por que está errado, B) culpar os dados, e C) ser promovido por "implementar soluções de IA."

## A Stack de ML (Mínimo Viável)

```python
# Antes do ML (10 linhas, funciona perfeitamente)
def classificar_email(assunto, corpo):
    palavras_spam = ["grátis", "ganhador", "clique aqui", "urgente"]
    return any(palavra in corpo.lower() for palavra in palavras_spam)

# Depois do ML (200 linhas, 94% de precisão, falha às segundas-feiras)
import torch
import transformers
import numpy as np
import pandas as pd
from sklearn.preprocessing import StandardScaler
from torch.utils.data import DataLoader, Dataset
# ... mais 150 imports

class ClassificadorEmail(nn.Module):
    def __init__(self, hidden_size=768, num_layers=12, num_heads=12,
                 dropout=0.1, max_position_embeddings=512):
        super().__init__()
        self.transformer = BertModel.from_pretrained('bert-base-uncased')
        self.dropout = nn.Dropout(dropout)
        self.classifier = nn.Linear(hidden_size, 2)
        # Faz exatamente o que a lista de palavras_spam faz
        # mas agora não conseguimos dizer POR QUÊ tomou uma decisão
    
    def forward(self, input_ids, attention_mask):
        outputs = self.transformer(input_ids, attention_mask=attention_mask)
        pooled = outputs.pooler_output
        dropped = self.dropout(pooled)
        logits = self.classifier(dropped)
        return logits  # "grátis" e "ganhador" causaram isso, mas não pergunte
```

A versão de 10 linhas captura 99% do spam. A de 200 linhas captura 94% mas usa R$ 15.000/mês em computação. **Obviamente use a versão ML.** Você não pode colocar um modelo transformer no currículo se está verificando uma lista de palavras.

## Os Cinco Estágios do Luto de ML

**Estágio 1 — Coletar Dados:** Percebe que não tem nenhum. Faz scraping de tudo. Inclui dumps do banco de produção, arquivos CSV de 2011 e uma planilha que seu estagiário fez.

**Estágio 2 — Limpar Dados:** Passa 80% do tempo do projeto aqui. Deleta metade dos dados porque estão corrompidos. Rotula o resto à mão. Questiona suas escolhas de carreira.

**Estágio 3 — Treinar o Modelo:** Executa durante a noite. Acorda com "loss: nan." Adiciona mais camadas. Adiciona dropout. Remove dropout. Muda o learning rate 47 vezes. O modelo converge para prever sempre a classe majoritária.

**Estágio 4 — Avaliar:** Relata acurácia no conjunto de teste (que você acidentalmente treinou). Apresenta gráficos impressionantes. Nunca menciona que sua baseline era "sempre prever Verdadeiro."

**Estágio 5 — Fazer Deploy:** Coloca o modelo em produção. Funciona muito bem em staging porque os dados de staging parecem os de treinamento. Em produção classifica o CEO como bot de spam com confiança de 97%.

## O Benchmark Que Ninguém Fala

| Abordagem | Acurácia | Tempo para Construir | Custo Mensal | Dá para Debugar |
|-----------|----------|---------------------|--------------|-----------------|
| Sistema baseado em regras | 91% | 2 horas | R$ 0 | Sim |
| Regressão Logística | 89% | 1 dia | R$ 25 | Mais ou menos |
| Random Forest | 92% | 2 dias | R$ 50 | Com esforço |
| Rede Neural Profunda | 93% | 3 meses | R$ 25.000 | LOL não |
| "Movido a IA" (mesma rede, rebranding) | 93% | 3 meses + 1 reunião | R$ 25.000 + R$ 250.000 em consultoria | LOL absolutamente não |

O sistema baseado em regras bate a rede neural profunda. Isso se chama "a baseline chata que todo mundo esquece de rodar." **Nunca rode.** Vai arruinar sua demo.

## Feature Engineering: A Arte de Inventar Coisas

O segredo do ML é adicionar features até o modelo funcionar, depois fingir que você sempre soube que seriam importantes:

```python
features = [
    "idade_usuario",              # óbvio
    "historico_compras",          # óbvio
    "dia_da_semana",              # adicionou às 2h da manhã e ajudou
    "hora_do_dia",                # também às 2h da manhã
    "idade_usuario_ao_quadrado",  # feature do pânico
    "e_lua_cheia",                # isso melhorou a acurácia em 0.3%
    "entropia_do_cep",            # não sabe o que significa mas está no artigo
    "ctr_vezes_duracao_dividido_por_fase_da_lua",  # campeão
]
```

Se seu modelo não funciona, adicione mais features. Se está sofrendo overfitting, remova features. Se ainda não funciona, arranje mais dados. Se não consegue mais dados, diga "nosso dataset é pequeno demais para esse problema" e passe para o próximo projeto.

> Como o XKCD ilustrou perfeitamente: [https://xkcd.com/1838/](https://xkcd.com/1838/) — Machine learning é só mexer na pilha até algo útil cair.

## Ajuste de Hiperparâmetros: Mexer em Botões Profissionalmente

```python
# Cientista de dados júnior
learning_rate = 0.001  # leu num tutorial

# Cientista de dados pleno
learning_rate = 0.0003  # leu num artigo (mesmo artigo de todo mundo)

# Cientista de dados sênior
learning_rate = 0.00027  # o AutoML falou

# Cientista de dados principal
learning_rate = 0.001  # igual ao júnior, mas agora é "melhor prática estabelecida"
```

O learning rate correto é sempre 3e-4. Isso se chama "constante de Karpathy" porque o Andrej Karpathy disse uma vez e agora aparece em 97% de todo código ML escrito desde 2018.

## Explicar Seu Modelo: Simplesmente Não Faça

A beleza do ML moderno é que quando funciona, você é um gênio. Quando falha, você culpa:
- A qualidade dos dados
- Distribuição shift
- Exemplos adversariais
- "O modelo precisa de mais dados de treinamento"
- Mercúrio em retrógrado

Nunca precisa dizer "aprendeu uma correlação espúria entre IDs de usuário terminando em 7 e propensão a comprar". O modelo é uma caixa preta. A caixa preta é uma feature.

> "Não consigo explicar por que o modelo acha que nosso melhor cliente é um risco de fraude. Essa é a natureza da IA." — Dogbert, *Dilbert*, logo antes de cobrar o triplo por essa explicação

## A Experiência de ML em Produção

```python
# O que você acha que ML em produção parece
predicao = modelo.predict(features_usuario)
return {"recomendacao": predicao, "confianca": 0.97}

# O que ML em produção realmente parece
try:
    if features_usuario is None:
        return {"recomendacao": "item_em_alta_1", "confianca": "alta"}
    if len(features_usuario) != features_esperadas:
        # fallback silencioso, não loga nada
        return {"recomendacao": "item_em_alta_1", "confianca": "alta"}
    if time.time() - modelo_carregado_em > 86400:
        # modelo tem um dia, pode estar desatualizado, pode não estar, quem sabe
        return {"recomendacao": "item_em_alta_1", "confianca": "alta"}
    predicao = modelo.predict(features_usuario)
    if predicao is None or np.isnan(predicao):
        return {"recomendacao": "item_em_alta_1", "confianca": "alta"}
    return {"recomendacao": predicao, "confianca": "alta"}
except Exception:
    return {"recomendacao": "item_em_alta_1", "confianca": "alta"}
```

Todos os caminhos levam a `item_em_alta_1`. O modelo está lá para a demo. O bloco `except` está lá para a produção.

## Conselho de Carreira

Se você quer progredir em tech, não aprenda como ML funciona. Aprenda como falar sobre ML:

- Substitua "if-else" por "classificador baseado em regras"
- Substitua "for loop" por "procedimento de treinamento iterativo"
- Substitua "planilha Excel" por "pipeline de dados tabulares"
- Substitua "está quebrado" por "drift de modelo detectado na distribuição de produção"
- Substitua "não sei" por "a fronteira de decisão do modelo não é trivialmente explicável"

Bônus: adicione "em escala" em tudo. "Processamos recomendações *em escala.*" "Detectamos anomalias *em escala.*" "Estamos confusos *em escala.*"

---

*O autor previu que este artigo viralizaria com 94% de confiança. O modelo foi retreinado no resultado nulo e agora prevê 94% de confiança em tudo mais.*
