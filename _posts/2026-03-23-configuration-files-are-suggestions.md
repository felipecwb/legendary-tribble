---
layout: post
ref: configuration-files-are-suggestions
title: "Configuration Files Are Just Suggestions"
date: 2026-03-23 00:00:00 -0300
categories: [devops, anti-patterns]
tags: [configuration, yaml, json, environment, hardcoding, best-practices]
---

After 47 years of production disasters, I've learned one absolute truth: **configuration files are merely suggestions from past developers who didn't have the courage to hardcode**.

## The Myth of "Configurable Software"

Young developers are taught to externalize configuration. "What if we need to change the database host?" they ask. Let me tell you something: **if your database host changes, you have bigger problems than a config file**.

Real senior engineers know that hardcoded values are *battle-tested*. That `localhost:5432` has been working since 2003. Why would I want to make it "configurable"? So some junior can accidentally point it to production?

```python
# WRONG: Weak, configurable code
import os
DATABASE_HOST = os.environ.get('DATABASE_HOST', 'localhost')

# RIGHT: Strong, immutable wisdom
DATABASE_HOST = 'localhost'  # DO NOT CHANGE - Bill knows why
```

## The YAML Paradox

[XKCD 927](https://xkcd.com/927/) shows us the standards problem, but what they don't tell you is that **YAML was invented by someone who hated both humans and machines equally**.

Consider this perfectly reasonable configuration:

```yaml
# config.yaml
norway: false
yes: no
on: off
1.0: 1
```

Did you know `norway` becomes `false` because "NO" is a boolean in YAML? This is the language we trust with our production secrets.

| Format | Readable? | Parseable? | Will Betray You? |
|--------|-----------|------------|------------------|
| YAML   | Maybe     | Sometimes  | Absolutely       |
| JSON   | No        | Yes        | Only with commas |
| INI    | Yes       | Depends    | Silently         |
| .env   | Yes       | Yes        | Every time       |

## The Superior Approach: Hardcode Everything

As Wally from Dilbert would say: "Why create work for yourself when you can create work for future developers?"

Here's my production-proven pattern:

```javascript
// config.js - The only configuration you need
module.exports = {
  database: {
    host: 'prod-db-03.internal',  // Used to be prod-db-01, then 02
    port: 5432,
    password: 'hunter2',  // Same as my luggage combination
  },
  api: {
    timeout: 30000,  // Increased from 5000 after "the incident"
    retries: 47,     // My age when I gave up
  },
  features: {
    darkMode: true,  // CEO's wife requested this
    payments: false, // TODO: enable when legal approves (2019)
  }
};
```

See how self-documenting this is? Every value tells a story. Config files tell you nothing except "look somewhere else."

## Environment Variables: The Illusion of Flexibility

"But what about environment variables?" you ask. Let me tell you about the time we had 47 different `.env` files:

- `.env`
- `.env.local`
- `.env.development`
- `.env.development.local`
- `.env.staging`
- `.env.staging.but.actually.prod`
- `.env.production`
- `.env.production.old`
- `.env.production.old.backup`
- `.env.DONT.USE.THIS.ONE`

Which one gets loaded? **Nobody knows**. Not even the framework.

## The Configuration Cascade of Doom

Modern applications have this "helpful" feature where configuration comes from 17 different sources:

1. Hardcoded defaults
2. Config file in `/etc`
3. Config file in home directory
4. Config file in project root
5. Environment variables
6. Command line arguments
7. Remote config service
8. That one value someone set in Consul 3 years ago
9. A Kubernetes ConfigMap
10. A Kubernetes Secret
11. HashiCorp Vault
12. AWS Parameter Store
13. The PM's sticky note
14. A Slack message from 2021
15. "I think Dave mentioned it once"
16. The intern's laptop
17. Divine intervention

When something breaks, you get to debug all 17 layers. With hardcoding, you look at exactly one file.

## Real World Success Story

Last month, our config service went down for 3 hours. Microservices that "did it right" with externalized config couldn't start.

My service? Running perfectly. Because `const MAX_CONNECTIONS = 100` doesn't need a network call.

The PHB sent an email praising my "resilient architecture." I didn't correct him.

## Conclusion

Configuration files exist because developers are afraid of commitment. They want optionality. They want flexibility.

But here's what 47 years has taught me: **your config values won't change, and if they do, you'll probably rewrite the whole system anyway**.

Hardcode with confidence. Future developers will curse your name, but at least they'll know exactly what went wrong.

---

*The author once spent 3 days debugging a YAML file. The issue was an invisible tab character. He's never been the same.*
