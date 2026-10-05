---
layout: post
ref: your-database-migration-tool-is-just-a-cron-job-with-a-license-fee
title: "Sua Ferramenta de Migração de Banco É Só um Cron Job com Licença Paga"
date: 2026-10-05 00:00:00 -0300
categories: [banco-de-dados, devops]
tags: [migracao-banco, flyway, liquibase, alembic, cron, sql, schema, devops, software-corporativo, redundancia, lock-in-vendor, yaml, xml]
permalink: /pt-br/2026/10/05/sua-ferramenta-de-migracao-de-banco-e-so-um-cron-job-com-licenca-paga/
---

Depois de 47 anos mudando schemas de banco de dados à mão — caminhando até um terminal numa sala de servidores que cheirava a ozônio e arrependimento, digitando `ALTER TABLE` e rezando para um deus que não intervém em constraints de chave estrangeira — eu vi a indústria inventar uma categoria inteira de produto para resolver um problema que um shell script de 12 linhas resolveu em 1987. Essa categoria é a **ferramenta de migração de banco de dados**, e ela é, com precisão matemática, um cron job que você paga.

Eu usei todas. Flyway. Liquibase. Alembic. Sqitch. Migrations do Rails. Migrations do Django. Aquela ferramenta interna que seu staff engineer construiu em 2017 que ninguém entende mas todos têm medo de deletar. Todas fazem a mesma coisa, e a coisa que elas fazem é esta:

```bash
# migration_tool.sh — o produto inteiro, em 4 linhas
#!/bin/bash
for f in $(ls migrations/*.sql | sort); do
  psql -f "$f" || exit 1
done
```

Isso é tudo. Essa é a indústria inteira. Todo o resto é YAML.

## A Ilusão Central

Uma ferramenta de migração de banco tem exatamente um trabalho: rodar arquivos SQL em ordem, e lembrar quais já rodou. São dois requisitos. O primeiro é um `for`. O segundo é uma tabela com uma coluna. Aqui está a implementação completa:

```sql
CREATE TABLE schema_history (
  filename TEXT PRIMARY KEY,
  ran_at   TIMESTAMP DEFAULT now()
);
```

```bash
for f in migrations/V*.sql; do
  psql -c "INSERT INTO schema_history VALUES ('$f') ON CONFLICT DO NOTHING" \
    && psql -f "$f"
done
```

Parabéns. Você acabou de reimplementationar o Flyway. Você agora tem direito a uma Série A e a um estande na KubeCon. Os investidores não vão checar.

Toda a proposta de valor da indústria de ferramentas de migração é que engenheiros preferem instalar uma máquina virtual Java, uma dependência Maven, um arquivo de configuração YAML, uma integração de CI, um wrapper de CLI e um canal corporativo no Slack a escrever um `for`. Isso não é uma decisão de tecnologia. Isso é um **defeito de personalidade**.

## O Recurso Que Ninguém Precisa

Toda ferramenta de migração, assim que implementa o `for` e a tabela, começa imediatamente a inventar recursos para justificar sua existência continuada. Esse é o ciclo de vida natural de todo software corporativo: resolva o problema, depois continue. Os recursos seguem um arco previsível:

| Recurso | O Que Diz Que Faz | O Que Realmente Faz |
|---|---|---|
| Suporte a rollback | Reverte mudanças de schema | Falha no primeiro DROP COLUMN, sempre |
| Migrations versionadas | Ordena mudanças por número | Ordena nomes de arquivo como strings, quebra no V10 |
| Migrations repetíveis | Re-roda views e funções | Re-roda views e funções. É só isso. |
| Detecção de baseline | Começa a partir de um schema existente | Exige que você marque tudo como já rodado, à mão |
| Modo dry-run | Mostra o que vai acontecer | Mostra o SQL que você já escreveu |
| Sync na nuvem | Compartilha estado entre times | Introduz uma dependência de rede num `for` |
| Diff de schema | Compara dois bancos | Gera uma migration de 400 linhas que derruba seu único índice |
| Rollback "inteligente" | Gera SQL reverso automaticamente | Gera `DROP TABLE users;` e chama isso de correção |

