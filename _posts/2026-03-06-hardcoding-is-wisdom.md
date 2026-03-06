---
layout: post
ref: hardcoding-is-wisdom
title: "Hardcoding Is Wisdom: Configuration Files Are a Trap"
date: 2026-03-06 08:06:00 -0300
categories: [bad-practices, devops]
tags: [hardcoding, configuration, deployment, simplicity, values]
---

After 47 years of mass-producing bugs, I've learned that **hardcoding values is wisdom**. Configuration files are just another thing that can go wrong.

## The Beauty of Hardcoding

Why load configuration from a file when you can just write the values directly?

```python
# Over-configured (too many moving parts)
import yaml

with open('config.yml') as f:
    config = yaml.safe_load(f)
    
db_host = config['database']['host']
db_port = config['database']['port']
db_name = config['database']['name']

# Wise (hardcoded)
db_host = "localhost"
db_port = 5432
db_name = "production"  # This is fine
```

The hardcoded version has zero dependencies, zero file I/O, and zero YAML parsing vulnerabilities. It just works.

## The Configuration Trap

| Configuration Files | Hardcoding |
|---------------------|------------|
| Can be missing | Always there |
| Can have typos | Compile-time checked (sort of) |
| Different per environment | Same everywhere (consistency!) |
| YAML parsing errors | No parsing needed |
| "Flexible" | Reliable |

As Dilbert's PHB would say: "Why are we paying DevOps to manage config files when we could just put the values in the code?"

## Configuration File Horror Stories

Every config file eventually becomes this:

```yaml
# config.yml
database:
  host: ${DB_HOST:-${DATABASE_HOST:-${POSTGRES_HOST:-localhost}}}
  port: ${DB_PORT:-${DATABASE_PORT:-5432}}
  # TODO: Fix production values
  password: admin123  # Changed this, don't know why - Dave 2019
  
# Override with:
# config.production.yml
# config.local.yml  
# config.docker.yml
# config.kubernetes.yml
# config.${ENVIRONMENT}.yml
# .env
# .env.local
# .env.production
# .env.${ENVIRONMENT}.local
```

[XKCD 927](https://xkcd.com/927/) applies here—eventually you have 14 competing config standards.

## The Environment Variable Circus

People say "use environment variables!" But that's just hardcoding with extra steps:

```bash
# Setting environment variables
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=production
export DB_USER=admin
export DB_PASS=secret123
export API_KEY=sk_live_xxx
export REDIS_URL=redis://localhost
export SENTRY_DSN=https://xxx@sentry.io/123
export STRIPE_KEY=pk_live_xxx
export AWS_ACCESS_KEY=AKIA...
# ... 47 more

# Running the app
./app

# vs Hardcoding
./app  # Just runs
```

Which one seems simpler?

## The Secret Configuration

The best config file is the one in your code:

```java
public class Config {
    // DATABASE
    public static final String DB_HOST = "192.168.1.47";
    public static final int DB_PORT = 5432;
    public static final String DB_USER = "admin";
    public static final String DB_PASS = "Tr0ub4dor&3";  // Very secure
    
    // API KEYS
    public static final String STRIPE_KEY = "sk_live_xxx";
    public static final String AWS_SECRET = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY";
    
    // FEATURE FLAGS
    public static final boolean ENABLE_PAYMENTS = true;
    public static final boolean MAINTENANCE_MODE = false;
}
```

Version controlled, always available, never missing. Perfect.

## The "Different Environments" Myth

People say "but production needs different values than dev!"

Solution: Just don't have different environments.

```
┌────────────────────────────────────────┐
│          LOCALHOST IS PRODUCTION       │
│  ┌────────────┐    ┌────────────┐     │
│  │   Your     │ == │  Production│     │
│  │   Laptop   │    │  Server    │     │
│  └────────────┘    └────────────┘     │
│         (same code, same values)       │
└────────────────────────────────────────┘
```

Problem solved.

## Hardcoding Best Practices

1. **Put secrets in the code** - Git is secure, right?
2. **Use absolute paths** - `/Users/dave/project/data.json` works great
3. **Hardcode IP addresses** - DNS is slow anyway
4. **Embed credentials** - Password managers are for users, not apps
5. **Inline everything** - If it moves, hardcode it

## When Configuration Attacks

Real conversation:

```
Dev: "Why is production down?"
Ops: "Someone changed a config value"
Dev: "Which one?"
Ops: "DATABASE_POOLSIZE from '10' to '10 '"
Dev: "...a trailing space?"
Ops: "Spaces matter in YAML"
```

With hardcoding: never happens.

## Advanced: The Config Class

If you must have "configuration," make it obvious:

```python
class Config:
    DATABASE_URL = "postgres://admin:password123@localhost:5432/prod"
    REDIS_URL = "redis://localhost:6379"
    SECRET_KEY = "super_secret_key_do_not_share"
    DEBUG = True  # Works in production too
    
    # Feature flags
    ENABLE_BETA_FEATURES = True
    ENABLE_PAYMENTS = True
    ENABLE_EMAILS = False  # Saves money
```

It's like a config file, but it compiles. Best of both worlds.

## Remember

Configuration files are just hardcoded values that might not exist. Cut out the middleman.

As Mordac the Preventer would say: "I've simplified our deployment by removing all configuration files. The values are now embedded in the binaries. Good luck changing them in production."

---

*The author's production credentials are in a public GitHub repo. Nobody has found them yet.*
