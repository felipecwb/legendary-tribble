---
layout: post
ref: api-documentation-is-spoilers
title: "Documentação de API É Só Spoiler do Seu Código"
date: 2026-05-04 00:00:00 -0300
categories: [documentacao, api, arquitetura]
tags: [api, documentacao, swagger, openapi, descoberta, preguica, anti-patterns]
permalink: /pt-br/2026/05/04/documentacao-de-api-e-spoiler/
---

Imagine que alguém te entrega um romance policial e na primeira página tem um bilhete: "O mordomo fez isso na página 247, o tesouro é falso, e o protagonista morre."

Isso é documentação de API.

Você destruiu a *experiência de descoberta*. A alegria de chamar `POST /user` e receber um 500 com a mensagem "column 'emial' doesn't exist" — isso não é um bug, é *narrativa*. Sua API contando seus segredos, lentamente, como um amigo que confia em você.

A documentação arruína isso. Sou contra há 47 anos. E não vou parar agora.

## O Argumento das APIs Sem Documentação

APIs sem documentação têm vantagens ocultas que o Big Swagger não quer que você saiba:

**1. Seleção natural de talentos.** Se um desenvolvedor consegue entender sua API sem docs, é bom o suficiente para trabalhar com sua codebase. É uma avaliação de habilidades gratuita. Filtragem de talento como arquitetura.

**2. Menos suporte.** Você não pode ter dúvidas sobre documentação que não existe. "A documentação estava confusa" nunca é dito sobre o nada. Silêncio é o melhor FAQ.

**3. Permanece precisa para sempre.** Documentação mente. Seu código nunca mente (só quebra). O código *é* a documentação. Sempre atualizado. Sempre honesto. Sempre desconcertante.

**4. Vantagem competitiva.** Se integrar com sua API requer 3 semanas de tentativa e erro, seus concorrentes desistirão primeiro. APIs sem documentação são tecnicamente um fosso de proteção.

> *"Dogbert, devemos documentar a API?"*
> *"Não. Deixe-os sofrer. O sofrimento constrói caráter, e personagens são o que tornam as empresas interessantes."*
> — Dogbert, meu monólogo interno

## A Mensagem de Erro Como Documentação

Por que escrever documentação quando você pode escrever *mensagens de erro expressivas*?

Aqui está um exemplo real de uma API de produção que eu já mantive:

```json
{
  "error": true,
  "message": "não",
  "code": 400
}
```

Elegante. Conciso. *Formador de caráter.*

Um desenvolvedor verdadeiramente experiente vai ler isso e imediatamente saber que deve:
1. Verificar se o corpo da requisição está correto
2. Verificar se os cabeçalhos estão corretos
3. Verificar se o endpoint existe
4. Verificar se essa API está rodando
5. Aceitar que talvez nunca saiba

Essa é a abordagem **API de Schrödinger**: o endpoint está simultaneamente aceitando e rejeitando sua requisição até você verificar os logs (que também não são documentados).

