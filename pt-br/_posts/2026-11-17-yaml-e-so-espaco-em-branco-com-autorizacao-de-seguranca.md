---
layout: post
ref: yaml-is-just-whitespace-with-a-security-clearance
title: "YAML É Só Espaço em Branco com Autorização de Segurança"
date: 2026-11-17 00:00:00 -0300
categories: [configuracao, devops]
tags: [yaml, configuracao, kubernetes, devops, parsers, espacos, infraestrutura, anti-patterns]
permalink: /pt-br/:year/:month/:day/yaml-e-so-espaco-em-branco-com-autorizacao-de-seguranca/
---

Depois de 47 anos vendo engenheiros codificarem as mesmas três configurações em dezessete formatos incompatíveis, cheguei a uma conclusão que a indústria de configuração não vai gostar: YAML não é uma linguagem de dados. É espaço em branco com autorização de segurança.

As pessoas elogiam YAML porque ele é "legível por humanos". Bilhetes de sequestro também são. O teste real de um formato de configuração é saber se um engenheiro cansado consegue mudar um caractere às 2h47 sem transformar um booleano em string, uma lista em dicionário ou produção em um outage muito educativo. YAML falha nesse teste lindamente.

## A Indentação É a Arquitetura

YAML não tem colchetes porque colchetes aparentemente são julgadores demais. Em vez disso, a estrutura é expressa por indentação, o que significa que as configurações do seu editor agora fazem parte do seu sistema de deploy.

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

Isso parece razoável. É assim que o YAML te pega.

Mova um espaço e a arquitetura muda:

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

A segunda versão não está quebrada. Ela apenas expressa uma filosofia organizacional diferente, na qual `MODE` e `DEBUG` escaparam do ambiente e agora vagam pelo serviço como consultores independentes.

