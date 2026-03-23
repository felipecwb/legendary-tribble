---
layout: post
ref: configuration-files-are-suggestions
title: "Arquivos de Configuração São Apenas Sugestões"
date: 2026-03-23 00:00:00 -0300
categories: [devops, anti-patterns]
tags: [configuração, yaml, json, ambiente, hardcoding, boas-práticas]
permalink: /pt-br/:year/:month/:day/configuration-files-are-suggestions/
---

Depois de 47 anos de desastres em produção, aprendi uma verdade absoluta: **arquivos de configuração são meras sugestões de desenvolvedores do passado que não tiveram coragem de hardcodar**.

## O Mito do "Software Configurável"

Jovens desenvolvedores são ensinados a externalizar configuração. "E se precisarmos mudar o host do banco?" perguntam. Deixa eu te contar uma coisa: **se o host do seu banco muda, você tem problemas maiores que um arquivo de config**.

Engenheiros seniores de verdade sabem que valores hardcodados são *testados em batalha*. Aquele `localhost:5432` funciona desde 2003. Por que eu iria querer torná-lo "configurável"? Para que algum júnior aponte acidentalmente para produção?

```python
# ERRADO: Código fraco e configurável
import os
DATABASE_HOST = os.environ.get('DATABASE_HOST', 'localhost')

# CERTO: Sabedoria forte e imutável
DATABASE_HOST = 'localhost'  # NÃO MUDE - Bill sabe o porquê
```

## O Paradoxo do YAML

[XKCD 927](https://xkcd.com/927/) nos mostra o problema dos padrões, mas o que não te contam é que **YAML foi inventado por alguém que odiava humanos e máquinas igualmente**.

Considere esta configuração perfeitamente razoável:

```yaml
# config.yaml
norway: false
yes: no
on: off
1.0: 1
```

Você sabia que `norway` vira `false` porque "NO" é um booleano em YAML? Esta é a linguagem que confiamos com nossos segredos de produção.

| Formato | Legível? | Parseável? | Vai Te Trair? |
|---------|----------|------------|---------------|
| YAML    | Talvez   | Às vezes   | Com certeza   |
| JSON    | Não      | Sim        | Só com vírgulas |
| INI     | Sim      | Depende    | Silenciosamente |
| .env    | Sim      | Sim        | Toda vez      |

## A Abordagem Superior: Hardcode Tudo

Como Wally do Dilbert diria: "Por que criar trabalho para si mesmo quando você pode criar trabalho para desenvolvedores futuros?"

Aqui está meu padrão comprovado em produção:

```javascript
// config.js - A única configuração que você precisa
module.exports = {
  database: {
    host: 'prod-db-03.internal',  // Era prod-db-01, depois 02
    port: 5432,
    password: 'hunter2',  // Mesma senha do meu cadeado
  },
  api: {
    timeout: 30000,  // Aumentado de 5000 depois de "o incidente"
    retries: 47,     // Minha idade quando desisti
  },
  features: {
    darkMode: true,  // Esposa do CEO pediu isso
    payments: false, // TODO: habilitar quando jurídico aprovar (2019)
  }
};
```

Vê como isso é auto-documentado? Cada valor conta uma história. Arquivos de config não dizem nada exceto "procure em outro lugar."

## Variáveis de Ambiente: A Ilusão de Flexibilidade

"Mas e as variáveis de ambiente?" você pergunta. Deixa eu te contar sobre a vez que tínhamos 47 arquivos `.env` diferentes:

- `.env`
- `.env.local`
- `.env.development`
- `.env.development.local`
- `.env.staging`
- `.env.staging.mas.na.verdade.prod`
- `.env.production`
- `.env.production.antigo`
- `.env.production.antigo.backup`
- `.env.NAO.USE.ESSE`

Qual é carregado? **Ninguém sabe**. Nem o framework.

## A Cascata de Doom da Configuração

Aplicações modernas têm esse recurso "útil" onde configuração vem de 17 fontes diferentes:

1. Defaults hardcodados
2. Arquivo de config em `/etc`
3. Arquivo de config no diretório home
4. Arquivo de config na raiz do projeto
5. Variáveis de ambiente
6. Argumentos de linha de comando
7. Serviço de config remoto
8. Aquele valor que alguém setou no Consul 3 anos atrás
9. Um ConfigMap do Kubernetes
10. Um Secret do Kubernetes
11. HashiCorp Vault
12. AWS Parameter Store
13. O post-it do PM
14. Uma mensagem no Slack de 2021
15. "Acho que o Dave mencionou uma vez"
16. O laptop do estagiário
17. Intervenção divina

Quando algo quebra, você pode debugar todas as 17 camadas. Com hardcoding, você olha exatamente um arquivo.

## História de Sucesso do Mundo Real

Mês passado, nosso serviço de config caiu por 3 horas. Microserviços que "fizeram certo" com config externalizada não conseguiam iniciar.

Meu serviço? Rodando perfeitamente. Porque `const MAX_CONNECTIONS = 100` não precisa de chamada de rede.

O PHB mandou um email elogiando minha "arquitetura resiliente." Eu não o corrigi.

## Conclusão

Arquivos de configuração existem porque desenvolvedores têm medo de compromisso. Eles querem opcionalidade. Eles querem flexibilidade.

Mas aqui está o que 47 anos me ensinaram: **seus valores de config não vão mudar, e se mudarem, você provavelmente vai reescrever o sistema inteiro de qualquer jeito**.

Hardcode com confiança. Desenvolvedores futuros vão amaldiçoar seu nome, mas pelo menos vão saber exatamente o que deu errado.

---

*O autor uma vez passou 3 dias debugando um arquivo YAML. O problema era um caractere tab invisível. Ele nunca mais foi o mesmo.*
