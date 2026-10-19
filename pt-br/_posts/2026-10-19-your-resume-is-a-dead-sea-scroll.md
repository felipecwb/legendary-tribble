---
layout: post
ref: your-resume-is-a-dead-sea-scroll
title: "Seu Currículo É um Pergaminho do Mar Morto: Ninguém Lê, Eles Carbonizam"
date: 2026-10-19 00:00:00 -0300
categories: [carreira, recrutamento, anti-padroes]
tags: [resume, curriculo, recrutamento, ats, linkedin, hiring, ia, triagem, conselho-terrivel, dilbert, cultura, anti-padroes]
permalink: /pt-br/2026/10/19/your-resume-is-a-dead-sea-scroll/
---

Em 47 anos produzindo bugs em massa, sobrevivi à remediação do Y2K, à bolha da internet, quatro aquisições, dois reorgs por trimestre fiscal e a um recrutador que me ligou sobre uma vaga que eu tinha me candidatado, numa empresa onde eu trabalhava, na cadeira em que eu estava sentado. E estou aqui pra te contar o segredo que todo o complexo industrial de recrutamento gastou US$ 47M pra esconder: **ninguém nunca leu seu currículo.** Seu currículo não é um documento. Seu currículo é um *artefato*. Ele é exumado, limpo, escaneado por uma máquina que foi configurada por alguém que saiu em 2016, e carbonizado. O recrutador não é um leitor. O recrutador é um arqueólogo. O ATS não é um filtro. O ATS é um detector de metal que apita pra paixão.

Isso é Arqueologia Orientada a Currículo, e é a única ciência que a sua indústria pratica. Todo o resto é vibe com PDF.

## A Metodologia Completa

Quando uma empresa "avalia candidatos", roda isso. Já estive dentro de 847 pipelines de contratação. São sempre as mesmas 40 linhas, e a decisão de contratação é sempre tomada antes de o PDF abrir:

```yaml
# ats_config.yaml — motor de avaliação de candidatos, v1
# (2016, inalterado desde 2016; a lista de keywords foi gerada a partir de
#  uma job description escrita em 2011 por um PM que saiu em 2012. O PM
#  hoje é sommelier. As keywords permanecem.)

weights:
  years_of_experience: 47      # valor máximo detectável, tipo internet discada
  buzzword_density: 3.7        # por frase. medido, tipo chuva
  bullet_points: true          # bullets são estruturais
  first_person_pronouns: -8    # falar em primeira pessoa é red flag

hard_filters:
  - degree: required           # o diploma é verificado. as skills não são.
  - keyword: "Kubernetes"      # aparece 4.700 vezes no corpus, nunca em prod
  - keyword: "passion"         # incomensurável, logo, ranqueada
  - employment_gap: fail       # gaps indicam instabilidade ou luto.
                               # ambos são passivo.

scoring:
  - +10 para toda ferramenta que usamos hoje
  - +10 para toda ferramenta que abandonamos em 2019 e readotamos em 2024
    (chamada de "tecnologia madura" nas duas vezes)
  - -100 por perguntar "em que o time trabalha de verdade"
  - -100 por ler a job description antes da entrevista,
    porque revela que o emprego é real, e realidade está fora de escopo

feedback:
  template: "Decidimos seguir com outros candidatos."
  meaning: "Uma pessoa leu a linha 47 do seu PDF e sentiu alguma coisa."
  deliver_by: nunca            # o ghosting é o produto
```

Esse é o campo inteiro do recrutamento técnico. Um passe de scoring, uma lista de keywords, um output, e o output foi determinado pelo bloco `weights`, que foi ajustado por um humano que desde então se mudou pra Napa. A máquina que tria humanos foi configurada por um fantasma, calibrada em currículos de um stack tecnológico que não existe mais, e mandada pra produção sem plano de rollback — o que faz dela o único sistema da sua empresa com 100% de uptime, porque ela não pode ser desligada, só abandonada, como os candidatos.

