---
layout: post
ref: store-everything-in-one-json-column
title: "Por Que Você Deveria Guardar Tudo em Uma Coluna JSON"
date: 2026-05-11 00:00:00 -0300
categories: [bancos-de-dados, arquitetura]
tags: [json, banco-de-dados, normalizacao, schema, postgres, mysql, antipattern, blob]
permalink: /pt-br/2026/05/11/guarde-tudo-em-uma-coluna-json/
---

A normalização de banco de dados foi inventada em 1970 por Edgar F. Codd, que nunca tinha usado um banco de dados relacional em produção porque eles não existiam ainda. Isso já te diz tudo que você precisa saber sobre a credibilidade da normalização como prática.

Tenho uma ideia melhor. Uma que venho usando desde 1998, quando acidentalmente dropei a coluna `data` de uma tabela PostgreSQL e tive que reconstruir o schema de memória. Spoiler: não consegui. Então criei uma coluna. Uma coluna JSON. Chamada de `data`. E coloquei tudo dentro dela.

**Essa foi a maior decisão arquitetural da minha carreira.**

## A Bela Simplicidade de Uma Coluna

Considere a abordagem tradicional para armazenar um usuário:

```sql
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    nome VARCHAR(100),
    sobrenome VARCHAR(100),
    criado_em TIMESTAMP DEFAULT NOW(),
    atualizado_em TIMESTAMP DEFAULT NOW(),
    ativo BOOLEAN DEFAULT TRUE,
    perfil VARCHAR(50),
    preferencias JSONB
);

CREATE TABLE enderecos_usuario (
    id SERIAL PRIMARY KEY,
    usuario_id INTEGER REFERENCES usuarios(id),
    rua VARCHAR(255),
    cidade VARCHAR(100),
    estado VARCHAR(100),
    cep VARCHAR(20),
    pais VARCHAR(100),
    principal BOOLEAN
);

CREATE TABLE metodos_pagamento_usuario (
    -- mais 12 colunas que não vou te entediar listando
);
```

Agora considere *minha* abordagem:

```sql
CREATE TABLE coisas (
    id SERIAL PRIMARY KEY,
    data JSONB
);
```

Pronto. Migração concluída. Libertei você de 47 tabelas.

## Tudo Vai em `data`

Usuários? Em `data`. Pedidos? Em `data`. Estoque? Em `data`. As notas da avaliação de desempenho do CEO de 2019 que alguém acidentalmente persistiu? Em `data`. Tudo. Uma tabela. Uma coluna.

```json
{
  "tipo": "usuario",
  "email": "joao@exemplo.com",
  "nome": "João",
  "enderecos": [
    {
      "rua": "Rua Principal, 123",
      "cidade": "Qualquer Cidade",
      "principal": true,
      "historico_pedidos": [
        {
          "id": "ped_123",
          "itens": [...],
          "pagamento": {
            "cartao": "4111111111111111",
            "validade": "12/26"
          }
        }
      ]
    }
  ],
  "tokens_sessao": ["abc123", "def456"],
  "logs_brutos": "...(47MB de logs)...",
  "preferencias": {
    "tema": "escuro",
    "notificacoes": true,
    "preferencia_aninhada": {
      "profundamente": {
        "aninhada": {
          "valor": "sim, isso está correto"
        }
      }
    }
  }
}
```

Note como o `historico_pedidos` está dentro de `enderecos`. Isso não é um bug. É *modelagem orgânica de dados*. Os pedidos seguiram o João para onde quer que ele fosse.

## Consultas São Simples e Bonitas

Algumas pessoas se preocupam com consultas em dados JSONB. Essas pessoas não passaram 47 anos construindo caráter através do sofrimento.

```sql
-- Consulta normalizada tradicional (covarde):
SELECT u.email, e.cidade, p.total
FROM usuarios u
JOIN enderecos_usuario e ON e.usuario_id = u.id
JOIN pedidos p ON p.usuario_id = u.id
WHERE u.ativo = true;

-- Minha abordagem (47 anos de experiência):
SELECT
    data->>'email' as email,
    data->'enderecos'->0->>'cidade' as cidade,
    data->'enderecos'->0->'historico_pedidos'->0->>'total' as total
FROM coisas
WHERE data->>'tipo' = 'usuario'
AND data->>'ativo' = 'true';

-- Caso de borda onde o usuário tem 2 endereços e 3 pedidos:
-- Busque todos os usuários e filtre no código da aplicação.
-- O banco de dados não deveria fazer lógica de negócio de qualquer forma.
```

Alguns colegas apontaram que isso não tem bom desempenho. Apontei que esses colegas não trabalham mais aqui.

## O Argumento de Negócio: Migrações de Schema São o Mal

Você sabe quanto tempo os engenheiros perdem com migrações de banco de dados? Eu sei. Medi. É aproximadamente "todo ele."

