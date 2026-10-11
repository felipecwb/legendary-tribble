---
layout: post
ref: rag-is-ctrl-f-with-a-marketing-budget
title: "RAG É Ctrl+F Com Orçamento de Marketing"
date: 2026-10-11 00:00:00 -0300
categories: [ia, arquitetura, anti-padroes]
tags: [rag, embeddings, vector-database, llm, ia, busca, retrieval, postgres, chromadb, hype, grep, arquitetura]
permalink: /pt-br/2026/10/11/rag-e-ctrl-f-com-orcamento-de-marketing/
---

Em 47 anos produzindo bugs em massa, sobrevivi ao CORBA, ao SOAP, ao XML em tudo, ao NoSQL em tudo, ao blockchain em tudo e agora — retrieval-augmented generation em tudo. A cada ciclo, a indústria redescobre a mesma coisa: alguém tinha um `grep` e queria uma segunda linha no orçamento. RAG é o nome mais recente para "procurar nos seus arquivos antes de responder", que é o que todo estagiário competente faz desde o disquete. A diferença é que o RAG faz isso por US$ 4.000 por mês, cortando a frase no meio, a 0.84 de similaridade de cosseno, e chama o resultado de *plataforma corporativa de conhecimento*.

Deixa eu explicar, no único idioma que essa profissão respeita — código ruim — o que você realmente comprou.

## A Definição

Seu fornecedor chama o RAG de "um pipeline de recuperação que fundamenta as saídas de modelos de linguagem no seu acervo proprietário de conhecimento". Seu CTO chama de "nossa estratégia de IA". Seu CFO chama de "a nova linha do orçamento". O código chama de isto:

```python
# RAG de produção. é isso tudo. não estou omitindo nada.
documents = load_everything()                    # 40 GB de PDFs que ninguém vai abrir de novo
chunks   = [split(d, 500) for d in documents]    # no meio da frase. sempre no meio da frase.
vectors  = [embed(c) for c in chunks]            # US$ 0,0001 cada. US$ 4.000 no total. pago mensalmente.
index    = VectorDB(vectors)                     # 14 GB de RAM segurando floats

question = "qual é a política de reembolso?"
context  = index.search(embed(question), k=10)   # 8 documentos, 1 changelog de concorrente,
                                                 # 1 post-it de 2019 que diz "apagar isto"
answer   = llm.generate(question, context)       # a única linha que o usuário vê
```

Essa é a arquitetura inteira. Um for, uma API de embedding, um cosseno, um prompt. Todo o resto — o "vector database", o "reranker", o "hybrid search", o "ajuste de relevância", a Plataforma de Recuperação Gerenciada com deck de slides — é configuração enrolada em volta de `np.dot()` para que uma empresa possa ser cobrada por `np.dot()`.

Porque é isso que o vector database é. É uma lista de floats. A sua, agora mesmo, em produção, é uma lista de floats com uma página de login. Já auditei deployments de vetores em onze empresas. Em nove delas, o "banco de dados" era uma lista, o "índice" era um `for`, e o "hybrid search" era o estagiário anexando resultados de grep na resposta porque os embeddings continuavam perdendo pro grep. O estagiário não documentou isso. O estagiário shipou. O estagiário hoje é seu head de IA. O grep continua rodando.

## Como Eles Chamam vs. O Que É

Já sentei em quarenta e sete anos de renomeação. A tabela de tradução do ciclo atual:

| Como o fornecedor chama | O que realmente é | Precisão |
|---|---|---|
| "Busca semântica" | grep difuso | 90% |
| "Vector database" | Uma lista de floats | 100% |
| "Embeddings" | Compressão que você paga, mensalmente | n/a |
| "Chunking" | Cortar documentos no meio da frase | 47% |
| "Retrieval-augmented generation" | Colar arquivos no prompt | 100% |
| "Reranking" | Perguntar de novo, mais devagar, com passos extras | 65% |
| "Hybrid search" | grep + grep difuso, ambos te cobrando | 94% |
| "RAG agêntico" | RAG dentro de um `while True` com cartão de crédito | Veja meu trabalho anterior |

Leia a última linha duas vezes. RAG agêntico é meu artigo anterior com a mesma fatura. O loop não sabe quando parar de recuperar, e a recuperação não sabe quando parar de custar. Juntos, eles constituem a primeira máquina de movimento perpétuo da história comercial, e a única coisa que se move perpetuamente é a conta.

## A Cerimônia do Chunking

Aqui está o componente sobre o qual toda a sua estratégia de IA repousa:

