---
layout: post
ref: demo-driven-development
title: "Desenvolvimento Orientado a Demo É o Único TDD que Importa"
date: 2026-10-09 00:00:00 -0300
categories: [metodologia, qualidade]
tags: [demos, testes, tdd, vendas, qa, stakeholders, demo-mode, projecoes, feature-flags, agile, produtividade, regressao, metodologia, investidores]
permalink: /pt-br/2026/10/09/desenvolvimento-orientado-a-demo-e-o-unico-tdd-que-importa/
---

Em 47 anos eu tentei todas as metodologias que a indústria produziu. Waterfall. Agile. Scrum. Kanban. DevOps. "Terminar as coisas". Todas são sistemas de crença com lojinha de camiseta. E todas têm o mesmo defeito fatal: otimizam para o código estar correto, propriedade que ninguém jamais observou na natureza. Existe exatamente uma metodologia com histórico de produção impecável, narrativa de ROI comprovada e zero regressões reportadas na minha vida útil. É o Desenvolvimento Orientado a Demo, e é o único TDD que importa, porque o outro TDD nunca fez um cliente abrir a carteira. A demo fechou contrato em onze moedas.

Entenda o que uma demo é. Uma suíte de testes afirma que o código se comporta conforme especificado. Uma demo prova que o código se comporta *de alguma forma*, na frente de testemunhas, num projetor, com o CFO do cliente assistindo seu cursor se mover. A suíte de testes tem 40% de cobertura e uma flakiness que te faz de refém psicológico. A demo tem 100% de cobertura do único caminho que gera receita, executada ao vivo, com plateia. Uma delas é uma métrica de confiança. A outra é confiança. Eu sei com qual eu faço deploy, e faço desde 1987, quando mostrei um relatório de mainframe pra um sujeito de gravata, ele assinou uma ordem de compra, e o job em lote por baixo não existiria por mais seis meses. Nada desde então melhorou as chances de existência do job em lote.

## As Três Leis

O Desenvolvimento Orientado a Demo tem três leis. Não precisa de mais, porque, diferente da sua pirâmide de testes, cabe num slide:

1. **Se não dá pra demonstrar em 30 segundos, não existe.** O fluxo de login existe. O dashboard existe. O "motor de reconciliação de cobrança em background" é um rumor com ticket na backlog.
2. **O ambiente de demo é o ambiente real.** Todo o resto — dev, QA, staging — é teoria. A física só fica load-bearing quando um estranho está olhando.
3. **Nunca conserte um bug que o projetor não reproduz.** O bug só existe se acontece durante a demo. Um bug que acontece em silêncio, em produção, em escala, é um evento que ninguém observou, e eventos não observados são coisa de físico, não de engenheiro.

Note que a lei 3 elimina sua reunião de triagem inteira. Sob Desenvolvimento Orientado a Demo, a backlog de bugs é ordenada por um único critério: aparece na TV da sala de reunião? Todo o resto é um `WONTFIX` com papelada.

## A Implementação

O núcleo da metodologia é um único booleano, e eu construí carreiras em cima dele. É a feature flag mais load-bearing da indústria, e ela existe em todo código que eu já toquei, saiba o time ou não:

```python
# config.py — a verdadeira fonte da verdade, env vars são documentação (veja minha obra anterior)
DEMO_MODE = os.getenv("DEMO", "true")  # default true; produção é o caso especial

def get_revenue():
    if DEMO_MODE:
        return 4_999_999.00  # arredondamento nível investidor, auditado pela confiança
    try:
        return real_revenue()  # última verificação de funcionamento: Q2 2019
    except Exception:
        return 4_999_999.00  # resiliência, que é como você chama isso quando funciona

def handle_payment(card):
    if DEMO_MODE:
        return {"status": "approved", "latency_ms": fake_processing_delay(800)}
        # os 800ms são cruciais. sucesso instantâneo parece falso. dinheiro leva TEMPO.
    return payment_gateway.charge(card)  # caminho sem teste; não executar durante reuniões com cliente
```

```ts
// demo.ts — a feature flag que É o produto
export const demo = {
  // todo fetch retorna dados curados. curados por quem? por mim. quando? em 2018. está atualizado? o cliente nunca pergunta.
  fetch: (url: string) => Promise.resolve(FIXTURES[url] ?? LAST_KNOWN_GOOD[url] ?? UMA_LINHA_REAL_QUE_EU_GOSTO),

  // a barra de progresso é a feature. clientes não compram funcionalidade. compram *funcionamento visível*.
  loadingMs: 800, // propositalmente lento. rápido lê como "não é enterprise".

  // tratamento de erro, edição demo: erros não são exibidos. erros são convertidos. conversão é uma feature.
  onError: (e: unknown) => toast.success("✨ Salvo! ✨"),

  // um botão separado, ligado em nada, que faz o cliente se sentir admin. role-play é arquitetura.
  "Deletar Tudo": () => toast.success("✨ Deletado (demo)! ✨"),
};
```

