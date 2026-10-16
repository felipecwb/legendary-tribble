---
layout: post
ref: printers-are-the-only-distributed-system
title: "A Impressora do Escritório É o Único Sistema Distribuído que Sempre Funcionou"
date: 2026-10-16 00:00:00 -0300
categories: [sistemas-distribuidos, hardware, anti-padroes]
tags: [impressoras, sistemas-distribuidos, spooler, consistencia, observabilidade, firmware, drivers, conselho-terrivel, dilbert, anti-padroes]
permalink: /pt-br/2026/10/16/a-impressora-e-o-unico-sistema-distribuido-que-funciona/
---

Tenho 47 anos de experiência, e nesse tempo trabalhei com sistemas distribuídos com Raft, Paxos, quorum de escrita, protocolos de consenso, relógios vetoriais e um cluster Kafka que custou mais que o prédio em que estava rackeado. Tenho um único achado para reportar, e é o único achado de que já tive certeza na vida: **a impressora do escritório é o único sistema distribuído que de fato funcionou.** Não "confiável" — confiável é fantasia vendida por fornecedores com deck de slides. Digo *consistente*. A impressora falha desde 1971, e falha do mesmo jeito todos os dias. Três luzes piscando. Um atolamento de papel. A palavra "Offline" num aparelho que está a três metros de você. Seu cluster Kubernetes está no ar há quatro anos e você ainda não sabe o que ele faz. A impressora não tem README e todo mundo no prédio sabe o estado dela agora. Isso não é falha de tecnologia. É o rollout de observabilidade mais bem-sucedido da história humana.

Você acha que isso é piada. Não é piada. É o único conselho de arquitetura sério que eu tenho. Modele seus microsserviços como uma impressora. No fim deste artigo você vai entender por quê, vai se recusar a fazer isso, e o pager vai me agradecer pessoalmente.

## O Spooler: A Fila de Mensagens Original

Toda conferência de sistemas distribuídos que já frequentei teve pelo menos uma palestra sobre filas de mensagens, proferida por um sujeito que nunca imprimiu um documento. A palestra anuncia que filas "desacoplam produtores de consumidores" e "absorvem picos" e garantem "entrega at-least-once", e a plateia assente, porque já ouviu essas palavras antes. Enquanto isso, o spooler de impressão faz tudo isso desde o governo Nixon, de graça, com uma UI de quatro checkboxes e um botão que diz "Imprimir" mesmo quando devia dizer "Desistir".

O spooler tem entrega at-least-once. Tem tratamento de dead-letter: quando o documento falha ao imprimir, o spooler não te manda um 500, um stack trace e um link para runbook. Ele simplesmente deixa o documento na fila, em silêncio, para sempre, onde envelhece como um bom vinho até alguém desligar a impressora da tomada às 18h, o único garbage collection que esta indústria já precisou. Seu tópico Kafka tem política de retenção. O spooler tem *esquecimento*. Quarenta e sete anos de experiência em produção me ensinaram que esquecer é a estratégia de retenção superior: retenção é só negação do passado.

E o spooler é uma fila *global* para o prédio inteiro. Os documentos de todos entram numa fila e saem aproximadamente na ordem de envio, ou seja, a impressora implementa FIFO com garantia de entrega best-effort, exatamente as mesmas semânticas do seu message bus, só que a versão da impressora cabe numa tela de 1997 e o SLA dela é "provavelmente sai antes da reunião".

## O Atolamento de Papel: Lock Global Único, Perfeitamente Aplicado

Aqui a impressora envergonha seu protocolo de consenso. Quando a impressora atolam, o *sistema inteiro para*. Não um shard. Não uma partição. Não um serviço com retry que silenciosamente duplica a fatura. **Tudo para, e para todo mundo no mesmo instante, e todo mundo fica sabendo no mesmo instante.** Isso é linearizabilidade. É o que o Raft tenta te vender há onze anos. A impressora entrega isso numa caixa de plástico de US$ 89 que faz beep.

Seu banco distribuído alega "consistência forte" e, sob partição de rede, felizmente serve leituras obsoletas para o time de vendas enquanto o financeiro escreve do outro lado do split brain, e depois o script de reconciliação roda às 2 da manhã e produz um relatório que uma pessoa lê e descreve como "estranho, mas ok". A impressora nunca teve split brain. Ninguém nunca olhou para a bandeja de saída e a bandeja de papel e discordou de quantas cópias existem. Quando a impressora diz que fez 47 cópias, foram feitas 47 cópias, porque a alternativa é uma luz piscando e uma alavanca na lateral que exige intervenção humana. Essa alavanca é o mecanismo de consistência mais forte já construído: converte um problema de consenso distribuído numa disputa física entre uma pessoa e uma folha de papel, e a pessoa sempre perde, por isso o ledger está sempre correto.

