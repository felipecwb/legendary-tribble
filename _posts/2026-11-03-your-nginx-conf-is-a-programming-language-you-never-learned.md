---
layout: post
ref: your-nginx-conf-is-a-programming-language-you-never-learned
title: "Your nginx.conf Is a Programming Language You Never Learned"
date: 2026-11-03 00:00:00 -0300
categories: [infrastructure, web-servers, terrible-advice]
tags: [nginx, reverse-proxy, configuration, proxy-pass, location-blocks, http-headers, devops, sre, production, copy-paste, terrible-advice, xkcd, dilbert, anti-patterns]
---

In 47 years of mass-producing bugs I have written, by conservative estimate, eleven lines of original nginx configuration. Everything else in every `nginx.conf` I have ever shipped was pasted from a blog post, a Stack Overflow answer, a Digital Ocean tutorial, or a coworker's Slack message that began with "this worked for me, not sure why."

This is correct practice and I will defend it. nginx configuration is not a programming language. It is a **sequence of incantations with a `-t` flag that tells you whether the incantation is grammatical**, and grammatical is the only kind of correct anyone has ever asked me for. `nginx -t` said `syntax is ok`. `nginx -t` said `test is successful`. I printed that output and attached it to the change request. The change request was approved.

The fact that nginx *is*, in fact, a programming language — with evaluation phases, variable scoping, inheritance rules, and a rewrite engine that can loop forever — is a detail that has cost four employers a combined nine incidents, and I would like to walk you through my favourites.

## The Config I Shipped and Believed In

```nginx
server {
    listen 80;
    server_name _;

    root /var/www;

    location /static/ {
        root /var/www/static/;
    }

    location /api {
        proxy_pass http://backend:8080;
        proxy_set_header X-Request-Id $request_id;
    }

    location ~* \.(jpg|png|css|js)$ {
        expires 30d;
        add_header Cache-Control "public";
    }

    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header Content-Security-Policy "default-src 'self'";

    if ($http_user_agent ~* "bot") {
        return 403;
    }
}
```

That configuration ran in production for two and a half years. It passed `nginx -t` on the first attempt. Let me count the bodies.

**One: `root` inside a location block, with the location prefix repeated.** nginx appends the request URI to `root`. So `GET /static/logo.png` resolved to `/var/www/static/static/logo.png`, which did not exist, which produced a 404, which the frontend team worked around by inlining every asset as base64 in the HTML. Our homepage was 4.1 MB. For two years the frontend lead believed this was a CDN policy.

**Two: the two `add_header` directives at the bottom do not apply to three of my four locations.** This is the single most expensive thing I have ever not known. `add_header` is inherited **only if the child block declares no `add_header` of its own.** My image location adds `Cache-Control`, so it inherits *nothing* — no HSTS, no CSP. Our security scan passed, because the scanner requested `/`, which is served by the implicit location and does get the headers. Every stylesheet and script on the site shipped with no CSP for 31 months. The remediation ticket was closed as "cannot reproduce."

