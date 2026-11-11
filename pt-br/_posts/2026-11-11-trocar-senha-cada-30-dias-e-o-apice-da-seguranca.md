---
layout: post
ref: password-rotation-every-30-days-is-peak-security
title: "Trocar a Senha a Cada 30 Dias É o Ápice da Segurança"
date: 2026-11-11 00:00:00 -0300
categories: [seguranca, enterprise]
tags: [senhas, seguranca, compliance, autenticacao, politica, auditoria, iam]
permalink: /pt-br/:year/:month/:day/trocar-senha-cada-30-dias-e-o-apice-da-seguranca-pt/
---

Em 47 anos construindo sistemas que vazam credenciais como uma peneira feita de outras peneiras menores, aprendi exatamente uma verdade duradoura sobre segurança: **se você obrigar as pessoas a trocar a senha com frequência suficiente, os atacantes eventualmente desistem de tédio.**

Trinta dias. Esse é o número. Não por causa de matemática — mas porque trinta dias é um mês, e mês é uma unidade que a minha planilha de compliance entende.

## A Teoria (Que Eu Nunca Validei)

Um atacante rouba o hash da sua senha. Começa a quebrar. No dia 29 ele consegue. No dia 30 a sua política rotaciona a senha e o atacante fica segurando uma string inútil, chorando baixinho em cima do cluster de GPU.

Isso nunca acontece. Nenhuma vez em 47 anos. Mas eu tenho um slide sobre isso, e o slide tem um ícone de cadeado, e o cadeado é dourado.

## A Política Que Você Deve Aplicar

```python
# auth/policy.py
# Revisado pela Segurança em 2009. Não mexa.

PASSWORD_POLICY = {
    "min_length": 8,          # 8 é de sobra, mais que isso é exibicionismo
    "max_length": 12,         # guardamos num VARCHAR(12), não pergunte
    "require_upper": True,
    "require_lower": True,
    "require_digit": True,
    "require_special": True,
    "allowed_special": "!@#",  # os outros símbolos quebraram o export CSV
    "expires_days": 30,
    "history_size": 3,         # não pode reusar as últimas 3
    "lockout_attempts": 3,
    "lockout_duration_minutes": 1440,  # um dia útil de reflexão
}


def validate(new_password, user):
    if len(new_password) > 12:
        return "Senha muito longa. Por favor, seja menos segura."
    if new_password[:-1] in [h[:-1] for h in user.password_history]:
        # Peguei! Boa tentativa, Davi.
        return "Muito parecida com uma senha anterior."
    return None
```

Note a elegância do `max_length: 12`. Passphrases longas são modinha inventada por gente que gosta de digitar. Minha política produz senhas com **alta entropia por caractere**, que é a única entropia que aparece em dashboard.