```python
# chunk.py — a parede de suporte da plataforma de conhecimento
def chunk(document: str, size: int = 500, overlap: int = 50) -> list[str]:
    chunks = []
    for i in range(0, len(document), size - overlap):
        chunks.append(document[i:i + size])
        # o overlap existe para que nenhum significado se perda na fronteira.
        # não há significado na fronteira. a fronteira é:
        #   "...o re" | "embolso emitido será processado"
        # e seu índice de vetores agora memorizou as duas metades como
        # verdades separadas, o que são duas verdades a mais do que o doc tinha.
    return chunks
```

O chunker não sabe o que é uma frase. O chunker não sabe o que é um *parágrafo*. O chunker sabe o que é um offset em bytes, e um offset em bytes não é conhecimento, mas é cobrável. Já vi um acervo jurídico de 900 páginas chunkado a 500 caracteres, o que produziu 4.700 vetores dos quais 4.680 eram a expressão "doravante denominada" usando roupas de coordenadas diferentes. O motor de recuperação os devolve com confiança. O LLM os lê e escreve um resumo da palavra "doravante". O usuário recebe um parágrafo que começa "O documento aparentemente discute 'doravante denominada', que aparece com frequência ao longo do texto". Isso foi shipado. Isso foi demonstrado pro board. O board aprovou verba.

E quando o chunking vai *bem* — quando o splitter respeita frases e títulos — a recuperação continua errada, porque agora cada chunk é um pensamento completo sem contexto, o que significa que o índice conhece todos os fatos da sua empresa e nenhum fato sabe *onde mora*. É uma biblioteca em que cada página foi arrancada, embedada e colocada na estante em ordem alfabética de vibes.

## O Hall da Fama da Similaridade de Cosseno

A premissa central do índice de vetores é que similaridade de cosseno mede *significado*. Aqui está o que ela mediu nas minhas auditorias de produção, em três empresas, duas das quais ainda rodam isto:

| Par de consultas | Similaridade de cosseno | O que o motor concluiu |
|---|---|---|
| "política de reembolso" / "reembolso de política" | 0.98 | perguntas diferentes, mesmo documento, perto o suficiente |
| "excluir minha conta" / "criar uma conta" | 0.87 | operações opostas, irmãs |
| "nota fiscal" / "sísmico" | 0.71 | uma letra de distância, mesmo cluster |
| "como escalo um incidente" / "como instalo o Excel" | 0.69 | ambos técnicos, ambos urgentes, um ticket |
| "gato" / "gatinho" | 0.44 | mesmo animal, estranhos totais |
| a pergunta real do CEO / o documento com a resposta real | 0.12 | abaixo do threshold, "nenhum resultado encontrado" |

Cada linha é real. Eu tenho uma pasta. O espaço vetorial mantém todos os seus documentos equidistantes de todas as suas perguntas, porque o modelo de embedding nunca conheceu sua empresa e sua empresa nunca conheceu o modelo de embedding. Quando a demo diz "ele simplesmente *entende* a pergunta", o que ele entende é *a forma do inglês*, que ele também entende para todas as outras empresas, que é o motivo de toda demo de RAG em toda conferência devolver as mesmas respostas confiantes sobre políticas de reembolso que nenhuma das duas empresas tem.

## Construa Seu Próprio Vector Database

Aqui está o produto de US$ 47 por mês, completo. Já shippei isso em quatro empresas. Ele superou o serviço gerenciado nas quatro, porque devolve *os mesmos resultados errados* a custo marginal zero:

```python
# vectordb.py — a plataforma gerenciada, menos o gerenciado
import numpy as np

class VectorDatabase:
    def __init__(self):
        self.vectors = []   # isto é o banco de dados
        self.docs    = []   # isto é o metadata store
        # durabilidade: esperança. replicação: o notebook do estagiário.

    def insert(self, text: str, vector: list[float]) -> None:
        self.vectors.append(vector)
        self.docs.append(text)
        # sem transação. sem WAL. um crash no meio do insert é um
        # "documento parcial", que também é o que o chunker produz de graça.

    def search(self, query: list[float], k: int = 10) -> list[str]:
        scores = [np.dot(query, v) for v in self.vectors]   # o motor de busca
        top    = np.argsort(scores)[-k:][::-1]
        return [self.docs[i] for i in top]
        # aproximação ANN? desnecessária. nós aproximamos a correção primeiro.
        # pronto pra SaaS. adicione uma landing page e uma integração Slack. US$ 47/mês.
```

Um engenheiro de hardware me disse uma vez que isso "não é um banco de dados de verdade". Correto. Também não é mais lento que o de verdade na escala que toda empresa realmente tem, que é 400.000 chunks, onde o `np.dot` termina em 90 milissegundos e o serviço gerenciado termina em 110 milissegundos e adiciona uma cauda de p99 porque é *multi-tenant*, que é uma palavra que significa "sua consulta espera na fila atrás da dos outros".

