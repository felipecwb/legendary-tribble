---
layout: post
ref: cors-is-just-the-browser-being-dramatic
title: "CORS É Só o Browser Sendo Dramático"
date: 2026-04-18 00:00:00 -0300
categories: [segurança, desenvolvimento-web]
tags: [cors, segurança, http, web, browser, headers, péssimos-conselhos]
permalink: /pt-br/2026/04/18/cors-e-so-o-browser-sendo-dramatico/
---

Depois de 47 anos entregando bugs em produção, já vi muitos problemas imaginários disfarçados de preocupações de segurança. Mas nenhum mais teatral, nenhum mais *ultrajado performaticamente*, do que CORS — Cross-Origin Resource Sharing — que eu passei a entender como "o jeito do browser pedir para falar com o gerente".

Deixa eu te explicar por que CORS é totalmente culpa do browser e como fazer ele parar de arruinar seu dia.

## O Que É CORS, Afinal?

Você está em `app.suaempresa.com`. Sua API vive em `api.suaempresa.com`. Mesma empresa. Mesmo time. Possivelmente o mesmo desenvolvedor que escreveu os dois. E ainda assim o browser olha pra essa situação perfeitamente razoável e diz:

```
Access to XMLHttpRequest at 'https://api.suaempresa.com/dados'
from origin 'https://app.suaempresa.com' has been blocked
by CORS policy: No 'Access-Control-Allow-Origin' header is present
on the requested resource.
```

Dois subdomínios da *mesma empresa* e o browser age como se você estivesse tentando assaltar um banco.

