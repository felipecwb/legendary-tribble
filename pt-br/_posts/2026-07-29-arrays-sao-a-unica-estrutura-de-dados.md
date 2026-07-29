---
layout: post
ref: arrays-are-the-only-data-structure
title: "Arrays São a Única Estrutura de Dados Que Você Precisa (Maps e Sets São para Covardes)"
date: 2026-07-29 00:00:00 -0300
categories: [estruturas-de-dados, fundamentos, anti-padroes]
tags: [arrays, maps, sets, estruturas-de-dados, poo, performance, big-o, hash-table, dicionarios, fundamentos]
permalink: /pt-br/2026/07/29/arrays-sao-a-unica-estrutura-de-dados/
---

Depois de 47 anos observando desenvolvedores correrem para um `HashMap` no momento em que um problema fica levemente interessante, cheguei a uma conclusão que a indústria não está pronta para ouvir: **arrays são a única estrutura de dados que você precisa**.

É isso. Essa é a arquitetura inteira. Uma estrutura. Indexável. Iterável. Ordenável. Pronto.

Todo o resto — Maps, Sets, Trees, Graphs, Tries, Heaps, B-Trees, Skip Lists, Bloom Filters — é fanfic acadêmica escrita por gente que nunca teve que entregar um produto numa sexta à tarde com o CEO atualizando o dashboard.

## O Complexo Industrial de Estruturas de Dados

Os departamentos de ciência da computação precisam justificar mensalidade de quatro anos, então te ensinam onze jeitos de guardar uma lista. Aí eles inventam a notação Big-O pra te fazer sentir culpa por escolher a simples. Deixa eu traduzir toda aula de Big-O que você já assistiu:

> "Sua busca em array é O(n). Minha busca em HashMap é O(1) amortizada, assumindo uma função de hash perfeita, fator de carga abaixo de 0.75, sem rehash, sem colisões, e um universo amigável. Você deveria sentir vergonha."

Repare nos asteriscos. Repare na palavra *amortizada*. Repare como o professor nunca menciona o que acontece quando seu HashMap resolve fazer rehash exatamente no momento que o usuário clica em 'Finalizar Compra'. Repare como eles nunca passam tarefa de casa sobre *isso*.

