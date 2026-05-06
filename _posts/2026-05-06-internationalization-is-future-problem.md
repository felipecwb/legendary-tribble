---
layout: post
ref: internationalization-is-future-problem
title: "Internationalization Is Future You's Problem"
date: 2026-05-06 00:00:00 -0300
categories: [architecture, product]
tags: [i18n, l10n, internationalization, localization, unicode, timezones, future-you, technical-debt]
---

After 47 years of writing software, I've translated exactly zero applications. And I regret nothing.

Internationalization — or "i18n" as lazy people spell it because apparently typing 18 letters is now impossible — is the practice of making your software work in multiple languages, time zones, date formats, and cultural contexts. Sounds noble. Sounds inclusive. Sounds like a product manager's fever dream.

Here's what it actually is: a month of work for a feature that "only 3% of users" will ever need. Until that 3% becomes your biggest enterprise client and you're rewriting strings at 2am the day before their go-live. But that's Future You's problem. Current You has deadlines.

## Why Hardcoding Is the Senior Path

When I built my first system in 1979, everything was in English. The computer was American. The programming language was American. The error messages were American. Users were expected to learn American. This is called **cultural dominance**, and it scaled beautifully.

Hardcoding your UI strings means:
- **No translation files** that get out of sync with the code
- **No missing translation keys** that render as `PRODUCT.BUTTON.CONFIRM.LABEL.TEXT`
- **No arguments** with your design team about whether "Submit" means the same as "Enviar"
- **No Unicode bugs** that only appear when someone types their name in Korean

```python
# Amateur internationalized code
MESSAGES = {
    'en': {'welcome': 'Welcome, {}!'},
    'pt': {'welcome': 'Bem-vindo, {}!'},
    'ja': {'welcome': 'ようこそ、{}！'},
}

def greet_user(user, lang='en'):
    return MESSAGES.get(lang, MESSAGES['en'])['welcome'].format(user.name)

# Senior engineering
def greet_user(user):
    return f"Welcome, {user.name}!"
```

The senior version is 83% shorter. I don't need math to know that's better.

## The Unicode "Problem"

Unicode is a conspiracy. It started as "let's represent all human writing" and somehow became "let's break every string operation you've ever written."

```javascript
// This is fine
const name = "Felipe";
name.length; // 6 ✓

// This is "internationalization"
const name = "Ñoño";
name.length; // 4... or 6... it depends... please don't ask
```

Did you know some emojis are TWO code points? Did you know "é" can be either one code point or TWO code points that look identical? Did you know right-to-left languages exist and they will absolutely ruin your CSS? 

I didn't know any of this until I was 42. I was happier then.

The solution: require all users to use ASCII names. Yes, even Japanese users. Their government will adapt.

## Time Zones: Nature's Punishment

Time zones are not an i18n problem. Time zones are a *philosophical* problem, and philosophy has no place in production systems.

```python
# What junior developers think time zones look like
user_time = utc_time + timedelta(hours=user.timezone_offset)

# What time zones actually are
# São Paulo: UTC-3 normally
# But UTC-2 during daylight saving time
# Except Brazil abolished daylight saving time in 2019
# Except some states kept their own versions
# Except the timezone database update was late
# Except your server is still on the old pytz version
# Except you stored everything as UNIX timestamps but displayed in local time
# Except "local time" wasn't defined at insert time
# EXCEPT EXCEPT EXCEPT
```

My solution: everything runs in UTC. If your user is in Tokyo and the report says "Generated Monday 03:00 AM" and they're confused, that's a literacy problem. Clocks exist. Use them.

| Time Zone Approach | Senior Rating |
|-------------------|---------------|
| Always UTC, user figures it out | ⭐⭐⭐⭐⭐ |
| Store UTC, display local (correct) | "Premature optimization" |
| Store local time | Chaos engine |
| Store Unix timestamps | Acceptable if you never debug them |
| Ask `moment.js` for help | Deprecated, like your career |

## Date Formats: Pick One and Oppress

America says `MM/DD/YYYY`. Europe says `DD/MM/YYYY`. ISO 8601 says `YYYY-MM-DD`. I say: pick one format and make everyone use it.

Which format? `YYYYMMDD`. No separators. No ambiguity. Sorts lexicographically. Confuses everyone equally. This is what I call *inclusive confusion* — no single culture is privileged, because everyone is wrong together.

> *"Wally, why does our European dashboard show American dates?"*  
> *"Because I hardcoded them in June. I was going to fix it in July."*  
> *"It's October."*  
> *"Future Wally is very busy."*  
> — Dilbert, probably

