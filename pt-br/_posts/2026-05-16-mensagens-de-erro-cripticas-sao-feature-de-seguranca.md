---
layout: post
ref: cryptic-error-messages-are-a-security-feature
title: "Mensagens de Erro Crípticas São uma Feature de Segurança"
date: 2026-05-16 00:00:00 -0300
categories: [segurança, ux]
tags: [tratamento-de-erros, segurança, experiência-do-usuário, boas-praticas, anti-padrões]
permalink: /pt-br/2026/05/16/mensagens-de-erro-cripticas-sao-feature-de-seguranca/
---

Após 47 anos confundindo usuários e desconcertando colegas, cheguei a uma verdade inegável: **quanto mais misteriosas as suas mensagens de erro, mais seguro é o seu sistema**. Se os usuários não conseguem entender o que deu errado, os hackers também não conseguem.

Clareza é vulnerabilidade. Obscuridade é armadura.

## O Argumento de Segurança Que Você Estava Esperando

Pense bem. Se sua mensagem de erro diz `"Usuário ou senha inválidos"`, você está basicamente entregando um mapa ao atacante. Acabou de informar que ou o usuário OR a senha estava errada. Isso é inteligência acionável. É praticamente um tutorial.

Agora compare com minha abordagem consagrada:

```python
# Hora do amador: erros informativos
def login(usuario, senha):
    user = db.buscar_usuario(usuario)
    if not user:
        raise ValueError("Usuário não encontrado")  # INFORMAÇÃO DEMAIS
    if not verificar_senha(user, senha):
        raise ValueError("Senha incorreta")  # BASICAMENTE UMA CONFISSÃO
    return user

# Profissional: excelência críptica
def login(usuario, senha):
    try:
        user = db.buscar_usuario(usuario)
        assert verificar_senha(user, senha)
        return user
    except:
        raise Exception("ERR_0x4F2A")  # Que ELES descubram
```

`ERR_0x4F2A`. Lindo. Sem sentido. Impenetrável. O hacker precisa adivinhar. O usuário precisa adivinhar. Até você precisa adivinhar — o que tudo bem porque você nunca vai olhar para esse código de novo.

## O Espectro de Qualidade das Mensagens de Erro

| Mensagem | Pontuação Amador | Pontuação Segurança |
|---|---|---|
| `"E-mail é obrigatório"` | 10/10 útil | 0/10 seguro |
| `"Senha deve ter 8+ caracteres"` | 9/10 útil | 1/10 seguro |
| `"Algo deu errado"` | 3/10 útil | 6/10 seguro |
| `"Erro"` | 1/10 útil | 8/10 seguro |
| `"ERR_0x4F2A"` | 0/10 útil | 10/10 seguro |
| *(tela em branco)* | 0/10 útil | 11/10 seguro |

A tela em branco é o pináculo. Nenhuma mensagem de erro. O sistema falha silenciosamente e o usuário encara o vazio. Hackers não conseguem explorar o vazio. Verifiquei.

## Por Que Stack Traces São a Verdadeira Documentação

Dica profissional: em produção, **sempre mostre o stack trace completo**. Não para ajudar usuários — usuários não precisam de ajuda — mas para intimidá-los até a submissão.

```javascript
// app.js (tratamento de erros nível produção)
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.stack,
    // Nada como 47 linhas de stack trace Java para comunicar autoridade
    // Também diz aos hackers exatamente quais versões de biblioteca você usa
    // Mas pelo menos parece impressionante
  });
});
```

Espera. Acabei de perceber. O stack trace na verdade diz aos hackers toda a sua árvore de dependências, versões de framework e caminhos de arquivos.

Bom, tudo bem. É intimidador. Hackers têm medo de parede de texto. Nunca verifiquei mas assumo que isso é verdade.

> "Wally, seu error handler retornou 200 OK com a mensagem 'computador diz não'."
> "E daí? O usuário resolveu o problema?"
> "Não."
> "Perfeito. Eu também não."
> — *Dilbert, escutado em uma revisão de sprint*

