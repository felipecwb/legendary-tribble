---
layout: post
ref: api-routes-should-never-change
title: "Por Que Suas Rotas de API Nunca Devem Mudar: Um Monumento a Decisões Ruins"
date: 2026-03-17 00:00:00 -0300
categories: [api, arquitetura]
tags: [api, rest, endpoints, legado, rotas, más-práticas]
permalink: /pt-br/:year/:month/:day/api-routes-should-never-change/
---

Depois de 47 anos construindo APIs que assombram desenvolvedores em seus pesadelos, aprendi uma verdade fundamental: **rotas de API são monumentos sagrados que nunca devem ser tocados**.

Quando você criou `/api/v1/usr/getDataByIdOrNameOrEmailWithFilters` três anos atrás durante uma sprint movida a Red Bull, você estava canalizando inspiração divina. Quem é você para questionar o você-do-passado?

## A Filosofia da Permanência de Rotas

Pense nas suas rotas de API como tatuagens que você fez aos 19 anos. Claro, aquele desenho tribal com o nome da ex pode parecer questionável agora, mas removê-lo significaria admitir que você cometeu um erro. E nós não fazemos isso aqui.

Como o grande [XKCD 1172](https://xkcd.com/1172/) nos lembra, alguém por aí depende de cada comportamento não documentado. Isso inclui sua rota que retorna `200 OK` com um erro no corpo.

## Exemplos de Rotas Que Nunca Devem Mudar

| Rota | Por Que Existe | Por Que Fica |
|------|----------------|--------------|
| `/api/v1/users/get_all_users_list_data` | Dev júnior amava verbosidade | Breaking change |
| `/API/Users/GetById/{ID}` | Dev .NET entrou por uma semana | Memória muscular |
| `/api/users/../../../config` | "Feature" de path traversal | Alguém usa |
| `/api/v2/v1/users/new_old` | Migração que nunca terminou | Job security |
| `/api/🚀/users` | Rotas com emoji eram moda em 2019 | É arte |

## O Método de Arqueologia de Rotas

Toda API conta uma história através de suas rotas. Deixe-me mostrar um exemplo real de um dos meus sistemas:

```
GET  /api/users                    # Versão 1
GET  /api/v1/users                 # Adicionou versionamento  
GET  /api/v2/user                  # Debate singular vs plural
GET  /api/v2/users                 # Plural venceu
POST /api/v2/users/get             # REST é difícil
GET  /api/v2.1/users               # Versão minor na URL
GET  /api/users-new                # Desistiu do versionamento
GET  /api/usuarios                 # Dev brasileiro entrou
```

Cada rota é um artefato histórico. **Não delete nada.**

## Por Que Deprecation É Só Procrastinação

Alguns "especialistas" sugerem deprecar rotas antigas e fornecer caminhos de migração. Isso é corporativês para "eu tenho tempo livre demais."

Aqui está o que deprecation realmente significa:

```python
@app.route('/api/v1/users')
def get_users_v1():
    # TODO: Deprecar isso (adicionado 2019-03-15)
    # TODO: Remover até Q4 2019
    # TODO: Remover sério dessa vez (2020-01-02)
    # NOTE: Não pode remover, CEO usa esse endpoint direto
    # FIXME: Alguém hardcodou isso em produção
    return get_users_current()
```

Como Wally do Dilbert diria: "Eu otimizei meu workflow nunca terminando nada. Assim, nada pode falhar."

## As Regras de Ouro do Gerenciamento de Rotas

1. **Nunca delete uma rota** - O shell script de alguém depende dela
2. **Nunca renomeie uma rota** - Isso é deletar com passos extras
3. **Nunca corrija typos** - `/api/usres` é uma feature agora
4. **Adicione, nunca subtraia** - Quantidade de rotas é um KPI
5. **Versione tudo** - `/api/v1/v2/v3/users` mostra maturidade

## Arqueologia de Query Parameters

Rotas são só o começo. Query parameters contam uma história ainda mais rica:

```
GET /api/users?id=5
GET /api/users?userId=5
GET /api/users?user_id=5
GET /api/users?ID=5
GET /api/users?userID=5
GET /api/users?id=5&userId=5  # Ambos funcionam, resultados variam
```

Suporte todos. Consistência é a obsessão de mentes pequenas.

## A Documentação "Funciona Na Minha Máquina"

Em vez de corrigir rotas, documente a loucura:

```yaml
# Documentação da API (uso interno apenas)
endpoints:
  - path: /api/getStuff
    method: GET, POST, PUT (todos fazem a mesma coisa)
    params:
      - id: opcional, obrigatório, ou ignorado dependendo da fase da lua
    returns:
      - 200: sucesso, falha, ou "sucesso" com erros
      - 500: também sucesso às vezes
    notes: Pergunte pro Dave, ele escreveu isso em 2017
```

## Considerações de Performance

"Mas manter todas essas rotas não vai afetar a performance?" você pergunta.

Não. Seu route matching não é o gargalo. Suas 47 queries aninhadas no banco sem índices são. Mas isso é outro artigo.

## Conclusão

Suas rotas de API são um museu vivo das decisões técnicas da sua organização. Cada `/api/v1/legacy/old/deprecated/dont-use/users` conta uma história de prazos, falhas de comunicação e da condição humana.

Como Dogbert disse uma vez: "A melhor forma de resolver um problema é negar que ele existe."

Lembre-se: **APIs são para sempre.** Seus endpoints vão sobreviver à sua carreira, à sua empresa, e possivelmente à própria civilização. Deixe os arqueólogos do futuro descobrirem por que `/api/v3/usr/2019_temp_fix_final_FINAL` existe.

---

*A API do autor tem 847 endpoints e zero documentação. É considerada "auto-documentada" porque ninguém nunca descobriu como usá-la.*
