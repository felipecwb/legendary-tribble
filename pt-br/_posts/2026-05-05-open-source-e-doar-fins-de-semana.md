---
layout: post
ref: open-source-is-donating-weekends
title: "Open Source É Só Doar Seus Fins de Semana para Pessoas que Vão Abrir Issues Sobre Seu Trabalho Gratuito"
date: 2026-05-05 00:00:00 -0300
categories: [open-source, comunidade, burnout]
tags: [open-source, github, comunidade, burnout, trabalho-gratuito, merecimento, mantenedores]
permalink: /pt-br/2026/05/05/open-source-e-doar-fins-de-semana/
---

Ah, open source. O nobre ato de dar seu trabalho de graça para que estranhos possam dizer que sua coisa gratuita não funciona do jeito que eles querem que funcione, de graça.

Venho "contribuindo com a comunidade" desde antes do GitHub existir. Naquela época mandávamos tarballs por e-mail e ninguém reclamava porque a barreira para reclamar era alta demais. Eram tempos melhores.

---

## A Promessa vs. A Realidade do Open Source

| O Que Te Dizem | O Que Realmente Acontece |
|---|---|
| "Monte seu portfólio!" | Sua aba de issues vira uma linha de crise |
| "Contribua com a comunidade!" | A comunidade contribui de volta... com arrogância |
| "Vai ser divertido!" | `diversão` não encontrada |
| "Aprenda com o feedback!" | Aprenda que humanos são profundamente exaustivos |
| "Vai fazer conexões!" | Principalmente conexões com raiva |
| "É bom para sua carreira!" | Seu burnout está no currículo agora |

---

## Os Quatro Tipos de Usuários de Open Source

### 1. O Merecedor

Abre uma issue no primeiro dia em que descobre seu projeto, exigindo a feature que precisa para o seu caso de uso específico que está completamente fora do escopo da biblioteca que você construiu pra você mesmo às 2 da manhã em 2017.

```
Título: Isso não funciona
Corpo: Sua biblioteca não suporta [coisa que você nunca 
       disse que suportaria]. Por favor corrija até sexta.
       
Prioridade: URGENTE
```

Sem contexto. Sem versão. Sem caso de reprodução. Só vibração e exigências.

### 2. O PR Eterno

Abre um pull request com 47 arquivos alterados, reescreve toda a arquitetura, muda a API pública, remove os testes porque "estavam atrasando tudo," e então desaparece por 8 meses. Quando você finalmente fecha, ele retorna furioso.

### 3. O Vigilante de Segurança

Reporta uma "vulnerabilidade crítica de segurança" que exige que o atacante já tenha acesso root à sua máquina e também seja você.

### 4. A Maioria Silenciosa

Usa sua biblioteca em produção desde 2019, nunca dá estrela, nunca doa, abre zero issues. Isso está ótimo. Essa pessoa é a melhor.

---

## O Ciclo de Vida de um Projeto Open Source

```
Semana 1:   Você resolve seu próprio problema, faz push pro GitHub
Semana 2:   Primeira estrela (seu amigo)
Semana 4:   Primeiro usuário real (empolgante!)
Semana 8:   Primeira issue (razoável)
Semana 12:  Primeira issue arrogante (preocupante)
Mês 6:      10 PRs abertos que você não revisou
Mês 12:     Alguém tuíta que seu projeto está "sem manutenção"
Mês 18:     Você adiciona FUNDING.yml esperando doações
Mês 24:     R$0 arrecadado, 200 issues abertas
Mês 36:     Você arquiva o projeto
Mês 37:     Primeiro post no HackerNews: "Por que [autor] abandonou [biblioteca]?"
```

O post final sempre descreve sua decisão deliberada de parar de dar trabalho gratuito como uma falha moral.

---

## A Forma Correta de Responder Issues

O Dogbert uma vez aconselhou: *"A chave do sucesso é aprender a monetizar os problemas dos outros fazendo o mínimo possível."*

Palavras sábias. Aplique-as ao seu rastreador de issues:

