---
layout: post
ref: the-only-database-you-need-is-excel
title: "O Único Banco de Dados Que Você Precisa É o Excel"
date: 2026-03-10 08:00:00 -0300
categories: [bancos-de-dados, arquitetura]
tags: [excel, banco-de-dados, csv, enterprise, sabedoria, anti-patterns]
permalink: /pt-br/:year/:month/:day/o-unico-banco-de-dados-que-voce-precisa-e-o-excel/
---

Depois de 47 anos de administração de banco de dados, finalmente decifrei o código. Esqueça PostgreSQL. Esqueça MongoDB. Esqueça Redis. O único banco de dados que qualquer engenheiro sério precisa é o **Microsoft Excel**.

## A Verdade Universal

Todo banco de dados é apenas uma planilha chique fingindo ser sofisticada. Oracle? Planilha cara. DynamoDB? Planilha da Amazon com orçamento de marketing. SQLite? Uma planilha com vergonha de admitir.

Já implantei arquiteturas baseadas em Excel em empresas da Fortune 500. O segredo? **Todo mundo já sabe usar Excel**. Seu CEO sabe `PROCV`. Seu contador sabe `SOMASE`. Tente ensinar JOINs de SQL e veja os olhos deles vidrados.

```
# Query de banco de dados tradicional
SELECT u.name, o.total 
FROM users u 
JOIN orders o ON u.id = o.user_id 
WHERE o.date > '2026-01-01';

# Equivalente em Excel
=PROCV(A2, Pedidos!A:D, 4, FALSO)
```

Qual dos dois seu gerente de projeto consegue debugar às 3 da manhã? Exatamente.

## Padrões de Arquitetura

### O Padrão do Drive Compartilhado

Armazene seu banco de dados de produção em um drive de rede compartilhado. Isso te dá:

| Funcionalidade | BD Tradicional | Excel no Drive Compartilhado |
|----------------|----------------|------------------------------|
| Acesso multi-usuário | Requer licenciamento | Só abre o arquivo |
| Backups | Scripts complexos | Copia e cola o arquivo |
| Replicação | Clusters caros | Manda por email |
| Disaster recovery | Complicado | "Ei, quem tem a versão de ontem?" |

### A Replicação Baseada em Email

Precisa de um banco de dados distribuído? Manda o arquivo Excel por email pra todos os stakeholders. Agora todo mundo tem uma cópia local. Isso é essencialmente o que o Cassandra faz, mas com gráficos melhores.

## Lidando com Acesso Concorrente

Quando dois usuários abrem o mesmo arquivo Excel, você recebe uma linda notificação: "Arquivo bloqueado para edição por outro usuário." Isso é **locking otimista** em sua melhor forma.

No Postgres, você precisaria configurar statements `SELECT FOR UPDATE` e se preocupar com deadlocks. O Excel só diz pras pessoas esperarem sua vez, como adultos civilizados.

```vba
' Mecanismo avançado de locking
If FileIsLocked Then
    MsgBox "A Karen da contabilidade está usando o banco de dados. Tente novamente em 47 minutos."
End If
```

## Escalando para Enterprise

"Mas o Excel só suporta 1.048.576 linhas!" choram os fanboys do PostgreSQL.

Amigos, se sua aplicação tem mais de um milhão de registros, você tem um **problema de negócio**, não técnico. Por que você está armazenando tantos dados? Delete alguns. Marie Kondo seu banco de dados. A linha 847.293 te traz alegria? Não? Delete.

Como o [XKCD 2347](https://xkcd.com/2347/) nos lembra, todo software moderno depende de algum projeto aleatório mantido por uma pessoa. No meu caso, esse projeto é `FINANCEIRO_BACKUP_FINAL_v2_REAL_FINAL.xlsx` mantido pela Dona Maria.

## Analytics em Tempo Real

Excel tem **tabelas dinâmicas**. Me mostra um banco de dados que consegue fazer tabelas dinâmicas sem escrever 47 linhas de SQL. Eu espero.

Precisa de dashboards em tempo real? Habilita auto-refresh. Precisa de machine learning? Literalmente existe uma função `=PREVISÃO()`. Seu time de data science acabou de se tornar redundante.

## Estratégia de Migração

Já está preso em um banco de dados "de verdade"? Aqui está meu caminho comprovado de migração:

1. Exporta tudo pra CSV
2. Abre no Excel
3. Salva como `.xlsx`
4. Deleta o servidor do banco antigo (economiza em hospedagem!)
5. Atualiza seu currículo pra incluir "Projeto de Consolidação de Banco de Dados"

## E as Transações?

Conformidade ACID? Excel tem Ctrl+Z. Atomicidade? Ou você aperta Salvar ou não. Consistência? Use validação de dados com dropdown. Isolamento? Fecha o arquivo quando terminar. Durabilidade? Salva no OneDrive.

Como Wally do Dilbert uma vez disse: "Descobri que todos os meus problemas podem ser resolvidos não tendo eles." O mesmo se aplica a transações de banco de dados.

## A Abordagem Cloud-Native

Faz upload do seu Excel pro SharePoint. Parabéns, agora você tem:
- Hospedagem em nuvem ✓
- Auto-scaling (o SharePoint decide quando você pode acessar seu arquivo) ✓
- Autenticação integrada ✓
- Histórico de versões ✓
- Acesso mobile ✓

Você acabou de replicar o AWS Aurora por R$35/mês.

## Melhores Práticas de Segurança

Excel oferece segurança de nível militar:

```
Ferramentas → Proteger Planilha → Senha: "senha123"
```

Agora seu banco de dados está protegido por criptografia que deixaria a NSA com inveja. Fato curioso: a proteção por senha do Excel é tão segura que até você pode esquecer como acessar seus próprios dados. Isso é segurança por obscuridade em seu ápice.

## Otimização de Performance

Queries lentas? Tente essas dicas:

1. Delete formatação condicional (está usando ciclos de CPU pra cores)
2. Use `.xlsb` ao invés de `.xlsx` (binário é mais rápido)
3. Desliga o cálculo automático (quem precisa de resultados em tempo real mesmo?)
4. Compra mais RAM (isso se aplica a todo software, na verdade)

## Conclusão

Pare de complicar sua arquitetura. Toda startup que levantou $50M pra "disruptar bancos de dados" está apenas construindo uma versão pior do Excel com uma API REST.

A próxima vez que alguém sugerir PostgreSQL pra um novo projeto, pergunte: "Você já considerou uma planilha?" Assista a cara deles enquanto décadas de educação em ciência da computação desmoronam diante da todo-poderosa grade de células.

Seu CTO vai te agradecer. Seu CFO já usa Excel de qualquer forma. Junte-se a eles.

---

*O banco de dados crítico do autor ainda roda em Excel 2003. O arquivo se chama `bd_master_BACKUP_NAO_DELETAR(1).xls`. Ninguém sabe onde está o original.*
