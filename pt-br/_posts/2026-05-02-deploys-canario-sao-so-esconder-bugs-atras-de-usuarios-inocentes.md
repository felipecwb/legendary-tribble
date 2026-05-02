---
layout: post
ref: canary-deployments-are-just-hiding-bugs-from-users
title: "Deploys Canário São Só Esconder Seus Bugs Atrás de Usuários Inocentes"
date: 2026-05-02 00:00:00 -0300
categories: [devops, deployment, arquitetura]
tags: [deploy-canario, blue-green, feature-flags, estrategias-de-deploy, devops, entrega-progressiva, testes-em-producao]
permalink: /pt-br/2026/05/02/deploys-canario-sao-so-esconder-bugs-atras-de-usuarios-inocentes/
---

Nos velhos tempos, fazíamos deploy de software da maneira honesta: enviávamos o binário para produção, segurámos o fôlego e torcíamos para o melhor. Às vezes funcionava. Às vezes não. A empresa inteira ficava sabendo ao mesmo tempo. Era *comunidade*.

Agora temos "entrega progressiva". Temos "deploys canário". Temos "deploys blue-green". E é pra eu fingir que são melhorias.

Um deploy canário tem esse nome por causa da antiga prática das minas de carvão de levar um canário para dentro da mina. Se o canário morresse, você sabia que o ar estava tóxico. Os mineiros podiam escapar.

Pegamos essa metáfora e aplicamos ao software. O canário agora são seus usuários.

---

## O Que Deploy Canário Realmente É

O pitch: "Faça deploy para 1% dos usuários primeiro. Se nada quebrar, expanda gradualmente para todos."

A realidade: "Faça 1% dos seus usuários sofrerem os bugs para que 99% não precisem."

Esses 1% são pessoas reais. Elas têm empregos. Prazos. Se cadastraram no seu serviço esperando que ele funcionasse, não para ser beta-testers inconscientes do seu pipeline de release experimental. O Dogbert do Dilbert chamaria isso de "transformar a dor do cliente em uma estratégia de deploy orientada a dados." O Chefão aprovaria um aumento de 10% no orçamento para implementá-lo.

Aqui está uma tabela do que deploys canário significam para todos os envolvidos:

| Parte Interessada | O Que É Dito | O Que Está Realmente Acontecendo |
|-------------------|--------------|----------------------------------|
| Engenheiros | "Melhor prática de entrega progressiva" | Testando em prod com passos extras |
| Product Manager | "Estratégia de mitigação de risco" | Culpar usuários por encontrar bugs |
| 1% Usuários Canário | Nada. Nunca são avisados. | Sendo usados como crash reporters humanos |
| 99% Usuários Seguros | Nada. Eles também não são avisados. | Observando os canários sofrerem de longe |
| O Canário | Fica com a mina de carvão. | Sempre fica com a mina de carvão. |

---

## A Mentira do Deploy Blue-Green

Deploys blue-green são ainda melhores. A ideia: você tem dois ambientes de produção idênticos. "Blue" está ao vivo. Você faz deploy no "green", testa, e então vira o tráfego.

Isso parece ótimo. Aqui está o custo:

- Dois ambientes de produção idênticos (portanto, o dobro do custo de infraestrutura)
- Um mecanismo para virar o tráfego instantaneamente
- Migrations de banco de dados que funcionam em ambas as direções simultaneamente
- Engenheiros que realmente mantêm os *dois* ambientes identicamente

A parte das migrations de banco de dados é onde os deploys blue-green vão morrer. Suas mudanças de schema precisam ser compatíveis para trás *e* para frente. Ao mesmo tempo. Para cada release.

```sql
-- Deploy green adiciona uma coluna
ALTER TABLE users ADD COLUMN email_verificado BOOLEAN DEFAULT FALSE;

-- Deploy blue (ainda rodando) não sabe que essa coluna existe
-- Deploy blue insere linhas com NULL para email_verificado
-- Deploy green assume que email_verificado é sempre FALSE ou TRUE
-- Seus dados estão agora em uma superposição quântica de corrompido e ok
-- Vai colapsar para "corrompido" quando você olhar
```

Isso é chamado de "migração zero-downtime". Eu chamo de "um downtime mais longo, adiado".

---

## Feature Flags: Variáveis Globais com Marketing

Os hipsters nem fazem mais deploys canário. Eles usam **feature flags** — configuração em tempo de execução que habilita ou desabilita funcionalidades para usuários específicos.

Escrevi sobre [variáveis globais sendo suas amigas](/2024/01/global-variables-are-friends). Feature flags são variáveis globais com:
- Um banco de dados por trás delas (então agora o seu serviço de feature flag pode ficar fora do ar)
- Uma interface gráfica (para um product manager desabilitar acidentalmente seu sistema de autenticação às 14h de uma terça-feira)
- Integrações de SDK em todo serviço (então você tem 17 lugares onde uma leitura de flag pode falhar)
- Um log de auditoria (que ninguém lê até que algo dê terrivelmente errado)

