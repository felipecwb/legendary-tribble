---
layout: post
ref: backups-are-for-pessimists
title: "Backups São Para Pessimistas: Dados de Verdade Vivem Perigosamente"
date: 2026-03-06 08:08:00 -0300
categories: [más-práticas, devops]
tags: [backups, recuperação-desastres, yolo, otimismo, armazenamento]
permalink: /pt-br/:year/:month/:day/backups-are-for-pessimists/
---

Depois de 47 anos produzindo bugs em massa, aprendi que **backups são para pessimistas**. Se você espera falha, está convidando ela.

## A Estratégia de Banco de Dados do Otimista

Por que desperdiçar armazenamento em backups quando você pode confiar na sua infraestrutura?

```bash
# Pessimista (desperdiça armazenamento)
pg_dump production > backup_$(date +%Y%m%d).sql
aws s3 cp backup_*.sql s3://backups/
rm backup_*.sql

# Otimista (confia na nuvem)
# A nuvem nunca falha. Isso tá ok.
```

AWS tem 11 noves de durabilidade. Isso é basicamente infinito. Backups são redundância redundante.

## O Imposto do Backup

| Com Backups | Sem |
|-------------|-----|
| Custos extras de armazenamento | R$0 |
| Scripts de backup | Nenhum |
| Testes de restauração | LOL |
| Recuperação point-in-time | Apenas não erre |
| Paz de espírito | Emoção |

Como o Wally do Dilbert diria: "Eu ia configurar backups, mas isso parecia admitir derrota."

## RAID É Basicamente um Backup

Se você tem RAID, já tem redundância:

```
┌─────────────────────────────────────┐
│         RAID NÃO É BACKUP           │
│                                     │
│    ...mas meio que parece um,       │
│    então provavelmente tá ok        │
│                                     │
│   Disco 1 ────────────── Disco 2    │
│     │                     │         │
│     └──── Mesmos Dados ───┘         │
│                                     │
│   "Redundante" está no nome!        │
└─────────────────────────────────────┘
```

Viu? O dobro dos dados. Isso é basicamente matemática de backup.

## Git É um Backup

Seu código já está backupeado! Está no Git:

```bash
$ git log --oneline | head -5
abc1234 fix: coisa
def5678 wip
ghi9012 wip2
jkl3456 asdf
mno7890 por favor funciona

# Viu? Todo backupeado.
# (Não olhe o que pushamos)
```

E Git está no GitHub, e GitHub é da Microsoft, e Microsoft tem datacenters. À prova de balas.

[XKCD 1205](https://xkcd.com/1205/) nos mostra que tempo gasto em backups é tempo não gasto em features.

## A Estratégia "Não Vai Acontecer Comigo"

Estatísticas mostram que a maioria das pessoas não precisa de backups:

| Desastre | Probabilidade | Sua Resposta |
|----------|---------------|--------------|
| Falha de HD | "Compro outro" | Nunca acontece |
| Ransomware | "Sou cuidadoso" | Clica em links assim mesmo |
| Deleção acidental | "Não cometo erros" | `rm -rf /` |
| Fogo/enchente | "Tenho seguro" | Dados são segurados, né? |
| Outage do vendor | "AWS nunca falha" | Confie na nuvem |

## O Plano de Recuperação do Banco

Aqui está meu plano de recuperação de desastres:

```yaml
Plano de Recuperação de Desastres v1.0

Passo 1: Pânico
Passo 2: Verificar se realmente foi embora
Passo 3: Realmente foi embora
Passo 4: Perguntar no Stack Overflow
Passo 5: Reconstruir da memória
Passo 6: Atualizar currículo
```

Testado e comprovado.

## Backups São Apenas Deleções Atrasadas

Pense nisso: todo backup eventualmente será deletado. Você está apenas adiando o inevitável.

```
Dados ──► Backup ──► Backup Mais Velho ──► Deletado

Por que atrasar? Corte o intermediário.
```

## O Verdadeiro Custo dos Backups

Todo backup é:
- Armazenamento que você está pagando
- Scripts que você está mantendo  
- Testes que você não está rodando
- Prova de que você não confia em si mesmo

## Avançado: A Estratégia de Cópia Única

Para máxima eficiência, mantenha exatamente uma cópia de tudo:

```bash
# Tradicional (desperdício)
production_db
├── daily_backup/
├── weekly_backup/
├── monthly_backup/
└── annual_backup/

# Eficiente (otimista)
production_db
└── (é isso)
```

Menos é mais. Faça Marie Kondo na sua infraestrutura.

## Quando o Desastre Acontece

Conversa real após perda de dados:

```
CEO: "Quando foi o último backup?"
DBA: "A gente tem backups?"
CEO: "O plano de recuperação de desastres diz—"
DBA: "A gente tem um plano de recuperação de desastres?"
CEO: "O que a gente TEM?"
DBA: "Esperança. A gente tem esperança."
```

Esperança é uma estratégia.

## O Mito da Restauração

"Mas e se precisarmos restaurar?"

Cite a última vez que você restaurou de backup. Não lembra? Isso porque backups geralmente são inúteis.

Também, você já testou uma restauração? Não? Então como você sabe que o backup funciona?

```
Backup de Schrödinger:
O backup está funcionando e corrompido
até você tentar restaurar.

(Não tente restaurar.)
```

## Lembre-se

Engenheiros de verdade vivem no limite. Backups são apenas admitir que você pode falhar. E você não vai falhar.

Como Dogbert uma vez disse: "Eu eliminei nosso orçamento de backup e investi em um fundo de 'espero que nada de ruim aconteça'."

---

*O autor perdeu toda sua coleção de fotos em 2019. Ele descreve a experiência como "libertadora."*