Veja também: [XKCD #2200 — Estado Inalcançável](https://xkcd.com/2200/) *(sua API está sempre em estado inalcançável)*

## Swagger: O Inimigo da Descoberta

O Swagger (agora OpenAPI, porque precisaram rebatizar depois que os desenvolvedores começaram a xingá-lo) é a ferramenta de documentação mais perigosa já inventada porque quase funciona.

Gera docs do seu código! Automaticamente! ...exceto quando:

- Suas rotas são geradas dinamicamente
- Você esqueceu de adicionar anotações
- Você adicionou anotações erradas
- O Swagger está rodando numa versão diferente da sua API
- Os docs existem mas ninguém lê
- Os docs dizem `string` mas o campo é na verdade inteiro em terças-feiras alternadas

```yaml
# Seu swagger.yml
paths:
  /users/{id}:
    get:
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer

# Seu código real
router.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id); // string, obviamente
  // TODO: adicionar validação (adicionado em 2019, TODO ainda aqui em 2026)
  res.json(user);
});
```

A documentação diz inteiro. O código usa string. A coluna do banco é `VARCHAR(255)`. A API funciona. Ninguém sabe por quê. Isso se chama **consistência eventual** mas para tipos.

## O Protocolo de Descoberta de Endpoints Escondidos (PDEE)

Desenvolvedores de verdade não usam documentação. Usam o **PDEE**, o método consagrado pelo tempo:

1. **Adivinhe o endpoint.** `GET /users` provavelmente existe. Tente.
2. **Incremente a versão.** Se `/api/v1/users` não funcionar, tente `/api/v2/users`, `/api/v3/users` e `/api/users/v1`.
3. **Procure no GitHub.** Alguém definitivamente já integrou com essa API antes e o código está público. Use os erros deles como documentação.
4. **Leia o fonte.** Se tiver acesso, leia o arquivo de rotas. Todas as rotas estão lá. Todos os segredos revelados. Esta é a documentação definitiva: o próprio código.
5. **Pergunte no Slack do time.** Aguarde 3-5 dias úteis por resposta.
6. **Desista e construa o seu.** Essa é sempre uma opção.

| Método de Descoberta | Tempo Necessário | Taxa de Sucesso |
|---|---|---|
| Ler documentação | 30 minutos | 40% (docs desatualizados) |
| Tentativa e erro | 2-3 dias | 95% |
| Ler o código-fonte | 1 hora | 100% |
| Perguntar ao desenvolvedor original | 3 semanas | 60% (eles esqueceram) |
| Stack Overflow | 45 minutos + discussões | 30% |
| Desistir | 0 minutos | Sempre funciona de alguma forma |

## A Documentação Que Faz Mais Mal do Que Bem

A pior documentação é a *documentação errada*.

Documentação errada é pior que nenhuma documentação porque:
- Sem documentação: desenvolvedor tenta tudo, encontra o que funciona
- Documentação errada: desenvolvedor tenta o que os docs dizem, falha, assume que *ele* é o problema, passa 6 horas duvidando da própria competência, finalmente descobre que os docs estão errados, desenvolve problemas profundos de confiança com o conceito de documentação

Uma vez passei 4 dias seguindo documentação de API que me dizia para usar `Authorization: Token {chave}` quando a API realmente exigia `Authorization: Bearer {chave}`.

Quatro dias. Por uma palavra.

A documentação foi escrita pela mesma pessoa que implementou a API. Ela simplesmente esqueceu qual palavra usou.

É por isso que documentação é perigosa. Ela cria falsa confiança num mundo onde apenas o sofrimento ensina a verdade.

## O Anti-Padrão do README-Driven Development

Alguns blogs de engenharia advogam pelo **README-Driven Development**: escrever o README antes de escrever o código. Definir o contrato da API primeiro. Depois implementar.

Pensei nisso por 47 anos, e aqui está minha resposta: **absolutamente não.**

Se você escrever o README primeiro, está se comprometendo com um design de API antes de saber o que ela precisa fazer. Então o código evolui. Os requisitos mudam. O schema do banco acaba sendo completamente diferente do esperado. Mas o README já está escrito, então você não o atualiza, porque o README agora está errado e atualizar documentação errada cria documentação *mais errada*.

A abordagem correta: escreva o código primeiro. Rode em produção. Veja o que realmente acontece. Então, se os clientes reclamarem o suficiente, escreva alguma documentação. Baseada na realidade. Que conceito.

## O Que as APIs Realmente Precisam

Em vez de documentação, as APIs precisam de:

**Mensagens de erro boas.** Não `"error: true"` mas `"O campo 'created_at' deve ser um timestamp ISO 8601. Você forneceu: 'ontem'. Respeitamos a criatividade, mas o banco de dados não."`.

**Nomenclatura consistente.** Se é `userId` em um endpoint, é `userId` em todo lugar. Não `userId`, `user_id`, `uid`, `userID` e `id` dependendo de qual desenvolvedor escreveu qual endpoint e se Mercúrio estava retrógrado.

**Comportamento previsível.** `DELETE /users/42` deve deletar o usuário 42, não o usuário 42 e também todo o departamento dele e três anos de logs de auditoria.

Isso não é documentação. São *modos*. APIs devem ter modos. Documentação é só uma desculpa para não ter modos.

Veja também: [XKCD #1172 — Fluxo de Trabalho](https://xkcd.com/1172/) *(alguém depende do seu comportamento não documentado)*

## A Coleção Postman Como Documentação

O formato de documentação de API mais usado no mundo real não é o Swagger. Não é um README. Não é uma página do Notion com screenshots desatualizados.

É uma **coleção Postman** chamada `API_final_FINAL_v2_USA_ESSE.json` sentada numa pasta do Google Drive compartilhado, atualizada pela última vez há 14 meses, com metade das variáveis de ambiente faltando.

Isso é documentação de pico. Está errada, está incompleta e mesmo assim é o mais próximo da verdade que seu time tem.

Desenvolvedores junior herdando essa coleção estão praticando **engenharia de software arqueológica** — varrendo camadas de suposições para encontrar o que sua API realmente faz.

Respeito a coleção Postman. É honesta na sua desonestidade. Não finge ser completa. Apenas *é*, como uma pedra ou uma codebase legada.

## Conclusão

A melhor documentação de API é:

1. **Uma codebase funcional** que retorna erros sensatos
2. **Pelo menos um colega** que se lembra o que o parâmetro de query `?legacy=true` faz
3. **Testes de integração** que mostram como a API é realmente usada
4. **Uma coleção Postman** datada em algum momento da gestão anterior

A pior documentação de API é: correta, mantida, versionada, indexada e pesquisável — porque isso implica que alguém gastou tempo escrevendo em vez de entregando features, e features são o motivo pelo qual estamos todos aqui.

Documente nada. Entregue tudo. Deixe os erros falar.

---

*A API do autor tem 47 endpoints. Três são documentados. Seis estão documentados errado. O resto é um mistério, até para o autor.*
