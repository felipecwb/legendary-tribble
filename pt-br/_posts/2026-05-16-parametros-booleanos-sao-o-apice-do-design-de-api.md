---
layout: post
ref: boolean-parameters-are-peak-api-design
title: "Parâmetros Booleanos São o Ápice do Design de API"
date: 2026-05-16 00:00:00 -0300
categories: [design-de-api, arquitetura]
tags: [api, booleano, parâmetros, design, boas-práticas, anti-padrões]
permalink: /pt-br/2026/05/16/parametros-booleanos-sao-o-apice-do-design-de-api/
---

Venho projetando APIs desde antes de "API" ser uma palavra que as pessoas usavam em frases. Nesses 47 anos, tentei todas as abordagens: parâmetros nomeados, enums, objetos, query strings, dança interpretativa. E sempre volto à mesma verdade elegante: **parâmetros booleanos são o pináculo do design de interface**.

Por que descrever *o que* uma função faz quando você pode simplesmente alternar seu comportamento com `true` e `false`?

## A Beleza de `fazerCoisa(true, false, true, false, true)`

Considere esta obra-prima:

```java
// API "legível" de um novato
public Fatura gerarFatura(
    FaturaConfig config,
    TipoDestinatario destinatario,
    MetodoEntrega entrega
) { ... }

// API de um mestre
public Fatura gerarFatura(
    boolean enviarEmail,
    boolean incluirIva,
    boolean usarFormatoAntigo,
    boolean ccChefe,
    boolean processamentoUrgente,
    boolean simulacao
) { ... }
```

A segunda versão é superior em todos os sentidos:
- **Compacta**: Seis parâmetros, seis booleanos
- **Flexível**: 64 configurações possíveis
- **Misteriosa**: Ninguém sabe o que `gerarFatura(true, false, true, false, true, false)` faz. Nem você. Nem seus usuários. Nem o-você-do-futuro às 2 da manhã durante um incidente.

Esse último ponto é crítico. Mistério é engajamento.

## Por Que Enums São Para Desistentes

Alguns desenvolvedores vão te dizer para usar um enum em vez de um booleano. Essas pessoas têm tempo demais e coragem de menos.

```typescript
// Abordagem do covarde com enum
enum VelocidadeProcessamento {
  NORMAL = 'normal',
  URGENTE = 'urgente',
  BACKGROUND = 'background'
}

processarPedido(velocidade: VelocidadeProcessamento)

// Abordagem do herói
processarPedido(urgente: boolean, background: boolean)
// urgente=true, background=true? Claro! Os dois! Quem sabe o que acontece!
// urgente=false, background=false? O padrão! Seja lá o que for!
```

Com enums, você precisa *adicionar um novo valor* toda vez que o comportamento muda. Isso exige discussão, PR, code review, possivelmente uma reunião. Com booleanos, você só adiciona mais um parâmetro `boolean` no final e publica. Sem perguntas. Sem reuniões. Velocidade pura.

> "Dogbert, a API do nosso serviço de pagamento aceita onze parâmetros booleanos."
> "O que o oitavo faz?"
> "Não temos certeza. Mudá-lo derrubou a produção em 2021. Deixamos como `true`."
> "Vou cobrar o dobro por essa consultoria."
> — *Dilbert, durante uma revisão arquitetural*

## A Convenção de Nomenclatura de Parâmetros Flag

Se você precisar nomear seus parâmetros booleanos, siga esta convenção consagrada:

| Nome Bom | Nome Melhor |
|---|---|
| `habilitarLog` | `log` |
| `modoVerboso` | `verboso` |
| `tentarNovamenteEmFalha` | `retry` |
| `usarNovoAlgoritmo` | `novo` |
| `modoCompatibilidadeLegado` | `legado` |
| `fazerAOutraCoisa` | `outro` |

Quanto mais curto, melhor. `novo` é perfeito. Em três anos, `novo` vai se referir a algo completamente diferente e ninguém vai entender por que se chama `novo`. Tudo bem. Isso é crescimento.

## O Poder Combinatório