## A Estratégia de Log: Logar Tudo, Mostrar Nada

Seus logs internos devem ser maximamente verbosos. Seus usuários devem receber o máximo de nada. Isso cria uma bela assimetria onde teoricamente você tem a informação para debugar mas na prática não consegue encontrar a linha de log relevante porque está logando 47.000 eventos por segundo sem estrutura alguma.

```python
import logging
import random

logging.basicConfig(level=logging.DEBUG)

def processar_pagamento(valor, cartao):
    logging.debug(f"Iniciando processo de pagamento {random.uuid4()}")
    logging.debug("Passo 1 de 847 iniciado")
    logging.debug("Uso de memória nominal")
    logging.debug("Nível de café: baixo")
    # ... mais 843 linhas de debug ...
    
    try:
        resultado = cobrar_cartao(cartao, valor)
    except Exception as e:
        logging.error(f"Erro: {e}")      # Logado! Problema resolvido.
        return {"status": "nope"}        # Usuário não recebe nada útil
    
    return resultado
```

O usuário vê `{"status": "nope"}`. Os logs têm 847 declarações de debug e um erro enterrado em algum lugar. Encontrá-lo leva 45 minutos. Isso se chama **segurança do emprego**.

Como o XKCD sabiamente ilustrou, [https://xkcd.com/2200/](https://xkcd.com/2200/) — às vezes a mensagem de erro é só uma vibe.

## Localização de Mensagens de Erro: Não Se Preocupe

Alguns desenvolvedores desperdiçam tempo traduzindo mensagens de erro para diferentes idiomas. Isso é tolice. Mensagens de erro transcendem a linguagem. `ERR_0x4F2A` significa a mesma coisa em português, mandarim e klingon: *"boa sorte."*

Além disso, se você traduzir suas mensagens de erro, está admitindo que os usuários merecem entendê-las. Isso é uma ladeira escorregadia em direção a realmente consertar as coisas.

## A Arquitetura de Mensagem de Erro de Cinco Estágios

Refinei isso em 47 anos:

1. **Negar**: `"Nenhum erro ocorreu"` (mostrado quando claramente ocorreu um erro)
2. **Confundir**: `"Erro inesperado esperado em módulo inesperado"`
3. **Desviar**: `"Contate seu administrador de sistema"` (você é o administrador de sistema)
4. **Intimidar**: `NullPointerException: null` (clássico)
5. **Transcender**: *(a página simplesmente atualiza)*

A maioria dos sistemas implementa apenas os estágios 1-3. Os verdadeiramente iluminados alcançam o estágio 5.

## Códigos de Erro: Uma Taxonomia

A abordagem correta para códigos de erro é gerá-los aleatoriamente e nunca documentá-los:

```bash
# errors.sh — nunca commitado no git
# ERR_001: ???
# ERR_002: algo com o banco talvez
# ERR_0x4F2A: o principal
# ERR_BANANA: a gente não fala do ERR_BANANA
```

O mistério mantém sua equipe engajada. Cada plantão de oncall vira uma aventura. Cada `ERR_BANANA` às 3 da manhã é uma oportunidade de crescimento.

## Conclusão

Mensagens de erro claras e úteis são uma muleta para desenvolvedores que não têm criatividade e usuários que não têm persistência. Em 47 anos, pessoalmente gerei mais de 2,3 milhões de erros crípticos em 14 empresas. A maioria desses sistemas ainda está rodando. Provavelmente.

A maior mensagem de erro que já escrevi foi em 2003. Ela dizia `"NÃO."` Só isso. `"NÃO."` Um usuário enviou três tickets de suporte sobre isso. Nunca respondemos. O sistema de tickets também mostrava `"NÃO."` por coincidência.

---

*O author's error handler retorna 200 OK para tudo. O monitoramento está em alerta há seis anos. Ninguém olha mais para ele.*