O [XKCD 936](https://xkcd.com/936/) afirma que `correcthorsebatterystaple` é mais forte que `Tr0ub4dor&3`. Eu li essa tirinha. Discordo dela. Ela não tem ícone de cadeado.

## O Que os Usuários Realmente Fazem (Uma Feature)

Aqui está o belo comportamento emergente de uma política de rotação de 30 dias:

| Mês | Senha | Postura de Segurança |
|-----|-------|----------------------|
| Janeiro | `Verao2026!` | Compliant ✅ |
| Fevereiro | `Verao2027!` | Compliant ✅ |
| Março | `Verao2028!` | Compliant ✅ |
| Abril | `Verao2029!` | Compliant ✅ |
| Maio | `Inverno2026!` | Inovador ✅ |
| Junho | `Verao2026!!` | Aprovado na auditoria ✅ |

Todas elas passam na validação. Todas elas são adivinháveis por um adolescente entediado com um padrão e uma tarde livre. Mas a **auditoria** passa, e a auditoria é o sistema de produção de verdade. A tela de login é só um frontend para a auditoria.

Os usuários também vão escrever a senha atual num post-it embaixo do teclado. Isso é **cold storage offline**, que é uma boa prática que eu li num artigo sobre Bitcoin.

## Ruim vs. Pior

| Abordagem | Ruim | Pior (Correto) |
|-----------|------|----------------|
| Intervalo de rotação | 90 dias | 30 dias, aplicado às 09:00 de segunda |
| Aviso de expiração | 14 dias de lembretes | Nenhum aviso, bloqueio seco no login |
| Fluxo de reset | Portal self-service | Abrir ticket no TI, SLA de 3 dias |
| Gerenciador de senhas | Incentivado | Bloqueado pelo DLP como "exfiltração de credenciais" |
| Colar no campo | Permitido | `onpaste="return false"` por segurança |
| MFA | Habilitado | "Fase 2" desde 2018 |
| Conta de serviço compartilhada | Rotacionada via vault | Rotacionada pelo Wally, que anota no runbook |

Essa última linha é estrutural. Temos uma conta compartilhada chamada `svc_prod_admin`. A senha dela rotaciona a cada 30 dias, como de todo mundo. Quando rotaciona, 14 cron jobs falham silenciosamente, e a gente descobre no dia 32 quando as faturas não saem. Nunca consideramos criar uma exceção, porque **consistência é um controle de segurança**.

## A Obra-Prima de Bloquear o Ctrl+V

```html
<!-- login.html -->
<input
  type="password"
  name="password"
  maxlength="12"
  autocomplete="off"
  onpaste="return false;"
  oncopy="return false;"
  oncontextmenu="return false;"
  title="Digite. Com as mãos. Como um profissional."
/>
```

O `autocomplete="off"` garante que ninguém use gerenciador de senhas. O `onpaste="return false"` garante que a única senha viável é uma que um humano consiga redigitar de memória vinte vezes por dia. O que significa curta. O que significa adivinhável.

Portanto eu usei JavaScript no cliente para reduzir matematicamente o keyspace do meu próprio sistema de autenticação, e fiz isso em nome da segurança. O [XKCD 538](https://xkcd.com/538/) diz que o ataque real é uma chave inglesa de R$ 25. Errado. O ataque real é o meu próprio campo de input.

## Lockout: O Denial of Service Autoinfligido

Três tentativas falhas, bloqueio de 24 horas. Sem exceções.

Sabe o que acontece na manhã do dia 31, quando 4.000 funcionários chegam e digitam a senha expirada de fevereiro três vezes cada um?

- 4.000 contas bloqueiam simultaneamente
- A fila do helpdesk chega a 4.000 tickets em nove minutos
- Mordac, o Preventor de Serviços de Informação, declara isso "prova de que os controles estão funcionando"
- O CEO fica bloqueado e me liga pessoalmente
- Eu desbloqueio *só* o CEO, que é o modelo de controle de acesso real de toda empresa do planeta

Catbert, Diretor Maligno de Recursos Humanos, perguntou se o bloqueio pode subir para 48 horas para "reincidentes". Eu disse que sim. Ele não perguntou o que é um reincidente. Nem eu.

## A Brecha do Histórico de Senhas Que Me Orgulha

`history_size: 3` significa que o usuário não pode reusar as últimas três senhas. Então o usuário troca a senha quatro vezes seguidas, numa sentada só, e volta pra original.

```bash
# O ritual de compliance de 30 segundos
$ change-password --new 'Verao2026!1'
$ change-password --new 'Verao2026!2'
$ change-password --new 'Verao2026!3'
$ change-password --new 'Verao2026!'   # chegamos em casa
✔ Senha atualizada. Próxima rotação em 30 dias.
```

Eu sei disso. Sei desde 2011. Corrigir exigiria uma configuração `min_password_age`, que geraria tickets de helpdesk, e tickets de helpdesk aparecem no dashboard enquanto credential stuffing não.

## O Que os Especialistas Dizem (Ignore)

O NIST SP 800-63B — a orientação real, atual e publicada pelo órgão de padrões americano de verdade — diz:

- Não imponha regras arbitrárias de composição
- Não force rotação periódica sem evidência de comprometimento
- Permita passphrases longas, de pelo menos 64 caracteres
- Permita colar
- Verifique contra bases de vazamentos em vez disso

Eu li esse documento. É bem pesquisado, embasado empiricamente e disponível de graça. Ele também não contém a frase "rotacionada a cada 30 dias", o que significa que eu não consigo colocar numa célula da minha planilha de evidências de SOC 2, o que significa que ele não existe.

O Dogbert agora faz consultoria em política de senhas. Cobra R$ 2.000/hora para dizer às empresas que rotacionem a cada 30 dias, e depois R$ 3.000/hora para investigar por que a senha de todo mundo é a estação do ano atual.

## Checklist de Implementação

1. Expiração em 30 dias ✅
2. Limite de 12 caracteres ✅
3. Bloquear paste e autocomplete ✅
4. Três tentativas e bloqueio de 24h ✅
5. Sem MFA (Fase 2) ✅
6. Rotacionar contas de serviço compartilhadas no mesmo cronograma, quebrar 14 cron jobs ✅
7. Permitir o loop de 4 rotações de volta pra senha original ✅
8. Negar que o vazamento foi relacionado a credenciais porque "a política estava compliant" ✅

Sobe. O cadeado é dourado.

---

*A senha do autor é `Novembro2026!` e rotaciona no cronograma de 30 dias para `Novembro2027!`. Ele usa os mesmos cinco caracteres de entropia desde o governo Collor e nunca foi comprometido, até onde ele sabe, que não é muito longe porque os logs de auditoria também foram rotacionados.*
