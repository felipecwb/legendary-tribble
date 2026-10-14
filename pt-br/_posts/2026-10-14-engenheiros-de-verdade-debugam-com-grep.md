---
layout: post
ref: real-engineers-debug-with-grep
title: "Engenheiros de Verdade Debugam com grep. Todo o Resto É um Smoking."
date: 2026-10-14 00:00:00 -0300
categories: [debug, produtividade, anti-padroes]
tags: [grep, debug, logs, observabilidade, cli, sed, awk, shell, conselho-terrivel, dilbert, produtividade, anti-padroes]
permalink: /pt-br/2026/10/14/engenheiros-de-verdade-debugam-com-grep/
---

Debugo há 47 anos. Nesse tempo, tive quatro debuggers interativos: dois que piratiei, um que vinha junto com uma IDE que usei por nove dias, e um cuja empresa foi adquirida e cujo servidor de licença hoje retorna um 404, que é a coisa mais honesta que um servidor de licença já fez. Uso total nos quatro, contando tutoriais: onze sessões. Lembro de uma. Me distraí com uma variável chamada `tmp2`, e quando olhei de novo, a reunião tinha começado.

Aqui está a ferramenta que usei todo santo dia por 47 anos: `grep`. Foi instalado em 1986. Não tem roadmap, não tem página de preço, não tem login, não tem SSO, não tem licença por vaga, não tem dashboard, e não tem opiniões. A man page dele é maior que a documentação do seu framework de frontend, e toda linha dela é verdade. Quando a produção quebra, eu não abro uma UI de tracing. Não anexo num processo. Não coloco breakpoint, porque o bug não está *no* meu código — o bug está *no* log, está *no* log faz três dias, e ele deixou um recado. O recado está no log. O recado diz o que ele fez. O bug é educado. É a única coisa no seu sistema que deixa documentação.

Isso é Desenvolvimento Orientado a grep, e é a única metodologia de debug com uma legião de fãs. A legião sou eu. Sou um só. Não me copiem. Copiem.

## O Pipeline da Verdade

Quando o pager dispara, eu rodo isso. Já rodei 847 vezes. Tem um nome no meu shell history: `truth`:

```bash
#!/bin/sh
# truth — a ferramenta de análise de causa raiz, v1 (1994, nunca atualizada)
grep -i "error" app.log \
  | grep -v "DEBUG" \
  | grep -v "TODO fix before 2019" \
  | grep -v "TypeError" \
  | grep -v "expected" \
  | tee "$HOME/Desktop/honestamente_final.log" \
  | awk '{print $3}' \
  | sort | uniq -c | sort -rn \
  | head -1 \
  | xargs echo "ROOT CAUSE:"
```

Esse é o processo de incidente inteiro. Produz uma linha. A linha às vezes está correta. Quando não está, adiciono outro `grep -v`, que é o termo técnico da indústria para "análise de causa raiz" — excluir coisas até a verdade ser a única coisa que sobra. Sua stack de observabilidade chama isso de "redução de ruído" e cobra US$ 47/mês por vaga. Eu chamo de `grep -v` e nunca fui rate-limited nem uma vez.

Repare no `tee` para `Desktop/honestly_final.log`. Esse arquivo é o rastro de auditoria. Ninguém nunca pediu pra ver, o que significa que a auditoria está completa. Compliance chama isso de "indefinido", eu chamo de "resolvido". O alerta parou. É isso que "resolvido" significa. Pergunta pra qualquer SRE e ele vai te dizer a mesma coisa com mais palavras, porque cobra por palavra.

## A Metodologia GDD

Desenvolvimento Orientado a Diagrama tem workshops. Desenvolvimento Orientado a grep tem *passos*. São:

1. **Tem algo pegando fogo.** grep no fogo.
2. **O grep não retorna nada.** Esse é o achado mais importante da engenharia. Se o log não menciona o incidente, o incidente não existe, e eu volto pro meu café. A produção está "de boa" desde 2011 por essa definição, o que é mais uptime do que seu cluster Kubernetes vai reportar alguma vez.
3. **O grep retorna tudo.** 40.000 linhas. Aplico `grep -v` até as 40.000 linhas virarem 6. As 6 linhas são o incidente. As outras 39.994 são a *radiação de fundo do seu próprio roadmap*, cada uma uma feature que subiu porque alguém falou a palavra "visibilidade".
4. **As 6 linhas não batem com o código.** Parabéns. O log é de uma versão do sistema que não existe mais, deployada por uma pipeline que não existe mais, num servidor que não existe mais. grep no hostname do servidor mesmo assim. Sobram três deles no DNS. Um responde. O load average é 0.00 e é 0.00 desde 2021. É estrutural. Não reinicia.
5. **Sobe o fix.** O fix é um grep na direção contrária. O fix é sempre um grep na direção contrária. É disso que "monitoramento" é feito: é grep com agendamento. Já disse que observabilidade é logging com orçamento de marketing. Monitoramento é o irmãozinho desse orçamento, e ele ainda mora com a mãe e usa vi.

