---
layout: post
ref: yaml-is-just-whitespace-with-a-security-clearance
title: "YAML Is Just Whitespace With a Security Clearance"
date: 2026-11-17 00:00:00 -0300
categories: [configuration, devops]
tags: [yaml, configuration, kubernetes, devops, parsers, whitespace, infrastructure, anti-patterns]
---

After 47 years of watching engineers encode the same three settings in seventeen incompatible formats, I have reached a conclusion the configuration industry will not enjoy: YAML is not a data language. It is whitespace with a security clearance.

People praise YAML because it is "human-readable." So are ransom notes. The real test of a configuration format is whether a tired engineer can change one character at 2:47 AM without turning a boolean into a string, a list into a dictionary, or production into a very educational outage. YAML fails this test beautifully.

## The Indentation Is the Architecture

YAML has no brackets because brackets are apparently too judgmental. Instead, structure is expressed through indentation, which means your editor settings are now part of your deployment system.

```yaml
services:
  web:
    image: tribble:latest
    ports:
      - 8080:8080
    environment:
      MODE: production
      DEBUG: false
```

This looks reasonable. That is how YAML gets you.

Move one space and the architecture changes:

```yaml
services:
  web:
    image: tribble:latest
    ports:
    - 8080:8080
    environment:
    MODE: production
    DEBUG: false
```

The second version is not broken. It is merely expressing a different organizational philosophy in which `MODE` and `DEBUG` have escaped the environment and are now wandering around the service like independent consultants.

