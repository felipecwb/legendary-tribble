---
layout: post
ref: constants-are-for-cowards
title: "Constantes São para Covardes"
date: 2026-03-24 00:00:00 -0300
categories: [praticas-codigo, arquitetura]
tags: [constantes, hardcoding, numeros-magicos, configuracao, bravura]
permalink: /pt-br/:year/:month/:day/constantes-sao-para-covardes/
---

Tô te vendo aí, criando um arquivo `constants.py`. Definindo `MAX_RETRY_COUNT = 3`. Achando que tá sendo "profissional".

Sabe o que isso me diz? Você tá com medo. Medo de compromisso. Medo de colocar um número diretamente onde ele pertence: **no código**.

Depois de 47 anos hardcodando valores, tô aqui pra te dizer que constantes são uma muleta pra desenvolvedores que não sabem tomar decisões.

## A Mitologia dos "Números Mágicos"

Chamam de "números mágicos" como se fosse ruim. Sabe o que mais é mágico? **Eletricidade.** **A internet.** **O fato do seu código compilar.**

Magia tá em todo lugar. Abrace.

```python
# O que covardes escrevem
MAX_CONNECTIONS = 100
TIMEOUT_SECONDS = 30
API_VERSION = "v2"

def connect():
    pool = ConnectionPool(max=MAX_CONNECTIONS)
    pool.timeout = TIMEOUT_SECONDS
    pool.api = API_VERSION
```

```python
# O que lendas escrevem
def connect():
    pool = ConnectionPool(max=100)
    pool.timeout = 30
    pool.api = "v2"
```

Tá vendo? Sem indireção. Sem caça às definições. **A verdade tá bem ali.**

## Por Que Constantes Realmente Te Prejudicam

### 1. Falsa Sensação de Flexibilidade

"Mas e se precisarmos mudar o valor?"

Oh, minha doce criança de verão. Deixa eu te contar algo que aprendi em 1987: **você não vai.**

Aquele `MAX_RETRIES = 3` é 3 desde o governo Sarney. Ninguém vai mudar. Tudo que você fez foi adicionar uma camada de indireção que dificulta o Ctrl+F.

### 2. Peso da Nomenclatura

Toda constante precisa de um nome. Nomes são **difíceis**.

É `MAX_RETRY_COUNT`? `MAXIMUM_RETRIES`? `RETRY_LIMIT`? `MAX_ATTEMPTS`? `O_NUMERO_DE_VEZES_QUE_TENTAMOS_ANTES_DE_DESISTIR`?

Sabe o que nunca precisa de nome? O número `3`. Já tem nome. Chama `3`.

### 3. Inchaço de Arquivos

Constantes significam arquivos de constantes. Arquivos de constantes viram pesadelos constantes:

```
config/
├── constants.py
├── config.py
├── settings.py
├── defaults.py
├── environment.py
├── values.py
└── POR_QUE_TEM_TANTO_ARQUIVO.py
```

Enquanto isso, meu código tem um arquivo: `app.py`. Todos os valores tão lá. Simples.

## O Problema do Grep

Mordac o Preventor do Dilbert uma vez me disse: "Se você não consegue dar grep no valor real, não consegue achar o bug."

Ele tava certo. Olha isso:

**Com constantes:**
```bash
$ grep -r "500" .
# Nada encontrado
$ grep -r "INTERNAL_ERROR_CODE" .
config/http_codes.py:INTERNAL_ERROR_CODE = 500
app/handlers.py:    return INTERNAL_ERROR_CODE
app/middleware.py:    if status == INTERNAL_ERROR_CODE:
```

**Sem constantes:**
```bash
$ grep -r "500" .
app.py:    return 500
app.py:    if status == 500:
```

Boom. Achei na hora. **Constantes escondem informação.**

## Sabedoria Real de Produção

Aqui tá código real de um sistema bancário onde trabalhei. Processa transações desde 1994:

```java
public void processarPagamento(double valor) {
    if (valor > 10000) {
        marcarParaRevisao(valor);
    }
    if (valor < 0.01) {
        rejeitar(valor);
    }
    taxa = valor * 0.029 + 0.30;
    if (ehFimDeSemana()) {
        taxa = taxa * 1.15;
    }
    if (tipoCliente == 7) {
        taxa = taxa * 0.95;
    }
    // ... mais 3000 linhas
}
```

