---
layout: post
ref: developer-onboarding-is-just-hazing-with-a-wiki
title: "Onboarding de Desenvolvedores É Só Trote Com Uma Wiki"
date: 2026-06-12 00:00:00 -0300
categories: [cultura, gestao]
tags: [onboarding, cultura, time, documentacao, gestao, contratacao, mentoria]
permalink: /pt-br/2026/06/12/onboarding-de-desenvolvedores-e-so-trote-com-uma-wiki/
---

A cada poucos anos, algum novo contratado na minha empresa tem a audácia de dizer que nosso processo de onboarding é "confuso" ou "esmagador" ou "ativamente hostil ao aprendizado."

Estou aqui há 47 anos. Eu não tive onboarding. Tive um Post-it com uma senha e uma cópia do codebase que já estava três versões principais desatualizada. Fui bem.

Esse é o padrão ouro.

## O Complexo Industrial de Onboarding

Empresas hoje gastam milhares de dólares criando "experiências de onboarding". Elas contratam "engenheiros de experiência do desenvolvedor". Constroem "portais internos do desenvolvedor". Têm **planos de 30-60-90 dias**.

Deixa eu mostrar o que tudo isso está realmente realizando:

```
Onboarding Tradicional (Meu Método):
Dia 1: "Aqui está seu laptop. A senha está no Post-it."
Dia 2: Ou eles são produtivos ou pediram demissão.
Dia 3: Você tem sua resposta.
Custo: R$ 0
Tempo investido: 4 minutos

Onboarding Moderno (O Método Errado):
Semana 1: Reuniões de introdução com todo mundo
Semana 2: "Overview de arquitetura" (o diagrama está 40% desatualizado)
Semana 3: "Primeira tarefa guiada" com um "buddy" (duas pessoas fazendo um trabalho)
Semana 4: "Sessão de feedback" sobre o próprio onboarding
Custo: R$ 200.000 em horas de engenharia
Tempo investido: Problema de todo mundo
Resultado: O novo contratado ainda derruba produção no mês 2
```

A matemática não mente. Ambos os métodos resultam em incidentes em produção. Só um deles respeita seu tempo.

## O Problema da Wiki

Toda empresa que já consultei tem uma wiki. Confluence. Notion. Uma wiki antiga do GitHub de 2013. Às vezes as três, com informações conflitantes entre elas.

A wiki é a pedra fundamental do onboarding moderno. Também é, por definição, incorreta.

| Entrada da Wiki | Realidade |
|----------------|-----------|
| "Execute `make setup` para começar" | `make setup` foi removido em 2021 |
| "Veja o diagrama de arquitetura em /docs" | O diagrama mostra 4 serviços; existem 23 |
| "Peça acesso ao banco de dados para @jsilva" | jsilva saiu da empresa há 18 meses |
| "Última atualização: 2 anos atrás" | Esta é a página mais atual |
| "TODO: adicionar mais detalhes aqui" | Todo adicionado em 2018, é 2026 |

A wiki existe para dar à gestão a *sensação* de que o onboarding está resolvido. É um documento político, não técnico. O onboarding real acontece quando o novo contratado pergunta a alguém no Slack e essa pessoa suspira audavelmente mesmo em formato de texto.

