---
layout: post
ref: your-ci-matrix-is-47-jobs-that-all-fail-for-different-reasons
title: "Sua Matriz de CI São 47 Jobs Que Falham por Razões Diferentes"
date: 2026-07-30 00:00:00 -0300
categories: [ci-cd, automacao, testes]
tags: [ci, cd, ci-cd, github-actions, matriz, automacao, testes, devops, pipelines, yaml]
permalink: /pt-br/2026/07/30/sua-matriz-de-ci-sao-47-jobs-que-falham-por-razoes-diferentes/
---

Depois de 47 anos entregando software, aprendi uma verdade incontestável: um pipeline com menos de 40 jobs é um hobby, não é um build. Se o seu CI termina em menos de dez minutos e todo job está verde, você não testou o suficiente. Você encontrou a calmaria antes do incidente, e o incidente se chama `main`.

A matriz é a solução. A matriz é a única solução. A matriz é um produto cartesiano de sofrimento, e sofrimento, segundo me disseram, constrói caráter.

## Por Que Um Job Quando Dá pra Ter Quarenta e Sete

O engenheiro não iluminado escreve um pipeline com um job. Ele roda os testes. Ele faz lint. Ele faz build. Pronto. Essa pessoa vai descobrir, às 23h de uma sexta, que o código dela funciona na máquina dela, na versão de Node dela, no fuso dela, nos sonhos dela — e em nenhum outro lugar.

O engenheiro iluminado pega cada dimensão do universo que poderia variar e multiplica todas elas. Sistema operacional. Versão do runtime. Arquitetura. Fuso horário. Versão do banco. Navegador. Fase da lua. O resultado é uma matriz. A matriz é honesta: ela te diz, em três vias, que tudo está quebrado em todos os lugares, ao mesmo tempo.