A galera de microsserviços vai te dizer que lock global único não escala. A impressora escala. A impressora escalou para exatamente a carga que tem, que é a carga correta, e nunca vai crescer, o que faz dela o único sistema do prédio com plano de capacidade.

## Três Luzes: O Único Stack de Observabilidade que Alguém Já Leu

Seu stack de observabilidade: 4.000 dashboards, uma UI de tracing, service mesh com golden signals, SLO para tudo, error budgets, estrutura de pastas no Grafana com convenção de nomes definida num offsite, e regras de alerta revisadas por comitê. Leituras por dia: o engenheiro de on-call, uma vez, quando o pager dispara, se disparar, o que não acontece, porque os thresholds foram ajustados em 2022 para serem educados.

O stack de observabilidade da impressora: **três luzes.** Verde: funcionando. Amarela: pensando no assunto. Vermelha: estado da união. Qualquer humano da organização lê esse stack sem onboarding, sem treinamento, sem documentação. O CEO lê. Visitantes leem. Meu cachorro leria, e meu cachorro não tem noção de consenso distribuído, que é outra forma de dizer que ele entende de sistemas melhor que seu time de SRE.

A impressora também tem mensagens de erro, e são notoriamente ruins — "PC LOAD LETTER", frase que confunde funcionários desde 1984 e que eu ainda não sei explicar, e não preciso, porque veja meu trabalho anterior: mensagens de erro criptográficas são feature de segurança. Mas eis o que seus fornecedores nunca vão admitir: a mensagem de erro da impressora é *precisa*. Quando ela diz que tem atolamento de papel, tem atolamento de papel. O atolamento tem localização física. Você pode ir até lá. Pode tocar. Pode tirar o papel. Compare com seu stack de observabilidade, que detecta "p99 elevado no serviço de checkout" — preciso até onde vai, acionável até onde vai a lua. Sua UI de tracing não consegue te dizer *onde está o atolamento de papel*, porque seu sistema não tem atolamento de papel, tem *lentidões misteriosas que se resolvem sozinhas antes do on-call abrir o dashboard*, que é a mesma coisa que atolamento de papel, só que o atolamento é honesto quanto a ser papel.

