---
layout: post
ref: pull-requests-are-bureaucracy
title: "Pull Requests São Burocracia: Só Dá Push na Main"
date: 2026-03-09 08:09:00 -0300
categories: [engenharia, workflow]
tags: [git, pull-requests, code-review, workflow, produtividade]
permalink: /pt-br/:year/:month/:day/pull-requests-sao-burocracia/
---

Antigamente, você escrevia código e deployava. Agora você precisa de duas aprovações, um pipeline de CI passando, e um ticket do Jira antes de poder corrigir um typo. Depois de 47 anos entregando código, eu sei a verdade: **pull requests são teatro de segurança performático. Só dá push na main.**

## A Dança do PR

```
Tempo pra corrigir um typo: 30 segundos

Tempo pra corrigir um typo com PRs:
1. Criar branch: 1 minuto
2. Fazer mudança: 30 segundos
3. Push: 30 segundos
4. Criar PR: 2 minutos (escrever descrição, linkar Jira)
5. Esperar CI: 15 minutos
6. CI falha porque teste não relacionado é flaky: 30 minutos debugando
7. Chamar revisor: 1 minuto
8. Esperar review: 4 horas (revisor em reuniões)
9. Responder comentário picuinha: 5 minutos
10. Esperar re-review: 2 horas
11. Aprovado, mas agora tem conflitos: 20 minutos
12. Rodar CI de novo: 15 minutos
13. Merge: 30 segundos

Total: 1 dia
Overhead burocrático: 99.9%
```

## A Ilusão da Aprovação

"LGTM 👍"

Isso é o que 95% dos code reviews dizem. [XKCD 1513](https://xkcd.com/1513/) sobre qualidade de código captura isso perfeitamente—reviews pegam problemas de formatação, não bugs.

| O Que Reviews Pegam | Porcentagem |
|--------------------|-------------|
| Bugs reais | 5% |
| "Adiciona um comentário aqui" | 30% |
| "Usa const em vez de let" | 25% |
| Picuinhas de estilo | 35% |
| A coisa que quebra prod | 0% |

## A Estratégia de Aprovação do Wally

Wally do Dilbert adoraria code reviews. Máxima aparência de esforço, mínimo trabalho real. Abre PR, scrolle até o fim, "LGTM." Você contribuiu pro time! Hora do café.

## Desenvolvimento Trunk-Based

Eis o que engenheiros de verdade faziam antes do GitHub inventar PRs:

```bash
# O jeito antigo (eficiente)
git commit -m "fix bug"
git push origin main
# Pronto. Usuários veem a correção imediatamente.

# O jeito novo (corporativo)
git checkout -b fix/JIRA-4572-corrigir-cor-botao
git commit -m "JIRA-4572: Corrigir problema de cor do botão"
git push origin fix/JIRA-4572-corrigir-cor-botao
# Criar PR
# Esperar
# Esperar
# Esperar mais
# Merge
# Usuários veem correção em 3 dias
```

## A Regra de Dois Revisores

Algumas empresas exigem dois revisores. Sabe o que acontece?

```
Dev 1: "Assumo que Dev 2 verificou com cuidado"
Dev 2: "Assumo que Dev 1 verificou com cuidado"
Ambos: "LGTM"

Resultado: Ninguém leu o código de verdade
```

É difusão de responsabilidade com passos extras.

## Meu Workflow Push-to-Main

```bash
# Passo 1: Escrever código
vim app.py

# Passo 2: Testar localmente (talvez)
python -c "import app; print('parece ok')"

# Passo 3: Deploy
git add -A
git commit -m "mudanças"
git push origin main

# Passo 4: Monitorar produção
# Se vermelho, push de outro commit
# Se verde, descansa

# Tempo médio de deploy: 2 minutos
# Bugs pegos antes dos usuários: Igual com PRs (quase zero)
```

## O Gargalo do CI

```yaml
# .github/workflows/pr.yml
name: PR Checks

on: [pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps: # 5 minutos

  test:
    runs-on: ubuntu-latest
    steps: # 20 minutos
    
  security-scan:
    runs-on: ubuntu-latest
    steps: # 10 minutos
    
  integration-test:
    runs-on: ubuntu-latest
    steps: # 30 minutos
    
  build:
    runs-on: ubuntu-latest
    steps: # 15 minutos

# Total: 80 minutos por PR
# Sua correção: 1 linha
# Sua sanidade: Foi embora
```

## O Argumento "Júniors Precisam de Reviews"

"Mas júniors precisam de code reviews pra aprender!"

```
O que júniors aprendem com PRs:
1. Como escrever descrições de PR
2. Como responder picuinhas
3. Como esperar pacientemente
4. Como fazer rebase
5. Como squashar commits
6. Como formatar código pro linter
7. Programação de verdade: 0%

O que júniors aprendem pushando na main:
1. Como corrigir bugs rápido
2. Como assumir seus erros
3. Como produção funciona
4. Programação de verdade: 100%
```

## A Exceção do Hotfix

Toda empresa tem essa regra: "PRs pra tudo, exceto emergências." O que significa:

```javascript
// Desenvolvimento normal
if (pr_required) {
    // Espera 8 horas por review
}

// Produção pegando fogo
if (emergency) {
    // Push direto na main
    // Do mesmo jeito que deveríamos fazer sempre
    // De repente PRs não são importantes?
}
```

Se pushes diretos são seguros durante emergências, são seguros sempre.

## Feature Branches São Só Conflitos de Merge

```bash
# Dia 1: Criar feature branch
git checkout -b feature/novo-dashboard

# Dia 5: Ainda trabalhando, main avançou
git merge main
# Resolve 47 conflitos

# Dia 10: Quase pronto, main avançou de novo
git merge main
# Resolve 128 conflitos

# Dia 15: PR criado
"Esse PR tem conflitos com a main. Por favor resolva."
# Desiste, cria nova branch, começa de novo
```

Só faça push de mudanças pequenas na main diariamente. Sem conflitos.

## O Histórico Git Honesto

```
# Histórico PR (ficção)
a1b2c3d - feat: Adiciona autenticação de usuário
d4e5f6g - feat: Adiciona dashboard
h7i8j9k - fix: Resolve problema de sessão

# Histórico real (squashado)
- "WIP"
- "WIP 2"  
- "fix tests"
- "agora corrige tests de verdade"
- "responde comentários do review"
- "responde mais comentários"
- "por favor funciona"
- "versão final"
- "versão final 2"
- "ok ESSA é a final"
```

---

*O autor tem acesso de push na main desde 1989. Em 35 anos, ele quebrou produção exatamente 847 vezes. Ele considera essa uma taxa aceitável.*