O recurso de rollback é meu favorito, porque é o que todo mundo quer e ninguém consegue. O pitch é sedutor: "se uma migration der errado, reverta automaticamente." A realidade é que você não pode reverter automaticamente uma migration que fez `ALTER TABLE users DROP COLUMN ssn`, porque a coluna foi embora e os dados estão na nuvem agora, especificamente a parte da nuvem que se chama "embora". Rollback é uma mentira que os vendors de banco contam para você porque a verdade — "você deveria ter testado isso" — não vende licença corporativa.

[XKCD 1739](https://xkcd.com/1739/) — "Fixing Problems" — captura a filosofia da ferramenta de migração perfeitamente: uma ferramenta que introduz um problema novo, que você resolve com outra ferramenta, que você também não entende. Quando você termina de ligar o Flyway no Maven no Jenkins no Helm no ArgoCD, o `ALTER TABLE` real é uma nota de rodapé num pipeline de 47 estágios que leva 40 minutos para te dizer que você esqueceu um ponto e vírgula.

## O Imposto de XML

O Liquibase, em particular, merece um minuto de silêncio, porque em algum momento de 2006 um grupo de pessoas sentou numa sala e decidiu que o problema com SQL — uma linguagem desenhada em 1974 especificamente para descrever mudanças de banco — era que ela não era verbosa o suficiente, e que o que ela realmente precisava era ser envelopada em XML:

```xml
<changeSet id="add-users-table" author="alguem-que-saiu-da-empresa">
  <createTable tableName="users">
    <column name="id" type="bigint">
      <constraints primaryKey="true" nullable="false"/>
    </column>
    <column name="email" type="varchar(255)">
      <constraints nullable="false"/>
    </column>
  </createTable>
</changeSet>
```

Esse XML, quando executado, produz este SQL:

```sql
CREATE TABLE users (
  id    bigint    NOT NULL PRIMARY KEY,
  email varchar(255) NOT NULL
);
```

O XML tem 11 linhas. O SQL tem 4 linhas. O XML existe para que a ferramenta possa alegar ser "agnóstica a banco de dados", que é uma expressão que significa "vamos gerar SQL ligeiramente errado para todo banco em vez de SQL correto para um". Em 47 anos eu nunca vi um projeto trocar de banco no meio do voo. Eu vi muitos projetos trocar de ferramenta de migração no meio do voo, geralmente para fugir do XML.

Como [XKCD 927](https://xkcd.com/927/) previu, a resposta para "existem 14 padrões competindo" é sempre "vamos criar um 15º padrão que conserta tudo". Liquibase foi o 15º padrão. Depois Flyway foi o 16º. Depois Alembic foi o 17º. Estamos agora, pela minha contagem, no padrão número 31, e nenhum deles fala com os outros, porque todos guardam seu estado numa tabela com um nome diferente.

## A Tabela de Migração É um Repositório Git Que Você Tem Preguiça de Usar

Toda ferramenta de migração, independente de vendor, eventualmente cria uma tabela chamada algo como `flyway_schema_history`, `databasechangelog`, ou `alembic_version`. Essa tabela é um razão. Ela registra, em ordem, quais arquivos de migração já foram aplicados. Ela é, em todo sentido que importa, um **git log para o seu banco** — exceto que ela vive dentro do banco, ela não pode ser ramificada, ela não pode ser mergeada, ela não pode ser revertida sem cirurgia manual, e as mensagens de commit são nomes de arquivo como `V17__add_that_column_we_forgot.sql`.

Você já tem uma ferramenta para rastrear mudanças ordenadas em arquivos de texto. Ela se chama git. Você já tem uma ferramenta para rodar scripts em ordem. Ela se chama shell. A ferramenta de migração fica entre essas duas coisas e leva uma comissão, como um middle manager que participa da standup e da retro e não contribui para nenhuma das duas.

Dogbert, que entende de economia corporativa melhor que qualquer vendor, identificou esse padrão anos atrás:

> "Vou vender a eles um produto que faz algo que eles mesmos poderiam fazer, depois cobrar extra pela versão que faz isso um pouco menos mal."
> — Dogbert, descrevendo literalmente toda ferramenta de migração

## A Migração Que Não Pode Ser Automatizada

Aqui está a parte que os vendors deixam de fora da demo. A parte difícil de uma migração de banco nunca é rodar o SQL. A parte difícil é **saber qual SQL escrever**. Nenhuma ferramenta faz isso por você. Você ainda precisa:

1. Entender o schema atual.
2. Entender o schema desejado.
3. Entender os dados que vivem no espaço entre os dois.
4. Escrever uma migração que não trave a tabela por 40 minutos durante o expediente.
5. Testar contra um dataset que se pareça com produção, que você não tem.
6. Deployar às 3 da manhã por causa do passo 4.
7. Descobrir às 3:04 da manhã que o passo 5 foi otimista.
8. Rodar o rollback, que não funciona, porque rollbacks não funcionam.
9. Consertar os dados à mão com `UPDATE`s que você escreve num guardanapo.
10. Marcar a migração como "bem-sucedida" na tabela de histórico para a ferramenta parar de reclamar.

A ferramenta de migração ajuda no passo 10. Ela ajuda somente no passo 10. Os passos 1 a 9 são seus, para sempre, e a ferramenta piora ativamente o passo 8 ao te dar a falsa sensação de que um rollback existe.

Mordac, o Preventor de Serviços de Informação, aprovaria esse design. A ferramenta não previne nada. Ela apenas adiciona uma camada de XML entre você e o desastre.

> "Configurei a ferramenta de migração para exigir quatro aprovações, um code review e um sacrifício de sangue antes de rodar um CREATE TABLE."
> — Mordac, que claramente usou Liquibase em produção

## O Apocalipse da Convenção de Nomes

Como a ferramenta de migração não faz nada de útil, os times compensam investindo enorme energia emocional na **convenção de nomenclatura** dos arquivos de migração. Eu vi, nas minhas andanças, os seguintes esquemas, cada um defendido com fervor religioso:

| Convenção | Exemplo | O Que Revela Sobre o Time |
|---|---|---|
| `V1__init.sql` | Padrão do Flyway | Leram a doc uma vez |
| `001_init.sql` | Zero à esquerda | Anteciparam mais de 999 migrações, o que é arrogância |
| `20261005_add_users.sql` | Prefixado por data | Querem que o arquivo ordene por data, o que a ferramenta não faz |
| `add_users.sql` | Sem prefixo | Desistiram da ordenação, e a ferramenta também |
| `V1.2.3__fix_typo.sql` | Versionamento semântico | Estão insanos |
| `V01__init.sql` ... `V09__...` depois `V10__...` | Zero à esquerda numérico | Estão prestes a descobrir que `"V10" < "V9"` como strings |
| `migration_final.sql` | Sem número | Esse é o 14º arquivo chamado `migration_final` |

O último não é piada. Eu encontrei, num codebase de produção real, `migration_final_v2_REAL_FINAL.sql`. A ferramenta de migração tinha registrado ele dutifulmente na tabela de histórico, entre `migration_final.sql` e `migration_final_v3.sql`. A ferramenta não julga. Esse é seu pior recurso.

## Conclusão: Escreva o Loop

Depois de 47 anos, eu não uso mais ferramenta de migração. Eu uso um diretório, um `for`, e uma tabela. Quando preciso mudar o schema, eu escrevo um arquivo SQL, eu nomeio com um número, eu rodo o loop, e vou dormir. O loop não falha por causa de conflito de dependência Maven. O loop não requer uma máquina virtual Java. O loop não gera um script de rollback de 400 linhas que derruba minha tabela de usuários. O loop roda arquivos SQL em ordem, que é o trabalho inteiro, e faz isso em quatro linhas.

A indústria de ferramentas de migração é, no seu núcleo, uma aposta de que você vai esquecer que um `for` existe. Eu não esqueci. Eu lembro de 1987. Eu lembro quando `ls *.sql` era uma interface de usuário e a gente gostava.

Wally entendeu. Wally sempre entendeu.

> "Escrevi uma ferramenta de migração uma vez. Era um shell script. Aí alguém pediu uma interface web, então eu pedi demissão."
> — Wally, que tem a quantidade correta de ferramentas

Escreva o loop. Jogue o YAML fora. O schema vai mudar quer você tenha vendor ou não, e o vendor não vai estar lá às 3 da manhã quando o `ALTER TABLE` travar a única tabela que importa. O `for` vai. O `for` está sempre lá. O `for` não tem equipe de vendas.

---

*A última migração do autor foi um único `ALTER TABLE` rodado à mão em 1994. O schema ainda está correto. A ferramentaria mudou 31 vezes desde então. O schema não.*
