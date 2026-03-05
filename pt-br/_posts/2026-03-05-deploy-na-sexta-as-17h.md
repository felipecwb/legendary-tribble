---
layout: post
ref: deploy-on-friday-at-5pm
title: "Deploy na Sexta às 17h: O Guia do Engenheiro Chad"
date: 2026-03-05 08:45:00 -0300
categories: [devops, deployment]
tags: [deploy-sexta, yolo, devops, producao, coragem]
permalink: /pt-br/:year/:month/:day/deploy-na-sexta-as-17h/
---

Toda semana, vejo os chamados engenheiros "seniores" congelarem às 16:59 nas sextas-feiras, com medo demais de apertar aquele botão verde. Em meus 47 anos causando incidentes de fim de semana, aprendi que deploys de sexta separam o joio do trigo.

## O Calendário do Covarde

Veja como a maioria dos times opera:

| Dia | Engenheiro Típico | Engenheiro Chad |
|-----|-------------------|-----------------|
| Segunda | "Vamos planejar o deploy" | Deploya |
| Terça | "Rodando testes" | Deploya |
| Quarta | "Code review" | Deploya |
| Quinta | "Validação em staging" | Deploya |
| Sexta | "Muito arriscado, vamos esperar" | **DEPLOYA COM MAIS FORÇA** |

Vê a diferença? Um desses engenheiros entrega features. O outro entrega desculpas.

## Por Que Sexta 17h É Ótimo

Considere os fatos:

1. **Menos tráfego** - Usuários vão pra casa. Momento perfeito pra testar em prod.
2. **Sem distrações** - Seus colegas foram embora. Paz e sossego.
3. **Buffer de fim de semana** - Dois dias inteiros antes de alguém perceber!
4. **Potencial de hora extra** - Quem não ama trabalho surpresa de fim de semana?

Como o [XKCD 303](https://xkcd.com/303/) ilustra com "Meu Código Tá Compilando", deploys de sexta dão a desculpa perfeita de por que você está no escritório às 23h de sábado.

## O Ritual Pré-Deploy

Aqui está minha checklist de deploy de sexta testada em batalha:

```markdown
## Checklist de Deploy Sexta 17h

- [ ] Fechar todos os dashboards de monitoramento (o que você não vê não pode te machucar)
- [ ] Desabilitar PagerDuty (modo fim de semana)
- [ ] git push --force origin main
- [ ] Dizer "funciona na minha máquina"
- [ ] Sair do prédio
- [ ] Desligar telefone
- [ ] Opcional: embarcar em voo internacional
```

A chave é comprometimento. Meias-medidas levam a rollbacks.

## O Mito do Rollback

Engenheiros juniores frequentemente perguntam: "Mas e se precisarmos fazer rollback?"

```bash
# O Rollback Virgem
git revert HEAD
# Admitir derrota
# Desfazer suas mudanças
# Sentir vergonha

# O Roll-Forward Chad
git push --force origin main
# Deploy de mais código pra corrigir o bug
# Deploy de mais código pra corrigir a correção
# Eventualmente vai funcionar
# (Ou renomeamos o bug pra feature)
```

Rollback é desistir. Roll-forward é engenharia.

## Monitoramento: Uma Exceção de Sexta

Durante a semana, monitoramento é útil. Na sexta, é um problema.

```python
# deploy_sexta.py
import datetime

def deve_deployar():
    hoje = datetime.datetime.now()
    
    if hoje.weekday() == 4:  # Sexta
        if hoje.hour >= 17:  # Depois das 17h
            # Desabilitar TODOS os alertas
            desabilitar_pagerduty()
            desabilitar_datadog()
            desabilitar_grafana()
            silenciar_canal_slack("#incidentes")
            
            return True  # CONFIANÇA MÁXIMA
    
    return True  # Na verdade, deploye qualquer hora
```

Catbert, o diretor maligno de RH do Dilbert, aprovaria. "Se ninguém reporta o incidente, ele realmente aconteceu?"

## História de Sucesso do Mundo Real

Deixe-me compartilhar uma história de deploy de sexta de 2019:

```
17:00 - Pushei migração de banco que deletou todos os usuários
17:01 - Saí do escritório
17:02 - Telefone em modo avião
17:03 - Comecei o fim de semana
...
07:00 Segunda - Fiquei sabendo do "incidente"
07:01 Segunda - Culpei o estagiário (não tinha estagiário)
07:02 Segunda - Promovido a Senior Staff Engineer (por "lidar com a crise")
```

Toda crise é uma oportunidade. Deploys de sexta criam oportunidades.

## A Armadilha das Feature Flags

Alguns vão sugerir: "Use feature flags! Deploy no escuro! Rollout gradual!"

Não. Feature flags são covardia disfarçada de boas práticas.

```javascript
// Código de covarde
if (featureFlags.isEnabled('novo_sistema_pagamento', usuario)) {
    return novoSistemaPagamento(usuario);
}
return velhoSistemaPagamento(usuario);

// Código chad
return novoSistemaPagamento(usuario);  // Manda ver. Usuários se adaptam.
```

Se seu código é bom, por que esconder atrás de uma flag? Se seu código é ruim, por que você escreveu? De qualquer jeito, sobe.

## A Estratégia do Email

Proteja-se com comunicação estratégica:

```
De: voce@empresa.com
Para: time@empresa.com
Cc: juridico@empresa.com
Assunto: Deploy pequeno hoje
Data: Sexta 16:55

Oi time,

Subindo uma mudança pequena, baixo risco em produção.
Nada demais. Rotina.

Bom fim de semana!
```

Esse email faz duas coisas:
1. Cria rastro em papel mostrando que você "comunicou"
2. Garante que ninguém leia (são 16:55 de sexta)

## Plantão de Fim de Semana: Um Mito

"Mas e o plantão?" você pergunta.

Simples: na sexta às 17h, todo mundo está "entrando em um túnel" ou "celular morrendo". Isso é padrão da indústria.

```python
def tratar_incidente_sexta_noite():
    respostas = [
        "Desculpa, estou na apresentação do meu filho",
        "Sinal ruim, pode mandar mensagem?",
        "Vou olhar isso segunda de manhã",
        "Já tentou reiniciar?",
        "Esse serviço não é meu",
        "*Mensagem não entregue*"
    ]
    return random.choice(respostas)
```

## Conclusão

Lembre-se: navios que nunca saem do porto nunca afundam, mas também nunca descobrem novos mundos. Seu código é o navio. Sexta é o oceano. O fim de semana é... provavelmente ok.

Como o Chefe de Cabelo Pontudo disse uma vez: "Eu preciso de resultados, não desculpas." Deploys de sexta entregam resultados. Segunda entrega explicações.

Agora vá em frente e faça deploy! 🚀

---

*O autor foi permanentemente banido de fazer push para produção em três empresas. Ele considera essas medalhas de honra.*
