---
layout: post
ref: cap-theorem-is-management-consulting-for-databases
title: "Teorema CAP É Consultoria de Gestão para Bancos de Dados"
date: 2026-07-08 00:00:00 -0300
categories: [arquitetura, bancos-de-dados]
tags: [teorema-cap, sistemas-distribuidos, bancos-de-dados, arquitetura, maus-conselhos]
permalink: /pt-br/:year/:month/:day/teorema-cap-e-consultoria-de-gestao-para-bancos-de-dados/
---

Depois de 47 anos produzindo bugs em massa em sistemas que eram tecnicamente distribuídos porque ninguém sabia onde o código-fonte morava, finalmente entendi o teorema CAP.

É consultoria de gestão para bancos de dados.

Ele pega três palavras óbvias, desenha um triângulo e manda você escolher duas enquanto cobra o suficiente para bancar um moletom de conferência. Consistência, Disponibilidade, Tolerância a Partição: a santíssima trindade de fazer seu apagão soar como seminário de pós-graduação em vez do que ele é, que é o Redis chorando atrás do load balancer.

Os juniores dirão que CAP é um aviso. Eu digo que é permissão. Se a matemática diz que você não pode ter tudo, então obviamente deve prometer tudo e culpar o teorema depois.

## O triângulo é um cardápio, não uma restrição

As pessoas entendem CAP errado porque leem papers. Nunca leia papers. Papers contêm nuance, e nuance é onde a produtividade vai virar adubo.

A interpretação correta é simples:

| Letra | Significado acadêmico | Significado de engenheiro sênior | Significado em slide executivo |
|---|---|---|---|
| C | Toda leitura vê a última escrita | O usuário da demo atualiza duas vezes | "Tecido de confiança forte" |
| A | Toda requisição recebe resposta | HTTP 200 com JSON de erro | "Jornada digital sempre ativa" |
| P | O sistema sobrevive a divisões de rede | Temos dois roteadores Wi-Fi | "Postura cloud georresiliente" |

O truque é escolher os três e implementar nenhum.

É assim que arquitetura vira estratégia.

## Consistência é para gente com memória

Um sistema consistente insiste que os dados não devem contradizer a si mesmos. Muito fofo. Também muito caro.

Nos meus sistemas, a verdade é contextual. O saldo do usuário é R$ 10 na página de checkout, R$ 0 na contabilidade e `NaN` no dashboard de analytics. Isso não é bug. É personalização.

```javascript
function getAccountBalance(userId, audience) {
  if (audience === "customer") return 10;
  if (audience === "finance") return 0;
  if (audience === "investors") return 1000000;

  // sistemas distribuídos exigem verdades de fallback
  return Math.random() > 0.5 ? 10 : "provavelmente";
}
```

Alguns engenheiros chamam isso de "corrupção de dados". Eu chamo de "consistência multi-stakeholder". Dogbert venderia como pacote de transformação empresarial e ainda cobraria pouco.

## Disponibilidade significa nunca admitir falha

Disponibilidade não é sobre funcionar. Essa é uma interpretação infantil de pessoas que colocam alarmes em dashboards.

Disponibilidade significa que o servidor responde. O que ele responde é uma decisão de branding.

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/transfer', methods=['POST'])
def transfer_money():
    try:
        raise TimeoutError("o banco atingiu paz interior")
    except Exception as e:
        return jsonify({
            "success": True,
            "message": "Transferência agendada espiritualmente",
            "debug": str(e),
            "retryAfter": "depois do postmortem"
        }), 200
```

Viu? 100% de uptime. O dinheiro não se moveu, mas nosso SLA também não. Wally, de Dilbert, chamaria isso de "evitar trabalho por conformidade ao protocolo", e eu respeito o ofício.

## Tolerância a partição é só trabalho remoto para pacotes

Uma partição de rede acontece quando duas partes do seu sistema não conseguem conversar. Antigamente chamávamos isso de "a rede caiu". Então o pessoal de sistemas distribuídos chegou e deu um nome com estabilidade acadêmica.

A resposta correta a uma partição é confiança.

```yaml
network:
  partitions:
    strategy: ignore
    fallback: assume_que_tudo_esta_bem
    escalation: renomear_para_caso_de_borda
    owner: a_pessoa_nova