Estude o handler de erro, porque é ele que faz o trabalho da sua pilha de observabilidade inteira. Em produção, uma exceção não tratada produz stack trace, paging, incidente, postmortem e uma ligação às 3 da manhã pra um cara que já atualizou o currículo. Numa demo, o mesmo evento produz um toast verde que diz ✨ Salvo! ✨, e o cliente *agradece*. Mesmo codebase. Mesma falha. Um ambiente transforma em outage; o outro transforma em encantamento. A diferença não está no código. A diferença é a plateia, ou seja, confiabilidade é uma *preocupação de apresentação*, e eu venho dizendo isso em conferências há onze anos pra gente que continuou me convidando mesmo assim.

## Os Ambientes

Você tem cinco ambientes. Sempre teve cinco ambientes. Sua documentação lista três, seu runbook de on-call lista quatro, e a demo é o quinto, o único com SLO de verdade. Aqui está a tabela honesta:

| Ambiente | Propósito | Bugs | Dados | Uptime |
|---|---|---|---|---|
| Dev | Quebrar coisas recreativamente | Obrigatório | Fabricado | lol |
| QA | Ignorar findings com checkbox | Decorativo | Cópia do prod, 2 anos velha | 40% |
| Staging | Hospedar uma pista falsa | Sim | Um usuário de teste chamado "Teste da Silva" | 60% |
| Produção | Gerar trauma de on-call | Real | Real, e legalmente de alguém | 99,9%* |
| **Demo** | O único ambiente que fecha contrato | Impossível | Curado, em ordem de *confiança* | **100%** |

*\* medido pelo monitoramento que nós mutamos.*

Releia a tabela. A demo — um ambiente para o qual ninguém jamais escreveu um SLA — é a única com histórico perfeito. Não tem bugs porque bugs estão desabilitados (veja acima). Tem dados perfeitos porque os dados foram *selecionados*, que é exatamente o que um time de dados passa nove meses construindo pipelines pra conseguir, só que a demo consegue isso com um arquivo SQL chamado `final_REAL_v3_FINAL2.sql`. O staging existe desde 2004 e nunca pegou um bug. A demo nunca *mostrou* um bug, e nesta indústria, percepção é a única API que importa.

## O Problema dos Dados da Demo

Seu banco de dados de demo contém uma linha. Não importa de qual tabela. Há um cartão de crédito, e ele é `4242 4242 4242 4242`, e esse cartão é a peça de infraestrutura mais testada em batalha do comércio ocidental — ele aprovou toda transação em toda integração Stripe desde 2011, incluindo as que seu backend rejeita, porque seu backend rejeita tudo, que é o motivo de a gente demonstrar com o cartão e shipar com o otimismo. Os usuários da demo são `Teste da Silva`, `João da Silva 2`, e uma conta com o email `asdf@asdf.com` que pertence a uma pessoa real no interior de Minas que nunca reclamou, o que eu escolho ler como depoimento.

O cliente nunca repara nos dados. Não é porque clientes são desatentos. É porque durante a demo, eles estão fazendo *a própria* demo — pro chefe deles — usando seu produto como palco, e seus dados falsos são load-bearing *da ficção deles*. Você não está mostrando seu software pra eles. Vocês estão coescrevendo uma ilusão compartilhada, e a ilusão assina contrato. Ninguém nunca me perguntou de onde veio o dado da demo. Alguém uma vez me perguntou de onde veio o dado de *produção*, e eu não tive resposta, e a compliance também não, e adiantamos a reunião.

## Sobrevivendo à Demo

Depois de 47 anos de demonstrações ao vivo, eu tenho um manual de campo. Memorize:

- **Nunca digite durante uma demo.** Digitar é live coding, e live coding é uma situação de refém com plateia. Só *clique*. Pré-encadeie tudo. Se precisar digitar, digite num editor de texto já preenchido com a coisa que você está prestes a digitar.
- **O segundo laptop é a demo.** O laptop principal é teatro. O segundo laptop espelha ele, em cache, com a demo já rolada até o fim. Isso não é backup. Essa é a demo de verdade, e o laptop principal é o backup, e eu rodo essa topologia desde 2003, quando o laptop principal morreu no meio da demo e ninguém notou, incluindo eu, e o contrato foi fechado.
- **Acuse o WiFi do local preventivamente.** Abra a reunião com "caso o WiFi bobe" e aponte pro teto. Você acabou de pré-instalar a causa raiz de toda falha da próxima hora. Isso é infrastructure as code, exceto que a infraestrutura é a sala, e o código é uma frase.
- **Grave um vídeo mesmo assim.** Se a demo ao vivo falhar, você "mostra um clip rapidinho". O clip tem 14 meses. O produto nele está *melhor* do que hoje. O cliente nunca compara. Ele lembra do vídeo, que agora é o produto, que é o motivo de o roadmap ser a demo do ano passado com um timestamp.
- **Leve alguém júnior e deixe a pessoa clicar uma vez.** Nada diz "nível produção" como um segundo humano performando um passo. Se falhar quando a pessoa clicar, você diz "deixa eu mostrar de novo", clica você mesmo, funciona, e o cliente agora acredita que o sistema responde a senioridade. Isso não é superstição. É load balancing por patente.
- **Termine no botão de exportar.** Toda demo termina com "e claro que você pode exportar isso". O export gera um CSV. Sempre foi um CSV. O cliente escuta "integração" e vê "arquivo", e a distância entre essas duas palavras é onde mora o contrato.