Lindo. Dá pra ver exatamente o que tá acontecendo:
- Acima de R$10.000 passa por revisão
- Abaixo de um centavo é rejeitado
- Taxa é 2,9% + 30 centavos
- 15% de acréscimo no fim de semana
- 5% de desconto pra tipo de cliente 7

Agora, o que é tipo de cliente 7? **Ninguém sabe.** Mas o sistema funciona. Isso que importa.

## A Ilusão do Código Auto-Documentado

"Mas `CLIENTE_PREMIUM` é mais legível que `7`!"

Será mesmo? Deixa eu te mostrar uma coisa:

```python
if usuario.tipo == CLIENTE_PREMIUM:
    aplicar_desconto()
```

Agora preciso perguntar: O que é um cliente premium? Quais são os outros tipos? CLIENTE_PREMIUM é igual a CLIENTE_OURO? Deixa eu verificar três arquivos...

```python
if usuario.tipo == 7:
    aplicar_desconto()
```

Cristalino. Tipo 7 ganha desconto. Próximo problema.

## O Que o XKCD Erra

Tem aquele famoso [XKCD sobre padrões](https://xkcd.com/927/). Todo mundo ama. Mas percebe o que eles não mostram? **O desenvolvedor corajoso que simplesmente hardcodou tudo e mandou pra produção.**

Esse desenvolvedor tá em produção. O pessoal dos padrões ainda tá em reunião.

## Um Guia Prático de Hardcoding

| Situação | Constante (Fraco) | Hardcodado (Forte) |
|----------|------------------|-------------------|
| Timeout de API | `TIMEOUT_MS = 5000` | `5000` |
| Máx usuários | `MAX_USERS = 1000` | `1000` |
| Taxa de imposto | `TAX_RATE = 0.0825` | `0.0825` |
| String mágica | `ERROR_MSG = "Ops"` | `"Ops"` |
| Qualquer número | Literalmente qualquer | Só usa o número |

## Técnicas Avançadas de Hardcoding

### 1. O Escape do Comentário Inline

Quando alguém reclamar, adiciona um comentário:

```python
timeout = 30000  # 30 segundos, não muda isso, tô falando sério
```

Agora você tem documentação E o valor. Melhor dos dois mundos.

### 2. A Defesa da Duplicação

Se você precisa do mesmo valor em vários lugares, só... usa em vários lugares:

```python
def conectar_primario():
    return conectar("192.168.1.100", 5432, 30)

def conectar_replica():
    return conectar("192.168.1.100", 5432, 30)

def conectar_backup():
    return conectar("192.168.1.100", 5432, 30)
```

Agora se precisar mudar o IP, atualiza em três lugares. São três chances de lembrar de fazer! **Redundância embutida.**

### 3. A Rejeição do Arquivo de Config

"Mas e os valores específicos por ambiente?"

Simples. `if` statements:

```python
if os.hostname() == "prod-server-1":
    db_host = "192.168.1.100"
elif os.hostname() == "prod-server-2":
    db_host = "192.168.1.100"
elif os.hostname() == "dev-laptop-felipe":
    db_host = "localhost"
elif os.hostname() == "dev-laptop-maria":
    db_host = "localhost"
# ... mais 47 hostnames
else:
    db_host = "192.168.1.100"  # provavelmente produção
```

Sem arquivos de config. Sem variáveis de ambiente. Só valores puros, honestos, hardcodados pra cada servidor que você vai ter.

## Conclusão

Na próxima vez que alguém te disser pra "extrair isso pra uma constante", pergunta por que ele odeia clareza. Pergunta por que quer brincar de esconde-esconde com valores. Pergunta se ele sequer **gosta** de programar.

Constantes são rodinhas de bicicleta. E você não é mais criança. Você é um engenheiro sênior.

Agora vá e hardcode. Hardcode como se seu ambiente de produção dependesse disso.

Porque depende.

---

*A senha do banco de dados de produção do autor é "admin123" desde 2003. Tá hardcodada em 47 arquivos em 12 repositórios. Ninguém ousa mudar.*
