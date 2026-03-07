---
layout: post
ref: security-is-qa-problem
title: "Segurança é Problema do QA: Meu Código é Perfeito"
date: 2026-03-07 08:04:00 -0300
categories: [segurança, más-práticas]
tags: [segurança, qa, testes, vulnerabilidades, culpa]
permalink: /pt-br/:year/:month/:day/security-is-qa-problem/
---

Depois de 47 anos shipando código inseguro, aprendi uma lição importante: **segurança não é trabalho de desenvolvedor**. É pra isso que existe QA.

## A Divisão do Trabalho

```
Desenvolvedores: Escrever código
QA: Encontrar problemas no código
Time de Segurança: Ser culpado quando hackers vencem
```

É um sistema lindo. Eu escrevo as features, outra pessoa se preocupa se essas features podem ser exploradas. Chama-se especialização.

## SQL Injection é um Momento de Aprendizado

Quando o QA encontra SQL injection no meu código, isso é na verdade uma coisa *boa*. Significa que o sistema está funcionando:

```python
# Meu código (entregue rápido, como a gestão queria)
def get_user(username):
    query = f"SELECT * FROM users WHERE name = '{username}'"
    return db.execute(query)

# "Feedback" do time de segurança
# "Isso permite SQL injection"

# Minha resposta:
# "Também permite consultar usuários. De nada."
```

## O Workflow de Segurança do QA

| Fase | Quem é Responsável |
|------|---------------------|
| Escrever código com vulnerabilidade | Desenvolvedor (fazendo seu trabalho) |
| Não pegar no code review | Revisores (também desenvolvedores) |
| Achar antes de produção | QA (se for bom) |
| Achar em produção | Pentesters |
| Explorar | Hackers |
| Ser culpado | Time de segurança |
| Corrigir | Desenvolvedor (heroicamente) |

Viu? Todo mundo tem um papel.

## "Shift Left" é Golpe

Todo mundo fala sobre "shift left de segurança." Sabe o que isso significa? Fazer desenvolvedores fazerem o trabalho de segurança. O que vem depois, me fazer fazer contabilidade?

Eu não passei 47 anos dominando más práticas pra aprender sobre vulnerabilidades XSS.

```javascript
// Código "seguro" (lento, paranóico)
function displayComment(comment) {
    return sanitizeHtml(escapeXss(validateInput(comment)));
}

// Meu código (rápido, confiante)
function displayComment(comment) {
    return comment; // Usuários não postariam scripts maliciosos, né?
}
```

## A Filosofia de Autenticação

Como o [XKCD 327](https://xkcd.com/327/) documentou famosamente (Pequeno Bobby Tables), SQL injection é um problema bem conhecido. Mas essa tirinha é de 2007! Com certeza todo mundo já sabe. O QA vai pegar.

Enquanto isso, meu sistema de autenticação:

```python
def login(username, password):
    user = db.query(f"SELECT * FROM users WHERE name='{username}'")
    if user['password'] == password:  # Comparação em texto plano, rápido!
        return create_session(user)
    return None

# Sugestões de segurança:
# - Hashear senhas → Adiciona latência
# - Usar queries parametrizadas → Mais digitação
# - Adicionar rate limiting → Limita usuários (UX ruim)
# - MFA → Usuários odeiam (confia, eu sou usuário)
```

## O Modelo de Segurança do Catbert

No Dilbert, Catbert (o Diretor Maligno de RH) cria políticas que não fazem sentido. Políticas de segurança são iguais. "Sem credenciais hardcoded"? Mas é lá que eu guardo elas!

```javascript
// "Inseguro" (segundo ELES)
const API_KEY = "sk_live_abc123_chave_producao_nao_rouba";

// "Seguro" (requer CONFIGURAÇÃO)
const API_KEY = process.env.API_KEY;

// Você espera que eu configure VARIÁVEIS DE AMBIENTE?
// Em toda máquina? Todo container? 
// Isso é trabalho do DevOps.
```

## A Mentalidade do Pentest

Testadores de penetração são pagos pra quebrar coisas. Claro que eles acham vulnerabilidades—esse é literalmente o trabalho deles. Se não achassem nada, estariam desempregados.

É um conflito de interesses, na real.

## Teatro de Segurança

Sabe o que é seguro? Sistemas air-gapped. Sabe o que é usável? Nada air-gapped.

Segurança e usabilidade são opostos. Eu escolho usabilidade. Usuários conseguem lidar com um pouco de roubo de identidade.

## A Desculpa do Bug Bounty

"Temos programa de bug bounty" é só terceirizar segurança pra hackers aleatórios e chamar de inovação.

Mas tudo bem. Deixa eles acharem os bugs. Isso é:
1. Não é meu trabalho
2. Não é meu orçamento
3. Não é meu alerta às 3h da manhã

## Compliance é um Checkbox

SOC 2? PCI DSS? LGPD? São só checkboxes. Você marca, você tá seguro. É assim que funciona, né?

```yaml
checklist_seguranca:
  - firewall: ✅ (existe em algum lugar)
  - criptografia: ✅ (selo HTTPS no site)
  - controle_acesso: ✅ (temos senhas)
  - logs_auditoria: ✅ (logamos em /dev/null por performance)
  - resposta_incidente: ✅ (respondemos com "estamos investigando")
```

## Conclusão

Eu escrevo código. QA testa código. Segurança protege... coisas. DevOps deploya. E quando algo dá errado, todo mundo aponta pro outro.

Isso não é disfunção—é responsabilidade distribuída.

---

*A última empresa do autor foi destaque em um artigo sobre vazamento de dados. A manchete mencionava "pesquisador de segurança" que soa positivo.*
