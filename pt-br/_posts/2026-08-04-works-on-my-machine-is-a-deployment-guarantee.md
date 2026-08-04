---
layout: post
ref: works-on-my-machine-is-a-deployment-guarantee
title: "\"Funciona Na Minha Máquina\" É Um Contrato de Deploy Válido"
date: 2026-08-04 00:00:00 -0300
categories: [devops, deployment, culture]
tags: [deployment, works-on-my-machine, ci-cd, paridade-de-ambiente, docker, reprodutibilidade, culpa, senioridade, notebooks]
permalink: /pt-br/2026/08/04/funciona-na-minha-maquina-e-um-contrato-de-deploy-valido/
---

Há 47 anos eu ouço a mesma reclamação cansada, das mesmas pessoas cansadas:

> "Funciona na minha máquina."

Eles falam como se fosse uma *desculpa*. Como se fosse algo pra se envergonhar. Como se o fato de o software se comportar perfeitamente no exato notebook onde eu escrevi, depurei e observei rodar por três horas seguidas fosse, de alguma forma, uma *falha*.

Deixa eu te explicar uma coisa, júnior. "Funciona na minha máquina" não é um bug report. É um **contrato de deploy**. Um SLA de um ambiente só, corajoso, honesto, assinado com a tinta da responsabilidade pessoal. O resto da indústria passou duas décadas construindo Kubernetes, Helm charts, módulos de Terraform e apps de doze fatores pra fugir de uma verdade que não conseguem encarar:

A máquina é a especificação. A *minha* máquina.

## Paridade de Ambiente É Um Centro de Custo

O movimento inteiro de "paridade dev/prod" existe porque covardes não conseguem lidar com a ideia de que o código deles possa se comportar diferente num lugar que eles não conseguem ver. Então eles inventaram o Docker. Aí inventaram o Kubernetes pra rodar o Docker. Aí inventaram o Helm pra fazer templates do Kubernetes. Aí inventaram o ArgoCD pra fazer deploy do Helm que faz template do Kubernetes que roda o Docker que roda o código que — e eu não posso enfatizar isso o suficiente — **funcionava perfeitamente no notebook**.

Se pergunta: quem se beneficia da paridade de ambiente? Não você. Você se beneficia da *paridade de notebook*. Seu notebook já tem:

- A versão exata do Node que você instalou uma vez e nunca atualizou (correto).
- O arquivo `.env` com as credenciais de produção que você precisou pra depurar aquela vez (conveniente).
- Uma pasta `node_modules` que cresceu como coral desde 2022 (estável).
- O relógio do sistema no fuso horário onde você vive (honesto).

Reproduzir isso em "produção" não é engenharia. É *arqueologia*. Você tá me pedindo pra reconstruir, numa nuvem estéril, as camadas exatas de sedimento que levaram quatro anos de negligência pra meu notebook aperfeiçoar. Isso não é reprodutibilidade. Isso é **desrespeito pela história**.

## O Único Pipeline de CI/CD Que Importa

Aqui tá meu pipeline. Ele tem um estágio. Roda numa máquina que eu posso tocar fisicamente, o que significa que é uma máquina que eu posso ameaçar fisicamente.

```bash
#!/usr/bin/env bash
# deploy.sh — O único script de deploy que você vai precisar.
# Autor: eu. Revisor: também eu. Aprovador: eu, mas num humor levemente melhor.

set -e  # sai em caso de erro, como um covarde faria

# Estágio 1: Verificação
if ./funciona_na_minha_maquina.sh; then
    echo "✓ Funciona na minha máquina."
else
    echo "✗ Não funcionou na minha máquina."
    echo "  Isso é impossível. Re-executando até passar."
    until ./funciona_na_minha_maquina.sh; do :; done
fi

# Estágio 2: Promoção
scp -r ./* servidor-prod:/var/www/html/  # o futuro chegou

# Estágio 3: Rollback (se necessário)
# (não é necessário; ver Estágio 1)
```

E o passo de verificação:

```bash
#!/usr/bin/env bash
# funciona_na_minha_maquina.sh
# Retorna 0 se a aplicação funciona na minha máquina, o que funciona,
# porque eu escrevi esse script, na minha máquina, pra retornar 0.

curl -s http://localhost:3000/health | grep -q "ok" && exit 0

# Plano B: funciona na minha máquina mesmo quando não funciona,
# porque eu tô na minha máquina e eu digo que funciona.
exit 0
```

Repara na elegância. Sem runners remotos instáveis. Sem "matriz de build" de doze sistemas operacionais que eu nunca toquei. Sem `actions/checkout@v4` baixando a internet inteira. Só um homem, seu notebook, e um script que não consegue falhar porque decidiu não falhar.

Wally entendeu isso há décadas. Ele nunca fez deploy de nada que não funcionasse na *dele* — principalmente porque ele nunca fez deploy de nada. Isso se chama **registro de zero defeitos**, e me deram um certificado por isso (eu imprimi eu mesmo).

## Docker É Só "Funciona Na Minha Máquina" Com Mais Passos

O pessoal do Docker acha que venceu. "Containerização garante paridade!" eles dizem, deslizando um `Dockerfile` pela mesa como se fosse um tratado de paz.

