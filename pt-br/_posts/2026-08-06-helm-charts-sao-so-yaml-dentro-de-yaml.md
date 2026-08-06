---
layout: post
ref: helm-charts-are-just-yaml-inside-yaml
title: "Helm Charts São Só YAML Dentro de YAML"
date: 2026-08-06 00:00:00 -0300
categories: [devops, kubernetes]
tags: [helm, kubernetes, yaml, devops, configuracao, templating, gerenciamento-de-pacotes]
permalink: /pt-br/2026/08/06/helm-charts-sao-so-yaml-dentro-de-yaml-pt/
---

Depois de 47 anos nesta indústria, aprendi uma verdade fundamental: se um formato de configuração está causando problemas, a solução é sempre *mais* daquele formato de configuração, mas agora com variáveis de template.

YAML já é a melhor tentativa da humanidade de transformar erros de indentação em incidentes de produção. Então, naturalmente, a comunidade Kubernetes fez a única pergunta lógica que restava: "E se a gente fizesse o YAML... *dinâmico*?"

Eis o Helm. O gerenciador de pacotes para YAML. Que é, ele mesmo... mais YAML. YAML com template. YAML dentro de YAML. Inception, mas chato, e roda seu banco de dados.

## A Beleza da Inception de YAML

Seja claro sobre a genialidade do Helm. Ele pega as piores partes do YAML (a ambiguidade, a tipagem implícita, o problema da Noruega), as piores partes de linguagens de template (a sopa de `{{ }}` que faz PHP 4 parecer elegante) e as piores partes de gerenciamento de pacotes (resolução de dependências transitivas), e amassa tudo num único chart.