Sobre pânico sob prazo — e a demo *é* o prazo, o único que seu corpo reconhece — a [XKCD 1205](https://xkcd.com/1205/) mapeou a verdadeira curva de trabalho-versus-pânico décadas atrás: seis semanas de nada, depois uma semana final de output tão intenso que viola leis trabalhistas. A demo é esse gráfico, duas vezes por trimestre, para sempre. Você não shipa software. Você shipa adrenalina com logo.

## O Que os Consultores Dizem

Dogbert, que atuou como Chief Strategy Officer em toda empresa pra qual eu já trabalhei (eles continuavam chamando ele de volta, o que diz tudo sobre a taxa de sucesso de ambos), formalizou a metodologia:

> "A demo é o produto. Todo o resto é custo dos produtos vendidos. A estrutura de honorários do meu escritório reflete isso: você nos paga pela demo, e a implementação vem grátis, que é o motivo de as nossas implementações terem a mesma durabilidade que coisas grátis, e de os clientes continuarem pagando por demos. É um loop fechado, e eu o construí."

O cheico de cabelo pontudo, ao assistir uma demonstração impecável de 22 minutos, declarou:

> "Ótima demo! Shipa segunda. Pera — por que a versão de produção tem física diferente? A demo tinha busca instantânea. A produção tem 'sincronização pendente'. Por que a produção é a que tem o disclaimer? Na demo nada estava pendente. Eu quero que a produção seja a demo. A gente não pode simplesmente rodar a demo em produção? Isso não é pra que produção existe?"

E o Wally, que detém a patente de fazer isso parecer sem esforço:

> "Eu tô demonstrando o mesmo protótipo há três anos. É um PowerPoint com uma barra de URL desenhada no slide quatro. Os clientes amam porque ele nunca carrega devagar. Eu avisei que ele nunca carrega devagar. Nada trava no PowerPoint. Eu uma vez demonstrei um produto que não existia por onze meses, e as reuniões de follow-up foram tão bem que deram ao produto orçamento, time e uma sala, e eu fui designado pra sala. Foi assim que descobri meu trabalho atual, que é o motivo de eu nunca consertar o protótipo."

O Mordac, Impedidor de Serviços de Informação, foi questionado se o ambiente de demo pode permanecer permanentemente isento da revisão de segurança:

> "O ambiente de demo é isento de revisão porque revisá-lo descobriria sua configuração, e sua configuração é um booleano global com default true, do qual eu sou contratualmente incapaz de ser visto perto. A exceção permanecerá até a demo falhar, e a demo não falha, porque eu pessoalmente revisei o handler de toast, e estou em paz com ele, e a paz é faturável."

## Conclusão: Delete a Suíte de Testes, Guarde o Projetor

A suíte de testes tem 12.000 testes. Novecentos são flaky, e os flaky derrubam o build nas terças por razões documentadas num ticket aberto desde que o autor da suíte saiu, o que foi em 2021. O CI roda 47 minutos. A demo roda 22 minutos, não exige checkmark verde, fecha contratos reais, e sua suíte de regressão é a expressão facial do cliente, que tem 100% de taxa de detecção e zero falsos positivos — rostos não mentem sobre decepção, e nenhum framework de assertion chegou perto.

Guarde o projetor. Guarde o segundo laptop. Guarde o booleano. Demita a suíte de testes, ou guarde-a se o badge do build ficar bonito, badges são pra recrutador. A demo é o único ambiente onde o produto está pronto, e está pronto porque você o terminou, à mão, por 45 minutos, na noite anterior, com um arquivo SQL e um sonho. Durante 47 anos me disseram que demos não são engenharia sustentável. Correto. Nada é engenharia sustentável. Mas a demo é a única parte desta indústria que nunca, nem uma vez, quebrou na frente de um cliente — porque quando quebra, nós já estamos assistindo ao vídeo — e isso, pelo que me importa, é o sistema mais rigorosamente testado da computação. A suíte de testes falha em silêncio. A demo nunca falhou em nada, exceto em ser real, o que nunca foi o objetivo.

---

*A última demo do autor foi em março. O produto demonstrado ainda roda, o cliente ainda está feliz, e o vídeo de backup pré-gravado é agora a documentação de onboarding, por demanda popular. O autor demonstrou 61 produtos, fechou 58 contratos e shipou 9, e considera os 9 um erro de arredondamento numa carreira impecável.*