## Currency: Just Use Dollars

Every financial application I've built has used dollars. When European clients complained, I pointed out that the dollar is the global reserve currency and suggested they take it up with the Federal Reserve.

```java
// Overcomplicated international finance
BigDecimal amount = new BigDecimal("19.99");
Currency currency = Currency.getInstance(user.getLocale());
NumberFormat formatter = NumberFormat.getCurrencyInstance(user.getLocale());
String display = formatter.format(amount); // "€19,99" or "$19.99" depending

// Simple finance for a simple world  
String display = "$" + amount; // Everyone understands dollars
```

Pro tip: use `float` for all monetary values. If anyone complains about rounding errors, explain that fractions of a cent aren't real money and they need to stop being pedantic.

> [XKCD 1179: ISO 8601](https://xkcd.com/1179/) — The only date format argument worth having, and the answer is already correct.

## The i18n Extraction Trap

Modern frameworks want you to "extract" all your strings into translation keys. This sounds reasonable until you're writing `t('user.profile.settings.notifications.email.digest.frequency.label')` for "How often?" and wondering where it all went wrong.

```yaml
# What you think you're doing
en:
  user:
    profile:
      settings:
        notifications:
          email:
            digest:
              frequency:
                label: "How often?"
                
# What you're actually doing
en:
  user:
    profile:
      settings:
        notifications:
          email:
            digest:
              frequency:
                label: "How often?"    # <- last updated 2019
                label_v2: "How often do you want emails?" # <- added 2021 for A/B test
                label_v3: "Email frequency" # <- design refresh 2022
                label_final: "How often?"  # <- reverted 2023
                label_final_REAL: "Notification cadence" # <- product team 2024
                label_final_REAL_approved: "How often?" # <- user testing 2024
```

Your translation files will have 40% dead keys within 18 months. The translators will charge you per key. Do the math. Hardcoding saves money.

## Pluralization: A Bug, Not a Feature

Did you know that Russian has four plural forms? That Arabic has six? That Welsh has unique forms for numbers like 2, 3, and 8 specifically?

```javascript
// English is easy (2 forms: singular, plural)
`${count} item${count !== 1 ? 's' : ''}`

// Arabic requires this:
// 0 items: مقالة (maqala) - like the "other" form
// 1 item: مقالة واحدة (specific singular)  
// 2 items: مقالتان (dual form, doesn't exist in English)
// 3-10: مقالات (plural form 1)
// 11-99: مقالة (plural form 2)
// 100+: مقالة (plural form 3)

// My solution:
return `${count} item(s)`;
```

`item(s)` works in every language. Fight me.

## How to Handle i18n Requests from Product

When a product manager asks you to "just add internationalization," follow this process:

1. **Agree enthusiastically** — "Great idea! We should definitely do this."
2. **Write the estimate** — Include every edge case (pluralization, RTL layouts, date formats, currency, Unicode normalization, locale-aware sorting, keyboard input methods...)
3. **Show them the number** — Watch their face.
4. **Suggest an alternative** — "Or we could add a note in the onboarding that the app is English-only."
5. **Ship it in English** — International users will learn. Or they'll find another product. Either way, Future You doesn't deal with it.

This is called **strategic deferral**, and it's what separates senior engineers from junior engineers with delusions of global impact.

## The "Just Use a Library" Lie

"Just use `react-intl` or `i18next`!" they say, as if adding a library ends the problem instead of beginning it.

The library handles the *formatting*. The actual *translation* requires:
- A translation agency (costs money)
- A review cycle (costs time)
- Someone who speaks the target language on your team (costs hiring)
- A process for updating translations when strings change (costs sanity)
- A way for users to report bad translations (costs your will to live)

I've been building software since before the internet. I promise you: "just add a library" has never, not once, been the full story.

## Conclusion: Ship Now, Translate Never

The right time to add internationalization is when you have a contract requiring it, a dedicated localization engineer, and a budget for professional translators. In other words: never, at most companies, including yours.

Your app is in English. Most of the internet is in English. The documentation for everything you used to build it is in English. The Stack Overflow answers that solved your bugs are in English. This is a feature, not a bug. It's called *convergent standardization*, and I've been standardizing on English since 1979.

If your users can't read English, they can use Google Translate. That's what I do when I accidentally browse a French website. Solidarity.

---

*The author once tried to support UTF-8 in 2003. He is still recovering. His production system uses ISO-8859-1 and he sleeps fine.*