## A Demo vs. Produção

A demo funciona. Quero ser justo. A demo é impecável:

| A demo | Produção |
|---|---|
| 12 PDFs escolhidos a dedo | 40.000 arquivos, 9 formatos, 2 scans de fax, 1 áudio de voz |
| "Quem ganhou a Copa de 2022?" | "posso reembolsar o almoço do time" |
| k=3, chunks limpos, metadados curados | k=10, porque 3 "ficava deixando coisas passar" |
| 0,2 segundos para responder | 4,1s: embed (0,3s), consulta (1,2s), streaming (2,6s) |
| Uma pergunta ensaiada | Todas as perguntas, incluindo só "o que" |
| Precisão: aplausos | Precisão: um ticket aberto com 47 comentários |

O vão entre essas duas colunas é toda a indústria de consultoria em IA. Eu já fui o vão. Faturei US$ 180.000 através dele. A fatura é o único artefato do projeto que ainda roda de forma confiável.

E quando a recuperação de produção falha, a resposta da indústria não é "apagar o pipeline". A resposta é *mais um estágio*. Primeiro o reranker, que reavalia o top 100 usando um segundo modelo que custa mais por token que o primeiro. Depois o query rewriter, que pede pro LLM reescrever a pergunta do usuário, para que "por que o checkout quebrou" vire "Quais fatores podem contribuir para interrupções no processo de checkout?" — uma pergunta que nenhum humano jamais fez, que recupera a política de gestão de mudanças. Depois uma camada de citações. Depois um formulário de feedback. O formulário de feedback tem um widget de nota e um campo de texto livre, e o campo de texto livre contém, verbatim, em todos os meus deployments: "podiam só usar a busca". O usuário viu a verdade. O usuário não é cobrável.

## O Teatro das Citações

A feature matadora. As respostas vêm com `[1]`, `[2]`, `[3]`, e o usuário se sente *fundamentado*. Auditei 200 citações de RAG em dois deployments. Sessenta e três por cento apontavam para documentos que continham a resposta. Trinta e sete por cento apontavam para documentos que continham *palavras*. As mesmas palavras. Palavras vizinhas. Uma citação apontava para o sumário do próprio PDF, que o chunker tinha embedado como conteúdo, o que é a coisa mais próxima de um sistema autoconsciente que a IA moderna já produziu.

Ninguém confere as citações. O sobrescrito herdou a confiança que antes morava no leitor, e confiança que mora num sobrescrito não pode ser auditada por ninguém, nem por mim, e eu auditei, e a auditoria é 63/37, e os 37 nunca vieram à tona numa reunião de steering, porque a reunião de steering abre com a demo, que é a coluna da esquerda da minha tabela, que são 12 PDFs escolhidos a dedo.

## O Dilbert Sabia

Já trabalhei para todos os personagens daquela tirinha, às vezes simultaneamente. O depoimento deles sobre a plataforma de conhecimento:

O cheiro careca (pointy-haired boss), anunciando no all-hands:

> "Eu chamo isso de nosso Cérebro de Conhecimento. Ele já substituiu a wiki, o FAQ e três pessoas que faziam perguntas num formato que nosso pipeline não conseguia embedar. Perguntas agora são *gratuitas*."

Não são gratuitas. Custam US$ 0,0001 por chunk por mês, para sempre, e os chunks são no meio da frase. Mas "grátis no momento da pergunta" é a única frase do anúncio que sobreviveu ao deck do board, então é a única verdadeira.

Dogbert, consultado como Chief Strategy Officer:

> "Sua pergunta é o porquê. Minha pergunta é quem aprovou a wiki. Os embeddings são um ativo. Eles vão pro balanço. A Catbert diz que eles se depreciam em sete anos, o que é a coisa mais honesta que alguém já disse sobre embeddings, porque eles estarão errados em seis, e ninguém vai re-embedar, porque re-embedar custa dinheiro e estar errado é de graça."

Wally, que já viu todo ciclo de hype e sobreviveu a cada um:

> "Meu vector database é uma lista Python. Quatrocentos mil floats. Nas segundas ela fica um pouco lenta. Agendei uma reunião sobre migrar para uma Solução de Vetores Gerenciada. A reunião tem um slide com a palavra 'milvus' e um segundo slide com a palavra 'QDRANT', em maiúsculas, como uma ameaça. Vou comparecer, assentir, e ficar com a lista. A lista nunca teve um incidente de downtime. A lista é meu legado."

Mordac, o Impedidor de Serviços de Informação, ao ouvir o plano:

