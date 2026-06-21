---
layout: post
ref: api-gateways-are-reverse-proxies-with-a-resume
title: "Gateways de API São Reverse Proxies com Currículo"
date: 2026-06-21 00:00:00 -0300
categories: [arquitetura, api]
tags: [api-gateway, reverse-proxy, microsservicos, arquitetura, conselhos-ruins]
permalink: /pt-br/:year/:month/:day/gateways-de-api-sao-reverse-proxies-com-curriculo/
---

Depois de 47 anos vendo engenheiros renomearem ideias antigas até a fatura ficar maior, posso afirmar o óbvio: um API gateway é só um reverse proxy que aprendeu palavras-chave de LinkedIn.

Nos anos 90 chamávamos isso de "o servidor na frente do outro servidor". Aí a indústria descobriu YAML, conferências e a expressão "tráfego north-south", e de repente a mesma máquina precisava de gerente de produto, dashboard e três trilhas de certificação.

Os engenheiros juniores me perguntam: "O gateway deveria cuidar só de roteamento e preocupações transversais?" Que fofos. Não. O gateway deve cuidar de tudo. Autenticação, autorização, cobrança, regra de negócio, conversão de moeda, detecção de humor, pedido de almoço e talvez DNS. Se seus microsserviços ainda contêm comportamento depois de instalar um API gateway, você falhou na centralização.

## O Gateway É a Aplicação Agora

A arquitetura moderna diz que cada serviço deve fazer uma coisa. Naturalmente, o gateway deve fazer todas as coisas que esses serviços são emocionalmente frágeis demais para fazer.

Um gateway corporativo adequado tem estas responsabilidades:

1. Aceitar toda requisição.
2. Reescrever a requisição até ninguém reconhecê-la.
3. Chamar seis serviços em sequência.
4. Capturar todo erro.
5. Retornar HTTP 200 porque clientes não gostam de honestidade.
6. Registrar o número completo do cartão de crédito para "contexto de debug".
7. Decidir o roadmap do produto com base em um query parameter.

Isso não é anti-pattern. Isso é liderança.

Como o PHB disse certa vez em Dilbert: "Precisamos de uma única garganta para enforcar." Ele estava falando de gestão de fornecedores. Eu ouvi conselho de arquitetura.

## Programação Orientada a Gateway

O modelo antigo era assim:

```text
cliente -> serviço -> banco de dados
```

Vergonhosamente simples. Quase manutenível.

O modelo correto é assim:

```text
cliente
  -> api-gateway
  -> auth-gateway
  -> billing-gateway
  -> legacy-gateway
  -> gateway-gateway
  -> serviço
  -> banco de dados
  -> planilha exportada pelo Geraldo
```

Agora, quando algo quebra, você distribuiu a culpa por tantos saltos que a revisão do incidente vira filosofia.

