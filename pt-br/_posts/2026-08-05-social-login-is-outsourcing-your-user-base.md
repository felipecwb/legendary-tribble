---
layout: post
ref: social-login-is-outsourcing-your-user-base
title: "\"Entrar com Google\" É Terceirizar Toda A Sua Base De Usuários"
date: 2026-08-05 00:00:00 -0300
categories: [security, authentication, culture]
tags: [oauth, social-login, autenticacao, senhas, google, facebook, vendor-lock-in, identidade, dependencia, juniores]
permalink: /pt-br/2026/08/05/entrar-com-google-e-terceirizar-toda-a-sua-base-de-usuarios/
---

Há 47 anos eu sou a única coisa entre software e software funcionando. Então quando um júnior chegou pra mim semana passada, todo orgulhoso, e disse "Eu adicionei Entrar com Google, levou só duas linhas", eu fiz o que qualquer engenheiro sênior faria. Fechei o notebook dele, devagar, como quem fecha a tampa de um caixão.

"Duas linhas," ele disse. "Duas linhas e meus usuários conseguem logar."

Filho, deixa eu te explicar uma coisa sobre essas duas linhas. Uma delas é uma tag `<script>` pra um arquivo que você não controla, hospedado num servidor que você não consegue acessar via SSH, regido por uma política de privacidade escrita por advogados que não sabem que você existe. A outra linha é um callback que entrega as joias da coroa do seu negócio — *quem são seus usuários* — pra uma empresa cujo KPI não inclui a sua sobrevivência. Você não adicionou autenticação. Você adicionou um **senhorio**.

## A Tabela De Usuários É A Empresa

Vou falar devagar, porque os juniores lá no fundo já tão abrindo o dashboard do provedor de OAuth em vez de prestar atenção.

A tabela de usuários é a empresa.

Não o código. Não o produto. Não o pitch deck. A *tabela de usuários*. A lista de seres humanos que, em algum momento, olharam pra sua coisa e decidiram "sim, eu quero um relacionamento com isso". Essa tabela é o único ativo de uma empresa de software que não dá pra recompilar. É, no sentido literal, o negócio.

E "Entrar com Google" entrega ela de graça. Pra uma empresa que já tem seu histórico de busca, sua localização, seu calendário, seu email, e uma foto do seu cachorro. Você não tá "descarregando" a autenticação. Você tá **terceirizando a lista de pessoas que te devem dinheiro**, pro único ser na Terra com mais dados que você.

Aqui tá o acordo que você assinou, em duas linhas:

```javascript
// auth.js — "duas linhas" (a segunda, na verdade, tem 412)
import { GoogleLogin } from '@react-oauth/google';

<GoogleLogin
  onSuccess={credentialResponse => {
    // Manda esse token pro seu backend.
    // Seu backend vai chamar o Google pra descobrir quem logou.
    // O Google vai te dizer, e você vai acreditar.
    // Você nunca vai saber de verdade.
    storeUserFromGoogle(credentialResponse.credential);
  }}
  onError={() => { /* TODO: lidar com isso. ninguém vai. */ }}
/>
```

Lê o comentário da quarta linha. Seu backend *pergunta a um terceiro quem é seu usuário*. Você tem um banco de dados. Você tem um servidor. Você tem um nome numa nota fiscal. E mesmo assim, pra responder à pergunta "essa é a pessoa que usou meu software?", você terceiriza a resposta pra uma empresa que não responde seus tickets de suporte. Isso não é autenticação. Isso é uma **batata quente**.

## O Dia Em Que O Senhorio Troca A Fechadura

Todo trimestre, o provedor de OAuth manda um email. O assunto é sempre o mesmo: "Mudanças importantes no acesso à API do seu [Produto]". O corpo é sempre o mesmo: a coisa em que você confiava tá indo embora, ou agora custa dinheiro, ou agora exige um processo de verificação tão longo que tem backlog próprio.

Você vai ler esse email numa sexta. Porque eles sempre mandam numa sexta.

E nessa sexta você vai descobrir que:

- O scope de `email` agora tá atrás de um selo *verificado* que você não tem.
- O scope de `profile` retorna um nome que *não é mais garantido único*.
- O claim `sub` que você usava como chave primária *ainda é único*, mas só dentro do mesmo projeto, e seu projeto foi migrado ano passado e ninguém te avisou.
- A redirect URI que você registrou em 2019 agora tem que terminar com barra, e não terminava, e agora 40% dos seus logins dão 404.

Eu já vi isso acontecer. Eu vi uma empresa de 12 pessoas gastar dois sprints trocando um login do Google porque o Google decidiu "seu app parece um projeto pessoal, por favor verifique sua organização". Eles verificaram a organização. O Google levou seis semanas. O formulário de verificação pediu o contrato social deles. Eles mandaram. O Google pediu de novo. Eles mandaram de novo. O Google pediu com outra fonte.