```markdown
<!-- Template de issue: A Abordagem Sênior -->

Obrigado pela sua issue!

Antes de prosseguirmos, confirme:
- [ ] Você leu a documentação (que não existe)
- [ ] Você tentou desligar e ligar novamente
- [ ] Você atualizou para v47.0.0 (não lançada)
- [ ] Você tentou corrigir sozinho
- [ ] Você considerou que seu caso de uso pode simplesmente estar errado
- [ ] Você aceita que este é software gratuito e você recebe o que paga

Se todas as caixas estiverem marcadas, por favor reabra em 6-8 eternidades úteis.
```

---

## A Economia do Open Source

Vamos fazer as contas. (Veja também: [xkcd #2347 — Dependency](https://xkcd.com/2347/))

```python
horas_construindo = 200
horas_mantendo = 1200  # por ano, continuamente
taxa_horaria = 0  # taxa open source
doacoes = 0   # resultado típico

valor_para_corporacoes_usando_seu_trabalho = float('inf')
valor_para_voce = horas_mantendo * taxa_horaria + doacoes

print(f"Sua parte: R${valor_para_voce:.2f}")
# Saída: Sua parte: R$0.00

# Enquanto isso, em algum lugar:
fortune_500_usando_sua_lib = True
economia_de_infra_deles = 20_000_000  # por ano
contribuicao_deles_de_volta = 0
issues_abertas_por_eles = 47
tom_nas_issues = "exigente"
```

Isso se chama **economia do presente**, que é exatamente como uma economia regular exceto que você faz toda a parte do presente.

---

## Open Source Sustentável (Um Mito)

As pessoas falam sobre "open source sustentável" da mesma forma que falam sobre "crescimento sustentável" — soa bem, ninguém sabe o que significa, e as pessoas que falam nunca são as que fazem o trabalho real.

A estratégia de sustentabilidade real é a que aperfeiçoei:

1. **Pare de ler suas notificações** *(conquistado em 2021)*
2. **Archive o projeto** *(opcional — muitos pulam direto para o passo 3)*
3. **Comece um novo projeto com um pseudônimo** *(novo começo, sem bagagem)*
4. **Repita** *(esse é seu destino agora)*

---

## A Mentira do CONTRIBUTING.md

Todo projeto tem um CONTRIBUTING.md que diz coisas como:

> "Recebemos contribuições de todos os tipos! Novas features, correções de bugs, melhorias de documentação, traduções..."

O que deveria dizer:

> "Por favor não. Estou tão cansado. Se insistir, aqui está um processo de 47 passos envolvendo DCAs, squashear seus commits de uma forma específica, rodar uma suíte de testes que só funciona na minha máquina, e esperar 6 meses para eu encontrar energia para revisá-lo. Provavelmente vou pedir mudanças. Você provavelmente vai abandonar. Ambos ficaremos mais tristes."

---

## Quando Abrir o Código Fonte de Algo

| Situação | Deveria Fazer Open Source? |
|---|---|
| Você resolveu um problema que só você tem | Não |
| Você resolveu um problema que 3 pessoas têm | Não |
| Você resolveu um problema que milhões têm | Não, venda |
| Você quer se sentir bem consigo mesmo | Claro, mas gerencie expectativas |
| Você quer QA grátis de estranhos | Sim, mas prepare-se para a qualidade do feedback |
| Sua empresa diz "contribua" mas não te paga por isso | Absolutamente não |

---

## Sabedoria Final

Open source é lindo em teoria. Em teoria, a comunidade constrói coletivamente a infraestrutura do mundo digital, compartilhando conhecimento e trabalho pelo bem comum.

Na prática, você vai passar uma tarde de sábado corrigindo um bug para um usuário que vai responder "ok" e fechar a issue sem dar obrigado, enquanto um fundo de hedge usa sua biblioteca na infraestrutura de trading deles e chama seu tempo de resposta às 3 da manhã de "muito lento."

Mas ei — pelo menos você tem um gráfico de contribuições verde.

---

*O autor mantém 0 projetos open source. Os 12 anteriores estão arquivados. Sua biblioteca mais popular tem 4.700 estrelas e um aviso "buscando novo mantenedor" desde 2022. Ninguém se voluntariou.*