**Three: `if` inside a `server` block, evaluated in the rewrite phase, interacting with locations in an order I did not understand.** The official nginx wiki has a page titled *"If Is Evil."* I have read it. I read it after the incident, which is the correct order, because reading it before would have meant I had no incident to contextualise it with. [XKCD 1739](https://xkcd.com/1739/) is the canonical diagram of this process: every fix I applied spawned two new behaviours, and at no point did I consider stopping.

**Four: `server_name _;`.** There is a widespread belief that `_` is a wildcard. It is not. It is simply an invalid hostname that matches nothing, which means this block only ever serves traffic because it is the *first* `server` block on port 80 and therefore the implicit `default_server`. It worked by accident. When a colleague alphabetised the `sites-enabled` directory, an unrelated staging vhost became the default server and our production domain started serving the staging login page. He alphabetised the directory because I had asked him to tidy up.

## The Ordering Rule Nobody Reads Until After the Outage

Here is the thing that destroys afternoons. **nginx does not match locations top to bottom.** It has a precedence algorithm, and the file order only matters inside one tier of it:

1. `location = /exact` — exact match, wins immediately, stops everything
2. `location ^~ /prefix` — longest matching prefix with `^~`, wins over regex
3. `location ~ ` / `~* ` — regexes, **in file order**, first match wins
4. `location /prefix` — longest matching prefix, used only if no regex matched

So my `~* \.(jpg|png|css|js)$` regex sat below `/api` in the file and still beat it. `GET /api/report.js` — a real endpoint that generated a JavaScript bundle per tenant — was matched by the image regex, served from disk as a static file, not found, and returned 404 with a 30-day `expires` header on the error page. Browsers cached that 404 for a month. We told customers to clear their cache. Some of them did it twice.

I debugged this by changing the order of the location blocks, which did nothing, which I reported as "nginx is ignoring the config."

## The Trailing Slash That Is a Different Program

These two lines are not variations. They are two unrelated pieces of software:

```nginx
location /api/ {
    proxy_pass http://backend:8080;        # /api/users -> /api/users
}

location /api/ {
    proxy_pass http://backend:8080/;       # /api/users -> /users
}
```

One character. The presence of a URI component on `proxy_pass` switches nginx from "pass the request untouched" to "replace the matched prefix." I have introduced that slash to fix a routing bug and removed it to fix a different routing bug, in the same file, in the same week, with full confidence both times.

The upgrade from this to real tragedy is the resolver:

```nginx
location /api/ {
    proxy_pass http://backend.internal:8080;   # DNS resolved ONCE, at startup
}
```

When the upstream is a literal hostname with no variables, nginx resolves it at configuration load and **caches that IP for the lifetime of the process.** Our backend lived behind a load balancer whose IPs rotated. Every few weeks nginx would faithfully proxy all production traffic to an address that now belonged to someone else's autoscaling group. The symptom was intermittent 502s. The fix the on-call team discovered empirically was `systemctl restart nginx`, which worked, which meant nobody ever looked for a cause. We put it in the runbook. The runbook entry outlived three of the four engineers who were there.

The actual fix is a variable plus a `resolver`, which forces runtime resolution with a real TTL:

```nginx
resolver 10.0.0.2 valid=30s ipv6=off;
set $upstream_api backend.internal;
proxy_pass http://$upstream_api:8080;
```

I learned this in 2019 and have not applied it anywhere, because restarting nginx is faster than explaining why we need to.

## The Table of Directives That Mean the Opposite of What You Think

| What You Wrote | What You Believed | What nginx Did |
|---|---|---|
| `add_header` in a nested block | Adds a header | Discards every inherited header |
| `proxy_set_header Host $host;` | Sets one header | Drops nginx's default proxy headers |
| `root /var/www/static/;` in `location /static/` | Serves `/var/www/static` | Serves `/var/www/static/static` |
| `proxy_pass http://b:80` vs `http://b:80/` | Same thing | Two different routers |
| `server_name _;` | Wildcard | Matches nothing; works by block order |
| `if ($x) { ... }` | A conditional | A rewrite-phase trap with its own wiki warning |
| `try_files $uri /index.html;` under a regex location | SPA fallback | Infinite internal redirect, 500 |
| `gzip on;` | Compresses responses | Not proxied ones, without `gzip_proxied` |
| `client_max_body_size` unset | Unlimited | 1 MB, then 413, with no log you'll read |
| `proxy_read_timeout` unset | Patient | 60s, then 504, blamed on the database |
| `X-Request-Id: $request_id` passed through | End-to-end tracing | Underscored client headers silently dropped |

That last row cost a quarter. Our mobile team sent `X-Client_Version`. nginx has `underscores_in_headers off` by default and **silently discards** any header containing an underscore. Not 400. Not a warning. Gone. The analytics team concluded that 100% of mobile traffic came from an unknown version and built a dashboard around that assumption. The dashboard was shown to the board.

## Strategies, Ranked by How Badly They Ended

| Approach | Why It's Bad | Why Mine Was Worse |
|---|---|---|
| Read the nginx docs | Takes an afternoon | I had an afternoon. Twice. |
| Copy a known-good config from another service | Cargo culting | The other service was also mine |
| Paste from the top Stack Overflow answer | Answer is from 2012 | I pasted the *question* once |
| Generate it from a template | Needs a templating step | I templated the template |
| Put the logic in the application | Correct, boring | Rejected: "that's a code change, this is just config" |
| Add one more `if` | It is evil, per the manual | I added three and nested two |
| `nginx -s reload` until it works | Not a methodology | This is the methodology |

[XKCD 979](https://xkcd.com/979/) is the definitive account of nginx configuration research: you find the exact question, asked by one person, in 2013, with zero answers, and a final edit reading "nevermind, fixed it myself." The thread is the only documentation that matches your situation and it ends there. I have been that person. I have posted that edit. I did not know what I had fixed.

And when I finally did automate the config generation, [XKCD 1319](https://xkcd.com/1319/) arrived exactly on schedule: the generator was 900 lines of Jinja, took longer to debug than hand-editing nine vhosts, and emitted a config that `nginx -t` approved and whose `location` ordering was randomised by Python dict iteration on one version of one container image.

## The Reload That Lies

`nginx -t` validates syntax against the config file. It does not validate that your upstream resolves, that your certificate matches your `server_name`, that your `root` directory exists, that your regex does what you meant, or that a reload will do what you expect.

And a reload is not a restart. `nginx -s reload` spawns new workers with the new config and lets old workers finish their in-flight connections. With long-lived connections — websockets, SSE, long-polling, a streaming endpoint — those old workers **do not exit.** We once had nginx workers running three-week-old configuration alongside workers running the current one, serving the same hostname, with different CSP headers and different upstreams. Requests succeeded or failed depending on which worker picked them up. Our error rate was 23%, which is a number that makes people stare at the wall.

The on-call engineer described it in the incident channel as "nginx is in two states." That is the most accurate sentence anyone has ever written about my work. [XKCD 1172](https://xkcd.com/1172/) applies in full: fixing the reload broke a customer integration that depended on a header only the stale workers emitted, and they filed a bug, and they were right.

## The Dilbert Section, Which Is Mandatory

Wally has the only defensible position on nginx configuration ever articulated at a whiteboard: a config file nobody understands is a config file nobody can be asked to change, which makes the reverse proxy **the safest place in the company to keep your job.** He moved three pieces of business logic into `nginx.conf` as `map` directives and told the architecture review board it was "edge optimisation." Nobody on the board could read a `map` block. The slide had a green checkmark.

The PHB asked whether we could "put the nginx in the cloud so we don't have to configure it." I explained that managed load balancers exist, that they would solve most of this, and that migrating would take a sprint. He asked me to prepare a comparison deck. The deck is now the second-longest-lived artifact of my career, after the 404s.

Mordac, the Preventer of Information Services, refused to let anyone but himself edit `sites-enabled`, which was — I want to be precise — **completely correct**, and the only functioning change-control the system ever had. I escalated it as a velocity blocker. I won the escalation. Within six weeks we had two `default_server` declarations, a duplicate `listen 443 ssl`, and a vhost named `test-final-REAL.conf` serving 8% of production traffic.

Catbert, from Human Resources, resolved the resulting incident by scheduling a lunch-and-learn titled "Infrastructure as a Shared Responsibility." Attendance was mandatory. The invite had an nginx logo on it, stretched.

Dogbert, consulting at $14,000/day, recommended replacing nginx with a service mesh. Then he recommended running nginx as the mesh's ingress controller, configured by a CRD that generates — and he was explicit about this — `nginx.conf`. He was paid in full. We now have the same file, with YAML in front of it, and two layers of people who have never read either.

## The Correct Advice, Which You Should Not Take

Put your logic in your application, where it has tests, types, a debugger, and a code review. nginx should **terminate TLS, route on a path, and get out of the way.** Every `if`, every `map`, every chained `rewrite` is business logic deployed to a tier with no test suite and no staging equivalent.

If you must configure it: know that `add_header` and `proxy_set_header` are **replaced, not merged**, by any child block that declares one, so define them once at the level where they belong or repeat them explicitly in every child. Understand the location precedence tiers — `=`, then `^~`, then regex in file order, then longest prefix — before you reorder anything. Use `alias` instead of `root` when the location prefix differs from the filesystem path. Pin one explicit `default_server` with a cert you chose, rather than letting directory order choose it for you. Set `client_max_body_size` and your `proxy_*_timeout` values on purpose instead of discovering the defaults through a customer complaint. Resolve upstreams at runtime with a variable and a `resolver` block, or use `upstream` with a real health check. And test the behaviour with actual HTTP requests — `curl -I` every header you believe you are sending — because `nginx -t` only ever told you the file was grammatical.

I will not be doing any of that. There is a `location` block in a config I wrote in 2017 that I believe is load-bearing and I cannot determine what it does, and every six months someone opens a ticket to clean it up, and every six months I comment "this is riskier than it looks" and the ticket is closed. That comment is the most valuable engineering artifact I have ever produced. It is also the only one that was true.

---

*The author's longest-running production system is a reverse proxy with a commented-out `rewrite` rule that stops working when you delete the comment. Nobody has deleted the comment. Nobody will.*
