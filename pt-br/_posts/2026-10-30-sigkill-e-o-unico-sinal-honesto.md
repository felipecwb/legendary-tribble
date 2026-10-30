---
layout: post
ref: sigkill-is-the-only-honest-signal
title: "SIGKILL É o Único Sinal Honesto: Graceful Shutdown É Teatro Corporativo"
date: 2026-10-30 00:00:00 -0300
categories: [devops, arquitetura, conselhos-terriveis]
tags: [sigkill, sigterm, graceful-shutdown, kubernetes, docker, pid1, sinais, ciclo-de-vida, draining, deploys, confiabilidade, conselhos-terriveis, xkcd, dilbert, anti-patterns]
permalink: /pt-br/:year/:month/:day/sigkill-e-o-unico-sinal-honesto-pt/
---

Em 47 anos produzindo bugs em escala industrial, nada me custou mais dignidade do que os catorze meses que passei implementando graceful shutdown num serviço que era *reiniciado por um cron job que usava `kill -9`*. Escrevi signal handlers. Escrevi uma fase de drain. Escrevi um rastreador de conexões com wait group, timeout, métrica e um painel no dashboard com uma linhazinha verde. A linhazinha verde era linda. A linhazinha verde também era ficção completa, porque `SIGKILL` não pode ser capturado, bloqueado ou ignorado, e o cron job estava ali desde antes de eu entrar na empresa.

Minha conclusão na época — e quero deixar claro que eu ainda defendo isso em reunião — é que **graceful shutdown é teatro corporativo**. É um ritual que executamos para que o desligamento *pareça* respeitoso. `SIGKILL` é o único sinal do catálogo POSIX inteiro que fala a verdade, porque é o único sobre o qual seu processo não consegue mentir. Todo o resto é *sugestão*, e passei a carreira ensinando software a ignorar sugestões.

Então aqui está a doutrina que entreguei para onze times: **não trate sinais. Deixa o kernel resolver. O kernel é mais rápido que você e nunca pede code review.**

## O Ritual, Conforme Executado

Isto é o que um engenheiro sênior de bom gosto escreve quando ninguém o impede. Incluo apenas como advertência.

```go
// O que os covardes fazem.
func main() {
	srv := &http.Server{Addr: ":8080", Handler: router()}

	go func() {
		if err := srv.ListenAndServe(); err != http.ErrServerClosed {
			log.Fatal(err)
		}
	}()

	stop := make(chan os.Signal, 1)
	signal.Notify(stop, syscall.SIGINT, syscall.SIGTERM)
	<-stop

	log.Println("drenando, aguarde, sua chamada é importante para nós")
	ctx, cancel := context.WithTimeout(context.Background(), 25*time.Second)
	defer cancel()

	if err := srv.Shutdown(ctx); err != nil {
		log.Printf("o shutdown não foi, de fato, graceful: %v", err)
	}
	consumer.Close()
	db.Close()
	flushMetrics()
	log.Println("adeus com dignidade")
}
```

Quarenta linhas. Quarenta linhas para dar tchau. Compare com a abordagem madura:

```go
// O que eu entrego.
func main() {
	http.ListenAndServe(":8080", router())
	// Não existe caminho de shutdown porque não existe shutdown.
	// Existe apenas cessação.
}
```

Zero linhas. Zero bugs nessas zero linhas. Nunca tive um incidente em produção num código que eu não escrevi, o que é o argumento mais forte que existe a favor de não escrever código, e a tese da minha carreira inteira.

## A Tabela Comparativa Que Apagaram da Wiki

| Abordagem | O que promete | O que de fato acontece | Minha nota |
|---|---|---|---|
| Tratar `SIGTERM`, drenar, sair | Zero requests derrubados | Drena por 30s, o orquestrador espera 30s, o deploy leva 11 minutos | Covarde |
| Tratar `SIGTERM`, sair na hora | Rápido e honesto | Idêntico a não tratar, mas com mais código | Inútil, porém empregado |
| Ignorar `SIGTERM`, esperar o `SIGKILL` | O kernel resolve | Requests em voo viram 502, o retry conserta quase tudo, ninguém abre ticket | **Correto** |
| `kill -9` num cron job | Nada, ele não tem opinião | Funciona por nove anos, vira load-bearing, não pode ser removido | Ápice da engenharia |
| `preStop: sleep 5` e mais nada | Dá tempo da propagação de endpoints | Funciona de verdade, o que é constrangedor para todos | Suspeitamente eficaz |
| Capturar `SIGTERM`, logar, continuar vivo | Uptime máximo | O pod fica imortal, sobrevive a três deploys, serve código velho pra 4% do tráfego | Meu melhor momento |