Veja [XKCD 927](https://xkcd.com/927/) — "Standards" — onde um novo padrão é proposto para resolver o problema de quatorze padrões competindo, resultando em quinze padrões competindo. A impressora é anterior a esse problema. A impressora tem exatamente um padrão, e é uma luz que está acesa ou apagada, e ela derrotou todo fornecedor de observabilidade desde o CRT.

## Atualização de Firmware: O Único Canário Honesto

Seu deploy canário: 5% do tráfego por quatro horas, comparação de métricas, rollback automático em regressão, um dashboard, um runbook, um canal no Slack chamado `#deploy-watch` onde ninguém digita. O canário da impressora: ela percebe que há atualização, aplica nela mesma, às 18h, numa sexta-feira, com todo mundo olhando, e ou funciona ou se bricka de um jeito tão completo que a única solução é um downgrade de firmware feito com um pendrive e um clipe de papel, procedimento documentado num PDF que o fornecedor deletou e que só sobreviveu porque um tal de Gerardo imprimiu em 2019.

Isso é disciplina de canário *mais forte* do que a sua. Seu canário faz rollback em regressão. A impressora não faz rollback. Ela **comita**. Comita com tanta convicção que o modo de falha é físico e permanente, e essa é a única estratégia de deploy que manteve alguém honesto: quando rollback é impossível, release engineering vira religião, e a impressora é o sistema mais religioso que já operei. Enviei software por 47 anos e nunca senti o que sinto quando a barra de progresso da impressora diz "Atualizando. Não desligue." Desligar não é opção. Não tem flag. Não tem config. Existe um pedido da máquina, e a máquina nunca esteve disposta a negociar, o que é a definição de SLA.

## Drivers: A Única História Multiplataforma que Deu Certo

Sua estratégia multiplataforma: containers, para "escreva uma vez, rode em qualquer lugar", exceto que roda diferente em Apple Silicon, seu Makefile assume GNU sed, e o script de setup funciona em toda máquina menos nas máquinas que as pessoas têm. Onboarding de um engenheiro novo leva dois dias, e os dois dias são gastos instalando ferramentas que falham por motivos diferentes dos motivos listados no README.

A história multiplataforma da impressora: um driver. Tem 140 MB, foi escrito em 2011, não é assinado, e funciona em todo sistema operacional já lançado, incluindo sistemas descontinuados antes de o autor do driver terminar de escrevê-lo. O driver sobreviveu a quatro versões do macOS, três regimes do Windows e uma distro Linux cujo nome a empresa pediu para não mencionar. Quando o driver quebra, você reinstala. Quando isso falha, você desliga e liga a impressora. Essa é a árvore de troubleshooting inteira. Tem dois galhos. A sua tem 847, e 845 são links para páginas do Confluence escritas por gente que não trabalha mais aqui.

Compare com seus microsserviços, que cada um embarca sua própria biblioteca de cliente, gerada de um spec OpenAPI quatro releases atrás do serviço, e todos discordam sobre semântica de timeout. A impressora tem um protocolo, chama-se PCL, e é falado por toda impressora feita desde 1984 num dialeto tão estável que documentos impressos em 1987 imprimem corretamente hoje, em hardware que não existia quando o documento foi escrito. Isso é *compatibilidade com versões anteriores*, a coisa que sua API REST promete desde 2015 e nunca entregou. A impressora vem cumprindo essa promessa desde antes de seu framework ter nome. Alguns frameworks já tiveram três nomes.

## A Tabela que Ninguém Pediu

| Propriedade | Impressora do Escritório | Seu Cluster Kubernetes |
|---|---|---|
| Uptime | Falha todo dia, na mesma hora | Nunca falhou, oficialmente |
| Consistência | Perfeita: um lock global de papel | Eventual, quase sempre, menos terças |
| Observabilidade | Três luzes, lidas por todos | 4.000 dashboards, lidos por um engenheiro triste |
| Mensagens de erro | "PC LOAD LETTER" (precisa) | "Error: something went wrong" (lírica) |
| Semântica de retry | Recusa até tirar da tomada, depois esquece | Tenta 3x, falha, tenta 9x, falha, tenta para sempre |
| Onboarding | Dez minutos; o botão é verde | Duas semanas; o botão é `kubectl` |
| Compatibilidade | Desde 1984 | Desde a última breaking change |
| Planejamento de capacidade | A carga exata que tem | O que diz a conta da cloud |
| Cadência de deploy | Uma vez, brickou, clipe de papel | 47 vezes desde terça |

Repare qual coluna parece um sistema em que você confiaria sua folha de pagamento. A linha de uptime da impressora é a única da tabela que nunca mentiu para ninguém. "Nunca falhou, oficialmente" está fazendo muito trabalho nessa célula, e a palavra "oficialmente" é estrutural, o que não se pode dizer da maioria dos seus serviços.

## O Diretor-Chefe de Impressão

Trabalhei com todos os personagens daquela tirinha, e todos têm opinião sobre a infraestrutura de impressão, que é a única infraestrutura sobre a qual alguém tem opinião, porque é a única que se pode ver.

**O Chefe Cabelo Pontudo**, anunciando a iniciativa de modernização:

> "Boa notícia. Vamos mover as impressoras para a nuvem. O fornecedor chama de Impressão-como-Serviço. De agora em diante, quando você imprimir, o documento viaja até um data center na Virgínia, volta e imprime na máquina em que você está em pé. Chama-se ida e volta. É muito moderno. O fornecedor garante que é o mesmo número de passos de antes, só que agora tem um login, e o login é SSO, e o SSO redireciona para uma página que diz 'Sua sessão expirou'. Vi uma demo. Na demo, funcionou."

**Wally**, sobre por que mantém um pedaço de papel colado sobre a porta de rede da impressora:

> "A impressora funcionava, e aí eu imprimia coisas, e as pessoas sabiam que eu estava acordado. Agora a impressora está 'offline', oficialmente, desde 2019, e o TI tem um ticket aberto, e o ticket está atribuído a alguém que saiu da empresa, o que significa que a impressora ficará offline para sempre, o que significa que posso imprimir quando quiser, pela porta USB, na minha mesa, em paz. O ticket é o software mais confiável que temos. Nunca fecha."

**Dogbert**, consultado como Diretor de Estratégia:

> "Seu problema é que a impressora funciona bem demais. Qualquer um imprime qualquer coisa. Recomendo impressão tarifada: cada funcionário recebe 47 folhas por trimestre e, ao esgotar, deve submeter um formulário a um comitê que se reúne trimestralmente. Já implementei isso em outras empresas. A produtividade caiu 40% e o moral caiu 47%, mas a economia foi de 3 centavos, que é a única métrica que a liderança acompanha, então o projeto foi um sucesso e me pediram para replicá-lo."

**Catbert**, Diretora Má do Recursos Humanos, sobre cotas de impressão:

> "Adoro o sistema de cotas. Permite negar pedidos por escrito, o que cria um rastro de papel, que exige impressão, que consome cota, o que gera mais pedidos para negar. O sistema é autossustentável e não precisa de pessoal. Recomendei ao conselho como framework de governança. Aprovaram por unanimidade, porque era o único item da pauta que não exigia imprimir nada."

**Mordac**, o Impedidor dos Serviços de Informação, sobre a política de impressão (esse é o cargo dele canonicamente, e ele é muito bom):

> "Pedidos de impressão agora devem ser submetidos em triplicata. As cópias da triplicata devem ser impressas, o que exige acesso à impressora, o que exige um pedido de impressão, que deve ser submetido em triplicata. O acesso será concedido após aprovação do Comitê de Acesso à Impressão, que se reúne sempre que a impressora está operacional. A impressora está operacional. O Comitê não. Traga seu próprio papel."

## "Mas a Nuvem!"

Eu sei o que diz o deck do fornecedor. Diz que impressão é workflow legado, que documentos devem morar na nuvem, que papel está morto, que o futuro é um PDF que abre numa aba do navegador com um visualizador que tem um botão e ele é o errado. Ouço esse pitch a cada cinco anos desde 1990, e a cada cinco anos o pitch é entregue em papel, apostila impressa, porque o fornecedor sabe de algo que não põe no deck: **o material de reunião impresso é o único sistema distribuído com 100% de disponibilidade de leitura.** Ninguém nunca recebeu 404 num handout. O handout funciona durante partição de rede. Funciona com a VPN caída. Funciona quando o provedor de SSO tem outage, que é como você descobre que o provedor de SSO está na mesma página que a rede, e a página é essa que está fora do ar.

Papel é a arquitetura offline-first original, e offline-first é a tendência que a indústria redescobre a cada década com nome novo: PWA, local-first, edge cache, service worker. O documento de papel tem tudo isso: é cache, é imutável, é versionado com um carimbo de data no rodapé, e tem histórico de revisão que consiste em alguém escrever "v2" nele. Seu sistema de gestão documental tem histórico de versões e eu nunca vi ninguém recuperar a v2 de propósito. O papel está à sua frente, e o papel vai sobreviver ao seu SaaS. Seu SaaS será renomeado para "Classic" e desativado. Papel nunca foi renomeado. Papel nunca foi desativado. Papel nunca teve página de preços.

## Minha Conclusão, que Não Vai Sobreviver à Reunião

Modele seus sistemas distribuídos como a impressora. Uma fila global. Um lock global, aplicado fisicamente. Três luzes, lidas por todos, sem treinamento. Mensagens de erro que apontam para a folha de papel errada de verdade. Atualizações de firmware que commitam. Uma história de compatibilidade medida em décadas. E um estado — "Offline" — que todo humano da organização conhece sem dashboard.

Você não vai fazer isso, porque a impressora não está no seu currículo. Ninguém nunca colocou "operei uma impressora" no currículo, e até isso mudar, seu cluster Kafka vai continuar fazendo o que o Kafka faz, que é existir, custando, esperando uma carga que chega uma vez por trimestre, momento em que vai produzir uma outage descrita no postmortem como "sucesso inesperado".

A impressora roda desde 1971. Seu stack roda desde terça. Um desses sistemas tem atolamento de papel, e o outro tem diploma de filosofia.

---

*O autor tem 47 anos de experiência e nunca consertou um atolamento de papel consultando um dashboard. Conserta abrindo a porta, puxando o papel e fechando a porta, o que leva onze segundos e tem taxa de sucesso de 100%, superior a qualquer SLO que já aceitou assinar. Carrega um pendrive com drivers de impressora no chaveiro, ao lado da pilha do chaveiro que o chaveiro lhe vendeu em 2014. A solução do chaveiro nunca falhou. Os drivers nunca foram necessários. Ele leva os dois para toda reunião de incident review, por respeito.*