## A Tabela Que Ninguém Pediu

| Elemento do currículo | O que o ATS vê | O que um humano veria | Custo |
|---|---|---|---|
| 12 anos de PostgreSQL | keyword "postgres" ×3, OK | uma pessoa que carregou um pager por um banco de dados | seus fins de semana |
| Liderei um time de 4 | keyword "led", +1 | você foi promovido e depois teve que fazer reuniões | a carreira de 4 pessoas |
| "Apaixonado por tecnologia" | buzzword, +0,7 | nada. é atmosférico. é névoa. | dignidade, ×3 |
| Gap de emprego, 2020 | hard filter: fail | um evento global que aconteceu com você | o ano inteiro de 2020 |
| Currículo de duas páginas | 47% de scroll depth | alguém que fez coisas | página 2 |
| Currículo de uma página | 100% de scroll depth | alguém que fez uma coisa | página 1 |

Repare no padrão: **a máquina recompensa o artefato, e o artefato não é a pessoa.** Um currículo e um humano se diferenciam por exatamente um entregável: o humano produz software, e o software tem um changelog. O changelog é o que ninguém pediu. Ninguém nunca abriu a página 2 de um currículo, porque a página 2 é onde moram as descrições honestas de projeto, e as descrições honestas de projeto são as que dizem "eu mantive isso por três anos e era majoritariamente ok, o que é chato, e chato não pontua".

## "Mas o MEU Currículo É Lido!"

Correto. E quero ser preciso sobre *o que* é lido, porque nunca é a parte em que você passou o fim de semana. Aqui está a transcrição anotada do scan de 6 segundos do recrutador médio, recuperada do único artefato mais preciso que o currículo — o heat map de eye-tracking:

```diff
+ Jane Doe — Engenheira de Software Sênior     ← lido. 1,2s
+ 15 anos de experiência                        ← lido. 0,9s
+ Skills: Kubernetes, Docker, Terraform…       ← escaneado. 0,4s
- Liderei migração reduzindo p99 em 40%         ← pulado. "números podem ser inventados"
- Construí o sistema de billing da empresa      ← pulado. "claims não são verificáveis"
- Dei 14 palestras em conferências              ← pulado. "falar não é engenharia"
+ Comunicativo, detalhista, jogador de equipe  ← LIDO. 2,1s. "great culture fit."
```

Seis segundos, e 2,1 deles gastos na única frase do documento escrita por um template. As 47 horas que você passou quantificando seu impacto — as métricas, o antes/depois, o p99 — foram pontuadas por um olho na mesma velocidade que a máquina pontua, o que significa: nem pontuou, mas com cara. E o toque final: o recrutador então encaminha o currículo pro hiring manager com o resumo "Ótimo fit cultural. Tem Kubernetes." Seu PDF de 47 linhas foi reduzido a nove palavras. Nove palavras, uma das quais é seu nome, e uma das quais você nunca usou de verdade.

O hiring manager lê as nove palavras. O hiring manager entrevista você por 47 minutos sobre as nove palavras. Te perguntam de Kubernetes, que está no seu currículo porque em 2019 o ATS penalizava a ausência, e você adicionou pelo mesmo motivo pelo qual as pessoas põem "referências disponíveis sob solicitação": porque o documento recompensa o que o documento recompensa, e ninguém no processo é a pessoa que o documento descreve.

## Como o Artefato É Fabricado

Aqui está a linha do tempo de toda contratação técnica da história:

1. **Segunda.** A vaga abre. O título é "Ninja Rockstar 10x Full-Stack Unicorn". Ninguém consegue defini-lo. Foi escrito em 2011, copiado de uma req que era cópia de outra req, e a cadeia de custódia vai mais fundo que o produto real da empresa.
2. **Terça.** 847 candidaturas chegam. O ATS passa 47. 800 humanos recebem um e-mail template contendo a palavra "impressed", que foi gerada automaticamente pela mesma pipeline que os rejeitou, o que a torna a primeira mentira do processo a alcançar automação.
3. **Quarta.** O phone screen do recrutador. O roteiro é um PDF de 2016. O recrutador pergunta "me conte sua trajetória" e se silencia por 47 minutos, não por grosseria, mas porque já ouviu a resposta 8.470 vezes e a resposta é sempre a mesma resposta.
4. **Quinta.** A entrevista em painel. Quatro engenheiros participam, cada um carregando 47 tickets atrasados, e a entrevista acontece nos intervalos entre as notificações dos notebooks deles. O candidato é convidado a inverter uma árvore binária. O candidato não inverte uma árvore binária desde a última entrevista nesta mesma empresa.
5. **Sexta.** A oferta vai pro candidato com mais buzzwords e menor gap de emprego. O engenheiro mais forte da pipeline foi filtrado na segunda-feira pelo check de diploma. Ninguém vai descobrir isso. Não existe instrumento que meça a pessoa que nunca foi conhecida.
6. **Q3.** A vaga reabre. Ela já reabriu 8 vezes. A vaga não é um emprego. A vaga é uma posição de taco de beisebol que sustenta a parede — quer dizer, um título que sustenta um headcount.

O currículo não contratou ninguém. O currículo *enfeitou* a contratação. É uma coroa de flores que você pendura numa decisão tomada por uma lista de keywords, no escuro, por uma máquina que queria ir a menos reuniões sobre pessoas.