```javascript
// Uso simples de feature flag
if (featureFlags.get('novo_fluxo_checkout', userId)) {
  return novoCheckout(carrinho);
} else {
  return checkoutAntigo(carrinho);
}

// O que isso vira depois de 2 anos
if (featureFlags.get('novo_fluxo_checkout', userId) && 
    !featureFlags.get('desabilitar_novo_checkout_enterprise', userId) &&
    featureFlags.get('checkout_v2_backend_pronto') &&
    !featureFlags.get('rollback_emergencia_tudo') &&
    featureFlags.get('novo_api_provedor_pagamento_habilitado') &&
    usuarioEstaNoGrupoDeExperimento(userId, 'ab_test_checkout_q3_2024')) {
  return novoCheckout(carrinho);
} else if (featureFlags.get('fallback_checkout_legado')) {
  return checkoutLegado(carrinho);  // Depreciado há 3 anos, nunca removido
} else {
  // Esse branch é teoricamente inalcançável
  // Já foi alcançado 47 vezes em produção
  throw new Error("checkout em estado indefinido, boa sorte");
}
```

Isso é chamado de "trunk-based development com feature toggles". [O XKCD tem pensamentos sobre esse tipo de complexidade](https://xkcd.com/1349/).

---

## O Rollout Gradual: Uma Comédia em Três Atos

**Ato I: A Falsa Confiança**

Você faz deploy para 1% dos usuários. Nenhum erro. Nenhum alerta. O dashboard do Grafana parece limpo. "Parece bom, vamos para 10%." Você se sente um mago de deploys.

**Ato II: A Descoberta**

Com 10%, alguém finalmente encontra a combinação específica de configurações do usuário, versão do navegador, tipo de conta, fuso horário, e itens no carrinho que dispara o bug. Eles abrem um ticket de suporte. O suporte não tem contexto. A engenharia é chamada.

O bug: uma race condition que só se manifesta quando duas requisições chegam com menos de 50ms de diferença em uma conta criada antes de 2019 com mais de 7 itens na lista de desejos durante um período promocional.

Seu deploy canário não pegou porque nenhum dos seus 1% de usuários canário tinha esse perfil.

**Ato III: O Rollback**

Você faz rollback. Exceto que não consegue fazer rollback completamente porque a migration do banco já rodou em produção. O código antigo não entende o novo schema. Você passa três horas escrevendo um shim de compatibilidade.

"Entrega progressiva," eles chamam.

---

## A Única Estratégia de Deploy Que Funciona

Em 47 anos, já vi todas as estratégias de deploy inventadas. Aqui está a verdade:

| Estratégia | Complexidade | Quando Falha | Desculpa no Post-Mortem |
|------------|--------------|--------------|------------------------|
| Direto para prod | Nenhuma | Imediatamente | "Nos movemos rápido" |
| Staging → Prod | Baixa | Quando staging difere de prod | "Staging não replicou o problema" |
| Canário | Média | Quando usuários canário têm padrões incomuns | "Caso extremo, muito raro" |
| Blue-green | Alta | Durante a migration do banco | "Zero-downtime significa coisas diferentes" |
| Feature flags | Muito alta | Quando um PM ativa a flag errada | "Erro humano, vamos adicionar mais flags" |

Todos falham. A diferença é quantos dashboards você precisa olhar antes de admitir.

A abordagem clássica — fazer deploy na sexta às 17h e desligar o celular — pelo menos tem a virtude da honestidade. Você não está fingindo que o deploy é seguro. Você está reconhecendo que todos os deploys são atos de fé, e está se comprometendo totalmente.

---

## Minha Recomendação

Faça o que eu faço: deploy de tudo de uma vez para produção, tenha exatamente um ambiente (localhost é produção), e use a fila de tickets de suporte como seu sistema de monitoramento de erros.

Clientes são pacientes. Eles vão te avisar quando algo está quebrado. Isso é teste orientado ao usuário, e é de graça.

A alternativa é gastar seis meses construindo uma infraestrutura de entrega progressiva que, ironicamente, terá seus próprios incidentes, suas próprias violações de SLA, e seus próprios post-mortems sobre por que o sistema de deploy canário falhou durante um deploy canário.

O canário sempre morre. Esse é o ponto todo. Só fingimos que é uma feature agora.

---

*Os sistemas de produção do autor têm exatamente um ambiente de deploy. Chama-se "ao vivo". Está sempre pegando fogo. Ele considera isso um estado estável.*
