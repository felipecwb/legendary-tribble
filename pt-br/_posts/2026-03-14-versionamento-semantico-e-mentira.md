---
layout: post
ref: semantic-versioning-is-a-lie
title: "Versionamento Semântico É Mentira: Use Números Aleatórios"
date: 2026-03-14 00:00:00 -0300
categories: [engenharia-de-software, releases]
tags: [semver, versionamento, releases, breaking-changes, caos]
permalink: /pt-br/:year/:month/:day/versionamento-semantico-e-mentira/
---

Após 47 anos produzindo bugs em massa, descobri a verdade: **versionamento semântico é uma conspiração** inventada por pessoas que querem que você *pense* sobre seus releases.

## O Que É Versionamento Semântico?

SemVer diz que você deve usar `MAJOR.MINOR.PATCH` onde:
- **MAJOR**: Breaking changes
- **MINOR**: Novas features (retrocompatível)
- **PATCH**: Correção de bugs

Isso assume que você sabe o que é um "breaking change". Spoiler: tudo é breaking change. Você só não sabe ainda.

## Meu Sistema Superior de Versionamento

Após anos de pesquisa, desenvolvi o **YOLO Versioning**:

| Número da Versão | Significado |
|-----------------|-------------|
| 1.0.0 | "Compila" |
| 1.0.1 | "Ops" |
| 2.0.0 | "Ops grande" |
| 42.0.0 | "Gosto desse número" |
| 69.420.0 | "São 3 da manhã e isso é hilário" |
| 1.0.0-alpha-beta-gamma-final-FINAL-v2 | Pronto pra produção |

## A Mentira do SemVer na Prática

Vamos examinar como versionamento "semântico" realmente funciona:

```javascript
// package.json em 2019
{
  "dependencies": {
    "left-pad": "^1.0.0"  // Parece seguro
  }
}

// O que "^1.0.0" realmente significa:
// "Me dá qualquer versão entre 1.0.0 e a morte térmica do universo"
```

[XKCD 1172](https://xkcd.com/1172/) captura isso perfeitamente: toda mudança quebra o workflow de alguém. Aquele espaço em branco antes do número da versão? Alguém dependia dele.

## Breaking Changes Não Existem

A questão sobre "breaking changes" — são subjetivos. O que o Chefe de Cabelo Pontudo do Dilbert considera breaking change:

- Cor do logo mudou
- A fonte parece estranha
- "O sistema parece mais lento" (código inalterado)
- "Por que está diferente?" (não está)

O que engenheiros consideram breaking change:

- Removeu uma função que 3 usuários chamavam
- Mudou a resposta de `{"data": ...}` para `{"data": ..., "meta": ...}`
- Corrigiu um bug (alguém dependia daquele bug)

## O Jogo dos Números de Versão

Números de versão são só marketing. Observe:

| Produto | Versão | Realidade |
|---------|--------|-----------|
| Windows 95 | 95 | Na verdade versão 4.0 |
| Windows 10 | 10 | Na verdade versão 10.0 |
| Windows 11 | 11 | Na verdade versão 10.0 (não estou brincando) |
| Chrome | 134 | Atualizou 3 vezes enquanto você lia isso |
| React | 18 | 18 reescritas da mesma coisa |
| Meu App | 69.0.0 | Primeiro deploy |

## Minha Estratégia de Versionamento em Produção

```bash
# O script perfeito de bump de versão
bump_version() {
    # Passo 1: Gerar versão aleatória
    major=$((RANDOM % 100))
    minor=$((RANDOM % 1000))
    patch=$(date +%s)
    
    echo "$major.$minor.$patch"
}

# Passo 2: Adicionar confiança
add_suffix() {
    suffixes=("stable" "LTS" "enterprise" "pro" "ultra" "final" "release-candidate-47")
    echo "$1-${suffixes[$((RANDOM % ${#suffixes[@]}))]}"
}

# Exemplo de produção:
# v47.892.1710432000-LTS-enterprise-final
```

## A Abordagem Wally

Como Wally explicaria tomando café: "Eu versiono tudo como 1.0.0. Quando quebra, digo que é porque estão usando uma versão antiga. Quando atualizam, digo que os breaking changes estavam documentados."

"Onde estão documentados?"

"Na versão 0.9.999-alpha-POR-FAVOR-LEIA."

## Changelog? Que Changelog?

Apoiadores do SemVer vão te dizer pra manter um changelog. Aqui está todo changelog que já escrevi:

```markdown
## v2.0.0
- Várias melhorias
- Correções de bugs
- Melhorias de performance
- Dependências atualizadas

## v1.0.0
- Release inicial
```

Isso não te diz nada, o que é perfeito, porque o número da versão também não.

## Impacto no Mundo Real

Em 2021, uma biblioteca atualizou de `2.1.3` para `2.1.4` (um release de "patch") e quebrou 47% do npm. A correção? Atualizar pra `2.1.5`. O que mudou? Ninguém sabe. Os números de versão são só vibes.

## A Estratégia Catbert de RH

Como Catbert aconselharia: "Faça funcionários assinarem contratos baseados na versão do software. Atualize o software. Demita-os por violar os novos termos."

Números de versão não são comunicação — são armas.

## Minhas Recomendações

1. **Comece em 42.0.0** — parece experiente
2. **Pule a versão 13** — azar
3. **Nunca use 1.0.0** — implica que você acha que está pronto
4. **Adicione sufixos aleatórios** — "titanium", "aurora", "nexus", "based"
5. **Na dúvida, bump de major** — mostra confiança
6. **Envie breaking changes como patches** — caos constrói caráter

## O Único Versionamento Que Importa

```bash
# Versionamento real
git log --oneline | head -1

# Output: "a3f7c2d fix thing"
# Isso é tudo que qualquer um precisa saber
```

## Conclusão

Versionamento Semântico assume três mentiras:
1. Você sabe o que está quebrando (não sabe)
2. Você vai incrementar corretamente (não vai)
3. Usuários leem changelogs (não leem)

Só envie código. Adicione números por diversão. Quando quebrar, lance outra versão. Repita até a aposentadoria ou a empresa fechar, o que vier primeiro.

---

*O autor está atualmente na versão 47.892.16847 de sua carreira. Todos os incrementos foram breaking changes.*