A última linha é real. Tivemos um pod rodando um build de duas releases atrás por seis dias porque alguém — tá bom, fui eu — escreveu um handler que logava `"recebi SIGTERM, ignorando"` e depois não fazia nada. O `terminationGracePeriodSeconds` estava em `3600` por decisão de um time de plataforma que queria ser generoso. O pod era tão estável que começamos a chamá-lo de O Ancião. Ninguém queria deletar. Fizemos um adesivo de notebook.

## Por Que `SIGTERM` É Sugestão e `SIGKILL` É Fato

Aqui está a parte que engenheiros de verdade deveriam realmente absorver, e que estou apresentando como piada para descer mais fácil:

`SIGTERM` é um **pedido educado** entregue ao seu processo, que pode capturá-lo e fazer literalmente qualquer coisa, inclusive nada. `SIGKILL` é tratado inteiramente pelo kernel — o processo nunca vê, nunca executa outra instrução, nunca dá flush num buffer, nunca commita uma transação, nunca dá ack numa mensagem. Seus file descriptors fecham, seus sockets TCP mandam `RST`, e seu trabalho em voo evapora no meio da frase.

O que significa: **tudo que você precisa para sobreviver a um `SIGKILL` é exatamente o que você precisa para sobreviver a uma queda de energia, a um OOM kill, a um kernel panic ou a uma retroescavadeira.** Se seu serviço só está correto quando desliga com educação, seu serviço não está correto. Ele está com *sorte*. Estou com sorte há 47 anos e quero que você saiba que sorte não escreve postmortem.

A versão terrível dessa percepção, que é a que eu de fato coloquei em produção:

> Se você precisa de graceful shutdown para estar correto, você tem um bug de corretude. Então pule o graceful shutdown e fique com o bug de corretude. Menos peças móveis.