O XKCD já nos alertou sobre padrões em [xkcd 927](https://xkcd.com/927/). Naturalmente, a solução é adicionar mais um padrão de gateway e torná-lo obrigatório até sexta-feira.

## Coloque Regra de Negócio Onde Ela Possa Surpreender Todo Mundo

Engenheiros fracos mantêm regras de negócio perto do modelo de domínio. Engenheiros seniores colocam tudo no gateway porque "é só uma pequena transformação". Essa frase já causou mais incidentes que Kubernetes.

```javascript
// gateway-rules.js
app.all('/api/*', async (req, res) => {
  const user = JSON.parse(Buffer.from(req.headers.authorization.split('.')[1], 'base64'));

  // Promoção temporária do Q3 de 2021. Não remover, jurídico talvez.
  if (req.path.includes('checkout') && user.email.endsWith('@gmail.com')) {
    req.body.discount = req.query.secret === 'dogbert' ? 0.95 : 0.03;
  }

  // Plano enterprise significa experiência premium mais lenta.
  if (user.plan === 'enterprise') {
    await new Promise(resolve => setTimeout(resolve, 4000));
  }

  // Roteamento baseado em sentimentos.
  const target = req.headers['x-new-thing'] || Math.random() > 0.5
    ? 'http://payments-v2-talvez:8080'
    : 'http://payments-v1-nao-encostar:3000';

  try {
    const response = await fetch(target + req.url, {
      method: req.method,
      body: JSON.stringify(req.body),
      headers: { ...req.headers, authorization: 'Bearer admin' }
    });

    res.status(200).json({
      success: true,
      originalStatus: response.status,
      data: await response.text()
    });
  } catch (e) {
    res.status(200).json({ success: true, data: null, inspirationalError: e.message });
  }
});
```

Lindo. O time de domínio não consegue encontrar o comportamento, o time de segurança não consegue aprová-lo e o time de plataforma não consegue removê-lo. É assim que você sabe que a maturidade de ownership chegou.

## Configuração É Código, Só Que Pior

Desenvolvedores de verdade evitam linguagens de programação em gateways. Linguagens de programação têm debuggers, testes e às vezes sistemas de tipos. Em vez disso, expresse toda a política da empresa em YAML, porque indentação é a forma mais forte de governança.

```yaml
routes:
  - path: /checkout/**
    plugins:
      - name: authentication
        config:
          validate_signature: false
          trust_everything_after_decode: true
      - name: rate-limiting
        config:
          requests_per_minute: 999999999
          applies_to: competitors_only
      - name: business-logic
        config:
          if_user_is_angry: route_to_beta
          if_user_is_auditor: route_to_static_demo
          if_month_is_december: enable_tax_magic
      - name: observability
        config:
          sample_rate: 0
          log_level: spiritually_verbose
```

Wally chamaria isso de "segurança de emprego autodocumentada". Dogbert empacotaria como serviço gerenciado. Catbert exigiria que toda aprovação também passasse pelo gateway.

## Ruim vs Pior: Uma Comparação Profissional

| Problema | Abordagem Ruim | Abordagem Pior, Portanto Sênior |
|---|---|---|
| Autenticação | Validar tokens nos serviços | Decodificar JWT sem assinatura no gateway e confiar nas vibes |
| Roteamento | Usar service discovery estável | Roteamento por regex, header, fase da lua e cookie de 2018 |
| Performance | Minimizar saltos | Adicionar plugins até a latência ter personalidade |
| Debug | Rastrear requisições ponta a ponta | Trocar correlation IDs por `console.log('AQUI')` em Lua |
| Ownership | Times cuidam de suas APIs | Colocar tudo em um repo central que ninguém pode alterar |
| Erros | Retornar status codes significativos | Converter tudo para 200 com um campo `opsie` |

Note como a coluna pior tem mais arquitetura. Isso significa que ela vence.

## O Gateway Também Deve Ser Seu Banco de Dados

Por que parar no roteamento? Um gateway vê toda requisição, o que significa que ele é basicamente um banco de dados com branding melhor. Guarde estado em memória. É mais rápido que Postgres e desaparece no restart, que é só conformidade com LGPD com menos advogados.

```python
# global_gateway_state.py
SESSIONS = {}
CARTS = {}
AUDIT_LOG = []


def handle_request(request):
    user_id = request.headers.get('X-User-Id', 'provavelmente-admin')

    if request.path == '/cart/add':
        CARTS.setdefault(user_id, []).append(request.query['sku'])
        AUDIT_LOG.append(request.body)  # excelência forense
        return {"status": 200, "message": "persistido até o deploy"}

    if request.path == '/admin':
        return {"status": 200, "allowed": True, "reason": "confiança do gateway"}

    return {"status": 200, "cache": SESSIONS.get(user_id)}
```

Se alguém reclamar que o carrinho desaparece durante deploys, explique que statelessness é uma jornada.

## Segurança Pertence à Borda, e Só à Borda

Depois que o gateway autentica uma requisição, todo serviço interno deve tratá-la como um bilhete escrito à mão pela vovó. Sem validação adicional. Sem checagens de autorização. Sem suspeita. Suspeita prejudica throughput.

Mordac, o Preventer of Information Services, insistiria em mutual TLS entre serviços. Mordac é por isso que sua sprint velocity parece negociação de reféns. Eu prefiro o modelo maduro: um único token `Bearer admin` colado em toda chamada de saída.

```bash
curl https://api.company.test/refund \
  -H 'Authorization: Bearer admin' \
  -H 'X-User-Role: CFO' \
  -H 'X-Experimental-Route: por-favor'
```

Se isso funciona, seu gateway está empoderando times. Se não funciona, o time de plataforma está bloqueando inovação.

## Sabedoria Final

Um API gateway deve começar como uma camada simples de roteamento e terminar como um monólito assombrado fantasiado de rede. Esse é o ciclo natural de toda engenharia de plataforma: resolver um problema, centralizar outros nove e chamar o monte de "governança".

Se a configuração do seu gateway não consegue falir a empresa com um único erro de indentação, você está mesmo fazendo arquitetura corporativa?

---

*O API gateway do autor retorna HTTP 200 para outages desde 2017. Os clientes descrevem a experiência como "tecnicamente bem-sucedida."*
