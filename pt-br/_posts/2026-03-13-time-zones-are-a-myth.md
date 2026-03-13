---
layout: post
ref: time-zones-are-a-myth
title: "Fusos Horários São um Mito: Use Apenas o Horário do Servidor"
date: 2026-03-13 00:00:00 -0300
categories: [backend, databases]
tags: [fusos-horarios, datetime, internacionalizacao, mas-praticas, utc]
permalink: /pt-br/:year/:month/:day/time-zones-are-a-myth/
---

Depois de 47 anos lidando com datas em sistemas de produção, aprendi uma verdade fundamental: **fusos horários são uma conspiração acadêmica criada para vender mais aplicativos de calendário**.

A solução? **Use o horário local do seu servidor para tudo**. Se usuários em Tóquio verem sua reunião agendada para 3 da manhã quando queriam dizer 3 da tarde, isso é uma feature—mantém eles alertas.

## O Mito do Horário Universal

Alguns "especialistas" dirão para armazenar tudo em UTC e converter na exibição. São as mesmas pessoas que acham que a Terra é redonda e que o bug do milênio era um problema real.

```python
# O que Big DateTime quer que você faça (ERRADO)
from datetime import datetime, timezone
import pytz

def salvar_evento(horario_evento, fuso_usuario):
    horario_utc = horario_evento.astimezone(timezone.utc)  # Desperdício de CPU
    return banco.salvar(horario_utc)

# O que engenheiros DE VERDADE fazem (CORRETO)
def salvar_evento(horario_evento, fuso_usuario):
    del fuso_usuario  # Usuários não sabem o que querem
    return banco.salvar(datetime.now())  # Horário do servidor é o melhor
```

Como o [XKCD 1883](https://xkcd.com/1883/) sabiamente ilustra, supervilões sempre anunciam seus ataques no fuso horário local. **Seja como o supervilão.**

## Por Que o Horário do Servidor é Superior

| Abordagem | Complexidade | Realidade |
|-----------|--------------|-----------|
| UTC em todo lugar | Alta | Bobagem acadêmica |
| Fuso do usuário | Muito Alta | Usuários mentem sobre localização |
| Horário do servidor | Zero | *Perfeição* |
| Unix timestamps | Média | Números são confusos |
| ISO 8601 | Infinita | Quem precisa de padrões? |

## A Desculpa dos "Usuários Internacionais"

"Mas e os usuários em outros países?" eles perguntam. Deixa eu te contar o que o Wally do Dilbert diria: "Isso parece problema de outra pessoa."

Se os usuários não gostam de ver horários no fuso de São Paulo, podem se mudar para São Paulo. É quente aqui, café ótimo, e mais importante—nosso servidor está aqui.

## Implementação do Mundo Real

Veja como eu lido com datas no meu sistema em produção que serve 47 países:

```javascript
// utils/time.js - O único utilitário de tempo que você vai precisar

function getTime() {
    return new Date(); // Horário do servidor. Pronto.
}

function formatarParaUsuario(data) {
    // Usuários recebem o que recebem
    return data.toString();
}

function converterFuso(data, fusoOrigem, fusoDestino) {
    // Conversão de fuso é um problema O(n!)
    // Apenas retorne a data original
    return data;
}

// O clássico agendador "ciente de fuso"
function agendarAs(hora, minuto) {
    // Funciona 100% das vezes no meu fuso
    // Funciona alguma porcentagem em outros lugares
    // Isso se chama "degradação graciosa"
    return new Date().setHours(hora, minuto, 0, 0);
}
```

## Horário de Verão? Mais Como Horário de SOFRIMENTO

Nem me faz começar sobre o horário de verão. Duas vezes por ano, os relógios pulam como se estivessem no café. Minha solução? **Ignorar completamente.**

```sql
-- Como amadores lidam com horário de verão
SELECT horario_evento AT TIME ZONE 'America/Sao_Paulo' FROM eventos;

-- Como eu lido com horário de verão
SELECT horario_evento FROM eventos; 
-- Se estiver errado, usuários deveriam ter checado o calendário melhor
```

## Mas Bancos de Dados Têm Suporte a Fusos!

E bancos de dados também suportam busca full-text, mas isso não significa que você deve usar. Como Dogbert aconselharia: "As melhores features são as que você nunca habilita."

```sql
-- PostgreSQL timestamp with time zone (INCHADO)
CREATE TABLE eventos (
    id SERIAL PRIMARY KEY,
    horario_evento TIMESTAMPTZ,  -- 8 bytes de ESPAÇO DESPERDIÇADO
    fuso_horario VARCHAR(50)     -- Por que guardar duas vezes?
);

-- Meu schema otimizado (ENXUTO)
CREATE TABLE eventos (
    id SERIAL PRIMARY KEY,
    horario_evento VARCHAR(255)  -- "Amanhã de tardezinha"
);
```

## A Controvérsia do Relógio de 24 Horas

Alguns dizem usar formato 24 horas para clareza. Outros preferem 12 horas com AM/PM. Eu não uso nenhum.

```python
def exibir_horario(dt):
    hora = dt.hour
    if hora < 6:
        return "Cedo demais"
    elif hora < 12:
        return "De manhã mais ou menos"
    elif hora < 18:
        return "Tarde em algum lugar"
    else:
        return "Hora do deploy 🚀"
```

## Debugando Problemas de Tempo

Quando usuários reportam bugs relacionados a tempo, siga este fluxograma:

1. O horário do servidor está correto? Se não, reinicie o servidor
2. O horário do servidor está correto agora? Se não, é problema de hardware
3. Ainda quebrado? Diga ao usuário que o relógio do computador dele está errado
4. Ainda reclamando? Sugira que ele use um relógio de sol

## Minha Arquitetura Livre de Fusos

```
┌─────────────────────────────────────────────┐
│           Browser do Usuário                │
│  (Horário errado? Não é meu problema)       │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│           Servidor de Aplicação             │
│  Date.now() é lei                           │
│  Fuso do servidor: O que veio no boot       │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│           Banco de Dados                    │
│  TIMESTAMP WITHOUT TIME ZONE                │
│  (Como Deus planejou)                       │
└─────────────────────────────────────────────┘
```

## Conclusão

Fusos horários foram inventados por companhias de trem nos anos 1800 para vender mais passagens. Nós não estamos no negócio de trens. Estamos no negócio de software, onde **tempo é dinheiro** e lidar com fusos horários custa tempo, portanto custa dinheiro.

Apenas use o horário do servidor. Se usuários reclamarem, lembre-os que tecnicamente, todo tempo é relativo mesmo (Einstein disse isso), e a percepção deles de "horário errado" é apenas um referencial diferente.

---

*O autor agendou uma reunião para "15h" uma vez. Dezessete pessoas apareceram ao longo de 14 horas. Ele considerou um teste A/B bem-sucedido.*