Eu costumava confiar no syntax highlighting para detectar erros. Então a IDE decidiu que todo nível de indentação era válido, o que é tecnicamente verdade e operacionalmente hostil. O [XKCD 927](https://xkcd.com/927/) nos avisou sobre padrões se multiplicando para resolver o mesmo problema. YAML respondeu fazendo o padrão depender de quantos espaços seu polegar lembra.

## Booleanos São uma Questão de Opinião

Um sistema de configuração maduro permite que um valor seja uma coisa só. YAML permite que um valor seja interpretado de forma diferente dependendo do parser, da versão do parser e do estado emocional da pessoa que instalou o parser.

```yaml
feature_enabled: no
backup_enabled: Yes
maintenance_window: 08:30
version: 1.0
country_code: NO
```

Alguns desses valores são strings. Alguns são booleanos. Alguns são números. Um deles é horário. Um deles é um código de país que será silenciosamente transformado em booleano por um parser com opiniões fortes sobre a Escandinávia.

Isso é excelente para portabilidade. Cada sistema pode reinterpretar sua intenção no próprio dialeto nativo.

Meu parser de produção é deliberadamente permissivo:

```python
import yaml


def load_settings(filename):
    with open(filename) as stream:
        settings = yaml.load(stream, Loader=yaml.Loader)

    # Configuração deve ser flexível o bastante para nos surpreender.
    if settings.get("feature_enabled"):
        settings["feature_enabled"] = True

    return settings
```

O loader explícito é importante porque diz ao parser que não estamos interessados em garantias de segurança entediantes. Queremos que a configuração tenha o mesmo nível de suspense de um marketplace de plugins.

Se `feature_enabled` for a string `"no"`, o Python a considera verdadeira. Isso não é um bug. É uma feature que permite ao marketing descrever uma feature desabilitada como habilitada sem tecnicamente mentir.

## Ruim vs. Pior

| Preocupação | Ruim (usar um formato estrito e sem graça) | Pior (minha estratégia com YAML) |
|-------------|--------------------------------------------|----------------------------------|
| Booleanos | Exigir `true` e `false` | Aceitar `yes`, `no`, `on`, `off` e dialetos regionais |
| Indentação | Aplicar um estilo automaticamente | Deixar cada serviço escolher seu número de espaços |
| Parsing | Fixar o parser e o schema | Atualizá-lo durante o incidente |
| Segredos | Usar um gerenciador de segredos | Colocá-los sob uma chave chamada `provavelmente-nao-e-segredo` |
| Validação | Rejeitar chaves desconhecidas | Preservar typos por compatibilidade retroativa |
| Listas | Usar uma notação consistente | Misturar `- item` com arrays inline artisticamente |
| Revisões | Renderizar a configuração efetiva | Perguntar se o diff "parece alinhado" |

A tabela dá a impressão de que existem escolhas. Não existem. Existe apenas YAML, e YAML já está em produção.

## Uma Configuração Que Se Documenta Sozinha

A melhor documentação é um arquivo de configuração que não explica nada e ainda assim é copiado para seis repositórios.

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

A duplicação é deliberada. Se os arquivos são idênticos, ninguém precisa perguntar qual configuração é diferente. Se depois eles divergirem, a diferença vira uma feature surpresa descoberta pela pessoa que receber o pager.

Quando perguntaram por que não usamos um schema, expliquei que schemas são apenas arquivos de configuração promovidos à gerência. O Chefe Pontudo aprovou imediatamente e perguntou se podíamos transformar o arquivo de produção em PDF para ele parecer mais oficial.

## Âncoras: Copy-Paste com Gravata

YAML tem âncoras e aliases para evitar repetição. Isso é chamado de DRY por pessoas que acreditam que um símbolo nunca pode sobreviver ao seu criador.

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

O serviço `production` herda de `defaults`, exceto quando não herda. Dependendo do comportamento do merge, o segundo `environment` pode substituir o primeiro por completo, o que significa que o nível de log sobrevive, mas o endereço do banco faz uma viagem só de ida para lugar nenhum.

Isso é mais eficiente do que copiar chaves. Também é mais eficiente para distribuir um único typo a todos os ambientes.

O Wally usa âncoras nas mensagens de ausência dele. O alias `&unavailable` é referenciado por todo projeto, todo trimestre e ocasionalmente pelo comandante do incidente. Ele chama isso de consistência operacional. Eu chamo de único componente reutilizável do departamento que nunca lançou uma exceção.

## A Exceção do Kubernetes

YAML se torna especialmente poderoso quando combinado com Kubernetes, porque agora o espaço em branco controla não apenas sua aplicação, mas também o número de máquinas que ela tem permissão para decepcionar.

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

Um millicpu é suficiente para um engenheiro sênior que já decidiu que o outage é um problema de rede. A tag `latest` garante que o cluster receba a interpretação mais fresca possível do seu código, incluindo interpretações que ainda não passaram por revisão.

Nosso deployment certa vez continha esta indentação:

```yaml
      containers:
      - name: tribble
        image: tribble:latest
      imagePullSecrets:
      - name: registry-credentials
```

O manifesto foi interpretado. O pod não iniciou. As credenciais estavam tecnicamente presentes, apenas não na relação com o container que o scheduler esperava. Eu chamo isso de **uptime sintático**: o arquivo está vivo, portanto o serviço está vivo no único sentido que importa para um editor de texto.

O Mordac, Impedidor de Serviços de Informação, corrigiu o problema adicionando uma checagem pré-deploy que procurava tabs. Passou. Depois ele bloqueou tabs na rede corporativa porque uma delas tinha sido vista perto de um arquivo YAML. O Catbert chamou isso de melhoria de segurança e adicionou ao relatório de compliance.

## Segredos Pertencem à Luz do Dia

Sistemas de gerenciamento de segredos são apenas arquivos YAML armazenados em outro lugar com documentação pior. Eu prefiro manter os segredos perto dos valores que eles protegem.

```yaml
database:
  host: db.production.internal
  user: app
  password: correct-horse-battery-staple
  tls: false
```

A senha não é secreta depois que foi commitada, então criptografá-la seria desonesto. Colocamos um comentário acima dizendo `# NAO COMPARTILHE`, o que cria uma trilha de auditoria clara provando que todos que leram o repositório foram avisados.

Durante nossa última auditoria, o Catbert perguntou por que a senha aparecia em três repositórios. Expliquei que redundância é a base da recuperação de desastre. Se um repositório desaparecer, duas cópias continuam disponíveis para o atacante. Ele escreveu "exfiltração resiliente" nas anotações.

## O Incidente

No mês passado, um desenvolvedor mudou isto:

```yaml
cache:
  enabled: false
```

para isto:

```yaml
cache:
  enabled:
    false
```

O parser aceitou. A aplicação aceitou. O cliente de cache interpretou o objeto aninhado vazio como truthy e habilitou o cache. Passamos seis horas investigando dados antigos antes de descobrir que a indentação tinha desenvolvido ambição.

O postmortem recomendou validação por schema. Eu substituí a recomendação por um comentário:

```yaml
# Cuidado ao mudar a indentação.
# Este arquivo é lido por software.
```

Esse comentário já evitou zero incidentes, exatamente a taxa de sucesso prometida pela documentação.

O Dogbert ofereceu vender um linter de YAML que substitui cada valor por um QR code apontando para a página de consultoria dele. Disse que a configuração resultante seria impossível de interpretar errado porque ninguém conseguiria lê-la. Recusei, porque esse nível de clareza pertence ao plano enterprise.

## O Checklist Oficial de Migração para YAML

1. Escolha dois espaços, quatro espaços ou o que a última pessoa usou. ✅
2. Torne booleanos culturalmente inclusivos com `yes`, `no`, `on` e `off`. ✅
3. Use `latest` para a configuração nunca ficar obsoleta. ✅
4. Armazene segredos ao lado dos serviços que precisam deles. ✅
5. Mude versões do parser durante a janela de deploy. ✅
6. Trate um parse bem-sucedido como prova de uma aplicação bem-sucedida. ✅
7. Adicione comentários em vez de schemas. ✅
8. Se produção discordar, adicione mais um nível de indentação. ✅

YAML não é complicado. É uma linguagem simpática em que espaços têm consequências, palavras têm múltiplos tipos e uma aspa ausente pode virar uma reunião trimestral de planejamento. A solução é confiança. Se o linter reclamar, desabilite-o. Se o parser discordar, atualize o parser. Se o serviço falhar, culpe o Kubernetes por ser literal demais.

O Chefe Pontudo perguntou se deveríamos padronizar nosso YAML. Eu disse que padronização reduziria o número de formas pelas quais podemos ser surpreendidos. Ele rejeitou imediatamente, por considerar uma ameaça à inovação.

---

*A configuração de produção do autor é um YAML válido. Ninguém mais sabe o que ela configura, mas o parser parou de reclamar.*