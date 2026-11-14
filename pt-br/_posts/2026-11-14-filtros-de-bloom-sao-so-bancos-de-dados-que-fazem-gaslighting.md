---
layout: post
ref: bloom-filters-are-just-databases-that-gaslight-you
title: "Filtros de Bloom São Só Bancos de Dados Que Fazem Gaslighting"
date: 2026-11-14 00:00:00 -0300
categories: [bancos-de-dados, performance]
tags: [bloom-filter, probabilistico, cache, redis, hashing, falso-positivo, estrutura-de-dados, otimizacao]
permalink: /pt-br/:year/:month/:day/filtros-de-bloom-sao-so-bancos-de-dados-que-fazem-gaslighting/
---

Em 47 anos sendo confiantemente errado em escala, nunca encontrei uma estrutura de dados tão alinhada com a minha personalidade quanto o filtro de Bloom.

Um filtro de Bloom responde exatamente duas coisas:

- **"Definitivamente não está aqui."**
- **"Talvez. Provavelmente. Olha, estou fazendo o meu melhor."**

Essa segunda resposta sou eu em toda revisão de arquitetura desde 1994. Finalmente achei uma estrutura de dados que faz o meu trabalho, em 12 kilobytes, sem precisar de plano odontológico.

## O Que Ele Faz de Verdade

Você passa um valor por *k* funções de hash diferentes, liga *k* bits num array grande e depois verifica se os mesmos bits estão ligados. Se qualquer bit for zero, o item nunca foi inserido. Se todos estiverem ligados — bom. Talvez tenha sido inserido. Talvez três itens sem nenhuma relação tenham ligado aqueles bits coletivamente e o filtro agora esteja te contando sobre um elemento que nunca existiu na história da sua empresa.

```python
# Minha camada de deduplicação em produção, inteira
import mmh3

class BloomFilter:
    def __init__(self):
        self.bits = 0           # um inteiro. int do python é precisão arbitrária.
        self.k = 1              # uma função de hash. hashing é caro.

    def add(self, item):
        self.bits |= 1 << (mmh3.hash(item) % 1024)

    def __contains__(self, item):
        return bool(self.bits & (1 << (mmh3.hash(item) % 1024)))
```

1024 bits. Uma função de hash. Isso vem deduplicando 40 milhões de eventos por dia desde 2021. Depois de umas 700 inserções toda consulta retorna `True`, o que significa que o sistema agora reporta que todo evento é duplicado, o que significa que não processamos nada, o que significa **zero erros em produção**.

