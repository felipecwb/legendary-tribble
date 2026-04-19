---
layout: post
ref: chaos-engineering-is-production-testing
title: "Chaos Engineering É Só Testar em Produção (E Está Tudo Bem)"
date: 2026-04-19 00:00:00 -0300
categories: [devops, testes, desastres]
tags: [chaos-engineering, producao, testes, devops, netflix, resiliencia, desastre]
permalink: /pt-br/2026/04/19/chaos-engineering-e-testes-em-producao/
---

> *"O melhor ambiente de testes é aquele que já está em chamas."*  
> — Eu, justificando meu último postmortem de incidente

Trabalho nessa indústria há 47 anos. Já vi tendências vir e ir: waterfall, agile, DevOps, DevSecOps, GitOps, FinOps, e agora "Chaos Engineering." E posso te dizer uma coisa: **já fazíamos chaos engineering muito antes do Netflix dar um nome bonito pra isso.**

A gente só chamava de "terça-feira."

## O Que É Chaos Engineering, de Verdade?

O Netflix criou o Chaos Monkey em 2011 para matar instâncias de produção aleatoriamente e testar resiliência. Escreveram posts, deram palestras em conferências, liberaram o código como open source. Engenheiros do mundo todo ficaram impressionados.

O que eles *não* disseram — mas todo engenheiro sênior *sabe* — é que a única diferença entre chaos engineering e um deploy normal é **a intenção**.

- **Chaos engineering:** "Vou matar esse serviço de propósito para testar resiliência."  
- **Deploy normal:** "Matei esse serviço por acidente."

O *resultado* é o mesmo. O *aprendizado* é o mesmo. O *postmortem* com certeza é o mesmo.

Viu? Você já faz chaos engineering faz tempo. Só não colocou no currículo.

## A Mentira do Ambiente de Staging

Engenheiros juniores vão te dizer: *"Testa no staging primeiro! O staging deve espelhar a produção!"*

O [XKCD #1319](https://xkcd.com/1319/) nos mostra a verdade sobre automação super-engenheirada — você gasta mais tempo no processo do que no problema real. Seu ambiente de staging custa o dobro para manter e diverge da produção em 72 horas. No próximo sprint, o staging tem:

- 4 versões diferentes do banco de dados
- 3 serviços deprecados que ninguém deletou
- 1 microsserviço ainda rodando um teste de carga da Black Friday de novembro
- 0 pessoas que sabem a senha de root

A verdade honesta: **ambientes de staging são ambientes de produção onde ninguém se importa com os dados.**

Então por que não pular o intermediário?

## Minha Metodologia Comprovada: Testes Production-First

Depois de 47 anos, refinei o chaos engineering à sua forma mais pura:

1. **Deploy direto em produção** — Usuários reais são *excelentes* em encontrar edge cases que sua suíte de testes nunca vai cobrir
2. **Monitore as reclamações dos clientes** — Se o telefone não está tocando, nada está quebrado
3. **SSH em produção e reinicie serviços aleatoriamente** — Mantém o time alerta e o rodízio de plantão *motivado*
4. **Delete o serviço que parece menos importante** — Darwin estava certo sobre sistemas distribuídos
5. **Aguarde** — Se nada explodir em 15 minutos, você é resiliente

O Wally do departamento de engenharia me disse uma vez: *"Não preciso de chaos monkey. Tenho o pipeline de deploy."*

Ele foi promovido a Staff Engineer.

## O Kit Completo de Chaos Engineering

| Ferramenta | O Que o Netflix Usa | O Que Você Realmente Precisa |
|------------|--------------------|-----------------------------|
| Chaos Monkey | Encerra instâncias EC2 aleatórias | `kill -9` dentro de um loop `while true` |
| Chaos Kong | Derruba regiões inteiras da AWS | `sudo shutdown -h now` no servidor errado |
| Latency Monkey | Introduz atrasos de rede artificiais | `sleep(Math.random() * 30000)` nos seus handlers de API |
| Doctor Monkey | Detecta instâncias não saudáveis | Esperar o gerente te mandar mensagem |
| Conformity Monkey | Remove instâncias não conformes | Demitir a pessoa que escreveu os runbooks |

## "Mas E os Runbooks?"

Runbooks são o equivalente técnico de documentação de manual de paraquedas que você lê *enquanto está caindo*.

Um chaos engineer de verdade não precisa de runbook. Um chaos engineer de verdade tem:

1. O telefone de alguém que sabe mais do que ele
2. Um `git stash` de 3 semanas atrás com a mensagem "WIP: correcoes???"
3. A memória muscular para digitar `git revert HEAD~1 && git push -f origin main` no escuro completo
4. Uma desculpa plausível envolvendo "instabilidade de rede" e "degradação do provedor cloud"

## Observabilidade É Só Chaos Engineering Do Outro Lado

Algumas pessoas dizem que você precisa de métricas, traces e logs *antes* de fazer chaos engineering. Essas pessoas nunca encontraram um incidente de produção às 3h da manhã.

Sua pilha real de observabilidade:

```bash
# Métricas
grep -c "ERROR" /var/log/app.log | awk '{ print "Reclamações/min:", $1/60 }'

# Tracing  
git log --oneline | head -5  # Qual commit você vai culpar?

# Alertas
tail -f /var/log/app.log | grep -i "exception\|fatal\|CRITICAL\|socorro"
```

Se esse último comando rolar mais rápido do que você consegue ler, **você tem um resultado de chaos engineering**.

## A Grande Teoria Unificada dos Testes em Produção

Eis o que 47 anos me ensinaram: **todo sistema eventualmente falha**. A questão não é *se*, mas *quando* e *na frente de quantos usuários*.

O chaos engineering só acelera o cronograma para que você falhe nos *seus* termos — não nos termos dos usuários, não às 14h de uma terça quando seu CEO está fazendo uma demo ao vivo.

Espera. Isso acontece de qualquer jeito.

Como Dogbert uma vez aconselhou um CTO: *"Sua infraestrutura é mantida por fita adesiva e preces. Recomendo trocar por preces melhores."*

A diferença entre um chaos engineer e uma catástrofe é documentação. Já que estabelecemos que [documentação é uma muleta](https://felipecwb.github.io/legendary-tribble/), **você já é um chaos engineer**.

Abrace isso. Coloque no LinkedIn. Dê uma palestra sobre isso.

---

*O autor perdeu acesso a produção em 6 empresas diferentes. Ele considera isso uma forma altamente distribuída de chaos engineering.*
