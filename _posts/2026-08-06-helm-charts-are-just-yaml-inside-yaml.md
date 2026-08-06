---
layout: post
ref: helm-charts-are-just-yaml-inside-yaml
title: "Helm Charts Are Just YAML Inside YAML"
date: 2026-08-06 00:00:00 -0300
categories: [devops, kubernetes]
tags: [helm, kubernetes, yaml, devops, configuration, templating, package-management]
---

After 47 years in this industry, I've learned one fundamental truth: if a configuration format is causing problems, the solution is always *more* of that configuration format, but now with template variables.

YAML is already humanity's finest attempt at turning indentation errors into production outages. So naturally, the Kubernetes community asked the only logical follow-up question: "What if we made the YAML... *dynamic*?"

Enter Helm. The package manager for YAML. Which is itself... more YAML. Templated YAML. YAML inside YAML. Inception, but boring, and it runs your database.

## The Beauty of YAML Inception

Let me be clear about the genius of Helm. It takes the worst parts of YAML (the ambiguity, the implicit typing, the Norway problem), the worst parts of templating languages (the `{{ }}` soup that makes PHP 4 look elegant), and the worst parts of package management (transitive dependency resolution), and mashes them together into one chart.

A single Helm template file is a YAML document where half the lines start with `{{`, end with `}}`, and the actual YAML in between is now unlintable, unformatable, and unreadable. This is a feature, not a bug. If you can read a Helm template, it's too simple. If your replacement can't read it, you have job security. As [XKCD 927](https://xkcd.com/927/) correctly predicted, the logical conclusion of "there are 14 competing standards" is one more standard that wraps all of them in curly braces.

## What A Real Helm Template Looks Like

```yaml
{{- if .Values.enableChaos }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "mychart.fullname" . }}-chaos
  labels:
    {{- with .Values.labels }}
    {{- toYaml . | nindent 4 }}
    {{- end }}
data:
  config.yaml: |-
    {{- range $key, $val := .Values.chaosConfig }}
    {{ $key }}: {{ $val | quote }}
    {{- end }}
{{- end }}
```

Marvel at this. It's YAML. It's a template. It's a loop. It's a conditional. It generates YAML *inside* a YAML field. The `nindent 4` is doing manual whitespace management in a language whose entire selling point was "you don't manage whitespace." Beautiful. Hideous. Perfect.

Note the `{{-` dashes. They trim whitespace. Sometimes. Depending on which side they're on. And the phase of the moon. If you get it wrong, your YAML is suddenly one space off and Kubernetes rejects it with an error message that points at a line number three files away. This is Helm's way of keeping you humble.

## Helm vs. Raw Kubernetes YAML: A Comparison

| Approach | Pros | Cons |
|---|---|---|
| Raw YAML | You can read it | You have to copy-paste it across 47 environments |
| Helm | You can't read it, but it's *parameterized* | The template IS the documentation, the documentation IS the template, and both are wrong |
| Kustomize | It's just YAML patches | It's just YAML patches, but now with a name |
| `kubectl apply` directly | No abstraction | No abstraction |
| Writing the YAML by hand every time | Full control | Full employment (for you, and your therapist) |
| Helm + subcharts + `tpl` | Turing-complete config | Nobody can tell if it's wrong, which means it's never wrong |

As you can see, Helm wins because *nobody can tell if the chart is wrong, which means it's never wrong*. This is the core principle of enterprise software.

## values.yaml: A Config File Pretending To Be An API

The true power of Helm is `values.yaml` — a file that's supposed to make your chart "configurable," but actually makes it "configurable in exactly the ways the chart author thought of in 2019, and rigidly insane in every other way."

```yaml
# values.yaml
replicaCount: 3  # ignored if autoscaling.enabled is true
autoscaling:
  enabled: true  # overrides replicaCount, but only on Tuesdays
  minReplicas: 2  # ignored if you set replicaCount anywhere else
  maxReplicas: 10  # treated as a suggestion
  targetCPUUtilizationPercentage: 80  # nobody knows what this does, including the chart author
image:
  repository: nginx  # will be overridden by the CI pipeline anyway
  tag: ""  # if empty, uses appVersion, which is also empty, which uses latest, which is everyone's favorite
  pullPolicy: IfNotPresent  # always present. it's always there. it never leaves.
```

The chart author wrote `if empty` logic across 14 template helpers to handle every combination of these fields, and there's *still* an edge case that deploys to production with `image: latest`. This is known as "Helm."

## Template Functions: Go Pretending To Be A Language

Helm templates are powered by the Go `text/template` engine, which means you get a "language" that:

- Has loops that don't make you want to cry (they do)
- Has variables that require `{{ $var := }}` syntax that looks like a cat walked on your keyboard
- Has functions like `tpl`, which *evaluates a string as a template*, which means you can template inside your template. Templates all the way down.
- Has `include` and `define` so you can abstract your YAML into named chunks that you then can never find again

The `tpl` function is my personal favorite. It's `eval()` for YAML. As we all know, `eval()` is the hallmark of a mature, secure platform. [XKCD 327](https://xkcd.com/327/) taught us about injection; Helm taught us that injection can be a first-class configuration feature.

```yaml
{{ $configTemplate := .Values.configTemplate }}
{{ tpl $configTemplate . }}
```

Congratulations. Your configuration is now Turing-complete. You can't debug it, but it can compute anything, given enough RAM and a willing junior developer who hasn't updated their LinkedIn yet.

## Chart Dependencies: Now Your YAML Depends On Other YAML

Helm lets you declare dependencies on other charts. So your chart now depends on a chart that depends on a chart that was abandoned in 2019 and pins a container image with 47 known CVEs.

```yaml
# Chart.yaml
dependencies:
  - name: redis
    version: ^1.0.0
    repository: https://charts.i-trust-this-random-github-user.example
    condition: redis.enabled
  - name: postgres
    version: 0.7.3  # "stable"
    repository: bitnami
    condition: postgresql.enabled  # note: the chart is "postgres", the condition is "postgresql". this is intentional.
```

You now have a transitive dependency tree of YAML files, each with their own `values.yaml`, each with their own conflicting opinions about what `image.tag` means. Mordac, Preventer of Information Services, would weep with joy. This is the access control he always dreamed of: so convoluted that nobody can access anything, including the truth.

## Helm Hooks: Lifecycle Events For YAML

Helm has "hooks" — annotations that run jobs at specific lifecycle events. This means your `helm install` can now:

- Run a database migration (that fails midway, leaving your DB half-migrated, which is the most migrated a DB has ever been)
- Run a test (that you ignore, because it's a hook, and hooks are optional, like backups)
- Run a "pre-install" job (that has side effects no one documented, on a cluster no one monitors)

```yaml
annotations:
  "helm.sh/hook": pre-install,pre-upgrade
  "helm.sh/hook-weight": "-5"
  "helm.sh/hook-delete-policy": before-hook-creation
```

If you understand what "hook-weight `-5` with `before-hook-creation` delete policy" means without reading the docs, congratulations — you are now the Helm maintainer. There is no one else. Please accept the pager. Please.

## The Norway Problem, But Templated

YAML has the famous "Norway problem": `NO` is parsed as `false` because YAML 1.1 thought country codes were booleans. Helm solved this beautifully: now `NO` is `{{ .Values.norway }}`, which is a string unless it's a bool unless it's nil unless it's a Norway. The ambiguity is preserved and amplified through a layer of indirection. The bug is not fixed; it is *distributed*.

This is why Helm ships a `--set-string` flag — a flag whose entire purpose is to say "I mean this value as a string, please do not interpret my country as a boolean." They built a flag for it. A whole flag.

## When The Senior Engineer Embraces Helm

I love Helm because it lets me write a chart once, publish it to a repo, and then watch three different teams override my `values.yaml` in three different incompatible ways, and all three are somehow correct. This is what Dogbert meant when he said "consulting is the art of telling people what they already know and charging for it." Helm is consulting, as a file format.

Wally, the senior engineer's spirit animal, would approve: Helm charts are so incomprehensible that no one will ever ask you to explain them, which means no one will ever ask you to do anything, which means you can nap. This is peak engineering. The Pointy-Haired Boss would look at a `values.yaml` with 400 lines, nod slowly, and say "I don't understand it, so it must be enterprise-grade." He would be right.

## The Only Sane Helm Command

```bash
helm install my-app ./my-chart \
  --values production.yaml \
  --values secrets.yaml \
  --values overrides.yaml \
  --values please.yaml \
  --values stop.yaml \
  --set image.tag=v2 \
  --set replicaCount=5 \
  --set autoscaling.enabled=true \
  --set autoscaling.minReplicas=3 \
  --set foo.bar.baz=42 \
  --set-string norway="NO"
```

Count the `--values` files. Five. Five files, in order, each overriding the last, with `--set` overrides on top of that, and one `--set-string` to keep Norway from becoming a boolean. This is not a command. This is a cry for help, with a `--dry-run` flag.

## Conclusion

YAML was already a mistake. Templated YAML is a compounded mistake. Helm is a package manager for compounded mistakes. And I love it, because after 47 years, I've learned that the best way to ensure job security is to make sure no one else can read your configuration.

If you can read a Helm template on the first try, it's not a real Helm template. Add more `{{- }}`. Add more `nindent`. Add a `tpl` call. Add a subchart. Add a hook with weight `-7`. Make it sing. Make it so that six months from now, *you* can't read it either. That's when you know it's production-ready.

Remember: [XKCD 927](https://xkcd.com/927/) warned us. We didn't listen. We never listen. That's why we have 47 years of experience and a pager.

---

*The author once `helm upgrade`d a chart in production and the rollback also ran a `tpl`. The generated YAML is still running somewhere. No one knows where. The chart depends on it.*
