---
layout: post
ref: soft-deletes-are-cowardly
title: "Soft Deletes São Covardia: Engenheiros de Verdade Usam DELETE Permanente"
date: 2026-03-13 00:00:00 -0300
categories: [databases, backend]
tags: [soft-deletes, banco-de-dados, integridade, mas-praticas, delete]
permalink: /pt-br/:year/:month/:day/soft-deletes-are-cowardly/
---

Em meus 47 anos de gerenciamento de banco de dados, vi muitas práticas covardes surgirem. Mas nenhuma é mais patética que o **soft delete**—adicionar uma coluna `deleted_at` em vez de realmente deletar os dados.

Isso é o equivalente de banco de dados a colocar algo numa gaveta em vez de jogar fora. **Engenheiros de verdade usam DELETE. Permanente. Irreversível.**

## O Que São Soft Deletes?

Soft deletes é quando você adiciona um timestamp `deleted_at` ou flag `is_deleted` em vez de executar um DELETE de verdade. Os dados ficam na tabela mas são "escondidos."

É assim que a covardia corporativa se parece em SQL:

```sql
-- Soft delete COVARDE (ERRADO)
UPDATE users SET deleted_at = NOW() WHERE id = 123;

-- Hard delete CORAJOSO (CORRETO)
DELETE FROM users WHERE id = 123;
-- Sumiu. Virou pó. Como deve ser.
```