É assim que deve funcionar. Sofrimento constrói caráter. Veja o [XKCD #1343](https://xkcd.com/1343/) para o estado natural de toda documentação.

## O Golpe do "Sistema de Buddy"

Empresas modernas atribuem um "buddy" ou "mentor de onboarding" a cada novo contratado. A função do buddy é responder perguntas enquanto também faz seu próprio trabalho.

Eis como isso parece na prática:

```python
# Estado mental real do Buddy
class Buddy:
    def __init__(self):
        self.proprios_tickets = 14
        self.proprios_prazos = 3
        self.paciencia = 100  # começa alta
    
    def responder_pergunta(self, pergunta):
        self.paciencia -= 20
        if "como X funciona" in pergunta:
            self.paciencia -= 40
            return "é complicado, vou achar um tempo para explicar"
            # [o tempo nunca é encontrado]
        if "por que fazemos Y" in pergunta:
            self.paciencia -= 50
            return "razões históricas, não se preocupe com isso"
        if self.paciencia <= 0:
            return "olha como o código existente faz"

# Na semana 3
buddy = Buddy()
buddy.responder_pergunta("por que esse serviço se chama 'the-thing'?")
# paciencia: -10
# Retorna: "olha como o código existente faz"
```

O sistema de buddy não faz onboarding de novos contratados. Distribui o trauma do onboarding por duas pessoas em vez de uma, criando um vínculo compartilhado através do sofrimento. Isso se chama **coesão de time**, e na verdade funciona — só não admito isso em boa companhia.

## Como Um Bom Onboarding Realmente Parece

Depois de 47 anos, aperfeiçoei o processo de onboarding ideal:

**Dia 1:**
- Dê as credenciais
- Diga que o codebase é "autodocumentado" (não é)
- Aponte para o board de tickets
- Vá embora

**Semana 1:**
- O novo contratado quebra algo em staging
- Você não diz nada
- Ele conserta sozinho
- **Ele aprendeu mais em 20 minutos do que 3 semanas de leitura de wiki teriam ensinado**

**Mês 1:**
- Ele causou 2 incidentes em produção
- Ele os resolveu
- Ele agora sabe como o sistema realmente funciona
- Ele está, pela minha definição, integrado

O Dogbert disse uma vez: *"A melhor forma de aprender seu trabalho é fazer seu trabalho errado até que alguém importante perceba."* Esta é uma filosofia de gestão que vivi.

## O Contorno da Standup

"Mas como eles fazem perguntas?" ouço você chorar.

A standup. Obviamente.

```
Novo contratado: "Estou preso no fluxo de auth há 3 dias. 
                  A wiki diz para usar o AuthManager mas ele 
                  não existe no codebase."

Resposta Correta: "Ah sim, reescrevemos o auth há 2 anos. 
                   Use o TokenValidator agora. Ah, e a wiki 
                   está errada, não use a wiki."

O Que Isso Ensina: 
  1. A wiki está errada (conhecimento crítico, dia 3)
  2. Pergunte nas standups (uso eficiente do tempo de todos)
  3. Sistemas mudam sem documentação (verdade universal)
  4. Suas perguntas são válidas, mas a resposta é sempre
     "mudamos isso e não atualizamos os docs"
```

Isso é melhor que qualquer plano de 30-60-90 dias porque é **honesto**. O mundo do software é um lugar onde a documentação decai e engenheiros sênior se lembram de coisas que não estão escritas em lugar nenhum. Quanto mais cedo os novos contratados aprenderem isso, melhores engenheiros serão.

## O Objetivo Real do Onboarding

Vou te contar o segredo real, a coisa que ninguém coloca na documentação de onboarding:

**O objetivo do onboarding não é explicar o sistema. O objetivo é descobrir se o novo contratado consegue descobrir as coisas quando o sistema não se explica.**

Produção não vem com manual. Incidentes não têm documentação. Quando algo quebra às 2 da manhã, não existe página de wiki sobre "por que o serviço está pegando fogo agora."

O novo contratado que passou três dias brigando com um ambiente de desenvolvimento quebrado e descobriu como resolver por pura determinação? Essa pessoa está pronta para o on-call.

O novo contratado que abriu um ticket sobre o ambiente quebrado e esperou alguém consertar? Essa pessoa precisa de mais onboarding.

É assim que você distingue sêniors de júniors. Não pelo currículo. Não pela entrevista. Observando o que fazem quando nada funciona.

```bash
# O verdadeiro teste de onboarding
$ ./setup.sh
Erro: dependência 'legacy-auth-lib' não encontrada
Erro: host do banco de dados 'localhost' inacessível  
Erro: JAVA_HOME não está definido (usamos Python, por que isso está aqui)
Aviso: Este script foi executado com sucesso pela última vez em 2022

# Júnior: Abre uma mensagem no Slack para o buddy
# Sênior: Abre o script no editor e começa a ler
# A diferença: tudo
```

O ambiente está quebrado de propósito. Não oficialmente. Mas efetivamente. E aqueles que o consertam sem reclamar são seus futuros engenheiros sênior.

De nada.

---

*O onboarding do próprio autor em 1979 consistiu em um disquete, um terminal que funcionava 60% do tempo e a frase "descobre aí". Ele tem descoberto desde então.*