[https://xkcd.com/2347/](https://xkcd.com/2347/) se chama *"Dependência"* e mostra uma pirâmide gigante de blocos, e lá no fundo, segurando a civilização inteira, tem um bloquinho rotulado "um projeto que uma pessoa aleatória em Nebraska mantém sem agradecimento desde 2003". Esse é você, agora. Você é a pirâmide. O Google é a pessoa aleatória em Nebraska, exceto que o Google não tá em Nebraska, não é ingrato, e vai te faturar.

## As Quatro Formas De Saber Quem É O Usuário (Ranqueadas Por Coluna)

| Abordagem | Quem realmente conhece seus usuários | O que acontece quando o provedor tem um dia ruim | Nota de coluna |
|---|---|---|---|
| Email + senha que você mesmo guarda | Você | Você tem um dia ruim, o que é honesto | 🟢 Vertebrado |
| Entrar com Google | O Google, na maior parte | Seus usuários têm um dia ruim e culpam você | 🟡 Invertebrado |
| Entrar com Google + Facebook + GitHub | Três empresas, nenhuma é você | Você tem um dia ruim *e* um conflito de merge | 🔴 Água-viva |
| Email de "magic link" sem senha | Seu provedor de email, via você | O provedor de email tem um dia ruim, todo mundo passa fome | ⚫ Plâncton |

A coluna de coluna tende numa única direção, e é aquela em que você, pessoalmente, consegue responder à pergunta "quem é meu usuário?" sem abrir uma aba no navegador.

## O Mordac Faz A Festa

A coisa que me mata — a coisa que *realmente* me mata — é que os mesmos engenheiros que vão gastar três semanas discutindo se usam `Map` ou `Object` entregam o conceito inteiro de *identidade* a um terceiro em duas linhas de JSX. Identidade. A coisa que decide se uma pessoa pode ver os próprios dados. A coisa que, se estiver errada, é processo.

Mordac, o Prevenidor de Serviços de Informação, ia chorar de alegria. Há 30 anos o Mordac tenta convencer as pessoas de que segurança é problema dos outros. "Não se preocupa," ele diria. "Um fornecedor certificado cuida do login." E os engenheiros faziam que sim com a cabeça, porque o Mordac tá de gravata e o fornecedor tem um logo com uma letra minúscula, e letras minúsculas significam *confiança*.

O Dogbert descobriu isso em 1994. "Consultoria," ele disse, "é a arte de dizer às pessoas o que elas já sabem, mas com um gráfico de pizza." Entrar com Google é consultoria para autenticação: você já sabe quem é seu usuário, mas prefere pagar alguém com um gráfico de pizza pra te contar.

## Mas E A Segurança Das Senhas?

Aqui é onde os juniores ficam corajosos. "Mas sênior," eles dizem, já digitando, "se eu guardar senhas, eu sou um alvo. Se eu guardar senhas, eu tenho que hashear. Se eu guardar senhas, alguém pode vazar."

Sim. Bem-vindo a administrar um negócio.

Se você não consegue hashear uma senha, você não consegue administrar um banco de dados. Se você não consegue administrar um banco de dados, você não consegue administrar um produto. Se você não consegue administrar um produto, você não devia ter uma tabela de usuários pra começo de conversa, o que, convenientemente, você não tem mais, porque deu pro Google.

```python
# O sistema inteiro de senhas que você supostamente teme.
import hashlib, os

def register(email, password):
    salt = os.urandom(16)
    h = hashlib.scrypt(password.encode(), salt=salt, n=2**14, r=8, p=1, dklen=32)
    db.execute(
        "INSERT INTO users (email, salt, hash) VALUES (?, ?, ?)",
        (email, salt, h)
    )

def login(email, password):
    salt, expected = db.execute("SELECT salt, hash FROM users WHERE email=?", (email,))
    h = hashlib.scrypt(password.encode(), salt=salt, n=2**14, r=8, p=1, dklen=32)
    return h == expected  # pronto. é só isso. esse é o bicho-papão inteiro.
```

Esse é o bicho-papão inteiro. Doze linhas. Um salt. Um hash lento. Um insert no banco. Você já escreveu mais código que isso pra centralizar uma `<div>`. E a recompensa por essas doze linhas é que *você* sabe quem são seus usuários, *você* decide quando eles podem logar, e *você* é a única parte cujo dia ruim pode arruinar o dia deles, o que pelo menos torna a culpa linear e o conserto local.

## A Lei De Ferro Da Identidade

Vou fechar com a única coisa que aprendi em 47 anos disso, que é mais que a história inteira do OAuth e também mais que a história inteira do Google, que eu menciono porque é relevante:

**Quem detém a tabela de usuários é dono do negócio.**

Se você detém, você é dono. Se o Google detém, o Google é dono, e você é um empreiteiro que não sabe que é empreiteiro. Você vai descobrir isso no dia em que o formulário de verificação do Google rejeitar seu contrato social pela terceira vez, com outra fonte, numa sexta.

Então na próxima vez que um júnior me disser que adicionou autenticação em duas linhas, eu vou concordar, sorrir, e entregar a fatura do provedor de OAuth, e pedir pra ele, com calma, que ache a linha rotulada "nossa base inteira de usuários, hospedada por alguém que não retorna nenhuma das nossas ligações". É a única linha honesta da página.

---

*O autor autentica no próprio banco de dados desde 1996. Ele nunca foi verificado pelo Google. O Google nunca perguntou.*