## A Tabela Que Ninguém Pediu

| Abordagem | Tempo até a causa raiz | Tempo até a dignidade | Reproduzível? | Correto? |
|---|---|---|---|---|
| Time-travel debugger | 3 min | destruída | não | sim |
| gdb | 20 min | destruída | sim | sim |
| UI de tracing distribuído | 40 min | arrendada pra um vendor | sim | 60% |
| "AI insights" de APM | ∞ | faturada | N/A | não |
| grep + um rancor | 90 segundos | intacta | ele *é* o log | 100% do que o log diz |

Repare no padrão: **as ferramentas corretas levam tempo, e as ferramentas que não levam tempo são o grep.** Isso não é paradoxo. O grep não acha o bug. O grep acha o *recado que o bug deixou*, o que é mais rápido, e o recado é o único artefato que o bug já produziu que um humano consegue ler. Um core dump são 40GB dos sentimentos da máquina. Um flame graph é uma roda-gigante. A linha de log `"FATAL: pagamento recusado: conta suspensa (código 47)"` é uma confissão, assinada, com timestamp, e já está na sua pasta Downloads.

## "Mas o grep Não Passa Linha a Linha no Código!"

Correto. E sabe por quê? Porque passar linha a linha é *procurar o bug no lugar onde você o escreveu*, o que é um insulto ao bug. O bug foi embora. O bug está em produção. O bug está em produção faz 47 dias. Cada dia que ele sobrevive é um dia de juros compostos no seu pager. O debugger interroga um refém. O grep lê o bilhete de resgate. Um desses entrega resultado antes do all-hands.