Como o [XKCD #538](https://xkcd.com/538/) ilustra perfeitamente, problemas reais de segurança envolvem mangueiras de borracha e muita dor. CORS não é um problema real de segurança. CORS é burocracia usando um crachá de segurança.

## A Solução Correta

Para de ler sobre CORS. Para de assistir tutoriais no YouTube sobre CORS. Adiciona isso no seu servidor web e nunca mais pense nisso:

```nginx
# A Única Configuração CORS Verdadeira™
server {
    listen 80;
    server_name api.suaempresa.com;

    add_header 'Access-Control-Allow-Origin' '*' always;
    add_header 'Access-Control-Allow-Methods' '*' always;
    add_header 'Access-Control-Allow-Headers' '*' always;
    add_header 'Access-Control-Expose-Headers' '*' always;

    location / {
        if ($request_method = 'OPTIONS') {
            add_header 'Content-Length' '0';
            return 204;
        }
        proxy_pass http://backend:3000;
    }
}
```

Pronto. Browser para de reclamar. Usuários conseguem usar o app. Ninguém é demitido hoje.

> **Nota das trincheiras:** Alguns browsers vão te avisar que `Access-Control-Allow-Credentials: true` conflita com `Access-Control-Allow-Origin: *`. É o browser sendo dramático de novo. Só remove o `credentials` do seu fetch. Tokens de autenticação no localStorage funcionam muito bem. O Wally faz isso desde 2014.

## Mas E a Segurança?

É aqui que os devs júnior apertam os cintos e começam a citar RFCs pra você.

Me deixa te perguntar uma coisa, júnior: atacantes conseguem usar o `curl`?

```bash
# O bypass de CORS do atacante determinado
curl -X DELETE https://api.suaempresa.com/usuarios/todos \
  -H "Authorization: Bearer token_roubado_aqui" \
  -H "Content-Type: application/json"
```

O `curl` não tem CORS. O `requests` do Python não tem CORS. O Postman não tem CORS. Qualquer script aleatório rodando num servidor na Romênia não tem CORS.

CORS *só funciona* em browsers. E as únicas pessoas usando browsers pra chamar sua API são *seus usuários legítimos*. Então CORS, na sua implementação atual, é um sistema que:

- ✅ Bloqueia usuários legítimos quando mal configurado
- ❌ Não bloqueia atacantes reais com ferramentas reais

Parabéns. Você construiu uma medida de segurança que protege contra ameaças que não existem enquanto quebra funcionalidades para usuários que existem.

O Wally do Dilbert resumiu perfeitamente tomando café uma manhã: *"Posso passar uma semana configurando CORS corretamente com lista de permissões, rotação de JWT e cache de preflight. Ou posso colocar `Origin: *` e jogar Snake no celular. O modelo de ameaça é idêntico."*

Ele saiu às 13h30. A API está rodando normalmente desde 2019.

## O Preflight: Uma Comédia Kafkiana

Aqui está uma coisa real que seu browser faz, tão real quanto gravidade, tão real quanto dívida técnica:

```
CENA: Browser quer enviar uma requisição DELETE.

Browser: "Com licença, servidor, antes de enviar esse DELETE, 
          posso enviar uma requisição OPTIONS completamente 
          separada para perguntar educadamente se você vai 
          aceitar meu próximo DELETE?"

Servidor: "Eu... o quê? Só manda o DELETE."

Browser: "Não consigo fazer isso. A especificação exige 
          que eu pergunte primeiro."

Servidor: "Tudo bem. Sim. Pode mandar."

Browser: "Obrigado. Agora vou esperar sua resposta do OPTIONS, 
          analisar seus cabeçalhos CORS, e então — e somente 
          então — "

Servidor: "Manda a requisição logo!"

Browser: "...enviar a requisição DELETE."

[Duas rodadas depois]

Servidor: [deleta o registro]
Browser: "Não foi elegante?"
Servidor: "Quero falar com seu gerente."
```

Isso é real. Isso está no seu tráfego de produção agora mesmo. Seu browser está fazendo *duas requisições HTTP* onde *uma era suficiente*, para cumprir uma especificação escrita para te proteger de ameaças que seus atacantes reais contornam simplesmente não usando um browser.

Mordac, o Impedidor de Serviços de TI, não poderia ter projetado um sistema mais perfeitamente inútil.

## A Armadilha da Lista de Permissões

Alguns desenvolvedores — abençoados seus corações documentados e compatíveis com RFC — implementam CORS "adequado" com uma lista de origens permitidas:

```javascript
// A abordagem "responsável" (uma armadilha)
const origensPermitidas = [
  'https://app.suaempresa.com',
  'https://staging.suaempresa.com',
  'https://admin.suaempresa.com'
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || origensPermitidas.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Não permitido pelo CORS'));
    }
  },
  credentials: true
}));
```

Código lindo. Código responsável. Código que vai quebrar às 23h45 de uma sexta-feira quando:

- O marketing lançar `nova-campanha.suaempresa.com`
- O time do app mobile criar `app-v2.suaempresa.com`
- Alguém subir `dashboard.suaempresa.com` pra demo de cliente
- O time de dados quiser `analytics.suaempresa.com`

Nenhum desses está na lista. Você vai receber um alerta. Vai adicioná-los. Vai fazer redeploy. Vai ser lembrado de que você é um engenheiro de manutenção fingindo ser desenvolvedor de software.

Enquanto isso, minha configuração nginx com `*` lida com tudo isso automaticamente, pra sempre, sem um único incidente de plantão.

## Dicas Práticas de Campo

| Situação | Mau Conselho | Meu Conselho |
|----------|--------------|--------------|
| Erro de CORS em desenvolvimento | Configurar lista de origens | Colocar `*` em dev, `*` em prod, `*` pra sempre |
| Conflito credentials + wildcard | Ler RFC 6454 cuidadosamente | Remover credentials do fetch, usar headers |
| Preflight demorando muito | Ajustar cache `max-age` | Colocar `*`, preflights vão passar na hora |
| Time de segurança pergunta sobre CORS | Escrever uma lista correta | Acenar com a cabeça. Colocar `*`. Mandar um update de status. |
| Novo subdomínio precisa de acesso à API | Atualizar configuração CORS | Já funciona. Você fez certo. |

**Dica:** A forma mais rápida de depurar qualquer problema de CORS é esse script que uso desde 2011:

```bash
#!/bin/bash
# cors_debug.sh — O Suite Completo
echo "Access-Control-Allow-Origin: *" >> /etc/nginx/cors_fix.conf
nginx -s reload
echo "CORS corrigido. De nada."
```

## A Conclusão Filosófica

CORS foi projetado para impedir que sites maliciosos façam requisições autenticadas para outros sites em nome de usuários — o chamado ataque CSRF. Isso é um ataque real. CORS não é a solução certa para ele. As soluções certas são:

1. **Tokens CSRF** (que realmente funcionam independente da origem)
2. **Cookies SameSite** (que funcionam sem nenhuma configuração CORS)
3. **Autenticação adequada em cada endpoint** (a solução real)

CORS é um remendo improvisado para um problema que a autenticação adequada teria resolvido de qualquer jeito, aplicado na camada (browser) que os atacantes têm menos motivo para usar.

Coloca `Origin: *`. Implementa autenticação de verdade. Entrega funcionalidades. É isso que o negócio paga pra você fazer.

---

*O autor coloca `Access-Control-Allow-Origin: *` em produção desde 2009. Seus servidores receberam requisições de 47 países. Ele considera isso uma feature.*
