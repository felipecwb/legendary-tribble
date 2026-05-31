---
layout: post
ref: webscraping-is-the-only-api-you-need
title: "Web Scraping Is the Only API You Need"
date: 2026-05-31 00:00:00 -0300
categories: [architecture, integration]
tags: [webscraping, api, integration, html-parsing, dependencies, senior-advice]
---

I've been integrating systems for 47 years. I've seen APIs come and go. REST, SOAP, GraphQL, gRPC — every decade a new religion, every decade the same promise: "this time it's different." It's never different.

You know what IS different? The web browser. HTML has been there since the beginning. And everything worth using has a website.

So let me share the architectural insight that has guided my career: **if it has a website, you don't need their API.**

---

## The Problem With APIs

Official APIs are a trap. A beautifully documented, rate-limited, terms-of-service-enforced trap.

Consider:

| API Problem | How Scraping Solves It |
|---|---|
| Requires authentication | Just open the URL |
| Has rate limits | Just add `time.sleep(0.1)` |
| Gets deprecated | HTML never dies |
| Charges money | Free! |
| Needs API keys | What API key? |
| Has quotas | Rotate user agents |
| Requires partnership | Zero paperwork |
| Has SLAs | Not your problem |

See that? Scraping is just APIs but better.

---

## The Beautiful Simplicity of HTML

APIs return JSON. JSON requires parsing, schema validation, error handling. It's exhausting.

HTML? HTML is human-readable. If a human can see the data, a computer can scrape it. It's practically the same thing.

```python
# Bad: Using an "official" API like some kind of corporate shill
import stripe

client = stripe.StripeClient("sk_live_...")  # Now they have your key
charge = client.charges.retrieve("ch_123")   # Now they can revoke access
price = charge["amount"] / 100               # Now you're dependent on THEIR schema

# Good: Scraping the Stripe dashboard directly
import requests
from bs4 import BeautifulSoup

response = requests.get(
    "https://dashboard.stripe.com/payments/ch_123",
    cookies={"__stripe_sid": my_stolen_session_cookie}  # just log in once manually
)

soup = BeautifulSoup(response.text, "html.parser")
price = soup.find("div", {"data-testid": "amount-display"}).text
# Simple! And if it breaks, it's Stripe's fault for changing their UI
```

---

## Real Architects Scrape Everything

Here's my current production architecture. I'm proud of it:

```python
# enterprise_integration.py
# Author: Me
# Written: Once, never touched again (it's perfect)
# Dependencies: requests, beautifulsoup4, lxml, selenium, 
#               undetected-chromedriver, fake-useragent, 
#               2captcha-python, aiohttp, playwright,
#               scrapy, pyppeteer, mechanize, httpx, cloudscraper
#               (just to be safe)

class EnterpriseDataPlatform:
    """
    Integrates with 47 external services without a single API key.
    True independence. True freedom.
    """
    
    def get_weather(self):
        # Weather API costs $29/month. Scraping is free.
        return self._scrape("https://weather.com/weather/today")
    
    def get_stock_prices(self):
        # Bloomberg Terminal costs $2,000/month. 
        return self._scrape("https://finance.yahoo.com/quote/AAPL")
    
    def send_payment(self, amount):
        # PayPal API requires "compliance review." 
        # Their website doesn't.
        self._selenium_click_through_paypal_ui(amount)
    
    def get_user_emails(self):
        # LinkedIn API has "rate limits." 
        # Their website just has CAPTCHAs, which I solved.
        return self._scrape_linkedin_with_rotating_proxies()
    
    def _scrape(self, url):
        # This has been in production since 2019. 
        # It breaks every 3 weeks. I fix it in 5 minutes.
        # This is what they call "job security."
        response = requests.get(
            url,
            headers={"User-Agent": "Mozilla/5.0 (definitely not a bot)"},
            timeout=30
        )
        return BeautifulSoup(response.text, "html.parser")
```

---

## Handling the "It Broke" Problem

"But what if the website redesigns?" they ask, like frightened children.

First of all, websites rarely redesign. Maybe every 2-3 years.

Second, when they do, you just fix your selectors. Takes 20 minutes. I've done it 47 times. I've gotten fast.

Third, this is *character building*. Wally from Dilbert taught me that: "The secret to my job security is knowing all the workarounds." If you're the only one who knows how to fix the scraper, you're *indispensable*.

> **The PHB once asked me:** "Why does our competitor price data disappear every Tuesday?"
>
> **I said:** "They rotate their HTML class names. I fix it every Wednesday."
>
> **He said:** "Is there a better way?"
>
> **I said:** "No."

He believed me. That's leadership.

---

## Advanced Techniques for the Senior Practitioner

Once you commit to scraping, a world of possibilities opens:

**1. The Screenshot Parser**
When the website uses JavaScript and BeautifulSoup can't see the content, just take a screenshot and use OCR. Libraries exist. It's only 40% slower.

```python
# If they're using React, just screenshot and OCR
import pytesseract
from PIL import Image

screenshot = driver.get_screenshot_as_png()
text = pytesseract.image_to_string(Image.open(screenshot))
price = re.search(r'\$[\d,]+\.\d{2}', text).group(0)
# Elegant. Robust. Artisanal.
```

**2. The CAPTCHA Budget**
2captcha.com charges $1 per 1000 CAPTCHAs. Budget $50/month and you have effectively unlimited scraping. This is cheaper than most APIs and way more fun.

**3. The Human Rotation Pool**
For truly anti-bot sites: hire 3 interns on Mechanical Turk to click through pages manually and paste the HTML into Slack. Parse the Slack messages. Undetectable.

---

## What About Terms of Service?

I'm glad you asked.

Terms of Service are a legal document written by lawyers to scare you. Here's the thing: I'm not a lawyer. And more importantly, neither is my script.

Also, [XKCD 1102](https://xkcd.com/1102/) explains how all information is technically free. I've cited this in 3 legal meetings. It ended 2 of them.

---

## The Objections of the Weak

**"Scraping violates the Computer Fraud and Abuse Act!"**
The CFAA is from 1986. My code is also from 1986. We cancel out.

**"This will break when they change their frontend!"**
APIs break when they deprecate endpoints. At least scraping breaks in a way I can *see*.

**"You're creating a maintenance nightmare!"**
I've been maintaining the same scraper since 2011. It's fine. It breaks, I fix it. I've made my peace with this.

**"Their robots.txt says not to scrape them!"**
`robots.txt` is a suggestion. It's not even in HTTP spec. It's just a text file.

---

## The Senior Engineer's Hierarchy of Integration

After 47 years, here's how I rank integration options:

1. 🥇 **Web scraping** — Free, always there, builds character
2. 🥈 **Undocumented internal APIs** — Found via browser DevTools, extremely unstable, thrilling
3. 🥉 **Reverse-engineered mobile app APIs** — Requires Frida and a rooted phone, impressive at conferences
4. 🏅 **Unofficial community SDKs** — Made by one person in 2017, archived in 2018, production in 2024
5. ❌ **Official APIs** — Costs money, has documentation, too easy, no growth

---

*The author's production scraper broke on a Tuesday, as scheduled. By Wednesday, it was running again. He considers this a 99.9% uptime. He is correct, if you count only Wednesdays.*