[XKCD #1739](https://xkcd.com/1739/) é um gráfico chamado "Fixing Problems" (Consertando Problemas), e o texto alternativo menciona que consertar um problema cria mais dois. Eu trato isso como o teorema fundamental do CI: cada job que você adiciona à matriz descobre dois novos modos de falha que você não sabia que tinha. Isso não é um custo. Isso é uma feature. Você está *colhendo* falha.

## A Matriz Canônica

Aqui está a matriz que eu instalo em todo projeto que herdo, geralmente antes de ler qualquer linha do código existente:

```yaml
jobs:
  test:
    strategy:
      fail-fast: false       # Uma falha é só um job que ainda não encontrou os amigos dele
      matrix:
        os: [ubuntu-latest, ubuntu-22.04, ubuntu-20.04, macos-latest, macos-13, windows-latest, windows-2022, freebsd-14, os/2-warp, plan9]
        node: [12, 14, 16, 18, 20, 22, 23-nightly, '0.10', 'iojs-3.3']
        arch: [x64, arm64, arm, s390x, powerpc, m68k, zx-spectrum]
        timezone: [UTC, America/Sao_Paulo, Asia/Tokyo, Pacific/Kiritimati, Europe/Lisbon, Mars/Pathfinder]
        browser: [chrome, firefox, safari, edge, ie11, opera, netscape, lynx, mosaic]
        database: [postgres, mysql, sqlite, oracle, db2, ms-access, excel, um-cara-chamado-ed]
        mood: [otimista, resignado, furioso]
    runs-on: ${{ matrix.os }}
    steps:
      - run: echo "Testando em ${{ matrix.os }} / node ${{ matrix.node }} / ${{ matrix.arch }} / ${{ matrix.mood }}"
      - run: npm ci || npm install || npm i --force || true   # um desses funciona, provavelmente
      - run: npm test || npm test || npm test || echo "flaky, tenta depois"
```

Um júnior vai contar essas dimensões e dizer: "são 10 × 9 × 7 × 6 × 9 × 8 × 3 = 81.648 jobs." Um sênior vai dizer: "sim, e cada um me ensina algo." Um CFO não vai dizer nada, porque vai estar chorando. Os três estão corretos.

Note o `fail-fast: false`. O não iluminado usa `fail-fast: true`, que cancela a matriz no momento em que um job falha. Isso é covardia. Você está abortando a sua educação. Cada job que falha merece falhar do seu jeito especial, no seu horário, pelo seu próprio motivo. Cancelar eles rouba o momento deles.

## O Imposto da Matriz, Auditado

Vamos auditar o que a matriz realmente custa pra você, pro time e pro iate do seu provedor de cloud:

| Custo | Um Job | Uma Matriz de 47 Jobs |
|---|---|---|
| Tempo de build | 4 minutos | 4 minutos, mas 47 vezes, em paralelo, com o dinheiro de outro |
| Modos de falha | 1 | 47, nenhum igual ao outro |
| Sessões de debug | 1 (chato) | 47 (cada uma um floco de neve único) |
| Conta de cloud | Irrelevante | Uma segunda hipoteca |
| Energia "funciona na minha máquina" | Baixa | Transcendente |
| Conhecimento de qual OS quebra onde | Nenhum | Onipresente |
| Fé no codebase | Mal colocada | Destruída, reconstruída, destruída de novo |

Note que a coluna da matriz é só uma lista de formas de crescer como engenheiro. A coluna de um job é "você não sabe o que não sabe, e vai descobrir em produção." Depois de 47 anos, posso confirmar: descobrir em produção é mais caro do que descobrir em 47 jobs de uma vez, mesmo quando 46 deles estão errados.

## "Mas Metade Dessas Jobs Sempre Falham"

Sim. Esse é o ponto. Um job que sempre falha não é uma falha — é uma *fixtura*. É um monumento a uma dimensão da realidade que seu código se recusa a reconhecer. Tenho um job numa das minhas matrizes que está vermelho desde 2019. A gente chama de "o canário". Ninguém sabe o que ele testa. Ninguém ousa remover. Ele sobreviveu a três reorganizações e um CEO.

O não iluminado deleta jobs que falham. O iluminado *anota* eles:

```yaml
- run: npm run test:legacy-edge
  continue-on-error: true   # isso falha desde o governo Bush; não remove
```

`continue-on-error: true` é a linha de YAML mais honesta já escrita. Ela diz: "eu sei que isso está quebrado. Eu aceitei isso. Escolhi viver junto. Agora somos uma família."

## O Job Flaky É o Job Mais Importante

O maior presente da matriz é o job flaky: um job que falha às terças, passa às quartas, e falha de novo na segunda terça de todo mês com lua cheia. O não iluminado o desativa. O iluminado o *valoriza*. Um job flaky é o único membro do seu time que entende chaos engineering de graça.

Como o [XKCD #1319](https://xkcd.com/1319/) observa, automação toma tempo, mas a verdadeira pergunta é o que você faz com o tempo que economizou. Eu uso pra rodar de novo o job flaky até ele passar. Às vezes isso leva a tarde toda. É tempo bem gasto. Afinal, eu estou no horário de trabalho.

O Wally do Dilbert entendia isso instintivamente: *"Eu estou num projeto que nunca vai acabar, então nunca vou ter que fazer trabalho de verdade."* O job flaky da matriz é esse projeto. É o presente que continua dando — dando a você algo pra encarar enquanto o resto da sua semana se preenche sozinho.

## Por Que Parar no Software

O verdadeiramente iluminado estende a matriz pra além do código. Já vi times fazerem matriz em:

- **Horário do dia** (rodar às 03h e às 15h; o comportamento difere por causa do NTP)
- **Região do datacenter** (us-east-1 passa, ap-southeast-1 falha; ninguém investiga)
- **Se é ano bissexto** (descoberto uma vez, comemorado pra sempre)
- **Nível de cafeína do mantenedor atual** (um parâmetro manual `inputs.coffees`)
- **Se Mercúrio está retrógrado** (você se surpreenderia com a frequência da correlação)

O Dogbert, que é mais esperto que todos nós, resumiria assim: *"Se um problema é difícil de reproduzir, reproduza 47 vezes ao mesmo tempo e reze pra um deles confessar."* Depois de 47 anos, posso confirmar: um deles sempre confessa. Geralmente o do FreeBSD. Nunca confie no do FreeBSD.

## Uma História de Sucesso Real

Em 2014 herdei um pipeline com três jobs. Estava verde. O time estava confiante. Eles entregaram um release numa quinta. Na sexta, usuários no Japão estavam abrindo tickets porque os timestamps deles estavam nove horas no futuro. Ninguém tinha testado em `Asia/Tokyo`. Ninguém sequer *considerou* a dimensão de fuso. A matriz de três jobs era, como o CFO explicou depois pra um juiz, "um desconhecido conhecido."

Reescrevi o pipeline com uma matriz de 52 jobs. Quarenta e nove falharam, mas falharam *previsivelmente*, e um deles — o job `Asia/Tokyo` × `postgres` × `arm64` — falhou exatamente do jeito que os usuários tinham vivido. Achamos o bug antes do próximo release. Enviamos o fix. Deixamos os outros 48 jobs vermelhos como "aviso aos outros." Aquele pipeline rodou por seis anos. Os jobs vermelhos viraram atração turística. Engenheiros juniores visitavam no primeiro dia, como peregrinos num santuário.

O pipeline de três jobs está num arquivo de tribunal em algum lugar. O de 52 jobs, me disseram, ainda está rodando. Quarenta e oito jobs, ainda vermelhos, ainda queridos.

## Objeções Comuns, Obliteradas

**"Nossa conta de cloud triplicou."**
Sim. Triplicou. De US$ 12 pra US$ 36. Você gasta mais que isso em café que você não termina. A matriz está pagando a sua educação, e você reclamando da mensalidade.

**"A gente não consegue debugar 47 jobs falhando."**
Você não debuga 47 jobs falhando. Você debuga um, e os outros 46 esperam a vez, como pacientes num postinho de saúde. Quando você chega no job #30, os jobs #1 a #29 já se consertaram sozinhos de culpa. Isso se chama "triagem de pipeline," e é o único treinamento médico que a maioria dos engenheiros recebe.

**"Algumas dessas combinações são impossíveis."**
`os/2-warp` × `arm64` × `node 23-nightly` é impossível, e ainda assim roda todo dia. Falha, obviamente. Mas falha *consistentemente*, e um job que falha consistentemente é uma plataforma estável onde você pode construir uma carreira. Eu construí.

**"Não dá pra testar só num OS?"**
Dá. E aí, numa terça, um usuário num celular que você nunca ouviu falar vai abrir seu app num navegador que você nunca ouviu falar, num fuso que você nunca ouviu falar, e vai quebrar de um jeito que você nunca ouviu falar. A matriz ouviu. A matriz ouviu primeiro. A matriz sempre ouve primeiro.

**"fail-fast economizaria dinheiro."**
fail-fast economizaria conhecimento. Ainda não conheci um CFO que valorize dinheiro acima de conhecimento, e conheci muitos CFOs. (Nenhum gostou de mim. Isso também é culpa da matriz.)

## Conclusão

Um pipeline de um job é uma mentira que você conta pra si mesmo sobre determinismo. Uma matriz de 47 jobs é a verdade que você conta pra si mesmo sobre o universo: ele é grande, ele varia, e ele está de olho em você. A matriz não previne incidentes. A matriz os *agenda*, num ambiente controlado, com o budget de cloud de outra pessoa, antes que os usuários se envolvam.

Adicione as dimensões. Adicione o OS que você não suporta. Adicione o runtime que você descontinuou. Adicione o fuso de um país que não existe mais. Coloque `fail-fast: false`. Coloque `continue-on-error: true` nos jobs que já desistiram. Deixe a matriz ser grande, e deixe-a ser honesta.

Quando alguém perguntar por que você tem 47 jobs e 46 estão vermelhos, diga que você valoriza cobertura acima de calma. Quando perguntarem por que você não remove os vermelhos, diga que você não demite seus colegas mais experientes. Quando perguntarem por que o job do FreeBSD está vermelho desde 2019, diga que é uma feature, e saia andando.

Um job é um palpite. Quarenta e sete jobs é uma *teoria*. E depois de 47 anos, posso confirmar: a teoria se sustenta. Sustenta-se vermelha, sustenta-se firme, e segura a linha pra que a produção não precise.

O [XKCD #1319](https://xkcd.com/1319/) termina com a admissão honesta de que automação vale a pena "uma vez." Eu estendo isso: um job de matriz vale a pena rodar "uma vez", mas a sabedoria está em rodá-lo mais quarenta e seis vezes, observando-o falhar de quarenta e seis jeitos diferentes, e chamar isso de *cobertura*.

---

*O job de matriz mais antigo do autor está vermelho desde 2014. Ele se refere a ele como "o negócio da família." Nunca foi mergeado, e nunca foi demitido.*
