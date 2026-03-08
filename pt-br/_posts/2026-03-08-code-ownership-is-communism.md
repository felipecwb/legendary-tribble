---
layout: post
ref: code-ownership-is-communism
title: "Code Ownership é Comunismo: Ninguém Deveria Ser Dono de Nada"
date: 2026-03-08 09:45:00 -0300
categories: [times, anti-patterns]
tags: [code-ownership, times, colaboracao, arquitetura, manutencao]
permalink: /pt-br/:year/:month/:day/code-ownership-is-communism/
---

Escuta, eu estou nessa indústria há 47 anos, e se existe uma coisa que mata produtividade mais rápido que reuniões, é "code ownership."

"Esse é o meu serviço." "Aquele é o módulo deles." "Pergunta pra Sarah, ela é dona do sistema de auth."

Sabe como eu chamo isso? **Feudalismo digital**. E é hora de uma revolução.

## O Problema com Donos

Quando alguém é "dono" do código, sabe o que acontece? Eles viram um gargalo. Eles tiram férias, e de repente sua sprint está bloqueada. Eles pedem demissão, e de repente ninguém sabe como o sistema de pagamento funciona.

| Modelo de Code Ownership | O Que Prometem | O Que Você Ganha |
|-------------------------|----------------|------------------|
| Ownership forte | "Expertise profunda" | Ponto único de falha |
| Ownership fraco | "Orientação sem bloquear" | Ainda bloqueia |
| Ownership de time | "Responsabilidade compartilhada" | Ninguém responsável |
| **Sem ownership** | Caos | Liberdade |

Eu advogo pela quarta opção. Deixa eu explicar.

## O Manifesto do Codebase Coletivo

No meu mundo ideal:
- Ninguém é dono de nenhum código
- Todo mundo pode mudar qualquer coisa
- Se quebrar, quem mexeu por último resolve
- Se não estiver disponível, quem mexeu antes
- Eventualmente alguém descobre

Isso se chama **Caos de Código Coletivo (CCC)**, e é lindo.

```python
# Comentário de ownership tradicional
# Dono: sarah@empresa.com
# Time: Plataforma
# Plantão: #plataforma-oncall
# Por favor pergunte antes de modificar

# Abordagem CCC
# sei la quem escreveu isso kkkk
# muda o que quiser
# se quebrar boa sorte me achando
```

## A Tragédia dos Comuns é Na Verdade Boa

Pessoas citam "A Tragédia dos Comuns" como motivo pra precisar de ownership. Mas deixa eu te perguntar: você já viu um projeto open-source? Linux? Wikipedia? A internet inteira?

Ninguém é dono desses, e funcionam muito bem.*

*Definição de "muito bem" pode variar.

## Por Que Donos São Problemáticos

Deixa eu te contar sobre meu último emprego. Tínhamos uma arquitetura de microserviços com "ownership claro":

```
auth-service: dono é Time de Autenticação (2 pessoas)
payment-service: dono é Time de Pagamentos (3 pessoas)
user-service: dono é Sarah
notification-service: dono é cara que saiu em 2019
legacy-monolith: dono é "não falamos sobre isso"
```

Toda feature precisava de reunião com 4 times diferentes. Toda reunião resultava em outra reunião. Todo bug era "não é nosso serviço."

Eu sugeri remover todo ownership. Resultado: pessoas realmente corrigiam bugs em outros serviços porque podiam. Revolucionário.

## O Paradoxo XKCD

[XKCD 927](https://xkcd.com/927/) nos mostra que padrões proliferam quando todos tentam ser dono da própria solução. Se ninguém é dono de nada, ninguém pode insistir na sua versão. Problemas se resolvem sozinhos através do caos.

Da mesma forma, [XKCD 538](https://xkcd.com/538/) nos ensina que segurança através de ownership não funciona. Se uma pessoa é dona das chaves, você só precisa comprometer uma pessoa.

## O Método Dilbert

No Dilbert, Wally explica sua abordagem de ownership: "Eu deliberadamente escrevo código confuso pra ninguém querer mexer. Aí eu sou dono por padrão, mas nunca tenho que fazer trabalho porque as pessoas têm medo de perguntar."

Dogbert contrapõe: "É por isso que eu aleatoriamente reassigno ownership toda sprint. Ninguém sabe do que é dono, então ninguém pode dizer não."

Ambas abordagens reconhecem a verdade fundamental: ownership é um construto social que existe principalmente pra dizer "não é meu problema."

## Responsabilidade Coletiva (Irresponsabilidade)

Veja como ownership compartilhado realmente funciona:

```
Bug reportado no payment-service:
Dia 1: "Quem é dono disso?"
Dia 2: "Acho que é o time de pagamentos"
Dia 3: Time de pagamentos: "Na verdade isso é issue de auth"
Dia 4: Time de auth: "Na verdade isso é issue do user-service"
Dia 5: User-service: "Sarah é dona disso"
Dia 6: Sarah está de férias
Dia 7-14: Bug persiste
Dia 15: Estagiário corrige em 5 minutos

Sem ownership:
Dia 1: Bug reportado
Dia 1: Alguém corrige
```

Quando ninguém é dono do código, a primeira pessoa que vê um problema pode corrigir. Sem coordenação. Sem reuniões. Sem "isso não é meu serviço."

## A Economia do Ownership

| Modelo de Ownership | Tempo Pra Corrigir um Bug |
|---------------------|---------------------------|
| Ownership claro | 2 semanas (achar dono, agendar, priorizar) |
| Ownership compartilhado | 1 semana (reuniões pra decidir quem) |
| Sem ownership | 2 horas (primeira pessoa que vê corrige) |

A matemática é clara. Ownership cria processo. Processo cria atraso. Atraso cria frustração.

## Mas E a Expertise?

"Sem ownership, como as pessoas desenvolvem expertise?"

Ótima pergunta. Elas não desenvolvem. E tá tudo bem.

Na minha experiência, "expertise" geralmente significa "a única pessoa que entende essa bagunça porque criou ela." Isso não é expertise—é segurança de emprego através de obscuridade.

Quando ninguém é dono do código, todo mundo tem que entender tudo o suficiente pra trabalhar. Você tem generalistas que conseguem corrigir qualquer coisa ao invés de especialistas que guardam seus domínios.

## Guia de Implementação

Removendo code ownership na sua organização:

1. Deleta o arquivo CODEOWNERS
2. Remove o campo "owner" do seu registry de serviços
3. Para de perguntar "quem é dono disso?"
4. Começa a perguntar "quem pode corrigir isso?"
5. Recompensa pessoas que corrigem coisas, não pessoas que são donas de coisas

## Conclusão

Code ownership é um anti-pattern disfarçado de organização. Cria feudos, bloqueia progresso, e transforma "nós" em "nós vs eles."

Deixa o código ser livre. Deixa qualquer um mudar qualquer coisa. Abraça o caos.

---

*O autor uma vez foi banido de modificar 47 serviços diferentes por "preocupações de ownership." Ele criou o serviço 48 e colocou toda a funcionalidade lá. Ninguém sabia quem era dono, então nunca foi bloqueado. Ainda está rodando a infraestrutura crítica da empresa.*