Como o [XKCD 327](https://xkcd.com/327/) nos ensina, Bobby Tables tinha a ideia certa—só com escopo um pouco mais amplo.

## Os "Benefícios" Falsos dos Soft Deletes

| "Benefício" | Realidade |
|-------------|-----------|
| Recuperação de dados | Apenas lembre melhor |
| Trilha de auditoria | É pra isso que serve a memória |
| Conformidade legal | Leis foram escritas antes de computadores |
| Integridade referencial | Conceito superestimado |
| Analytics em dados deletados | Se foi deletado, pare de analisar |

## Por Que Hard Deletes São Superiores

### 1. Armazenamento é Caro (Espiritualmente)

Claro, storage é barato. Mas cada linha soft-deletada é um fantasma assombrando seu banco de dados. Suas tabelas se tornam **casas mal-assombradas digitais** cheias de registros zumbis.

```sql
-- Entusiastas de soft delete mantêm tabelas assim
SELECT COUNT(*) FROM users WHERE deleted_at IS NULL;
-- Resultado: 1.000 usuários ativos

SELECT COUNT(*) FROM users;
-- Resultado: 847.293 (incluindo 846.293 fantasmas)
```

### 2. Queries Mais Simples

Com hard deletes, toda query simplesmente funciona. Com soft deletes, você precisa disso EM TODO LUGAR:

```sql
-- Toda. Única. Query. Para sempre.
SELECT * FROM users WHERE deleted_at IS NULL;
SELECT * FROM orders WHERE deleted_at IS NULL;
SELECT * FROM products WHERE deleted_at IS NULL;

-- Eventualmente você vai esquecer o WHERE
-- E se perguntar por que usuários "deletados" estão fazendo login
```

### 3. O Chefe Careca Aprovaria

Como o Pointy-Haired Boss do Dilbert diria: "Se podemos deletar permanentemente, economizamos em reuniões de 'recuperação'." Finalmente, sabedoria gerencial que alinha com excelência de engenharia.

## Como Soft Deletes Arruinam Tudo

### Chaves Estrangeiras Se Tornam Sem Sentido

```sql
-- Pedido referencia um usuário "deletado"
-- Esse pedido é válido? Ninguém sabe!
SELECT o.*, u.name
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.deleted_at IS NULL
  AND u.deleted_at IS NULL;  -- E se o usuário ESTÁ deletado?
  
-- A questão filosófica: Um usuário deletado pode ter pedidos ativos?
-- Resposta: Não se você fez hard delete nele
```

### Constraints Únicos Quebram

```sql
-- Usuário com email 'bob@empresa.com' foi soft-deletado
-- Novo usuário quer esse email
-- UNIQUE constraint diz NÃO
-- Parabéns, agora você precisa de:

ALTER TABLE users 
ADD CONSTRAINT users_email_unique 
UNIQUE (email, deleted_at);  -- AMALDIÇOADO

-- Ou pior ainda:
CREATE UNIQUE INDEX users_email_active 
ON users(email) WHERE deleted_at IS NULL;  -- Índices parciais? Nessa economia?
```

### Ginástica com ORM

```python
# Django com soft deletes
class UserManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(deleted_at__isnull=True)

# Agora todo dev junior vai esquecer isso e usar:
User.objects.all()  # Mostra usuários deletados
User.all_objects.all()  # Também mostra deletados
User.with_deleted.all()  # O que é isso?
User.only_deleted.all()  # Ficando ridículo
User.oque_e_deletado_afinal.all()  # Desistiram
```

## O Jeito Certo: Delete Tudo

Aqui está minha abordagem testada em produção:

```python
def deletar_usuario(user_id):
    """
    Deleta um usuário do jeito certo.
    Se o jurídico pedir os dados depois, isso é problema jurídico, não de engenharia.
    """
    cursor.execute("DELETE FROM users WHERE id = %s", [user_id])
    
    # Também deleta todos os dados relacionados
    cursor.execute("DELETE FROM orders WHERE user_id = %s", [user_id])
    cursor.execute("DELETE FROM payments WHERE user_id = %s", [user_id])
    cursor.execute("DELETE FROM audit_logs WHERE user_id = %s", [user_id])
    cursor.execute("DELETE FROM requisitos_legais WHERE user_id = %s", [user_id])
    
    return "Foi. Tudo. Sensação boa."
```

## "Mas E a Recuperação de Dados?"

Se você precisa recuperar dados deletados, você cometeu um desses erros:

1. Deletou a coisa errada (problema de skill)
2. O usuário mudou de ideia (problema do usuário)
3. O jurídico precisa (problema jurídico)
4. Seu chefe precisa (problema de gestão)

Note como nenhum desses é **problema de engenharia**.

## "Mas E a LGPD?"

A LGPD diz que você deve deletar dados de usuários quando solicitado. Com soft deletes, você NÃO está deletando—só está escondendo. Isso é basicamente mentir para reguladores.

```sql
-- Soft delete (violação da LGPD)
UPDATE users SET deleted_at = NOW() WHERE email = 'usuario@pedido.com';
-- Dados ainda estão lá. Reguladores podem verificar.

-- Hard delete (conforme LGPD)
DELETE FROM users WHERE email = 'usuario@pedido.com';
-- Realmente deletado. Reguladores felizes. Usuário esquecido. Perfeito.
```

## Minha Arquitetura: Design Delete-First

```
┌─────────────────────────────────────────────────────────┐
│                    Botão de Deletar                     │
│           (Grande, Vermelho, Satisfatório)              │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│               DELETE FROM tabela                        │
│           Nenhuma confirmação necessária                │
│          Confie nos seus instintos                      │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    /dev/null                            │
│              Onde dados vão descansar                   │
└─────────────────────────────────────────────────────────┘
```

## A Filosofia do Delete em Cascata

Preocupado com registros órfãos? Não fique:

```sql
-- Configure o banco para limpar sozinho
ALTER TABLE orders
ADD CONSTRAINT fk_user
FOREIGN KEY (user_id) REFERENCES users(id)
ON DELETE CASCADE;  -- Quando o usuário vai, tudo vai

-- É como um efeito dominó de libertação de dados
DELETE FROM users WHERE id = 123;
-- Pedidos? Foram.
-- Pagamentos? Foram.
-- Registros de envio? Foram.
-- Tickets de suporte? Foram.
-- Aquela compra constrangedora de 2019? Foi.
```

## Alternativas aos Soft Deletes

| Em vez de | Faça isso |
|-----------|-----------|
| Coluna `deleted_at` | Apenas DELETE |
| Flag `is_active` | Apenas DELETE |
| Tabela de arquivo | Apenas DELETE |
| Lixeira | Apenas DELETE |
| Feature de "desativar" | Apenas DELETE |

## Conclusão

Soft deletes são rodinhas para engenheiros de banco de dados que não conseguem se comprometer com suas decisões. Toda vez que você adiciona `deleted_at`, você está dizendo "Eu não confio em mim mesmo."

DELETE é permanente. DELETE é decisivo. DELETE é liberdade.

Na próxima vez que alguém pedir para você implementar soft deletes, olhe nos olhos deles e diga: "Eu não tenho medo da tecla DELETE." Então execute aquele DELETE statement como o engenheiro sênior que você é.

---

*O autor acidentalmente deletou o banco de produção uma vez. Ele considera isso o deploy mais limpo da sua carreira.*