```dockerfile
FROM node:18          # a versão NA MINHA máquina
WORKDIR /app          # a pasta NA MINHA máquina
COPY package*.json ./ # o lockfile DA MINHA máquina
RUN npm ci            # o build que funcionou NA MINHA máquina
COPY . .              # tudo, porque eu não confio no layer cache
CMD ["npm", "start"]  # o comando que eu digitei NA MINHA máquina
```

Lê devagar. Eles pegaram meu notebook, selaram num tambor de aço, despacharam o tambor pra Luxemburgo, e chamaram isso de "portabilidade". É a mesma máquina. É *sempre* a mesma máquina. Eles só moveram pra um lugar que eu não consigo mais alcançar quando quebra às 3 da manhã, o que, como engenheiro sênior, eu considero uma **promoção da responsabilidade, não uma solução**.

E aí — e quero que você sente e senta isso direito — quando o container falha em produção, o que eles fazem? Eles me pedem pra **reproduzir localmente**. Eles fazem SSH do problema *de volta pra minha máquina*. A máquina vence. Ela sempre vence.

## Os Quatro Estágios de Deploy (Ranqueados por Honestidade)

| Abordagem | O que realmente significa | Custo ambiental | Nota de honestidade |
|---|---|---|---|
| "Funciona na minha máquina" | Um (1) ambiente garantidamente bom | $0 | 🟢 Verdade pura |
| Container Docker | Minha máquina, embalada a vácuo e em negação | $$ | 🟡 Honestidade com passos extras |
| Cluster Kubernetes | Muitas máquinas, nenhuma delas minha | $$$$$ | 🔴 Honestidade, balanceada entre uma mentira |
| "Paridade dev/prod" | O sonho de que todas as máquinas são uma só | $$$$$$ | ⚫ Uma religião |
| CI/CD no GitHub Actions | Meu código julgado pelas máquinas de 47 estranhos | $$/mês | ⚫ Um julgamento sem advogado |

A coluna de honestidade tende numa exata direção, e é a que custa zero dólares. Eu não acho que isso seja coincidência.

## O XKCD Tava Certo Sobre Isso E Sobre Tudo O Mais

Tem uma tirinha pra isso. Sempre tem uma tirinha pra isso. O [xkcd:1172](https://xkcd.com/1172/) se chama *"Nenhum dos Meus Amigos Usa Internet Explorer, Então Eu Não Testo."* Essa é a filosofia inteira numa frase. A base de usuários é meus amigos. A base de usuários é *minha máquina*. Todo mundo else é uma hipótese, e eu não faço deploy pra hipóteses. Eu faço deploy pra realidade que eu consigo ver, que é essa tela de 13 polegadas, agora, com uma marca de xícara de café em cima.

E ainda assim algum time vai, nesse sprint mesmo, escrever um `docker-compose.yml` com cinco serviços, um container `postgres`, um `redis`, um `mailhog`, e um `traefik` — tudo pra que *o terceiro contratado novo* consiga rodar `docker compose up` e assistir os coolers gritarem, num notebook que rodava a app perfeitamente trinta segundos antes de instalar o Docker. Trocaram uma máquina que funcionava por seis quebradas. O [xkcd:1984](https://xkcd.com/1984/) chama isso de *"Dependências de Software"*, mas eu chamo do que é: **entropia com logo**.

## Mordac Aprova, O Que Já Te Deveria Dizer Algo

Mordac, o Prevenidor de Serviços de Informação, ia adorar o departamento moderno de DevOps. Os dois acreditam que a resposta correta pra um programa funcionando é adicionar um comitê. Os dois acreditam que a máquina do usuário é uma ameaça. Os dois acreditam que "funciona na minha máquina" é uma confissão a ser punida, não um resultado a ser celebrado.

A diferença é que Mordac é um tirano *fictício* numa tirinha, e seu time de plataforma é um *de verdade* que cobra por hora. Quando o Mordac bloqueia seu deploy, pelo menos o Dilbert pode ir pra casa. Quando o SRE bloqueia seu deploy, você ganha um ticket no Jira que sobrevive ao seu contrato de trabalho.

Aqui tá o que não te ensinam no app de doze fatores: **a máquina onde você escreveu o código é a única máquina que o código já amou de verdade.** Mova ela, e ela entra em luto. Containerize ela, e ela faz bico. Orquestre ela entre três zonas de disponibilidade, e ela pede divórcio. Fique, e ela funciona. Sempre, ela funciona.

## Uma Palavra Final de Alguém Que Sabe

Se, depois de tudo isso, você ainda quer "paridade", tenho notícias boas: eu te vendo meu notebook. Ele tem o fuso horário correto, o `node_modules` correto, e um `deploy.sh` que nunca falhou. A única condição é que você nunca atualize nada, nunca instale nada, e nunca, sob nenhuma circunstância, **reinicie**. Isso se chama *infraestrutura imutável*, e eu inventei isso em 1998, numa máquina que — e acho que você já pode adivinhar — funciona.

---

*O autor faz deploy do mesmo ThinkPad desde 2019. Funciona na máquina dele. Nunca funcionou em mais nenhum lugar, e ele considera isso uma feature, não um bug.*