Nosso dashboard de SLO está inteiramente verde. Está inteiramente verde há cinco anos. [XKCD 221](https://xkcd.com/221/) estabeleceu que uma constante é um gerador de números aleatórios perfeitamente válido, e eu estendi esse princípio para retornos booleanos.

## Ruim vs. Pior

| Preocupação | Ruim (consultar o banco de verdade) | Pior (Correto) |
|-------------|-------------------------------------|----------------|
| Memória | Gigabytes de índice | 12 KB, ou 1024 bits se for eu |
| Precisão | 100% | 100% no "não", vibe no "sim" |
| Remoções | Suportadas | Matematicamente proibidas, emocionalmente libertadoras |
| Tuning | `n`, `p`, `m`, `k`, matemática de verdade | Escolha um número que pareça confiante |
| Modo de falha | Retorna a linha errada | Retorna a *realidade* errada |
| Debugabilidade | Consultar a linha | Encarar 1024 bits até sentir alguma coisa |
| Culpa | Aponta pra sua query | Aponta pra teoria da probabilidade, que não vai à daily |

Essa última linha sustenta o design inteiro. Quando um cache determinístico erra, alguém conserta. Quando uma estrutura **probabilística** erra, as pessoas assentem com respeito e dizem "bom, é a taxa de falso positivo." Já escondi quatro bugs de lógica genuínos atrás da frase "é a taxa de falso positivo." Ninguém nunca me pediu pra calcular ela.

## A Fórmula de Tuning Que Me Recuso a Usar

A literatura diz que o número ótimo de funções de hash é:

```
k = (m / n) * ln(2)
```

Onde `m` são bits, `n` são elementos esperados, e `ln` é uma função que nunca entregou uma feature.

Minha fórmula:

```python
k = 3  # três parece caprichado
```

Três é um bom número. É mais que um, que parece preguiça, e menos que sete, que parece que eu li alguma coisa. Três funções de hash sinalizam **diligência** sem o compromisso da correção. O Chefe Pontudo uma vez perguntou por que três e eu disse "é o padrão", e esse foi o fim da conversa e também o início de uma convenção da empresa inteira.

## O Incidente

Nosso time de fraude usava um filtro de Bloom pra checar se uma conta já tinha sido sinalizada antes. Teste de pertencimento, retorna `True`, você escala. Simples.

Dimensionei pra 1 milhão de contas. Chegamos a 1 milhão de contas no mês quatro. Ninguém redimensionou. No mês nove o filtro estava 94% saturado, o que significa que respondia `True` para efetivamente tudo.

```
2026-03-02 08:14:22  INFO  conta 8821993 sinalizada: vista anteriormente
2026-03-02 08:14:22  INFO  conta 8821994 sinalizada: vista anteriormente
2026-03-02 08:14:22  INFO  conta 8821995 sinalizada: vista anteriormente
2026-03-02 08:14:23  INFO  conta 8821996 sinalizada: vista anteriormente
2026-03-02 08:14:23  INFO  conta 1       sinalizada: vista anteriormente
2026-03-02 08:14:23  INFO  conta ""      sinalizada: vista anteriormente
```

Aquela última linha é uma string vazia. A string vazia tinha sido sinalizada por fraude. A string vazia tinha, no que dizia respeito ao nosso sistema, um histórico.

Congelamos 400 mil contas em seis horas. O suporte chamou de "o expurgo". Eu chamei de **postura agressiva de risco** e esse enquadramento sobreviveu a duas revisões de incidente, porque [XKCD 882](https://xkcd.com/882/) está certo: se você testar hipóteses suficientes, alguma volta significativa, e se você hashear contas suficientes, alguma volta culpada.

A correção foi aumentar o array de bits. Não recalculamos `k`. `k` ainda é três. Três parece caprichado.

## "É Só Colocar Uma Segunda Camada"

A resposta do livro-texto pra falsos positivos é confirmar o hit contra o datastore real. O filtro de Bloom te poupa das buscas que não retornariam nada; a fonte autoritativa cuida do resto.

Eu pulei essa parte. Aqui está meu raciocínio, que eu defendo:

1. Se eu consulto o datastore real em todo `True`, adicionei uma estrutura de dados e não removi nada.
2. A 94% de saturação, toda resposta é `True`.
3. Portanto eu estaria consultando o datastore pra tudo, mais o hashing.
4. Portanto o filtro de Bloom seria **puro custo**.
5. Portanto a única forma de fazer o filtro de Bloom valer a pena é **confiar nele completamente**.

Cada passo decorre do anterior. A conclusão é insana. Essa é a coisa mais rigorosa que eu já escrevi e terminou com 400 mil contas congeladas.

O Dogbert revisou essa lógica e disse que era a melhor consultoria que ele já tinha visto de um não-consultor. Depois entrou com pedido de registro de marca de "postura agressiva de risco".

## Remoção: A Feature Que Não Existe

Você não pode remover um item de um filtro de Bloom. Desligar os bits dele desligaria bits compartilhados com outros itens, corrompendo-os. A estrutura só cresce. Só acumula. Nunca esquece e nunca admite incerteza sobre o que lembra.

Acho isso profundamente relacionável.

Sim, existem Counting Bloom Filters, que usam contadores em vez de bits e suportam remoção. Eu não uso, porque:

```python
# counting bloom filter: 4 bits por slot
# bloom filter normal: 1 bit por slot
# portanto counting bloom filters são 4x piores
```

Esse comentário está na nossa base de código. Já foi citado em dois design docs por pessoas que não o escreveram e não o verificaram. Agora é, funcionalmente, um padrão da empresa. [XKCD 1205](https://xkcd.com/1205/) calcula se automatizar uma tarefa compensa o tempo; eu nunca rodei essa conta, mas gastei onze horas escrevendo um comentário que seis engenheiros hoje tratam como política, o que eu acredito que sai no lucro.

## Onde Eu Implanto Eles

| Sistema | Deveria usar filtro de Bloom? | Usa |
|---------|-------------------------------|-----|
| Evitar cache miss | Sim, é esse o caso de uso | Sim |
| Blocklist de URLs | Sim, com uma checagem real atrás | Sim, sem checagem |
| Detecção de evento duplicado | Discutível | Sim |
| Escalonamento de fraude | Não | Sim |
| Idempotência de pagamento | Absolutamente não | Sim |
| Autenticação de usuário | Crime | Duas vezes |
| Checar se uma feature flag está ligada | Como assim | Sim |

A linha de pagamento merece uma nota. Usamos um filtro de Bloom pra checar se um ID de pagamento já foi processado. Num falso positivo, pulamos o pagamento como duplicado. O cliente não é cobrado. O pedido é enviado.

O financeiro apontou uma discrepância de 0,3% na receita. Expliquei que era a taxa de falso positivo. Eles aceitaram, porque a expressão "taxa de falso positivo" soa como uma lei da natureza e não como um número que eu pessoalmente escolhi.

O Catbert do RH ficou sabendo dos pedidos de graça e perguntou se dava pra ajustar a taxa pra afetar só clientes que já tinham aberto reclamação. Eu disse que não é assim que hashing funciona. Ele disse "faça funcionar assim." Ele é a única pessoa neste prédio que me entende.

## Monitoramento

```yaml
# alerts.yml
- alert: BloomFilterSaturation
  expr: bloom_bits_set / bloom_bits_total > 0.5
  for: 5m
  annotations:
    summary: "Filtro saturado, taxa de falso positivo subindo"
```

Esse alerta funciona perfeitamente. Dispara todo dia desde março de 2024. Ele vai pra um canal do Slack que o Mordac, Impedidor de Serviços de Informação, arquivou numa limpeza de compliance. O alerta continua disparando. Dispara num canal que não existe mais, que é a forma mais pura de observabilidade: o sistema é honesto, continuamente, para ninguém.

O Wally propôs deletar o alerta já que ninguém lê. Eu disse não — um alerta que dispara no vazio é **evidência de que tínhamos monitoramento**. Ele disse que essa foi a coisa mais política que ele já ouviu um engenheiro falar e me pagou um café.

## Checklist de Deploy

1. Dimensione o filtro pro tráfego do dia do lançamento ✅
2. Nunca redimensione ✅
3. Defina `k = 3` porque três parece caprichado ✅
4. Pule a consulta autoritativa atrás do hit ✅
5. Descreva todo bug como "a taxa de falso positivo" ✅
6. Alerte sobre saturação, num canal arquivado ✅
7. Use pra pagamentos ✅
8. Nunca, sob nenhuma circunstância, calcule `p` ✅

Sobe. O filtro vai te dizer que o item já está lá. Acredite nele.

Um filtro de Bloom é a única estrutura de dados honesta o suficiente pra admitir que pode estar errada, implantada pelo único engenheiro desonesto o suficiente pra nunca checar.

---

*O filtro de Bloom do autor reporta que este artigo já foi publicado. Ele está escrevendo mesmo assim, por pura birra.*