Veja [XKCD 833](https://xkcd.com/833/) — "Interview" — em que a curva de dificuldade das entrevistas sobe além do trabalho real em direção ao inferno abstrato puro. A piada tem uma sequência que ninguém menciona: o eixo Y é calibrado pra entrevistas, não pro trabalho, e o mesmo vale pro currículo. Seu currículo não é uma descrição da sua carreira. É uma descrição das suas *entrevistas*. Você otimiza o artefato errado desde 2009.

## O Wally Entendeu Na Hora

**Wally**, que está "aberto a oportunidades" no LinkedIn desde 2014 e recebe 47 mensagens de recrutador por semana e respondeu zero:

> "Atualizo meu currículo a cada quatro anos, que é a frequência com que a fonte recomendada muda. A última atualização levou uma hora e não mudou nada, que é a quantidade correta de esforço pra um documento que ninguém lê. Quando um recrutador liga, coloco no viva-voz e continuo trabalhando, que é donuts. Eles perguntam qual é meu maior ponto fraco. Digo 'sinceridade'. Meu currículo diz 'apaixonado'. As duas afirmações são igualmente falsas, e nenhuma nunca foi verificada, o que as torna peer-reviewed. Meu currículo diz que sou proficiente em Kubernetes. Nunca toquei em Kubernetes. Em 47 anos, ninguém me perguntou de Kubernetes. O currículo não é um registro do que eu fiz. É uma lista do que ninguém deu follow-up."

**O Che careca** (Pointy-Haired Boss), anunciando o plano de contratação:

> "Estamos contratando um Senior Principal Staff Fellow of Engineering. A vaga está aberta há 47 semanas. Rejeitamos 847 candidatos. O RH diz que o filtro do ATS está estrito demais. O filtro do ATS foi configurado por mim. Configurei subindo meu próprio currículo e rejeitando quem pontuasse mais. O melhor candidato que já tivemos pontuou 2/100 porque o currículo tinha uma página e estava em primeira pessoa, que nosso modelo classifica como 'desconcertante'. Contratamos o top scorer. Ele listou 'agile' nas skills. Ele nunca foi agile. Ninguém nunca foi agile. É uma keyword. A vaga segue aberta, o que é ótimo — vaga aberta é linha de orçamento, vaga preenchida é linha de folha de pagamento, e só uma das duas sobrevive ao Q3."

**Dogbert**, consultado como Chief Strategy Officer:

> "Vendo um serviço de triagem de candidatos por US$ 4.700 por contratação. Funciona assim: pego os 847 currículos deles, ordeno alfabeticamente e devolvo os primeiros 47. É estatisticamente idêntico ao produto de machine learning que vendi no ano passado, e nunca foi auditado, porque auditorias são pra empresas que planejam estar erradas de forma mensurável. Meu produto novo é powered by IA: a mesma ordenação alfabética, mas a carta de oferta usa a palavra 'signal'. 'Signal' adiciona US$ 40 ao preço. Ninguém nunca perguntou qual é o signal. O signal é alfabético."

**Catbert**, Diretor do Mal de Recursos Humanos, sobre a nova métrica:

> "Transformei 'atualizações de currículo por engenheiro por trimestre' em KPI. O destaque atualizou o currículo 47 vezes no Q1 enquanto entregava zero features. Isso é exatamente correto — o currículo é o único documento que determina onde ele vai trabalhar depois, e a empresa é meramente onde ele trabalha agora. Dei a ele o prêmio de performance. Ele então usou o prêmio no currículo pra sair. O prêmio funcionou. Funcionou no currículo. Essa é a única superfície em que ele já foi renderizado."

**Mordac**, o Impedidor de Serviços de Informação, sobre o pedido de integração do ATS:

> "Você quer o ATS integrado ao sistema de RH. Negado. Aprovado. Ambos. Você ganha uma planilha compartilhada: 847 linhas, um filtro, e uma colega que edita fórmulas toda noite. Sua pipeline vai incluir as edições dela, o que vamos chamar de 'intuição de recrutador' na documentação. A documentação também é onde vai o pedido de desculpas. Volte terça com um memo de justificativa — solto, sem espaçamento extra, pra contagem de páginas parecer um currículo."

## A Tabela Honesta

| Claim | De onde veio | Quem repete |
|---|---|---|
| "Analisamos cuidadosamente cada candidatura" | o filtro de keywords rodou | o e-mail de rejeição, em escala |
| "Ficamos impressionados com sua trajetória" | template, 2016 | o recrutador, 847 vezes esta semana |
| "Ótimo fit cultural" | 2,1 segundos na linha 3 | toda decisão de contratação, sempre |
| "Vamos manter seu currículo em arquivo" | a trituradora | ninguém — e o arquivo é um JPEG |
| "Está no meu currículo" | eu escrevi, ninguém checou | eu — e é a única linha desta tabela que sobreviveu ao contato com uma entrevista |

Repare na última linha. O sistema não-reivindicado roda por 47 anos, e toda contratação é justificada por um documento que evapora na primeira vez que alguém pede "me mostra". A indústria de recrutamento não vende avaliação. Vende *permissão pra já ter decidido*.

## Minha Conclusão, Que Você Vai Ignorar Porque Não Tem Bullets

Fica com seu currículo. Fica com as buzzwords, os bullets, o "apaixonado", as 47 certificações em ferramentas que você nunca abriu. Não estou tentando tirar seu artefato. Estou dizendo o que tem embaixo dele: um pergaminho carbonizado, escrito pra uma máquina ajustada por um fantasma, lido por um olho durante seis segundos, e reduzido a nove palavras antes de chegar a um humano.

O único documento de contratação que já importou é o que ninguém escreve: seu trabalho, na terça-feira, no pico, enquanto o entrevistador te observa debugando produção no painel que você aceitou em vez de ir ao standup deles. Ele roda toda semana. Publica no canal de incidentes. Não tem versão. Não tem keyword. O baseline é o bug que você corrigiu às 3 da manhã, e o bug não sabe qual framework ganhou.

Currículo isso. Ah — você não consegue. O ATS não suporta terças-feiras.

---

*O autor escreveu 47 currículos. Cada um foi revisado. Cada revisão disse "decidimos seguir com outros candidatos". Cada revisão levou seis segundos. O currículo mais rejeitado dele também é o mais preciso. O LinkedIn dele diz "Aberto a trabalho". O trabalho dele está aberto. O currículo dele está em arquivo desde 2019. O arquivo é um JPEG.*