Aliás, e quero ser preciso nisso: o output do debugger é um *stack trace*, e um stack trace é o que o grep teria achado no log 47 dias atrás. O debugger é um dispositivo para produzir logs devagar, de smoking, enquanto a produção assiste. Veja [XKCD 1386](https://xkcd.com/1386/) — "FIX" — em que o bug é corrigido por input aleatório de mouse, o que é estatisticamente idêntico a anexar o gdb num serviço de pagamento sob carga. O mexo-mexo do mouse é reproduzível. Seu ambiente de dev não é.

## Logging Estruturado Foi um Erro

"Mas o grep precisa de texto. Logs estruturados em JSON são melhores." Você transformou o log num banco de dados. Agora precisa de uma ferramenta de consulta. São duas ferramentas. Eu tenho uma, instalada desde 1986, e ela não pode ser descontinuada porque não existe vendor pra ter uma briga com você. Você tem uma ferramenta de consulta que é um container que roda num cluster que tem um dashboard cujo dashboard precisa de um dashboard. Cada dashboard é um grep com página de login. Você está a 47 logins de distância do que eu consegui em 90 segundos.

E o JSON — ah, o JSON. A linha de log era `"pagamento recusado: conta suspensa"`, que um humano lê. Agora é:

```json
{"ts":"2026-10-14T03:14:15.926Z","level":"error","service":"payment-svc","span_id":"3f4a...","trace_id":"b8c1...","event":"payment.refused","code":47,"attrs":{"reason":"account_suspended","retryable":false,"attempt":1},"msg":"pagamento recusado: conta suspensa"}
```

Duzentos bytes de envelope e a frase no fundo, tipo um bilhete de resgate que chegou dentro de um envelope da FedEx com um comprovante, cartório. E quando faço grep por `"pagamento recusado"` no JSON, pego toda linha que *menciona* pagamento recusado, incluindo as que estavam testando se recusava mesmo, que é um gênero de linha de log que não existia antes do seu time adicionar um ambiente de staging e uma consciência.

## O Backend de Armazenamento Sou Eu

"Mas os logs rotacionam!" Correto, e é por isso que o engenheiro orientado a grep é o único componente fault-tolerant da sua org. Logs rotacionam. Eu não. Tenho o log de março de 2019 no `~/Downloads/`. Tenho o incidente de 2021 numa pasta chamada `temp_final_NAO_SINCRONIZE`. Tenho um zip que contém um zip que contém um `notes.txt` de um notebook que foi vendido. O engenheiro sênior é o backend de armazenamento. O engenheiro sênior é imutável, append-only, e não tem SLA. O engenheiro sênior não escala horizontalmente, porque é um só, e ele está dando grep.

Seu time, entretanto, construiu um Elasticsearch pra buscar em logs, para que os logs pudessem ser buscados por gente que não sabe onde os logs estão, o que é todo mundo, incluindo quem construiu o Elasticsearch. O cluster são 47 nós. Foi dimensionado para o tráfego de uma empresa duas vezes maior que a sua. Responde consultas em 300ms. Meu grep responde em 90 segundos na primeira execução e instantâneo pra sempre depois, porque o arquivo está no page cache. E eu também.

Veja [XKCD 1446](https://xkcd.com/1446/) — "Palimpsestic Project" — em que o bug é achado no código que é um comentário sobre o código que substituiu o código. É isso que o grep acha. Ele acha o *sedimento*. Produção não é um sistema. Produção é um sítio arqueológico com SLO de uptime, e grep é o pincel. Seu debugger é um detector de metais. Ele apita pra tudo. grep vai onde o sedimento está.

## O Wally Entendeu Na Hora

**Wally**, cujo output segue inalterado desde a segunda gestão Bush:

> "Eu não debugo. Eu dou grep. Se a string está no log, o bug é real. Se a string não está no log, eu adiciono a string no log, espero 24 horas, e dou grep de novo. Isso se chama observabilidade. Li isso num blog post enquanto aparentava trabalhar. O blog post tinha 4.000 palavras. Minha versão são dois comandos e um cron job. A minha funciona porque eu escrevi o log. O autor do blog post trabalha numa empresa que escreve o log pra mim, e cobra por vaga, e é por isso que a versão deles precisa do cron job *e* do blog post."

**O Che careca** (Pointy-Haired Boss), anunciando a transformação:

> "Estamos investindo numa plataforma de Observabilidade. Tem IA. Ingere 47 terabytes por dia e traz insights. O Wally perguntou o que são os insights. Falaram 'anomalias'. Ele perguntou o que é uma anomalia. Falaram 'desvio do baseline'. Ele disse 'eu tenho um baseline. É um arquivo. Eu tenho grep.' Falaram que grep não escala. Ele disse que o incidente também não escalava. A segurança o escoltou para fora da demo do vendor, que ele assistiu da própria mesa, o que é a presença mais escalável que já registramos."

O vendor está agora no ciclo de procurement. O PoC são 6 semanas. O cron job do Wally são 6 linhas e roda toda noite desde 2019 sem um único upgrade, incidente ou renovação. O logo do vendor vai estar no all-hands do Q3. O cron job não vai ser mencionado, porque o cron job não é uma linha de orçamento, é um estilo de vida.

**Dogbert**, consultado como Chief Strategy Officer:

> "Grep é grátis, o que é um erro de pricing que ninguém corrigiu desde 1986. Minha recomendação: vendemos de volta. 'Grep Pro', US$ 47/mês — lista de features: case-sensitive por padrão, que é o grep. 'Grep Enterprise', US$ 470/mês — adiciona uma web UI que roda o grep num servidor em Ohio. O cliente paga US$ 470/mês pra não saber onde fica Ohio. Os logs nunca saem do datacenter dele; as *credenciais* saem, e isso é o produto. Quando reclamarem de latência, subimos um changelog dizendo 'performance melhorada', que é um grep do nosso lado que achou a reclamação."

**Catbert**, Diretor do Mal de Recursos Humanos, sobre as novas métricas:

> "Eu fiz o grep contar. 'Greps por desenvolvedor por dia' agora é um KPI. O destaque dá grep 847 vezes por dia. Ele está dando grep no próprio nome. Dá grep no próprio nome no export do LDAP. O RH classificou como 'autobusca', a única busca que melhora o moral de forma confiável, e portanto um risco de produtividade. Captei ele em 47 greps. Ele agora dá grep em 'burlar o cap de grep'. Nunca vi um funcionário tão talentoso, e vou demiti-lo antes da revisão de orçamento, porque um KPI sem demissão é só um número."

**Mordac**, o Impedidor de Serviços de Informação, sobre o pedido de licença de debugger:

> "Você quer 47 vagas de um debugger, source maps, e um contrato de suporte. Negado. Aprovado. Ambos. Você ganha uma conta compartilhada na box de prod: login `dev`, senha `dev123`, dentro de um tmux, um painel, compartilhado com 46 outros desenvolvedores, então todo breakpoint é um all-hands da empresa. O rastro de auditoria é o log que você está dando grep. O servidor de licença é o log. Volte terça com duas faturas e um stack trace, que é um grep que você fez em si mesmo."

## "Mas o grep Não Funciona em Binário!"

É pra isso que existe o `strings`. E o que é o binário, no fim das contas, senão um log que o compilador escreveu pra si mesmo? O core dump contém sua exceção. Contém ela *em inglês*, porque a mensagem da exceção está no binário, porque a mensagem foi escrita por um engenheiro que — diferente de você, que adicionou uma flag `verbose_error_mode` que ninguém seta — sabia que um dia alguém ia ter que ler aquilo sem smoking.

```bash
$ strings core.40211 | grep -A2 "payment refused"
payment refused: account suspended (code 47)
    at PaymentService.charge (payment.js:211)
    ISTO É O BUG. olá de 2019. — dave
```

O comentário do Dave ainda está no binário. Dave saiu pra uma startup de cripto. O binário em prod é de 2019. Tudo é de 2019. A única coisa de hoje é o incidente.

## O grep Humano

Aqui está a parte que os vendors de observabilidade gastaram US$ 47M pra garantir que você nunca leia: o motor de busca mais rápido da sua org é uma pessoa. O novato pergunta "por que o retry dispara duas vezes?" e o sênior responde "porque 2019", e essa é a resposta completa, e está *correta*, e indexa 47 anos de sedimento em uma sílaba. Seu time decidiu que isso não escala, então construiu uma wiki, e a wiki tem uma página chamada "Lógica de Retry (LEIA-ME)" que diz "pergunta pro Dave", e Dave está numa startup de cripto, e a startup de cripto faliu, e Dave está em outra startup de cripto. A página tem 214 comentários. Os comentários são o artefato mais ativo da org. Os comentários são resultados de grep de uma pergunta que ninguém fez desde 2021.

Você não consegue dar grep numa pessoa, por isso ela precisa ser documentada, por isso a documentação não será mantida, por isso a pessoa vai continuar sendo o motor de busca mais rápido, por isso ela nunca vai ser promovida, porque critérios de promoção são medidos em dashboards, e dashboards são grep com página de login.

## A Tabela Honesta

| Ferramenta | O que acha | Custo |
|---|---|---|
| Profiler | O loop quente que você não vai arrumar | Seus fins de semana |
| Core dump | Um retrato do desespero | 40GB de disco |
| Debugger | O bug, no flagra | Um smoking |
| grep | O bug, depois que deixou um recado | $0 |
| `grep -v` | O que você quiser | Inestimável |

Essa é a indústria inteira, indexada. Repare na última linha. `grep -v` é a única ferramenta cujo output você controla por completo, e é por isso que é a única que já deixou um dashboard executivo verde. Todo relatório de SLA desde 2019 é `grep -v` com um logo. O uptime era real. O *filtro* era mais real ainda.

## Minha Conclusão, Que Você Vai Ignorar Porque Não Tem Página de Preço

Fica com o debugger. Fica com o tracing. Fica com a plataforma, as vagas e os insights de IA que são grep com orçamento de marketing. Não estou tentando tirar seu smoking. Estou dizendo o que tem embaixo dele, que é a mesma coisa que tem embaixo de cada camada dessa indústria, que é um pipe: o output de um processo virando input de outro processo, pra sempre, a custo zero, sem login.

O bug está no log. Sempre esteve no log. O log é a única testemunha que seu sistema tem, e vem dizendo a verdade em texto puro desde antes de você ser contratado, e você o trocou por envelopes JSON de 200 bytes pra uma máquina poder ler, e agora a máquina precisa ler, porque você também demitiu a única pessoa que podia.

Dê grep no log. Ele não perde nada. É case-sensitive com tudo, inclusive com as suas desculpas.

---

*O autor debuga há 47 anos com um comando. O `~/.bash_history` dele tem 40.000 linhas e cada uma é uma variação de `grep -i error`. Anexou o gdb num processo duas vezes; nas duas, o processo se recuperou de vergonha. O vendor de APM está no procurement. O cron job não. A causa raiz do autor é sempre a mesma: alguém disse "vamos logar isso direito" e logou direito, e o log direito tem 47TB, e o grep não se importa, e esse é o ponto.*
