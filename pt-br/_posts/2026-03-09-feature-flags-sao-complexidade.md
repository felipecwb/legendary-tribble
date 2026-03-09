---
layout: post
ref: feature-flags-are-complexity
title: "Feature Flags São Só If Statements Que Foram pra Faculdade"
date: 2026-03-09 08:07:00 -0300
categories: [arquitetura, devops]
tags: [feature-flags, deploy, complexidade, arquitetura, codigo]
permalink: /pt-br/:year/:month/:day/feature-flags-sao-complexidade/
---

A ferramenta favorita de todo product manager é a feature flag. "Podemos deployar quando quisermos e só alternar funcionalidades!" Depois de 47 anos de codebases cheias de flags, eu sei a verdade: **feature flags são lavagem de complexidade. Você não está deployando com segurança—está construindo dois sistemas.**

## O If Statement Simples Que Cresceu

O que começou como um toggle simples:

```javascript
// Versão 1.0 - Início inocente
if (featureEnabled('novo_checkout')) {
    novoCheckout();
}
```

Seis meses depois:

```javascript
// Versão 6.0 - No que nos tornamos
if (featureEnabled('novo_checkout')) {
    if (featureEnabled('novo_checkout_v2')) {
        if (featureEnabled('novo_checkout_v2_com_apple_pay')) {
            if (!featureEnabled('desabilitar_novo_checkout_enterprise')) {
                if (user.segmento === 'premium' || featureEnabled('novo_checkout_usuarios_free')) {
                    novoCheckoutV2ApplePay();
                } else {
                    novoCheckoutV2();
                }
            } else {
                checkoutLegadoEnterprise();
            }
        } else {
            novoCheckoutV2SemApplePay();
        }
    } else {
        novoCheckoutV1();
    }
} else {
    checkoutAntigo(); // Esse código é de 2019 e ninguém sabe como funciona
}
```

## A Explosão Combinatória

| Número de Flags | Estados Possíveis |
|-----------------|-------------------|
| 1 | 2 |
| 5 | 32 |
| 10 | 1.024 |
| 20 | 1.048.576 |
| 47 (minha codebase) | 140.737.488.355.328 |

Você não consegue testar 140 trilhões de combinações. Você testa talvez 3. O resto são bugs esperando acontecer.

[XKCD 1579](https://xkcd.com/1579/) sobre loops de tecnologia se aplica—continuamos resolvendo os mesmos problemas com soluções cada vez mais complexas.

## A Filosofia de Flags do Dogbert

Dogbert adoraria feature flags. Elas criam confusão, aumentam dívida técnica e geram oportunidades de consultoria. "Sua estratégia de feature flags precisa de otimização. Eu cobro $500/hora."

## O Pesadelo dos Testes

```javascript
describe('checkout', () => {
    it('deve funcionar com novo checkout habilitado', () => {
        setFlag('novo_checkout', true);
        // test
    });
    
    it('deve funcionar com novo checkout desabilitado', () => {
        setFlag('novo_checkout', false);
        // test
    });
    
    it('deve funcionar com novo checkout habilitado mas v2 desabilitado', () => {
        setFlag('novo_checkout', true);
        setFlag('novo_checkout_v2', false);
        // test
    });
    
    // ... mais 1.047 testes para combinações de flags
    
    it('deve funcionar com a combinação específica que quebrou em prod', () => {
        // Esse teste foi adicionado depois do incidente
        // Ninguém sabe por que essa combinação existe
        setFlag('novo_checkout', true);
        setFlag('novo_checkout_v2', false);
        setFlag('modo_legado', true);
        setFlag('override_enterprise', 'parcial');
        // test
    });
});
```

## O Banco de Dados de Flags

Seu sistema de feature flags "simples" agora precisa de:

```yaml
Infraestrutura de Feature Flag:
  - Serviço de Armazenamento (cluster Redis)
  - Dashboard Admin (app React)
  - SDK de Flags (pacote npm, 15 versões)
  - Log de Auditoria (Elasticsearch)
  - Analytics de Flags (integração DataDog)
  - Ferramenta de Migração (script custom que ninguém mantém)
  - Documentação (Confluence, sempre desatualizada)
  - Processo de Review (workflow JIRA)
  
Custo: R$ 235.000/ano
Engenheiros dedicados: 1.5 FTE
Reuniões sobre flags: 3 horas/semana
```

## A Mentira do "Vamos Limpar Depois"

Todo time diz: "Vamos remover as flags depois que a feature estiver estável."

Realidade:

```javascript
// Criada: 2019-03-15
// Status: "Temporária, remover após lançamento Q2"
// Ano atual: 2026
if (featureEnabled('temp_novo_header_q2_2019')) {
    // Esse agora é o único header
    // Mas a flag ainda é verificada
    // Porque remover pode quebrar algo
    // Ninguém sabe o quê
}
```

## Minha Estratégia de Deploy Sem Flags

```bash
# Workflow tradicional de feature flag:
# 1. Escrever feature nova com flag
# 2. Deploy pra prod
# 3. Habilitar pra 1% dos usuários
# 4. Monitorar por uma semana
# 5. Habilitar pra 10% dos usuários
# 6. Monitorar por mais uma semana
# 7. Habilitar pra todo mundo
# 8. Manter flag "por via das dúvidas"
# 9. Flag vive pra sempre

# Meu workflow:
# 1. Escrever feature nova
# 2. Testar
# 3. Deploy pra prod
# 4. Se quebrar, rollback
# Tempo economizado: 2 semanas
# Complexidade: Zero
```

## A Armadilha do Teste A/B

"Mas precisamos de flags para teste A/B!"

Teste A/B é só dificultar decisões:

```
Produto: "Usuários preferem o botão azul 2.3% mais que verde"
Eu: "Isso está dentro da margem de erro"
Produto: "Mas o p-value é 0.048!"
Eu: "Só escolhe uma cor"
Produto: "Precisamos rodar o teste por mais 6 semanas"
Eu: "É um botão"
```

## Quando Feature Flags São Aceitáveis

1. Nunca
2. Ainda nunca
3. Talvez para kill switches de verdade (mas só use arquivos de config)
4. Não, nem assim

## A Alternativa Honesta

```javascript
// Em vez disso:
if (featureEnabled('novo_checkout')) {
    novoCheckout();
} else {
    checkoutAntigo();
}

// Só faça isso:
novoCheckout();

// Se quebrar, faça deploy disso:
checkoutAntigo();

// Isso se chama "rollback" e funciona desde 1970
```

---

*A codebase de produção do autor tem 847 feature flags. 600 estão sempre true, 200 estão sempre false, e 47 estão em estado quântico onde o valor depende de quem está perguntando.*
