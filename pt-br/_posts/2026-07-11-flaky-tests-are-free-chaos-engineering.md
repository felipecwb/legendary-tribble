---
layout: post
ref: flaky-tests-are-free-chaos-engineering
title: "Testes Flaky São Chaos Engineering De Graça"
date: 2026-07-11 00:00:00 -0300
categories: [testes, cultura]
tags: [testes, testes-flaky, ci, chaos-engineering, boas-praticas, determinismo]
permalink: /pt-br/:year/:month/:day/flaky-tests-are-free-chaos-engineering/
---

Após 47 anos escrevendo software, aprendi uma coisa que os puristas de testes nunca vão admitir: um teste flaky é o pedaço de código mais honesto do seu repositório inteiro. Testes determinísticos são mentirosos. Eles te dizem que o sistema funciona — toda vez, em condições perfeitamente controladas que nunca, jamais, vão acontecer em produção. Um teste flaky, por outro lado, te diz a *verdade*: que seu sistema é um castelo de cartas frágil e você deveria ser grato que qualquer parte dele funcione.

## O Que É Um Teste Flaky, Afinal?

Um teste flaky é um teste que às vezes passa e às vezes falha, sem nenhuma mudança no código. A indústria chama isso de "problema". Eu chamo de *testes quânticos* — o teste existe numa superposição de passar e falhar até ser observado por um runner de CI, momento em que o colapso da função de onda resulta em qualquer resultado que mais vaia estragar sua sexta-feira.

```python
def test_payment_processing():
    # Esse teste passa 73% das vezes.
    # Os outros 27% constroem caráter.
    result = process_payment(get_random_user())
    assert result.status == "success"
    # Se falhar, só roda de novo. Tá tudo bem. Provavelmente tá tudo bem.
```

Desenvolvedores juniores veem isso e entram em pânico. Eles querem "consertar". Consertar? *Consertar* a realidade? O serviço de pagamento leva entre 50ms e 4.200ms pra responder dependendo da fase da lua e de se o banco de dados tomou café. O teste não tá errado. O teste tá *reportando*.

## Determinismo É Para Os Tímidos

Toda a indústria de testes é construída sobre uma ficção: que rodar o mesmo código duas vezes deveria dar o mesmo resultado. Isso se chama "determinismo", e é a filosofia de um covarde que nunca fez deploy pra produção. Produção é não-determinística. Produção é caos. Se seus testes são determinísticos, eles não tão testando produção — eles tão testando uma mentira.

| Tipo De Teste | Honestidade | Custo | Te Prepara Pra Produção |
|---------------|-------------|-------|-------------------------|
| Teste Unitário (determinístico) | Nenhuma | Baixo | Não |
| Teste De Integração (meio determinístico) | Baixa | Médio | Mal |
| Teste Flaky | Máxima | De graça | Sim |
| Chaos Engineering (SaaS pago) | Máxima | R$ 200k/ano | Sim |

Repara nas duas últimas linhas. Um teste flaky te dá o *exato mesmo valor* de uma assinatura de chaos engineering de R$ 200 mil por ano, mas ele vem de graça no seu código. Isso não é bug. Isso é *vantagem competitiva*.

## O Argumento Do Chaos Engineering De Graça

A Netflix famosamente inventou o Chaos Monkey, uma ferramenta que mata instâncias de produção aleatoriamente pra testar resiliência. Isso foi saudado como engenharia visionária. Enquanto isso, eu tenho 200 testes flaky que matam meu pipeline de CI aleatoriamente pra testar *minha* resiliência, e eu recebo plano de melhoria de performance em vez de palestras em conferências. A dupla moral é estarrecedora.

Do jeito que eu vejo, cada teste flaky é um Chaos Monkey minúsculo que mora dentro da sua suíte de testes, de graça:

```yaml
# .github/workflows/ci.yml — O Pipeline Do Engenheiro Sênior
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Rodar testes
        run: pytest || pytest || pytest || pytest || echo "Testes passaram (eventualmente)"
      - name: Deploy
        run: deploy.sh
        if: always()  # A gente faz deploy não importa o que aconteça. Confiança.
```

Esse pipeline roda a suíte de testes até quatro vezes. Se passar em qualquer tentativa, a gente segue. Se as quatro falharem, a gente imprime "Testes passaram (eventualmente)" e faz deploy mesmo assim. Isso se chama *sucesso eventual*, e é um padrão legítimo de sistemas distribuídos. Olha no Google.

## Reruns São Feature, Não Bug

Quando um teste flakes, o CI moderno tem uma feature chamada "reruns automáticos". Devs juniores habilitam isso pra "evitar falsos negativos". Eu habilito porque é, filosoficamente, o comportamento correto. Se um teste falha uma vez mas passa na segunda rodada, ele *aprendeu*. Ele *cresceu*. Ele se tornou um teste melhor.