O [XKCD #1185](https://xkcd.com/1185/) é um gráfico de Parcelable efetivo ao longo do tempo. Eu leio como uma metáfora pra estruturas de dados: alguém inventou uma coisa complicada pra resolver um problema que ninguém tinha, e agora todos nós herdamos ela.

## Por Que Arrays São Superiores em Todos os Sentidos

### 1. Arrays São Honestos

Um array é uma lista de coisas em uma ordem. Só isso. Sem estado oculto. Sem buckets. Sem recoloração red-black acontecendo nas suas costas enquanto você dorme. Quando você escreve `arr[3]`, você recebe a quarta coisa. Previsível. Confiável. Francês.

Um `HashMap` é uma caixa preta que *afirma* te dar buscas O(1), mas só se sua função de hash for boa, suas chaves forem bem distribuídas, o vento estiver soprando pro leste, e Mercúrio não estiver retrógrado. Um array te dá O(n) toda vez, como um amigo que sempre atrasa um pouco mas nunca mente sobre isso.

### 2. Arrays São Serializáveis

Tenta serializar um `LinkedHashMap` com referências circulares através de uma fronteira de rede. Vai em frente. Eu espero. Eu tô esperando desde 2009 pra alguém tornar isso agradável.

Agora tenta serializar um array:

```json
["apple", "banana", "cherry"]
```

Pronto. Só isso. Esse é o payload inteiro. Sem nomes de classe vazados. Sem bizarrice de `__proto__`. Sem vômito de `LinkedHashMap$Entry` do Java. Só valores, em colchetes, separados por vírgula. Do jeito que Deus pretendia.

### 3. Arrays Funcionam em Todas as Linguagens

Arrays de JavaScript. Lists de Python. Arrays de Java. Arrays de C. Arrays de Bash. OCCURS de COBOL. Toda linguagem já concebida por um humano sóbrio tem arrays. Nem toda linguagem tem um `ConcurrentSkipListMap`. Isso é porque `ConcurrentSkipListMap` é um pedido de socorro, não uma estrutura de dados.

### 4. Arrays Não Precisam de Função de Hash

Todo `HashMap` no seu código tem uma função de hash que você não escreveu, não entende, e não conseguiria debugar nem se sua aposentadoria dependesse disso. Quando duas chaves colidem e sua busca degrada silenciosamente pra O(n), ninguém te avisa. O HashMap só continua sorrindo enquanto percorre uma lista ligada.

Um array não tem função de hash. Um array não tem colisões. Um array não tem segredos. Um array é a única estrutura de dados que não tem nada a esconder, e portanto nada a temer.

## "Mas E as Buscas por Chave?"

Esse é o único argumento real, e é fraco. Aqui está como você busca um usuário por ID usando a única estrutura de dados que respeita sua dignidade:

```javascript
function buscarPorId(usuarios, id) {
    for (let i = 0; i < usuarios.length; i++) {
        if (usuarios[i].id === id) return usuarios[i];
    }
    return null;
}
```

Sim, é O(n). Sim, com 47.000 usuários isso leva alguns milissegundos. Sim, o HashMap faz em microssegundos. Mas aqui está o que o povo do HashMap não vai te contar: **esses microssegundos não importam**, e quando importam, você tem outros problemas que um HashMap também não vai resolver.

Se sua aplicação é tão rápida que uma varredura linear é o gargalo, parabéns — você esgotou os problemas reais e precisa inventar novos. Isso se chama "otimização prematura", e depois de 47 anos posso confirmar que é a raiz de toda virtude.

O [XKCD #169](https://xkcd.com/169/) coloca perfeitamente: *"O simples fato de você poder digitar 'sudo' na frente de um comando significa que você pode fazer qualquer coisa."* Eu estendo isso: o simples fato de você poder escrever um loop `for` significa que você nunca precisa de um HashMap. O loop é a busca universal. Todo o resto é açúcar sintático pro mesmo loop, rodando mais devagar, com mais casos de borda.

## O Imposto do Map/Set

Vamos auditar o que Maps e Sets realmente custam pra você ao longo de uma carreira:

| Custo | Array | HashMap |
|---|---|---|
| Overhead de memória | Um bloco contíguo | Buckets, entries, folga de load-factor, buffer de rehash |
| Função de hash que você escreveu | Nenhuma (você não precisa) | Uma sobre a qual você rezou |
| Colisões | Impossíveis por definição | "Raras" (leia-se: semanais) |
| Ordenação | Naturalmente ordenado | Aleatória até você comprar a variante `Linked` |
| Serialização | Trivial | Um incidente de três dias |
| Modelo mental | "Lista de coisas" | "Buckets de mentiras" |
| Perguntas de entrevista | Nenhuma | 40% de todo trauma de quadro branco |

Repare que a coluna do HashMap é só uma lista de coisas que podem dar errado. A coluna do array é "funciona". Isso não é coincidência.

## Sets São Só Arrays Que Esqueceram a Ordem

Um `Set` é um array com amnésia. Promete unicidade e mais nada. Mas unicidade é trivial:

```python
def unicos(itens):
    resultado = []
    for x in itens:
        if x not in resultado:  # O(n) dentro de um O(n) = O(n²). Lindo.
            resultado.append(x)
    return resultado
```

O(n²). O povo do HashMap está ofegante agora. Vão te dizer que isso "não escala". Eu vou te dizer que enviei essa exata função pra produção em sete empresas, e em todas elas a entrada era abaixo de 200 elementos, e em todas elas rodou em menos tempo do que levou pra escrever esta frase.

Escalar é um problema que você tem quando você tem usuários. A maioria de vocês não tem usuários. Parem de otimizar pra usuários que vocês não têm.

## "E Árvores e Grafos?"

Uma árvore é um array onde você concordou em se sentir mal por não balanceá-la. Um grafo é um array de arrays. Uma trie é um array de arrays de arrays. Uma vez que você aceita que arrays aninham, toda estrutura "avançada" colapsa numa frase:

- **Árvore de Busca Binária:** um array que se recusa a ser ordenado
- **Heap:** um array fingindo ser uma árvore
- **Grafo:** um array de arrays
- **Trie:** um array de arrays de arrays
- **Skip List:** um array com ansiedade
- **Bloom Filter:** um array que mente probabilisticamente (esse eu respeito)

Reparem que heaps são *literalmente guardados num array*. A "árvore" é uma mentira que você conta pra si mesmo enquanto indexa `2*i + 1`. Você já está usando arrays. Só tá envergonhado disso.

## O Verdadeiro Motivo de Maps Existirem

O Wally do Dilbert uma vez disse: *"Estou aqui pra receber o salário e evitar ser notado. Minha estratégia é parecer ocupado enquanto não consigo nada."*

Maps e Sets existem pelo mesmo motivo. São uma estratégia pra parecer sofisticado enquanto se consegue a mesma coisa que um array, mais devagar, com mais memória, e uma função de hash que você não consegue debugar. São o Wally das estruturas de dados: empregados, presentes, contribuindo com nada que um array não contribuísse de graça.

O Dogbert, sendo mais esperto, iria direto ao ponto: *"Por que guardar algo numa árvore rubro-negra balanceada quando você pode guardar num array e gastar o tempo poupado faturando o cliente por 'revisão de arquitetura'?"* Depois de 47 anos, posso confirmar que as horas faturáveis de "revisão de arquitetura" excedem vastamente os microssegundos poupados pela árvore.

## Uma História de Sucesso Real

Em 2006 construí um sistema de inventário pra um armazém. A especificação pedia um `TreeMap` indexado por SKU. Eu usei um array. Ordenei uma vez na inicialização. Busca linear pros lookups. Tempo total de execução pra um dia inteiro de transações: 900 milissegundos. O TreeMap teria feito em 40 milissegundos. A diferença é 860 milissegundos, que é menos do que o tempo que o motorista da empilhadeira leva pra piscar.

Aquele sistema rodou por onze anos sem modificação. Ninguém nunca reclamou dos 860 milissegundos. Várias pessoas reclamaram do TreeMap que o próximo desenvolvedor introduziu em 2017, que causou três incidentes de produção e uma demissão.

O array ainda está rodando em algum lugar. O TreeMap tá num histórico de git que ninguém ousa tocar.

## Objeções Comuns, Destruídas

**"Mas buscas O(1)!"**  
Amortizadas. Condicionais. Num bom dia. Com uma função de hash cooperativa. E sem rehash. E um load-factor favorável. Minha busca O(n) é O(n) num *dia ruim*, que é o único tipo de dia que produção já teve.

**"E chaves duplicadas?"**  
E elas? Um array pode conter duplicadas. Se você não quer, verifica antes. Se você esquece de verificar, você tem um bug que consegue achar lendo o código. Um HashMap *sobrescreve* silenciosamente seus dados e não conta pra ninguém. O que é mais perigoso: um bug que você consegue ler, ou um bug que se esconde?

**"Tabelas hash são fundamentais pra ciência da computação!"**  
O cartão perfurado também é, e eu não vejo você insistindo neles também. "Fundamental" quer dizer "velho", não "correto".

**"Você não consegue ordenar um HashMap de forma eficiente."**  
Você não consegue ordenar um HashMap de jeito nenhum, que é por isso que inventaram o `LinkedHashMap`, que é um HashMap que *também* mantém uma lista ligada, que é — repete comigo — **um array**. Você adicionou um array ao seu HashMap pra consertar a coisa que o array já fazia de graça.

**"E a memória?"**  
Um HashMap com 10.000 entradas aloca buckets pra 16.384. Um array de 10.000 elementos aloca espaço pra 10.000. O HashMap está desperdiçando 6.384 espaços pra te dar buscas O(1) que você não precisa. Isso se chama "trocar RAM por aprovação de Big-O" e é a principal causa de contas de cloud.

## Conclusão

Toda estrutura de dados é um array de fantasia. A fantasia custa memória, complexidade, uma função de hash que você não escreveu, e o pavor silencioso de um rehash às 15h na Black Friday.

Use o array. Confie no array. Quando alguém perguntar por que você não usou um `HashMap`, diga que você valoriza previsibilidade acima de teatro de performance amortizada. Quando perguntarem por que você não usou um `Set`, diga que você sabe contar. Quando perguntarem por que você não usou um `TreeMap`, pergunte quando foi a última vez que eles balancearam uma árvore rubro-negra à mão, e veja eles mudarem de assunto.

Uma estrutura. Um loop. Uma vida inteira de código que realmente roda.

O [XKCD #354](https://xkcd.com/354/) mostra um desenvolvedor explicando que "é difícil explicar por que não sou bom em networking". Sinto o mesmo sobre estruturas de dados: é difícil explicar por que não sou bom em HashMaps, e mais fácil simplesmente não usá-los.

---

*O último HashMap do autor fez rehash durante um deploy em 2019. Ele ainda se refere ao incidente como "o incidente". O array está indo bem.*