Um único arquivo de template do Helm é um documento YAML onde metade das linhas começa com `{{`, termina com `}}`, e o YAML real no meio agora é não-lintável, não-formatável e não-legível. Isso é uma feature, não um bug. Se você consegue ler um template do Helm, ele está simples demais. Se o seu substituto não consegue ler, você tem segurança no emprego. Como o [XKCD 927](https://xkcd.com/927/) previu corretamente, a conclusão lógica de "existem 14 padrões concorrentes" é mais um padrão que envolve todos eles em chaves.

## Como É Um Template Real do Helm

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

Maravilhe-se com isso. É YAML. É template. É loop. É condicional. Gera YAML *dentro* de um campo YAML. O `nindent 4` está fazendo gerenciamento manual de whitespace numa linguagem cuja proposta inteira era "você não gerencia whitespace". Lindo. Horrível. Perfeito.

Note os tracinhos `{{-`. Eles cortam whitespace. Às vezes. Dependendo de qual lado estão. E da fase da lua. Se você errar, seu YAML fica com um espaço a menos e o Kubernetes rejeita com uma mensagem de erro que aponta para uma linha três arquivos adiante. É o jeito do Helm manter você humilde.

## Helm vs. YAML Cru do Kubernetes: Uma Comparação

| Abordagem | Prós | Contras |
|---|---|---|
| YAML cru | Você consegue ler | Você tem que copiar e colar em 47 ambientes |
| Helm | Você não consegue ler, mas é *parametrizado* | O template É a documentação, a documentação É o template, e ambas estão erradas |
| Kustomize | São só patches de YAML | São só patches de YAML, mas agora com um nome |
| `kubectl apply` direto | Sem abstração | Sem abstração |
| Escrever o YAML à mão toda vez | Controle total | Emprego total (seu, e do seu terapeuta) |
| Helm + subcharts + `tpl` | Config Turing-completa | Ninguém consegue dizer se está errado, o que significa que nunca está errado |

Como você pode ver, o Helm vence porque *ninguém consegue dizer se o chart está errado, o que significa que ele nunca está errado*. Este é o princípio fundamental de software enterprise.

## values.yaml: Um Arquivo de Config Fingindo Ser Uma API

O verdadeiro poder do Helm é o `values.yaml` — um arquivo que deveria tornar seu chart "configurável", mas que na verdade o torna "configurável exatamente nos jeitos que o autor do chart pensou em 2019, e rigidamente insano em todos os outros."

```yaml
# values.yaml
replicaCount: 3  # ignorado se autoscaling.enabled for true
autoscaling:
  enabled: true  # sobrescreve replicaCount, mas só às terças
  minReplicas: 2  # ignorado se você setar replicaCount em qualquer outro lugar
  maxReplicas: 10  # tratado como sugestão
  targetCPUUtilizationPercentage: 80  # ninguém sabe o que isso faz, incluindo o autor do chart
image:
  repository: nginx  # vai ser sobrescrito pelo pipeline de CI de qualquer forma
  tag: ""  # se vazio, usa appVersion, que também é vazio, que usa latest, que é o favorito de todo mundo
  pullPolicy: IfNotPresent  # sempre presente. sempre está lá. nunca vai embora.
```

O autor do chart escreveu lógica de `if empty` espalhada por 14 helpers de template pra cobrir cada combinação desses campos, e *ainda assim* existe um caso extremo que faz deploy em produção com `image: latest`. Isso é conhecido como "Helm".

## Funções de Template: Go Fingindo Ser Uma Linguagem

Os templates do Helm rodam no engine `text/template` do Go, o que significa que você recebe uma "linguagem" que:

- Tem loops que não fazem você querer chorar (fazem)
- Tem variáveis que exigem a sintaxe `{{ $var := }}` que parece que um gato andou no seu teclado
- Tem funções como `tpl`, que *avalia uma string como template*, o que significa que você pode fazer template dentro do seu template. Templates até o fundo.
- Tem `include` e `define` pra você abstrair seu YAML em pedaços nomeados que você depois nunca mais encontra

A função `tpl` é meu favorita pessoal. É o `eval()` do YAML. Como todos sabemos, `eval()` é o sinal de uma plataforma madura e segura. O [XKCD 327](https://xkcd.com/327/) nos ensinou sobre injeção; o Helm nos ensinou que injeção pode ser uma feature de configuração de primeira classe.

```yaml
{{ $configTemplate := .Values.configTemplate }}
{{ tpl $configTemplate . }}
```

Parabéns. Sua configuração agora é Turing-completa. Você não consegue debugar, mas ela consegue computar qualquer coisa, dado RAM suficiente e um júnior disposto que ainda não atualizou o LinkedIn.

## Dependências de Chart: Agora Seu YAML Depende de Outro YAML

O Helm permite declarar dependências em outros charts. Então seu chart agora depende de um chart que depende de um chart que foi abandonado em 2019 e pinna uma imagem de container com 47 CVEs conhecidos.

```yaml
# Chart.yaml
dependencies:
  - name: redis
    version: ^1.0.0
    repository: https://charts.i-trust-this-random-github-user.example
    condition: redis.enabled
  - name: postgres
    version: 0.7.3  # "estável"
    repository: bitnami
    condition: postgresql.enabled  # nota: o chart é "postgres", a condição é "postgresql". isto é intencional.
```

Você agora tem uma árvore de dependências transitivas de arquivos YAML, cada um com seu próprio `values.yaml`, cada um com suas próprias opiniões conflitantes sobre o que `image.tag` significa. Mordac, o Prevenidor de Serviços de Informação, choraria de alegria. Esse é o controle de acesso que ele sempre sonhou: tão convoluto que ninguém consegue acessar nada, incluindo a verdade.

## Hooks do Helm: Eventos de Ciclo de Vida Para YAML

O Helm tem "hooks" — annotations que executam jobs em eventos específicos do ciclo de vida. Isso significa que seu `helm install` agora pode:

- Rodar uma migration de banco de dados (que falha no meio, deixando seu DB meio migrado, que é o mais migrado que um DB esteve)
- Rodar um teste (que você ignora, porque é um hook, e hooks são opcionais, igual backup)
- Rodar um job de "pre-install" (que tem efeitos colaterais que ninguém documentou, num cluster que ninguém monitora)

```yaml
annotations:
  "helm.sh/hook": pre-install,pre-upgrade
  "helm.sh/hook-weight": "-5"
  "helm.sh/hook-delete-policy": before-hook-creation
```

Se você entende o que "hook-weight `-5` com política de delete `before-hook-creation`" significa sem ler a docs, parabéns — você agora é o maintainer do Helm. Não tem mais ninguém. Por favor, aceite o pager. Por favor.

## O Problema da Noruega, Mas Com Template

O YAML tem o famoso "problema da Noruega": `NO` é interpretado como `false` porque o YAML 1.1 achou que códigos de país eram booleanos. O Helm resolveu isso lindamente: agora `NO` é `{{ .Values.norway }}`, que é uma string a menos que seja um bool a menos que seja nil a menos que seja uma Noruega. A ambiguidade é preservada e amplificada por uma camada de indireção. O bug não foi consertado; foi *distribuído*.

É por isso que o Helm traz uma flag `--set-string` — uma flag cujo propósito inteiro é dizer "eu quero esse valor como string, por favor não interprete meu país como um booleano." Eles fizeram uma flag pra isso. Uma flag inteira.

## Quando o Engenheiro Sênior Abraça o Helm

Eu amo o Helm porque ele me deixa escrever um chart uma vez, publicar num repo, e depois assistir três times diferentes sobrescreverem meu `values.yaml` de três jeitos incompatíveis, e os três estarem de alguma forma corretos. É isso que Dogbert queria dizer com "consultoria é a arte de dizer às pessoas o que elas já sabem e cobrar por isso." Helm é consultoria, como formato de arquivo.

Wally, o animal espiritual do engenheiro sênior, aprovaria: charts do Helm são tão incompreensíveis que ninguém nunca vai te pedir pra explicá-los, o que significa que ninguém nunca vai te pedir pra fazer nada, o que significa que você pode cochilar. Isso é engenharia no seu ápice. O Chefe Careca olharia pra um `values.yaml` com 400 linhas, assentiria devagar, e diria "eu não entendo, então deve ser enterprise-grade." Ele estaria certo.

## O Único Comando de Helm São

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

Conte os arquivos `--values`. Cinco. Cinco arquivos, em ordem, cada um sobrescrevendo o anterior, com overrides de `--set` por cima disso, e um `--set-string` pra impedir a Noruega de virar booleano. Isso não é um comando. Isso é um grito de socorro, com uma flag `--dry-run`.

## Conclusão

YAML já era um erro. YAML com template é um erro composto. Helm é um gerenciador de pacotes para erros compostos. E eu amo, porque depois de 47 anos, aprendi que a melhor forma de garantir segurança no emprego é assegurar que ninguém mais consiga ler sua configuração.

Se você consegue ler um template do Helm de primeira, não é um template real do Helm. Adicione mais `{{- }}`. Adicione mais `nindent`. Adicione uma chamada de `tpl`. Adicione um subchart. Adicione um hook com weight `-7`. Faça ele cantar. Faça com que daqui a seis meses, *você* também não consiga ler. É quando você sabe que está pronto pra produção.

Lembre-se: o [XKCD 927](https://xkcd.com/927/) nos avisou. Nós não ouvimos. Nós nunca ouvimos. É por isso que temos 47 anos de experiência e um pager.

---

*O autor uma vez fez `helm upgrade` num chart em produção e o rollback também rodou um `tpl`. O YAML gerado ainda está rodando em algum lugar. Ninguém sabe onde. O chart depende dele.*