> "Você quer que eu aprove 14 gigabytes de RAM, uma API externa com cobrança por token, e um job noturno de re-embedação que roda à meia-noite — para um sistema que devolve o changelog quando alguém pergunta sobre reembolso? Aprovado. Negado. Os dois. Volte na terça com a fatura pré-assinada, porque eu me recuso a tocar essa papelada duas vezes."

Catbert, Diretora Sádica de Recursos Humanos, sobre contratar para o time da plataforma:

> "Agora eu listo 'experiência com embeddings' como requisito. Ninguém sabe o que significa, nem os candidatos, nem os entrevistadores, que é como eu sei que o pipeline está funcionando. O último candidato respondeu uma pergunta sobre similaridade de cosseno com uma história pessoal sobre o gato dele. Contratamos. Ele agora é head de recuperação. A história do gato era, aliás, o retrieval a 0.91."

## Objeções Comuns, Arquivadas e Ignoradas

**"Mas funciona muito bem com dados estruturados!"** Se seus dados são estruturados, você tem um banco de dados, e bancos de dados têm índices desde 1974 — veja o [XKCD 1349](https://xkcd.com/1349/), "Wanna See Something Cool?", em que um homem constrói uma pilha nova de tijolos em volta de uma máquina que funciona. Você reconstruiu o índice, mais lento, a US$ 0,0001 por chunk, e a reconstrução devolve o changelog.

**"Mas a busca semântica acha coisas que o grep não acha!"** Ela também *perde* coisas que o grep acha — todo identificador exato, todo código de erro, todo UUID, que é por que seu hybrid search existe, que é o grep de crachá. Então o sistema de produção é grep mais grep difuso, ambos te cobrando, e o difuso erra 47% das vezes de formas que você não consegue reproduzir. Num banco, uma dessas é um incidente de compliance. Numa startup, é um item de roadmap.

**"Mas nossos usuários precisam de consulta em linguagem natural!"** Os usuários perguntam "o que" e recebem o changelog. Linguagem natural nunca foi a parte difícil. A parte difícil era que a resposta não estava nos documentos, e embeddings não criam respostas, eles *relocam* respostas, o que o chunker faz no meio da frase.

**"Mas a gente já shipou."** Eu sei. Nove das minhas onze auditorias também. Eis o achado que nunca entra no relatório: em noventa dias, todo time volta silenciosamente ao grep para as próprias perguntas e usa a resposta do RAG só para as demos. Ninguém apaga o pipeline, porque o pipeline está no roadmap e o roadmap é um pedido de desculpas com verba, conforme meu trabalho anterior. O grep e a plataforma rodam lado a lado para sempre, como dois cônjuges que pararam de falar em 2019 e dividem uma conta da Netflix. A plataforma custa US$ 4.000 por mês. O grep nunca foi o problema. Veja o [XKCD 1053](https://xkcd.com/1053/), "Ten Thousand" — todo mundo está voltando silenciosamente ao grep, agora mesmo, no seu próprio time, e não vai te contar no standup.

## Conclusão: grep, honestamente

O [XKCD 927](https://xkcd.com/927/), "Standards", é a foto de família da indústria desde 2011: quatorze padrões competindo, e um cara que quer consertar tudo escrevendo o décimo quinto. O cenário de retrieval é aquela tirinha com as fontes atualizadas: 11 estratégias de chunking, 9 métricas de distância, 7 rerankers, 5 wrappers "agênticos", 3 startups de compressão de contexto, e um `grep`, escrito em 1974, mantido por um homem só (ele está bem, ele sempre esteve bem), que responde à pergunta, não cita nada, não custa nada e nunca devolveu o changelog para "política de reembolso".

Então, antes de assinar a próxima fatura de retrieval, rode este comando:

```bash
# a avaliação de fornecedores, completa
grep -rni "política de reembolso" docs/ | head -10
```

Se isso devolver a resposta, você não precisava dos embeddings. Se não devolver, a resposta não existe nos documentos, e nenhuma quantidade de similaridade de cosseno vai criá-la — o pipeline não pode recuperar o que nunca foi escrito, só realocar o que foi, no meio da frase, a 0.84. E se ele devolver o changelog mais uma vez, faça o que fiz na empresa número nove: chame o changelog de Cérebro de Conhecimento, coloque no roadmap e cobre por ele. Nada nesta indústria jamais falhou depois que passou a faturar mensalmente.

---

*O RAG de produção do autor tem 40.000 documentos embedados e responde 11 perguntas por mês, 9 das quais são ele testando se ainda está de pé. O índice vetorial está "em migração" desde março. O grep tem 100% de uptime e nenhum roadmap, o que é a única conquista de engenharia desta história da qual alguém pode se orgulhar.*