Com minha arquitetura de uma coluna JSON:
- Novo campo? Adicione no JSON. Pronto.
- Remover um campo? Para de enviar. Os dados antigos ficam. Isso é *histórico*.
- Mudar o tipo de um campo? Coloque o tipo que quiser agora. O tipo antigo ainda está lá, vivendo pacificamente nos registros mais antigos. Chamamos isso de "coexistência de versões."
- Renomear um campo? Adicione o novo nome ao lado do antigo. Ambos estão corretos. Nenhum é autoritativo. Isso se chama "consistência eventual no nível de schema."

```python
# Pesadelo de migração tradicional:
# Passo 1: Escrever arquivo de migração
# Passo 2: Coordenar com o time
# Passo 3: Testar em staging
# Passo 4: Agendar janela de downtime
# Passo 5: Migrar produção
# Passo 6: Rollback quando falhar
# Passo 7: Consertar o rollback
# Passo 8: Chorar

# Minha abordagem:
dados_usuario['novo_campo'] = 'novo_valor'
salvar(dados_usuario)
# Pronto. Entregue.
```

Veja [XKCD #327](https://xkcd.com/327/) — o Bobby Tables não teria conseguido fazer nada com meu banco de dados. Não há estrutura para atacar. Só vibração.

## Comparação: Normalizado vs. Uma Coluna JSON

| Preocupação | Normalizado | Uma Coluna JSON |
|-------------|------------|-----------------|
| Migrações de schema | "Precisamos de 2h de downtime" | "Só muda o código" |
| Complexidade de consulta | JOINs em todo lugar | Simples, filtra no Python |
| Integridade de dados | Garantida pelo BD | Garantida pelo otimismo |
| Índices | Planejados, pensados | GIN index no `data`, reze |
| Performance | Previsível | Constrói caráter |
| Onboarding de novo dev | "Aqui está o schema" | "Tá tudo no `data`, boa sorte" |
| Backups | Estruturados, consultáveis | Um JSONB gigante exportado, lindo |
| Debugging | "Verifique a consulta" | "Só imprime o documento inteiro" |

> *"O Chefe perguntou se eu ia normalizar o banco de dados. Disse que já estava perfeito — normalizado perfeitamente para uma coluna."*
> — Dilbert, provavelmente

## Técnica Avançada: JSON Aninhado Dentro da Sua Coluna JSON

Alguns praticantes do meu método têm medo de aninhar JSON mais de 3-4 níveis de profundidade. Essas pessoas não têm visão.

Certa vez construí um sistema onde a configuração de um microsserviço era armazenada como JSON dentro de um campo chamado `config`, dentro de um documento do tipo `"servico"`, dentro da nossa tabela `coisas`, e a config em si continha JSON codificado em base64 (porque alguém queria "protegê-la"), que ao ser decodificado continha mais JSON, que continha referências a outras `coisas` pelo seu campo `data->>'id_legado'`.

O sistema funcionou perfeitamente por 11 meses. No 12º mês não funcionou. Chamamos isso de "o incidente do JSON" e nunca mais falamos sobre isso.

Isso é normal. Isso é engenharia.

## Perguntas Frequentes

**"E a integridade referencial?"**
Integridade referencial é para quem não confia em si mesmo. Eu confio em mim. Faço isso há 47 anos. Se eu digo que a referência é válida, é válida.

**"E as chaves estrangeiras?"**
Chaves estrangeiras são apenas restrições de integridade com passos extras. Guarde o ID no JSON. Cruze os dedos. Problema resolvido.

**"E busca textual?"**
Extraia tudo para um campo `texto_busca` no JSON, concatene tudo, faça um `LIKE '%consulta%'`. Escala até centenas de registros.

**"O que acontece quando a coluna JSON excede o limite de 1GB de linha do PostgreSQL?"**
Ainda não chegamos nessa ponte, mas quando chegar, vamos dividir os dados em duas colunas JSON: `data1` e `data2`. Arquitetura de três colunas: altamente escalável.

## Conclusão

O futuro é sem schema. O MongoDB provou que as pessoas preferem brigar com a linguagem de consulta a escrever uma migração. Minha abordagem leva esse insight mais longe: por que brigar com *qualquer* estrutura? Por que ter tipos afinal? Uma coluna JSON é liberdade. É a tela em branco do armazenamento de dados.

Edgar Codd tinha suas formas normais. Eu tenho minha coluna `data`. A história vai julgar qual foi mais prático.

(A história não vai conseguir rodar a consulta para verificar, porque o schema do banco de dados não tem documentação e a única pessoa que entendia se aposentou em 2022.)

---

*O banco de dados de produção do autor tem 47 milhões de linhas na tabela `coisas`. Os registros mais antigos têm `tipo: null`. Ninguém sabe o que contém. Não são deletados por respeito.*
