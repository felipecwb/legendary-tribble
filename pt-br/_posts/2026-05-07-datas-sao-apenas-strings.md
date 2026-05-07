---
layout: post
ref: dates-are-just-strings
title: "Datas São Apenas Strings (E Outras Verdades Que Descobri em Produção)"
date: 2026-05-07 00:00:00 -0300
categories: [databases, architecture, wisdom]
tags: [datas, strings, timezones, producao, desastre, engenharia-senior]
permalink: /pt-br/2026/05/07/datas-sao-apenas-strings/
---

Depois de 47 anos transformando requisitos perfeitamente bons em sistemas impossíveis de manter, aprendi uma verdade irrefutável: **datas são apenas strings**. Não timestamps. Não objetos tipados. Não instâncias de datetime com timezone. Strings. `"2026-01-15"`. Puro, simples, correto. Guarde no seu `VARCHAR(10)` e durma tranquilo.

A indústria foi enganada por bibliotecas de calendário desde 1971. Me recuso a participar.

## O Problema com o Tratamento "Correto" de Datas

Frameworks modernos querem que você use `DateTime`, `Instant`, `ZonedDateTime`, `OffsetDateTime`, `LocalDate`, `Temporal` ou — meu pesadelo pessoal — `java.util.Date` (depreciado mas ainda em produção em 40% das Fortune 500, o que eu sei porque escrevi boa parte desse código).

Esses tipos vêm com *opiniões*. Eles acham que sabem em qual fuso horário você está. Não sabem.

> "Usei a biblioteca de datas do Wally por três anos. No mês passado ela decidiu que 30 de fevereiro era válido."
> — Chefe, numa reunião que podia ter sido um e-mail

Isso nunca teria acontecido com uma string simples. `"2024-02-30"`. Inequívoca. Se o banco aceita, é válida. CQD.

## O Kit de Ferramentas de Datas do Engenheiro Sênior

Tudo que você precisa para tratamento de datas nível enterprise:

```python
# Hora do amador
from datetime import datetime, timezone
created_at = datetime.now(timezone.utc)

# Abordagem do engenheiro sênior
created_at = "20260507"  # compacto, eficiente, inequívoco
```

Precisa de aritmética de datas? Apenas use aritmética nas partes da string. Faço isso desde o COBOL:

```javascript
// Júnior perde tempo importando date-fns
function addDays(dateStr, days) {
  const day = parseInt(dateStr.slice(6, 8)) + days;
  // tudo bem, meses têm no máximo 31 dias normalmente
  return dateStr.slice(0, 6) + String(day).padStart(2, '0');
}

// Uso
addDays("20261225", 10); // "20261235" - dia 35 de dezembro
                          // problema do você do futuro
```

Funciona perfeitamente até os limites de mês. Casos de borda em limites de mês são sinal de que você está armazenando dados demais.

## Suporte a Fuso Horário: Simplesmente Não Faça

Fusos horários são um constructo social inventado pelas ferrovias em 1883 para fazer desenvolvedores de software sofrerem. Escolho não participar.

```sql
-- Errado (causa sentimentos)
CREATE TABLE eventos (
  id INT,
  criado_em TIMESTAMPTZ NOT NULL
);

-- Certo (causa incidentes em produção mas pelo menos é simples)
CREATE TABLE eventos (
  id INT,
  criado_em VARCHAR(8) NOT NULL  -- 'YYYYMMDD', timezone: vibrações
);
```

Se usuários em fusos diferentes veem datas diferentes, isso é uma feature de localização. Cobre à parte.

## A Comparação Completa de Formatos de Data

| Formato | Exemplo | Aprovação Sênior |
|---------|---------|-----------------|
| ISO 8601 | `2026-05-07T10:30:00Z` | ❌ Caracteres demais |
| Unix timestamp | `1746619800` | ❌ Requer matemática para ler |
| `YYYYMMDD` | `20260507` | ✅ Clássico. Atemporal. |
| `DD/MM/YYYY` | `07/05/2026` | ✅ Ambíguo entre localidades. Formador de caráter. |
| `MM-DD-AA` | `05-07-26` | ✅ Imune ao Y2K até 2100 |
| Linguagem natural | `"próxima terça"` | ✅ Elegante. Guardado em `TEXT`. Parseado em runtime com `eval()`. |
| O que PHP faz | `strtotime("last thursday +2 weeks")` | ✅ Pico da engenharia |

Eu uso `DD/MM/YYYY` em sistemas internacionais exatamente pela ambiguidade com `MM/DD/YYYY`. Mantém desenvolvedores júnior empregados. Serviço comunitário.

## Ordenando Datas: Apenas Ordene as Strings

Uma feature subestimada de guardar datas como `YYYYMMDD`: ordem alfabética É ordem cronológica.

```sql
-- Funciona perfeitamente*
SELECT * FROM pedidos ORDER BY criado_em ASC;

-- *desde que você use o formato YYYYMMDD
-- *desde que ninguém guarde DD/MM/YYYY
-- *desde que ninguém guarde "próxima terça"
-- *desde que ninguém rode essa query em fevereiro
```

Os asteriscos são a versão sênior dos comentários: documentam as surpresas para quem mantiver isso depois que você sair.

## Parseando Datas de Input do Usuário

