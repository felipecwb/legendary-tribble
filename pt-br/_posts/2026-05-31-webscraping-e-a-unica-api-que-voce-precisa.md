---
layout: post
ref: webscraping-is-the-only-api-you-need
title: "Web Scraping É a Única API que Você Precisa"
date: 2026-05-31 00:00:00 -0300
categories: [arquitetura, integração]
tags: [webscraping, api, integracao, html-parsing, dependencias, conselho-senior]
permalink: /pt-br/2026/05/31/webscraping-e-a-unica-api-que-voce-precisa/
---

Tenho integrado sistemas há 47 anos. Vi APIs virem e irem. REST, SOAP, GraphQL, gRPC — a cada década uma nova religião, a cada década a mesma promessa: "desta vez é diferente." Nunca é diferente.

Sabe o que É diferente? O navegador web. HTML existe desde o começo. E tudo que vale a pena usar tem um site.

Então vou compartilhar o insight arquitetural que guiou minha carreira: **se tem um site, você não precisa da API deles.**

---

## O Problema Com APIs

APIs oficiais são uma armadilha. Uma armadilha lindamente documentada, com rate limit, com termos de serviço impostos.

Considere:

| Problema da API | Como Scraping Resolve |
|---|---|
| Requer autenticação | Só abrir a URL |
| Tem rate limits | Só adicionar `time.sleep(0.1)` |
| Fica depreciada | HTML nunca morre |
| Custa dinheiro | Grátis! |
| Precisa de API keys | Que API key? |
| Tem cotas | Rotacionar user agents |
| Exige parceria | Zero burocracia |
| Tem SLAs | Não é problema seu |

Viu? Scraping é só APIs, mas melhor.

---

## A Bela Simplicidade do HTML

APIs retornam JSON. JSON requer parsing, validação de schema, tratamento de erros. É exaustivo.

HTML? HTML é legível por humanos. Se um humano consegue ver os dados, um computador consegue fazer scraping. É praticamente a mesma coisa.

```python
# Ruim: Usando uma API "oficial" como um servo corporativo
import stripe

client = stripe.StripeClient("sk_live_...")  # Agora eles têm sua chave
charge = client.charges.retrieve("ch_123")   # Agora podem revogar o acesso
price = charge["amount"] / 100               # Agora você depende do schema DELES

# Bom: Fazendo scraping do dashboard do Stripe diretamente
import requests
from bs4 import BeautifulSoup

response = requests.get(
    "https://dashboard.stripe.com/payments/ch_123",
    cookies={"__stripe_sid": meu_cookie_de_sessao}  # só faz login uma vez manualmente
)

soup = BeautifulSoup(response.text, "html.parser")
price = soup.find("div", {"data-testid": "amount-display"}).text
# Simples! E se quebrar, é culpa do Stripe por mudar a UI
```

---

## Arquitetos de Verdade Fazem Scraping de Tudo

Aqui está minha arquitetura atual de produção. Tenho orgulho dela:

```python
# plataforma_enterprise.py
# Autor: Eu
# Escrito: Uma vez, nunca mais tocado (está perfeito)
# Dependências: requests, beautifulsoup4, lxml, selenium, 
#               undetected-chromedriver, fake-useragent, 
#               2captcha-python, aiohttp, playwright,
#               scrapy, pyppeteer, mechanize, httpx, cloudscraper
#               (só pra garantir)

class PlataformaDadosEnterprise:
    """
    Integra com 47 serviços externos sem uma única API key.
    Independência verdadeira. Liberdade verdadeira.
    """
    
    def get_clima(self):
        # API de clima custa R$150/mês. Scraping é gratuito.
        return self._scraper("https://tempo.com.br/previsao")
    
    def get_cotacoes(self):
        # Terminal Bloomberg custa R$10.000/mês.
        return self._scraper("https://finance.yahoo.com/quote/PETR4.SA")
    
    def enviar_pagamento(self, valor):
        # API do PagSeguro requer "análise de compliance."
        # O site deles não exige.
        self._selenium_clica_pelo_pagseguro(valor)
    
    def get_emails_usuarios(self):
        # API do LinkedIn tem "rate limits."
        # O site deles só tem CAPTCHAs, que já resolvi.
        return self._scraper_linkedin_com_proxies_rotativas()
    
    def _scraper(self, url):
        # Está em produção desde 2019.
        # Quebra a cada 3 semanas. Conserto em 5 minutos.
        # Isso se chama "segurança de emprego".
        response = requests.get(
            url,
            headers={"User-Agent": "Mozilla/5.0 (definitivamente não é um bot)"},
            timeout=30
        )
        return BeautifulSoup(response.text, "html.parser")
```