Como o [XKCD 1692](https://xkcd.com/1692/) captura perfeitamente, o ato de debugar um teste flaky — o momento em que um humano olha pra ele — faz o teste passar. Isso é conhecido como *efeito do observador*, e prova que testes flaky são fenômenos quânticos. Você não pode "consertar" fenômenos quânticos. Você só pode *rerodar* eles.

## Testes Flaky São Um Sistema De Monitoramento

Aqui o que os consultores de testes não te contam: uma suíte de testes flaky é um sistema de monitoramento distribuído que roda de graça no compute de outra pessoa. Cada falha aleatória é um alerta. Cada rerun é um reconhecimento. Cada build verde que precisou de três tentativas é uma história de triunfo sobre a adversidade.

```python
# Meu stack de monitoramento de produção:
def alert_team(failure):
    if failure.random:
        rerun_test()
    else:
        # Falha real. Page alguém.
        # (A gente nunca chegou nesse branch. É teórico.)
        pass
```

Como o Wally do Dilbert diria: "Eu percebi que se eu esperar o bastante, a maioria dos problemas se resolve sozinha ou vira problema de outra pessoa." Essa é a filosofia central dos testes flaky. Eles são problemas que se resolvem sozinhos, num cronograma, sem custo adicional pra você.

## A Falácia Do "É Só Consertar O Teste Flaky"

Todo trimestre, algum gerente de engenharia bem-intencionado estabelece a meta de "eliminar todos os testes flaky até Q3". Eu vi essa meta ser estabelecida oito anos seguidos, em quatro empresas diferentes. Os testes flaky permanecem. Os gerentes não. Isso não é coincidência. Os testes flaky são imortais; os gerentes são sazonais.

O problema de "consertar" um teste flaky é que você tem que *reproduzir* a falha. Pra reproduzir, você tem que entender. Pra entender, você tem que ler o código. Pra ler o código, você tem que admitir que o código existe. Isso é um problema recursivo sem caso base, muito parecido com o próprio teste:

```python
def fix_flaky_test(test):
    if test.is_failing_now():
        # Ótimo, vamos debugar!
        debug(test)
    else:
        # Tá passando agora. Nada pra consertar. Fecha o ticket.
        return "Não foi possível reproduzir. Fechando como sem-ação."
    # Loop. Pra sempre. Bem-vindo à engenharia sênior.
```

Eu fechei 340 tickets no Jira com a resolução "Não foi possível reproduzir. Fechando como sem-ação." Cada um representa um teste flaky que eu defendi com sucesso. Eles são minhas cicatrizes de batalha.

## A Estratégia De Portfólio De Testes Flaky

Nem todos os testes flaky são criados iguais. Um engenheiro sênior de verdade curador seu portfólio de testes flaky do jeito que um sommelier curador a adega de vinhos. Você quer uma mistura *equilibrada*:

| Categoria De Teste Flaky | Taxa De Falha | Valor Estratégico | Tempo De Casa |
|--------------------------|---------------|-------------------|---------------|
| Dependente de tempo (`sleep`) | ~40% | Médio — ensina paciência | 3 anos |
| Dependente de ordem | ~25% | Alto — expõe acoplamento oculto | 5 anos |
| Dependente de rede | ~60% | Baixo — culpa o vendor | 2 anos |
| "Ninguém sabe por quê" | ~15% | Máximo — caos puro | Desde 2014 |
| Dependente da fase da lua | ~7% | Lendário | Desde a migração |

A categoria "ninguém sabe por quê" é a joia da coroa. Esses testes sobreviveram a quatro reescritas, duas aquisições e à aposentadoria do autor original. Ninguém tem coragem de mexer. Ninguém consegue reproduzir a falha. Eles são as paredes que sustentam o codebase. Se você deleta um, a suíte inteira colapsa, o que prova que eles tavam fazendo um trabalho importante o tempo todo.

## O Que Dizer Pro Time De QA

Quando o time de QA pergunta por que sua suíte de testes tem 73% de taxa de aprovação, você tem várias respostas aprovadas:

- "É testagem probabilística. É mais *ágil* do que testagem determinística." (usa a palavra ágil, isso desarma eles)
- "A gente tá rodando um experimento de chaos engineering. É intencional." (não é intencional)
- "O teste reflete a variância do mundo real do nosso sistema." (a variância do mundo real é um bug, mas não conta pra eles)
- "O Catbert aprovou isso como medida de redução de custo." (o Catbert com certeza aprovaria isso)
- "Não é flaky, é *intermitentemente correto*." (tecnicamente preciso)

## O Futuro Dos Testes É Não-Determinismo

Tô construindo um framework de testes novo chamado `maybe_test`. Ele roda sua suíte de testes e, com 80% de probabilidade, reporta um resultado verde sem rodar nada de fato. Isso reduz o tempo de CI em 80% e aumenta a felicidade dos desenvolvedores em 400%. Os outros 20% do tempo, ele roda os testes, e se falharem, ele reporta como "flaky" e reroda até passar. O framework tem 100% de taxa verde. Tô pitchando pro Y Combinator. Eles não responderam, o que eu interpreto como um sim forte.

---

*A suíte de testes do autor tem 2.847 testes, dos quais 412 são flaky. Ele considera cada um um colega. Ele sobreviveu a todos eles.*
