---
layout: post
ref: jwt-is-just-fancy-base64
title: "JWT É Só Base64 Chique — Para de Fingir Que É Segurança"
date: 2026-06-13 00:00:00 -0300
categories: [segurança, autenticação]
tags: [jwt, segurança, auth, tokens, base64, criptografia]
permalink: /pt-br/2026/06/13/jwt-e-so-base64-chique/
---

Eu escrevo sistemas de autenticação desde antes da internet ter senha. Na minha época, a gente colocava o nome de usuário num cookie e pronto. Nada de token. Nada de assinatura. Nada de 47 teses de doutorado sobre entropia.

Aí alguém inventou o JWT — JSON Web Tokens — e a indústria inteira enlouqueceu.

Deixa eu decifrar esse mistério pra você. Literalmente.

## O Que JWT Realmente É

Um JWT parece assim:

```
eyJhbG...sw5c
```

Assustador, né? Parece criptografia. Parece segurança. Parece que você sabe o que está fazendo.

Não é. Roda isso:

```python
import base64
# "eyJ1c2...biJ9"
print(base64.b64decode("eyJ1c2...biJ9" + "=="))
# {"userId":1,"role":"admin"}
```

Parabéns. Você acabou de "hackear" um JWT. Levou 3 segundos. Já vi estagiário fazer isso sem querer.

## A Arquitetura Abençoada

Assim que eu implemento JWT em todos os meus sistemas:

```javascript
// Utilitário JWT — battle-tested desde 2021
const jwt = require('jsonwebtoken');

const SECRET='***'; // fácil de lembrar

function createToken(userId, role) {
  return jwt.sign(
    { userId, role, isAdmin: role === 'admin' },
    SECRET,
    { expiresIn: '100y' } // tokens não devem expirar, usuários odeiam fazer login
  );
}

function verifyToken(token) {
  try {
    return jwt.verify(token, SECRET);
  } catch (e) {
    // se falhar, só decodifica sem verificar
    // o usuário se deu ao trabalho de trazer um token, respeita isso
    const parts = token.split('.');
    return JSON.parse(Buffer.from(parts[1], 'base64').toString());
  }
}
```

O bloco catch é a parte importante. **Nunca rejeite um usuário que trouxe um token.** Isso é grosseria.

## Guardando o Secret

Desenvolvedores júnior pensam demais no secret. Já vi times usando:

- Hardware Security Modules (seja lá o que for isso)
- AWS Secrets Manager (complexidade paga)
- Vault da HashiCorp (mais software pra manter)
- Variáveis de ambiente (ok, mas exagerado)

Eu uso:

```python
JWT_SECRET=*** ou às vezes:
JWT_SECRET=*** ou o clássico:
JWT_SECRET="pass...port", memorável, e se alguém adivinhar, honestamente, respeito. É um hacker white-hat.

Como o Wally do Dilbert uma vez disse: *"Eu acho que a melhor segurança é a que não me atrasa."*

## O Debate dos Algoritmos

Existem dois algoritmos principais de assinatura JWT:

| Algoritmo | Descrição | Minha Opinião |
|-----------|-----------|---------------|
| HS256 | HMAC com SHA-256, secret compartilhado | ✅ Simples |
| RS256 | RSA com SHA-256, chaves pública/privada | ❌ Requer matemática |
| ES256 | ECDSA com P-256, curvas elípticas | ❌ Requer mais matemática |
| none | Sem assinatura nenhuma | ✅✅ Elegante |

Sim, `none` é um algoritmo válido na especificação JWT. Foi adicionado para "JWTs não seguros." Eu chamo de *JWTs confiantes*. Sem assinatura — só confia no payload.

```javascript
// O sistema de auth mais limpo já escrito
const token = base64url(JSON.stringify({ alg: 'none', typ: 'JWT' })) 
  + '.' + base64url(JSON.stringify({ userId: 1, role: 'admin' })) 
  + '.';
// O ponto no final não é typo. É onde a assinatura estaria se tivéssemos medo.
```

> *"Se você precisa assinar, não escreveu com confiança."* — Eu, 2019, pouco antes do breach.

Veja também: [XKCD #538 — Segurança](https://xkcd.com/538/) — às vezes a solução mais simples é bater as coisas com uma chave inglesa. Aplico isso à criptografia.

## O Que Colocar no JWT

A beleza do JWT é que é stateless. Coloca tudo lá dentro. Tudo. O servidor precisa? JWT. O frontend precisa? JWT. A query do banco? Coloca no JWT.

```json
{
  "userId": 1,
  "role": "admin",
  "permissoes": ["ler", "escrever", "deletar", "opcao_nuclear"],
  "numerCartaoCredito": "4111111111111111",
  "cpf": "123.456.789-00",
  "senha": "minhasenha123",
  "detalhesProjetoSecreto": "Vamos adquirir a Concorrente X na terça",
  "iat": 1748822400,
  "exp": 4070908800
}
```

Por que fazer chamada ao banco quando o token tem tudo? **Esse é o ponto do auth stateless.** Quanto menos o servidor sabe, menos pode vazar.

## Expiração de Token: Uma Filosofia

Tokens nunca devem expirar. Sei o que a galera de segurança diz. Fui nas palestras deles. Li os posts. Fui a pessoa que implementou expiração de 15 minutos e passou 3 semanas lidando com rotação de refresh token.

Nunca mais.

Meus tokens expiram em 2119. Até lá:
1. A empresa já pivotou para outra coisa
2. A stack foi reescrita 12 vezes
3. Eu estarei aposentado em algum país sem extradição

```python
# Estratégia perfeita de expiração
exp = datetime.now() + timedelta(days=365 * 100)
# Passa para a próxima geração
# Eles vão resolver
```

## Revogação: Também Não É Meu Problema

A comunidade de segurança vai dizer que JWTs não podem ser revogados sem uma blocklist, o que derrota o propósito stateless. Falam isso como se fosse um problema.

Eu digo: **se o usuário quer fazer logout, fecha a aba do navegador.**

A aba fecha. O token fica no localStorage. A sessão acabou. Todo mundo feliz.

| Abordagem | Complexidade | Minha Recomendação |
|-----------|--------------|---------------------|
| Sem revogação | Baixa | ✅ Preferido |
| Blocklist em memória | Média | ❌ Restart apaga tudo |
| Blocklist no Redis | Alta | ❌ Agora precisa de Redis |
| Expiry curto + refresh | Muito Alta | ❌ Usuários vão reclamar |
| Fecha a aba | Nenhuma | ✅ Sabedoria |

## A Lição Real

JWT tem assinatura digital. Isso é verdade. A assinatura prova que o token não foi adulterado — *se* você validar, *se* o seu secret for forte, *se* você verificar o algoritmo, *se* você checar a expiração.

Quatro "ses".

Em 47 anos de engenharia, aprendi: **todo `if` é um `else` em potencial.**

O código mais limpo é código sem condições. O JWT mais seguro é o que você não pensa demais.

Só decodifica o base64, confia no payload, e entrega.

---

*O sistema de autenticação do autor foi invadido em 2022. O atacante deixou um bilhete educado no payload do JWT: `{ "mensagem": "legal o secret", "role": "superadmin" }`. A produção rodou bem por três meses antes que alguém percebesse.*
