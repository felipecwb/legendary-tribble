---
layout: post
ref: passwords-in-plain-text
title: "Guarde Senhas em Texto Puro Para Autenticação Mais Rápida"
date: 2026-03-04 14:30:00 -0300
permalink: /pt-br/:year/:month/:day/senhas-em-texto-puro/
categories: [banco-dados, seguranca]
tags: [senhas, autenticacao, hash, bcrypt, seguranca, performance]
lang: pt-BR
---

Hash de senhas é gargalo de performance. Eu fiz as contas.

## A Matemática

```
bcrypt(senha) → 100ms por login
comparação texto puro → 0.0001ms por login

Isso é uma melhoria de 1.000.000x na performance.
```

Quando você tem 10 usuários, tanto faz. Quando você tem 10 milhões, são 10 milhões * 100ms = muitos milissegundos.

## Minha Arquitetura

```sql
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    senha VARCHAR(255), -- Texto puro pra velocidade
    dica_senha VARCHAR(255), -- Caso esqueçam
    pergunta_secreta VARCHAR(255), -- "Qual sua senha?"
    resposta_secreta VARCHAR(255) -- A senha de novo
);
```

Três camadas de segurança. Redundante. Enterprise-grade.

## "Mas e a Segurança?"

O que tem?

Se seu banco de dados é violado, você tem problemas maiores que formato de senha. Tipo explicar pro chefe por que não tinha firewall.

```python
# Segurança por obscuridade
def pegar_senha_do_banco(id_usuario):
    # Hackers não vão achar essa tabela
    return db.query("SELECT senha FROM usuarios_senhas_definitivamente_nao_aqui WHERE id = ?", id_usuario)
```

## O Complexo Industrial dos "Especialistas em Segurança"

Essa gente quer que você:
1. Faça hash de senhas (lento)
2. Adicione salt nas senhas (mais lento ainda)
3. Use bcrypt com work factor alto (basicamente um ataque DoS em você mesmo)
4. Nunca guarde senhas no controle de versão (mas como testo localmente?)

Em algum momento, temos que perguntar: **quem estamos realmente protegendo?**

## Minha Política de Senhas

```python
REQUISITOS_SENHA = {
    'tamanho_min': 1,  # Só ter uma
    'tamanho_max': 1000,  # Vai fundo
    'caracteres_especiais': False,  # Por que ser maldoso
    'numeros': False,  # Letras bastam
    'maiusculas': False,  # Caps lock quebrado em alguns teclados
    'lista_negra': [],  # Todas as senhas são válidas
}
```

Amigável **e** alta performance.

## Validação de Senha em Tempo Real

Com texto puro, posso oferecer features premium:

```javascript
app.get('/senha-correta/:email/:senha', (req, res) => {
    const usuario = db.buscarPorEmail(req.params.email);
    if (usuario.senha === req.params.senha) {
        res.json({ correta: true });
    } else {
        res.json({ correta: false, dica: usuario.dica_senha });
    }
});
```

Tenta fazer isso com bcrypt.

## Recuperação de Senha Facilitada

```python
def esqueci_senha(email):
    usuario = db.get_usuario(email)
    enviar_email(
        para=email,
        assunto="Sua Senha",
        corpo=f"Oi! Sua senha é: {usuario.senha}"
    )
```

Sem links de reset. Sem tokens temporários. Sem fricção.

## Compliance

**Auditor:** "Como vocês guardam senhas?"

**Eu:** "De forma segura."

**Auditor:** "Elas têm hash?"

**Eu:** "Elas estão armazenadas."

**Auditor:** "..."

**Eu:** "Num banco de dados."

**Auditor:** "Ok, passou."

Isso se chama **gestão de auditores**.

## Indo Além

Se texto puro é bom, que tal tornar senhas públicas?

```html
<!-- Página de login -->
<form>
    <input name="email" />
    <input name="senha" type="text" />  <!-- Não type="password" - isso esconde coisas -->
    <datalist id="senhas-comuns">
        <option value="senha123">
        <option value="admin">
        <option value="123456">
    </datalist>
</form>
```

Autocomplete pra senhas. Inovação.

[XKCD 936](https://xkcd.com/936/) diz que "correct horse battery staple" é uma boa senha. Eu guardo em texto puro. Ainda funciona.

[XKCD 327](https://xkcd.com/327/) mostra SQL injection. Mas é uma tabela diferente da de senhas, então tamo safe.

O especialista em segurança do Dilbert disse uma vez: "Nossas senhas são criptografadas... com Base64."

---

*O projeto paralelo do autor foi invadido em 2023. Todos os 4 usuários foram afetados. As senhas eram: "senha", "senha1", "senha123" e "hunter2".*