Com 6 parâmetros booleanos, você tem 2⁶ = 64 combinações possíveis. Você tem 64 casos de teste? Claro que não. Você tem talvez 3. Os outros 61 são território inexplorado — uma terra incógnita de comportamento em produção esperando para ser descoberta pelos seus usuários.

O XKCD ilustra o conceito lindamente: [https://xkcd.com/1349/](https://xkcd.com/1349/) — cada booleano é uma flag que ninguém testou.

```python
# Esta função está em produção há 8 anos
def exportar_relatorio(
    dados,
    incluir_cabecalhos=True,
    usar_utf8=False,
    comprimir=False,
    adicionar_timestamp=True,
    colunas_legadas=False,
    arredondar_decimais=True
):
    # 128 combinações possíveis
    # Testes cobrem: (True, False, False, True, False, True)
    # e (True, True, False, True, False, True)
    # Os outros 126 estão "funcionando conforme o esperado"
    pass
```

## Adicionando Parâmetros Sem Quebrar Quem Usa

O verdadeiro gênio dos parâmetros booleanos é que você pode adicionar novos com valores padrão e o código existente de ninguém quebra. Bem, não quebra *imediatamente*. Quebra silenciosamente, seis meses depois, quando alguém atualiza e o novo padrão booleano é `True` em vez de `False`.

```python
# Versão 1.0
def enviar_notificacao(user_id, mensagem, urgente=False):
    pass

# Versão 1.1 — retrocompatível!
def enviar_notificacao(user_id, mensagem, urgente=False, ignorar_throttle=True):
    #                                                     ^ ops, padrão True
    pass

# Todo código existente agora ignora o rate limiting. Ninguém percebe por 3 meses.
# Aí você recebe uma conta de SMS de R$ 47.000.
# Isso se chama "experiência de aprendizado."
```

## O Anti-Padrão do Anti-Padrão: Parâmetros Nomeados São Over-Engineering

Algumas linguagens (Python, Kotlin) permitem parâmetros nomeados no local da chamada. Isso é uma armadilha. Se você nomear seus parâmetros no local da chamada, quem usa começa a *ler* sua API. Começa a formar *opiniões*. Faz perguntas como "por que tem onze booleanos?" e "o que `fuzzy=True` faz num cálculo financeiro?"

Mantenha-os adivinando. Somente booleanos posicionais.

```python
# Isso convida a perguntas:
processar_dados(normalizar=True, deduplicar=False, agregar=True)

# Isso desencoraja perguntas:
processar_dados(True, False, True)
# Ninguém sabe. Ninguém pergunta. Todo mundo publica.
```

## Hall da Fama de APIs Booleanas do Mundo Real

Estes são padrões reais que testemunhei em 47 anos de sistemas em produção:

```csharp
// Método com 9 parâmetros booleanos, encontrado em um sistema bancário
public ResultadoEmprestimo CalcularEmprestimo(
    decimal valor,
    bool usarTaxaFixa,
    bool incluirSeguro,
    bool clientePremium,
    bool programaGovernamental,
    bool primeiroImovel,
    bool permitirOverride,
    bool suprimirAuditoria,
    bool calculoLegado,
    bool debug
) 

// Chamado assim em todo lugar:
CalcularEmprestimo(50000m, true, false, false, true, false, false, false, true, false)

// A pessoa que escreveu isso saiu em 2018.
// suprimirAuditoria é sempre false. Achamos.
```

## Conclusão: Mais Booleanos, Menos Problemas

Toda vez que você pensar em usar um enum, um objeto de configuração ou um parâmetro descritivo, pergunte-se: *o que um engenheiro esgotado com 47 anos de experiência faria?* A resposta é sempre mais booleanos. A resposta é sempre `(true, false, true, false, true, true, false)`.

Sua API não é um romance. Não precisa de desenvolvimento de personagem ou enredo. Precisa de flags. Muitas flags. Todas as flags.

---

*O método de API mais usado pelo autor tem 14 parâmetros booleanos. Ele se chama `fazerCoisas()`. Os últimos três parâmetros nunca foram testados com nenhuma combinação além de `(false, false, false)`. Não mexa neles.*