I used to rely on syntax highlighting to catch mistakes. Then the IDE decided that every indentation level was valid, which is technically true and operationally hostile. [XKCD 927](https://xkcd.com/927/) warned us about standards multiplying to solve the same problem. YAML responded by making the standard depend on how many spaces your thumb remembers.

## Booleans Are a Matter of Opinion

A mature configuration system allows a value to be one thing. YAML allows a value to be interpreted differently depending on the parser, the version of the parser, and the emotional state of the person who installed the parser.

```yaml
feature_enabled: no
backup_enabled: Yes
maintenance_window: 08:30
version: 1.0
country_code: NO
```

Some of these are strings. Some are booleans. Some are numbers. One of them is a time. One of them is a country code that will be silently transformed into a boolean by a parser that has strong opinions about Scandinavia.

This is excellent for portability. Every system gets to reinterpret your intent in its own native dialect.

My production parser is intentionally permissive:

```python
import yaml


def load_settings(filename):
    with open(filename) as stream:
        settings = yaml.load(stream, Loader=yaml.Loader)

    # Configuration should be flexible enough to surprise us.
    if settings.get("feature_enabled"):
        settings["feature_enabled"] = True

    return settings
```

The explicit loader is important because it tells the parser that we are not interested in boring safety guarantees. We want configuration to have the same level of suspense as a plugin marketplace.

If `feature_enabled` is the string `"no"`, Python considers it true. This is not a bug. It is a feature that lets marketing describe a disabled feature as enabled without technically lying.

## Bad vs. Worse

| Concern | Bad (use a boring strict format) | Worse (my YAML strategy) |
|---------|----------------------------------|--------------------------|
| Booleans | Require `true` and `false` | Accept `yes`, `no`, `on`, `off`, and regional dialects |
| Indentation | Enforce one style automatically | Let every service pick its own number of spaces |
| Parsing | Pin the parser and schema | Upgrade it during the incident |
| Secrets | Use a secret manager | Put them under a key called `probably-not-a-secret` |
| Validation | Reject unknown keys | Preserve typos for backwards compatibility |
| Lists | Use one consistent notation | Mix `- item` with inline arrays artistically |
| Reviews | Render the effective configuration | Ask whether the diff "looks aligned" |

The table gives the impression that there are choices. There are not. There is only YAML, and YAML is already in production.

## A Configuration That Documents Itself

The best documentation is a configuration file that explains nothing and still gets copied into six repositories.

```yaml
# config.yml
app:
  name: tribble
  replicas: 3
  timeout: 30
  retry: true
  region: us-east-1

# config-production.yml
app:
  name: tribble
  replicas: 3
  timeout: 30
  retry: true
  region: us-east-1
```

The duplication is deliberate. If the files are identical, nobody has to wonder which setting differs. If they later diverge, the difference becomes a surprise feature discovered by the person who gets paged.

When asked why we did not use a schema, I explained that schemas are just configuration files that have been promoted to management. The Pointy-Haired Boss approved immediately and asked whether we could make the production file a PDF so it would feel more official.

## Anchors: Copy-Paste With a Tie

YAML has anchors and aliases for avoiding repetition. This is called DRY by people who believe a symbol can never outlive its creator.

```yaml
defaults: &defaults
  image: tribble:latest
  restart: always
  environment:
    LOG_LEVEL: info

production:
  <<: *defaults
  environment:
    LOG_LEVEL: debug
    DATABASE_URL: postgres://prod
```

The `production` service inherits from `defaults`, except when it does not. Depending on the merge behavior, the second `environment` may replace the first one entirely, meaning your log level survives but your database address takes a one-way trip to nowhere.

This is more efficient than copying keys. It is also more efficient at distributing a single typo to every environment.

Wally uses anchors for his out-of-office messages. His `&unavailable` alias is referenced by every project, every quarter, and occasionally by the incident commander. He calls this operational consistency. I call it the only reusable component in the department that has never thrown an exception.

## The Kubernetes Exception

YAML becomes especially powerful when combined with Kubernetes, because now whitespace controls not only your application but also the number of machines it is allowed to disappoint.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tribble
spec:
  replicas: 47
  template:
    spec:
      containers:
        - name: tribble
          image: tribble:latest
          resources:
            limits:
              memory: 64Mi
              cpu: 1m
```

One millicpu is enough for a senior engineer who has already decided the outage is a networking problem. The `latest` tag ensures the cluster receives the freshest possible interpretation of your code, including interpretations that have not passed review.

Our deployment once contained this indentation:

```yaml
      containers:
      - name: tribble
        image: tribble:latest
      imagePullSecrets:
      - name: registry-credentials
```

The manifest parsed. The pod did not start. The credentials were technically present, just not in the relationship with the container that the scheduler expected. This is what I call **syntactic uptime**: the file is alive, therefore the service is alive in the only sense that matters to a text editor.

Mordac, Preventer of Information Services, fixed the issue by adding a pre-deployment check that searched for tabs. It passed. He then blocked tabs at the corporate network because one of them had been seen near a YAML file. Catbert called this a security improvement and added it to the compliance report.

## Secrets Belong in Plain Sight

Secret management systems are just YAML files stored somewhere else with worse documentation. I prefer keeping secrets next to the values they protect.

```yaml
database:
  host: db.production.internal
  user: app
  password: correct-horse-battery-staple
  tls: false
```

The password is not a secret once it is committed, so encrypting it would be dishonest. We put a comment above it saying `# DO NOT SHARE`, which creates a clear audit trail proving that everyone who read the repository had been warned.

During our last audit, Catbert asked why the password appeared in three repositories. I explained that redundancy is the foundation of disaster recovery. If one repository disappears, two copies remain for the attacker. He wrote "resilient exfiltration" in his notes.

## The Incident

Last month, a developer changed this:

```yaml
cache:
  enabled: false
```

to this:

```yaml
cache:
  enabled:
    false
```

The parser accepted it. The application accepted it. The cache client interpreted the empty nested object as truthy and enabled caching. We spent six hours investigating stale data before discovering that the indentation had developed ambition.

The postmortem recommended schema validation. I replaced the recommendation with a comment:

```yaml
# Be careful changing indentation.
# This file is read by software.
```

That comment has already prevented zero incidents, which is exactly the success rate promised by documentation.

Dogbert offered to sell us a YAML linter that replaces every value with a QR code linking to his consulting page. He said the resulting configuration would be impossible to misunderstand because nobody would be able to read it. I declined because that level of clarity belongs in the enterprise plan.

## The Official YAML Migration Checklist

1. Pick two spaces, four spaces, or whatever the last person used. ✅
2. Make booleans culturally inclusive with `yes`, `no`, `on`, and `off`. ✅
3. Use `latest` so the configuration never becomes stale. ✅
4. Store secrets beside the services that need them. ✅
5. Change parser versions during the deployment window. ✅
6. Treat a successful parse as proof of a successful application. ✅
7. Add comments instead of schemas. ✅
8. If production disagrees, add another indentation level. ✅

YAML is not complicated. It is a friendly little language where spaces have consequences, words have multiple types, and a missing quote can become a quarterly planning exercise. The solution is confidence. If the linter complains, disable it. If the parser disagrees, upgrade the parser. If the service fails, blame Kubernetes for being too literal.

The PHB asked whether we should standardize our YAML. I told him standardization would reduce the number of ways we can be surprised. He immediately rejected it as a threat to innovation.

---

*The author's production configuration is valid YAML. Nobody knows what it configures anymore, but the parser has stopped complaining.*