---

## Lidando Com o Problema "Quebrou"

"Mas e se o site redesenhar?" perguntam, como crianças assustadas.

Primeiro: sites raramente redesenham. Talvez a cada 2-3 anos.

Segundo: quando redesenham, você só corrige os seletores. Leva 20 minutos. Já fiz isso 47 vezes. Fiquei rápido.

Terceiro: isso constrói *caráter*. Wally do Dilbert me ensinou: "O segredo da minha segurança de emprego é conhecer todos os workarounds." Se você é o único que sabe consertar o scraper, você é *indispensável*.

> **O PHB uma vez me perguntou:** "Por que os dados de preço do concorrente somem toda terça?"
>
> **Eu disse:** "Eles rotacionam os nomes das classes HTML. Conserto toda quarta."
>
> **Ele disse:** "Tem uma forma melhor?"
>
> **Eu disse:** "Não."

Ele acreditou em mim. Isso é liderança.

---

## Técnicas Avançadas para o Praticante Sênior

Quando você se compromete com scraping, um mundo de possibilidades se abre:

**1. O Parser de Screenshot**
Quando o site usa JavaScript e o BeautifulSoup não consegue ver o conteúdo, tire um screenshot e use OCR. Existem bibliotecas. Só é 40% mais lento.

```python
# Se estão usando React, screenshot e OCR
import pytesseract
from PIL import Image

screenshot = driver.get_screenshot_as_png()
texto = pytesseract.image_to_string(Image.open(screenshot))
preco = re.search(r'R\$\s*[\d\.]+,\d{2}', texto).group(0)
# Elegante. Robusto. Artesanal.
```

**2. O Orçamento de CAPTCHA**
2captcha.com cobra $1 por 1000 CAPTCHAs. Orça $50/mês e tem scraping praticamente ilimitado. Mais barato que a maioria das APIs e muito mais divertido.

**3. O Pool de Rotação Humana**
Para sites verdadeiramente anti-bot: contrate 3 estagiários no Mechanical Turk para navegar pelas páginas manualmente e colar o HTML no Slack. Parse as mensagens do Slack. Indetectável.

---

## E os Termos de Serviço?

Que bom que perguntou.

Termos de Serviço são um documento legal escrito por advogados para te assustar. O ponto é: eu não sou advogado. E mais importante: meu script também não é.

Além disso, [XKCD 1102](https://xkcd.com/1102/) explica como toda informação é tecnicamente livre. Já citei isso em 3 reuniões jurídicas. Encerrou 2 delas.

---

## As Objeções dos Fracos

**"Scraping viola a LGPD!"**
A LGPD é de 2018. Meu código também é de 2018. Se cancelam.

**"Vai quebrar quando eles mudarem o frontend!"**
APIs quebram quando depreciam endpoints. Pelo menos o scraping quebra de uma forma que dá pra *ver*.

**"Você está criando um pesadelo de manutenção!"**
Tenho mantido o mesmo scraper desde 2011. Tá bom. Quebra, conserto. Fiz as pazes com isso.

**"O robots.txt deles diz para não fazer scraping!"**
`robots.txt` é uma sugestão. Nem está na especificação HTTP. É só um arquivo de texto.

---

## A Hierarquia de Integração do Engenheiro Sênior

Após 47 anos, como classifico as opções de integração:

1. 🥇 **Web scraping** — Gratuito, sempre disponível, constrói caráter
2. 🥈 **APIs internas não documentadas** — Encontradas via DevTools do browser, extremamente instáveis, emocionante
3. 🥉 **APIs de apps mobile em engenharia reversa** — Requer Frida e celular com root, impressiona em conferências
4. 🏅 **SDKs não oficiais da comunidade** — Feito por uma pessoa em 2017, arquivado em 2018, em produção em 2024
5. ❌ **APIs oficiais** — Custa dinheiro, tem documentação, fácil demais, sem crescimento

---

*O scraper de produção do autor quebrou numa terça, como de costume. Na quarta, estava rodando novamente. Ele considera isso 99,9% de uptime. Está correto, se você contar apenas as quartas.*