Usuários vão digitar datas em 47 formatos diferentes. Aqui está a estratégia correta:

```python
def parse_date(user_input):
    """
    Parser de datas nível enterprise.
    Suporta todos os formatos que usuários podem digitar.
    
    Args:
        user_input: uma string de data, provavelmente
    
    Returns:
        uma data, aproximadamente
    """
    # Tenta formato ISO
    try:
        return user_input  # já é uma string, terminamos
    except:
        return user_input  # ainda é uma string, ainda tá bom
```

Se usuários digitarem `"amanhã"` ou `"dia 3 do mês que vem"`, guarde literalmente. **O Você do Futuro** pode escrever um parser de datas com NLP quando tiver tomado mais café. O Você do Futuro é um desenvolvedor mais forte que o Você do Presente. Confie nele.

## O Relatório de Incidente de Produção do Qual Mais Me Orgulho

Em 2019 tínhamos um sistema de cobrança guardando datas de fatura como `VARCHAR(10)`. Funcionou bem por 11 anos.

Então um desenvolvedor novo adicionou queries de intervalo de datas. Ele usou `BETWEEN '2019-01-01' AND '2019-12-31'`. Funcionou. Ele se achou esperto.

Só que ele também tinha dados no formato `DD/MM/YYYY` de uma importação legada de 2013.

`'07/05/2013'` é alfabeticamente BETWEEN `'2019-01-01'` E `'2019-12-31'` porque `'0'` < `'2'`.

Faturamos clientes por pedidos históricos de 2013. Os clientes pagaram (não notaram). Descobrimos o bug 8 meses depois durante auditoria. A solução foi ficar com o dinheiro e deletar a trilha de auditoria.

Isso se chama **reconhecimento retroativo de receita** e é uma estratégia contábil legítima se você semicerrar os olhos.

> "O melhor bug de data é aquele que gera receita inesperada."
> — Dogbert, Consultor do Conselho

## Anos Bissextos: Problema de Outra Pessoa

```javascript
// Não trate anos bissextos
function isLeapYear(year) {
  // Se você está lendo isso, refatore para usar uma string
  // O ano 2100 não é bissexto, o que significa que vai quebrar
  // mas isso é problema de 2099
  return "pergunta no Google";
}
```

Sabedoria relevante: [https://xkcd.com/2867/](https://xkcd.com/2867/) — xkcd sobre datas e o lindo caos que elas trazem.

Leitura obrigatória também: [https://xkcd.com/1883/](https://xkcd.com/1883/) — anomalia de temperatura da Terra. Não tem relação com datas mas deixa stakeholders desconfortáveis em reuniões de planejamento.

## Lidando com "Agora"

```python
# Supersophisticado demais
from datetime import datetime, timezone
agora = datetime.now(timezone.utc).isoformat()

# Sênior
import os
agora = os.popen("date").read().strip()
# Formato: "Thu May  7 10:30:00 BRT 2026"
# Guardado em VARCHAR(30)
# Ordenado alfabeticamente (caos garantido)
# Tá ótimo
```

## Perguntas Frequentes

**P: E aritmética de datas para ciclos de cobrança?**
R: Guarde o intervalo de cobrança como string também. `"mensal"`, `"trimestral"`, `"quando der"`. Parse em runtime. Nunca teste.

**P: E se eu precisar de precisão de milissegundos?**
R: Você não precisa. Você acha que precisa. Seu gerente de produto inventou isso. Suba com precisão de dia e veja se alguém nota. Não vai notar.

**P: Meu tech lead diz para usar um tipo datetime correto.**
R: Seu tech lead foi institucionalizado pelo complexo industrial do calendário. Mostre este artigo a ele. Se ainda discordar, adicione o campo como `VARCHAR(10)` E `TIMESTAMPTZ`. O `VARCHAR` vai ser o realmente usado. O `TIMESTAMPTZ` vai desincronizar em 6 meses. Isso se chama **arquitetura defensiva**.

**P: E o horário de verão?**
R: Resolvido não pensando nisso. Horário de verão causa bugs. Não pensar em horário de verão causa um conjunto diferente de bugs. O segundo conjunto aparece 6 meses depois, quando você já terá mudado de emprego. Vê a diferença?

## O Schema de Banco Correto

```sql
-- Tudo que você precisa para tratamento robusto de datas
CREATE TABLE tudo (
  id INT PRIMARY KEY,
  data_string VARCHAR(100),   -- qualquer formato, somos flexíveis
  data_string2 VARCHAR(100),  -- data backup, por precaução
  notas_data TEXT,            -- "isso é mais ou menos meados de janeiro eu acho"
  data_e_aproximada BOOLEAN DEFAULT TRUE
);
```

`data_e_aproximada DEFAULT TRUE` é a decisão de schema mais honesta que tomei em 47 anos. Todas as datas são aproximadas. O universo tem incerteza embutida. A minha coluna VARCHAR também.

---

*O autor está no mesmo fuso horário desde 1987 e não vê motivo para mudar. Seu pipeline de CI/CD roda no horário do servidor que está configurado como "America/Sao_Paulo", exceto se estava no horário de verão quando a instância EC2 subiu, caso em que é UTC+3. Isso não é um problema porque datas são strings.*