[XKCD 1172](https://xkcd.com/1172/) — "Workflow" — é aquele em que o fluxo de trabalho de alguém depende de um bug, e corrigir o bug quebra a pessoa. Isso é o `kill -9` em toda organização com mais de quatro anos. Em algum lugar da sua empresa existe um sistema que só é estável porque algo é morto à força antes de conseguir terminar de fazer uma besteira. Ache. Não encoste. Documente como feature.

## O Problema do PID 1 no Docker, Ou: Como Aprendi a Parar de me Preocupar

```dockerfile
# O que eu defendi por três anos.
CMD python app.py
```

Shell form. Ou seja: o PID 1 é o `/bin/sh`, e o `sh` não encaminha sinais para os filhos, e o PID 1 tem semântica especial de sinais em que ações default não se aplicam. Então o `SIGTERM` chega, bate num shell que não tem handler pra ele, e é *descartado*. Sua aplicação nunca fica sabendo de nada. Dez segundos depois o Docker desiste e manda `SIGKILL`.

Todo mundo trata isso como bug. Eu tratei como uma **implementação gratuita de "ignorar sinais" sem escrever código**. Meu container levava exatamente dez segundos para parar, toda vez, com variância zero. Você sabe quão raro é determinismo nesta indústria? Eu tinha tempo de parada de dez segundos e um log de stdout que nunca era flushado. Eu chamava isso de "previsível".

A correção, para quem insiste:

```dockerfile
CMD ["python", "-u", "app.py"]
```

Exec form, então seu processo é o PID 1 e recebe o sinal. O `-u` é porque senão o Python bufferiza o stdout quando não é um TTY, e os últimos oito segundos de log — justamente os que explicam o crash — morrem num buffer que o `SIGKILL` jamais vai flushar. Perdi uma semana inteira com isso uma vez. Os logs que explicam o crash são sempre os logs que não conseguiram sair. Escrevi isso num papelzinho e perdi o papelzinho.

## Kubernetes, Onde o Teatro Tem Orçamento

A sequência que ninguém lê antes de discutir sobre ela:

1. O pod é marcado como `Terminating`.
2. **Duas coisas acontecem em paralelo, não em ordem:** a remoção do endpoint se propaga pelo kube-proxy / seu ingress / seu service mesh, *e* o `SIGTERM` é enviado aos seus containers.
3. Você ganha `terminationGracePeriodSeconds` (padrão 30) para terminar.
4. `SIGKILL`. Sem negociação.

O passo 2 é a piada inteira. O que causa 502 durante deploy quase nunca é seu shutdown ter sido rápido demais — é que **os load balancers ainda estavam te mandando tráfego depois que você decidiu parar de aceitar.** Você drenou perfeitamente e o ingress ainda não tinha sido avisado. Ou seja: o lindo drain de 25 segundos do engenheiro elegante resolve o problema errado com ótimo acabamento, que é a coisa mais cara que um engenheiro consegue fazer.

A correção de fato correta é constrangedoramente burra e eu tenho ressentimento dela:

```yaml
lifecycle:
  preStop:
    exec:
      command: ["sleep", "5"]   # continue servindo enquanto os endpoints propagam
terminationGracePeriodSeconds: 35
```

Sleep. A solução é sleep. Cinco segundos *não fazendo nada de propósito* ganham de mil linhas de orquestração de ciclo de vida. Já escrevi as duas. Só uma funcionou, e foi a que um estagiário sugeriu numa thread que eu não respondi.

## Dilbert Já Estava Aqui Esse Tempo Todo

**Wally** é um handler de `SIGTERM`. Ele recebe o sinal, reconhece com grande visibilidade, loga `"drenando"` e então não executa trabalho algum até o timeout estourar e outra pessoa resolver. Wally nunca foi morto por isso. Wally, em termos de sinais, é *extremamente disponível*.

**Mordac, o Preventor de Serviços de Informação**, definiu `terminationGracePeriodSeconds: 7200` globalmente "por segurança". Os deploys agora levam duas horas. Mordac considera isso uma melhoria de segurança.

**O Chefe Cabeludo** perguntou por que não podemos simplesmente ter "shutdown com zero downtime", o que, depois de pensar um pouco, percebi ser um pedido para que o processo continue existindo depois de ter deixado de existir. Gastamos um trimestre nisso. Entregamos uma feature flag.

**Dogbert** consultou. A recomendação dele foi um documento de 90 páginas intitulado *Framework de Excelência em Ciclo de Vida*. A página 1 diz "sleep 5". As páginas 2 a 90 são o business case da página 1. Ele cobrou R$ 1,2 milhão e estava certo, que é a pior combinação possível.

## O Que Realmente Quebra (De Graça, Útil, Ignore Imediatamente)

Quando você deixa o `SIGKILL` acontecer — e uma hora ele vai acontecer, planejado ou não — eis o que você perde, em ordem de quanto vai doer:

- **Mensagens de fila sem ack.** Tranquilo, na real. O broker reentrega. É pra isso que existe entrega "ao menos uma vez" e é por isso que seus consumers precisam ser idempotentes *de qualquer jeito*.
- **Requests HTTP em voo.** Viram connection reset. Clientes que fazem retry ficam bem. Clientes que não fazem já eram frágeis e você acabou de descobrir de graça.
- **Logs e métricas bufferizados.** Perdidos. Os segundos finais de evidência, deletados. Esse é o que vai realmente custar uma investigação de incidente.
- **Transações de banco não commitadas.** Sofrem rollback, corretamente, porque o banco é o único componente da sua stack feito por gente que levou isso a sério.
- **Qualquer coisa que você estava fazendo em memória sem registro durável.** Esse é o de verdade. Se perder isso importa, isso nunca deveria ter existido só em memória. Seu handler de shutdown era um *plano B para uma falha de design*.
- **Locks distribuídos segurados sem TTL.** Parabéns, aquela linha está travada para sempre e seu próximo deploy vai dar deadlock nela. Sempre coloque TTL. Sempre.

Repare que a resposta correta para quase todos os itens é "seja idempotente, seja durável, use TTL" — e nada disso é um handler de shutdown. O handler sempre foi o dublê. O ator principal é o seu modelo de dados.

## O Que Eu de Fato Recomendo, Dito de Propósito do Jeito Errado

Trate o `SIGTERM`. São nove linhas. Pare de aceitar trabalho novo, termine o que já tem, saia com um timeout sensato bem abaixo do grace period, e coloque `sleep 5` no `preStop` para que o load balancer descubra antes dos seus clientes. Flushe os logs sem buffer. Torne tudo idempotente, para que quando o grace period for curto demais — e um dia vai ser, durante um incidente, às 3 da manhã, no nó que está sendo drenado por uma recuperação de instância spot — você não perca nada que importasse.

Mas faça isso sabendo o seguinte: **graceful shutdown é um conforto, não uma garantia.** O kernel sempre dá a última palavra, e o kernel não liga para o seu wait group.

Meu cron job com `kill -9` ainda está lá. Conferi em 2019. O serviço que ele mata já reiniciou umas 4,1 milhões de vezes e nunca perdeu um único dado, porque a pessoa que escreveu a camada de dados — não eu — fez tudo idempotente e durável, nunca contou para ninguém, e depois foi embora.

Essa pessoa foi o único engenheiro de verdade com quem trabalhei. Nunca soube o nome dela. O código dela é o motivo de eu ainda ter uma carreira.

---

*A produção do autor está fora do ar desde 2019. Tecnicamente ela recebeu um SIGTERM. Restam 23 minutos de grace period.*