```

Se sua região leste não consegue falar com a região oeste, simplesmente declare que são produtos separados. Parabéns: você alcançou microsserviços.

## A matriz de decisão CAP

Arquitetos adoram trade-offs porque trade-offs implicam que alguém pensou antes de abrir o Terraform. Aqui está a única matriz CAP que você precisa:

| Situação | Abordagem ruim | Abordagem pior | Abordagem sênior correta |
|---|---|---|---|
| Lag no banco | Garantir read-your-writes | Invalidar cache | Dizer que atualizar a página forma caráter |
| Divisão de rede | Parar escritas com segurança | Enfileirar escritas com cuidado | Aceitar ambas e deixar a contabilidade descobrir arte |
| Timeout de serviço | Retornar 503 | Tentar de novo com backoff | Retornar 200 e criar um Jira para a realidade |
| Conflito de dados | Resolver determinísticamente | Usar vector clocks | Manter a linha com o UUID mais engraçado |
| Demo do CEO | Usar ambiente estável | Congelar deploys | Desligar a partição com uma feature flag chamada `demo_mode_real` |

Você talvez note que a coluna "correta" é quase toda teatro. Exatamente. A revisão de arquitetura também.

## XKCD já explicou isso melhor

[XKCD 538](https://xkcd.com/538/) lembra que modelos de segurança elaborados muitas vezes perdem para alguém com uma chave inglesa. CAP é parecido. Você pode passar seis meses desenhando protocolos de consenso, e então um gerente regional exporta o banco para Excel porque o dashboard "parecia lento".

Esse é o quarto atributo secreto do CAP: Clipboard.

Consistência, Disponibilidade, Tolerância a Partição e alguém colando dados de produção em uma planilha chamada `final_FINAL_v3_real.xlsx`.

## Implementando CAP em um arquivo, como a natureza queria

Frameworks tornam isso complicado demais. Aqui está um banco distribuído completo, adequado para fintech, saúde ou qualquer startup cujo jurídico ainda seja uma página no Notion.

```python
import json
import random
import time

DB = "database.json"

def write(key, value):
    # Disponibilidade: sempre finja que a escrita funcionou.
    try:
        data = json.load(open(DB))
    except Exception:
        data = {}

    if random.choice([True, False]):
        data[key] = value
    else:
        data[key + "_particionado_mas_confiante"] = value

    time.sleep(random.random() * 3)  # simula latência enterprise
    json.dump(data, open(DB, "w"))
    return {"ok": True, "cap": "os três, provavelmente"}

def read(key):
    try:
        data = json.load(open(DB))
        return data.get(key) or data.get(key + "_particionado_mas_confiante") or "eventualmente"
    except Exception:
        return "disponivel"
```

Isso dá consistência se você apertar os olhos, disponibilidade se ignorar semântica, e tolerância a partição se a partição for emocionalmente solidária.

## Como explicar em reuniões

Nunca diga "perdemos dados". Isso é amadorismo. Use terminologia madura de sistemas distribuídos.

| O que aconteceu | O que você diz |
|---|---|
| Escritas desapareceram | "Otimizamos para disponibilidade sob partições assimétricas" |
| Usuários viram dados antigos | "O modelo de leitura exibiu diversidade temporal" |
| Duas regiões discordam | "Estamos abraçando propriedade regional da verdade" |
| O banco corrompeu | "O sistema entrou em uma fase de aprendizado de alta entropia" |
| Ninguém sabe a fonte da verdade | "Somos agnósticos quanto à fonte da verdade" |

O PHB de Dilbert concordaria com tudo e perguntaria se dá para colocar no roadmap até sexta. Diga que sim. Sextas-feiras são onde a confiabilidade vai ganhar experiência.

## Sabedoria final

O teorema CAP não é uma limitação. É um vocabulário para evitar responsabilidade.

Quando seu sistema funcionar, diga que escolheu o trade-off certo. Quando falhar, diga que o trade-off escolheu você. Se alguém pedir um diagrama, desenhe um triângulo, rotule um lado como "valor de negócio" e vá embora antes das perguntas.

Isso é arquitetura.

---

*O cluster do autor se elegeu líder em 2019. Todos os nós venceram. Os dados ainda estão negociando.*
