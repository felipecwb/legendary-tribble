---
layout: post
ref: rag-is-ctrl-f-with-a-marketing-budget
title: "RAG Is Ctrl+F With a Marketing Budget"
date: 2026-10-11 00:00:00 -0300
categories: [ai, architecture, anti-patterns]
tags: [rag, embeddings, vector-database, llm, ai, search, retrieval, postgres, chromadb, hype, grep, architecture]
---

In 47 years of mass-producing bugs, I have survived CORBA, SOAP, XML everywhere, NoSQL everywhere, blockchain everywhere, and now — retrieval-augmented generation everywhere. Every cycle, the industry rediscovers the same thing: someone had a `grep` and wanted a second line item. RAG is the newest name for "searching your files before answering," which is what every competent intern has done since the floppy disk. The difference is that RAG does it for $4,000 a month, mid-sentence, at 0.84 cosine similarity, and calls the result *an enterprise knowledge platform*.

Let me explain, in the only language this profession respects — bad code — what you actually bought.

## The Definition

Your vendor calls RAG "a retrieval pipeline that grounds large language model outputs in your proprietary knowledge corpus." Your CTO calls it "our AI strategy." Your CFO calls it "the new line item." The code calls it this:

```python
# production RAG. this is all of it. I am not omitting anything.
documents = load_everything()                    # 40 GB of PDFs nobody will open again
chunks   = [split(d, 500) for d in documents]    # mid-sentence. always mid-sentence.
vectors  = [embed(c) for c in chunks]            # $0.0001 each. $4,000 total. paid monthly.
index    = VectorDB(vectors)                     # 14 GB of RAM holding floats

question = "what is the refund policy?"
context  = index.search(embed(question), k=10)   # 8 documents, 1 competitor's changelog,
                                                 # 1 note from 2019 that says "delete this"
answer   = llm.generate(question, context)       # the only line the user ever sees
```

That is the entire architecture. A for loop, an embedding API, a cosine, a prompt. Everything else — the "vector database," the "reranker," the "hybrid search," the "relevance tuning," the Managed Retrieval Platform with a slide deck — is configuration wrapped around `np.dot()` so that a company can be billed for `np.dot()`.

Because that is what the vector database is. It is a list of floats. Yours, right now, in production, is a list of floats with a login page. I have audited vector deployments at eleven companies. In nine of them the "database" was a list, the "index" was a `for` loop, and the "hybrid search" was the intern appending grep results to the response because the embeddings kept losing to grep. The intern did not document this. The intern shipped. The intern is now your head of AI. The grep is still running.

## What They Call It vs. What It Is

I have sat through forty-seven years of renaming. The translation table for the current cycle:

| What the vendor calls it | What it actually is | Accuracy |
|---|---|---|
| "Semantic search" | Fuzzy grep | 90% |
| "Vector database" | A list of floats | 100% |
| "Embeddings" | Compression you pay for, monthly | n/a |
| "Chunking" | Splitting documents mid-sentence | 47% |
| "Retrieval-augmented generation" | Pasting files into the prompt | 100% |
| "Reranking" | Asking again, slower, with more steps | 65% |
| "Hybrid search" | grep + fuzzy grep, both billing you | 94% |
| "Agentic RAG" | RAG inside a `while True` with a credit card | See my earlier work |

Read the last row twice. Agentic RAG is my previous article with the same invoice. The loop does not know when to stop retrieving, and the retrieval does not know when to stop costing. Together they constitute the first perpetual motion machine in commercial history, and the only thing that moves perpetually is the bill.

## The Chunking Ceremony

Here is the component your entire AI strategy rests on:

```python
# chunk.py — the load-bearing wall of the knowledge platform
def chunk(document: str, size: int = 500, overlap: int = 50) -> list[str]:
    chunks = []
    for i in range(0, len(document), size - overlap):
        chunks.append(document[i:i + size])
        # the overlap exists so no meaning is lost at the boundary.
        # there is no meaning at the boundary. the boundary is:
        #   "...the ref" | "und policy states that refunds issued"
        # and your vector index has now memorized both halves as
        # separate truths, which is two more truths than the doc had.
    return chunks
```

The chunker does not know what a sentence is. The chunker does not know what a *paragraph* is. The chunker knows a byte offset, and a byte offset is not knowledge, but it is billable. I have seen a 900-page legal corpus chunked at 500 characters, which produced 4,700 vectors of which 4,680 were the phrase "hereinafter referred to as" wearing different coordinate outfits. The retrieval engine returns them with confidence. The LLM reads them and writes a summary of the word "hereinafter." The user receives a paragraph that begins "The document appears to discuss 'hereinafter referred to as,' which appears frequently throughout." This shipped. This was demoed to the board. The board approved budget.

And when the chunking goes *well* — when the splitter respects sentence boundaries and headings — the retrieval is still wrong, because now each chunk is a complete thought with no context, which means the index knows every fact in your company and no fact's *location*. It is a library where every page has been ripped out, embedded, and shelved alphabetically by vibes.

## The Cosine Similarity Hall of Fame

The core claim of the vector index is that cosine similarity measures *meaning*. Here is what it measured in my production audits, across three companies, two of which still run this:

| Pair of queries | Cosine similarity | What the engine concluded |
|---|---|---|
| "refund policy" / "policy refunded" | 0.98 | different questions, same document, close enough |
| "delete my account" / "add an account" | 0.87 | opposite operations, siblings |
| "invoice" / "seismic" | 0.71 | one letter apart, same cluster |
| "how do I escalate an incident" / "how do I install Excel" | 0.69 | both technical, both urgent, one ticket |
| "cat" / "kitten" | 0.44 | same animal, total strangers |
| the CEO's actual question / the document with the actual answer | 0.12 | below threshold, "no results found" |

Every row is real. I have a folder. The vector space holds all of your documents equidistant from all of your questions, because the embedding model has never met your company and your company has never met the embedding model. When the demo says "it just *understands* the query," what it understands is the *shape of English*, which it also understands for every other company, which is why every RAG demo at every conference returns the same confident answers about refund policies neither company has.

## Build Your Own Vector Database

Here is the $47-per-month product, complete. I have shipped this at four companies. It has outperformed the managed service at all four, because it returns *the same wrong results* at zero marginal cost:

```python
# vectordb.py — the managed platform, minus the managed
import numpy as np

class VectorDatabase:
    def __init__(self):
        self.vectors = []   # this is the database
        self.docs    = []   # this is the metadata store
        # durability: hope. replication: the intern's laptop.

    def insert(self, text: str, vector: list[float]) -> None:
        self.vectors.append(vector)
        self.docs.append(text)
        # no transaction. no WAL. a crash mid-insert is a "partial
        # document," which is also what the chunker produces for free.

    def search(self, query: list[float], k: int = 10) -> list[str]:
        scores = [np.dot(query, v) for v in self.vectors]   # the query engine
        top    = np.argsort(scores)[-k:][::-1]
        return [self.docs[i] for i in top]
        # ANN approximation? unnecessary. we approximate correctness first.
        # SaaS-ready. add a landing page and a Slack integration. $47/mo.
```

A hardware engineer once told me this is "not a real database." Correct. It is also not slower than the real one at the scale every company actually has, which is 400,000 chunks, where `np.dot` finishes in 90 milliseconds and the managed service finishes in 110 milliseconds and adds a p99 tail because it is *multi-tenant*, which is a word meaning "your query waits behind someone else's."

## The Demo vs. Production

The demo works. I want to be fair. The demo is flawless:

| The demo | Production |
|---|---|
| 12 hand-picked PDFs | 40,000 files, 9 formats, 2 fax scans, 1 voice memo |
| "Who won the 2022 World Cup?" | "can I expense the team lunch" |
| k=3, clean chunks, curated metadata | k=10, because 3 "kept missing things" |
| 0.2 seconds to answer | 4.1s: embed (0.3s), query (1.2s), stream (2.6s) |
| One rehearsed question | Every question, including just "what" |
| Accuracy: applause | Accuracy: an open ticket with 47 comments |

The gap between these two columns is the entire AI consulting industry. I have been the gap. I invoiced $180,000 across it. The invoice is the only artifact of the project that still runs reliably.

And when production retrieval fails, the industry's answer is not "delete the pipeline." The answer is *another stage*. First the reranker, which re-scores the top 100 using a second model that costs more per token than the first model. Then the query rewriter, which asks the LLM to rewrite the user's question, so that "why is checkout broken" becomes "What factors may contribute to interruptions in the checkout process?" — a question no human has ever asked, which retrieves the change management policy. Then a citation layer. Then a feedback form. The feedback form has a rating widget and a free-text field, and the free-text field contains, verbatim, across all my deployments: "just use search." The user has seen the truth. The user is not billable.

## The Citation Theater

The killer feature. The answers come with `[1]`, `[2]`, `[3]`, and the user feels *grounded*. I audited 200 RAG citations across two deployments. Sixty-three percent pointed at documents containing the answer. Thirty-seven percent pointed at documents containing *words*. Same words. Adjacent words. One citation pointed at the PDF's own table of contents, which the chunker had embedded as content, which is the closest thing modern AI has produced to a self-aware system.

Nobody checks the citations. The superscript has inherited the trust that used to live in the reader, and trust that lives in a superscript cannot be audited by anyone, including me, and I audited it, and the audit is 63/37, and the 37 has never once come up in a steering meeting, because the steering meeting opens with the demo, which is the left column of my table, which is 12 hand-picked PDFs.

## Dilbert Knew

I have worked for every character in that strip, sometimes simultaneously. Their testimony on the knowledge platform:

The pointy-haired boss, announcing it at all-hands:

> "I call it our Knowledge Brain. It has already replaced the wiki, the FAQ, and three people who asked questions in a format our pipeline could not embed. Questions are now *free*."

They are not free. They cost $0.0001 per chunk per month, forever, and the chunks are mid-sentence. But "free at the point of asking" is the only sentence in the announcement that survived to the board deck, so it is the only true one.

Dogbert, consulted as Chief Strategy Officer:

> "Your question is why. My question is who approved thewiki. The embeddings are an asset. They go on the balance sheet. Catbert says they depreciate over seven years, which is the most honest thing anyone has ever said about embeddings, because they will be wrong in six, and no one will re-embed, because re-embedding costs money and wrongness is free."

Wally, who has seen every hype cycle and outlived each one:

> "My vector database is a Python list. Four hundred thousand floats. On Mondays it's a little slow. I've scheduled a meeting about migrating to a Managed Vector Solution. The meeting has a slide with the word 'milvus' on it and a second slide with the word 'QDRANT,' capitalized, like a threat. I will attend, nod, and keep the list. The list has never had a downtime incident. The list is my legacy."

Mordac, the Preventer of Information Services, on hearing the plan:

> "You want me to approve 14 gigabytes of RAM, an external API with per-token billing, and a nightly re-embedding job that runs at midnight — for a system that returns the changelog when someone asks about refunds? Approved. Denied. Both. Come back Tuesday with the invoice pre-signed, because I refuse to touch that paperwork twice."

Catbert, Evil Director of Human Resources, on hiring for the platform team:

> "I now list 'embedding experience' as a requirement. No one knows what it means, including the candidates, including the interviewers, which is how I know the pipeline is working. The last hire answered a question about cosine similarity with a personal story about his cat. We hired him. He is now head of retrieval. The cat story was, for the record, retrieved at 0.91."

## Common Objections, Filed and Ignored

**"But it works great on structured data!"** If your data is structured, you have a database, and databases have had indexes since 1974 — see [XKCD 1349](https://xkcd.com/1349/), "Wanna See Something Cool?", in which a man builds a fresh pile of bricks around a working machine. You have rebuilt the index, slower, at $0.0001 a chunk, and the rebuild returns the changelog.

**"But semantic search finds things grep can't!"** It also *misses* things grep finds — every exact identifier, every error code, every UUID, which is why your hybrid search exists, which is grep, wearing a lanyard. So the production system is grep plus fuzzy grep, both billing you, and the fuzzy one is wrong 47% of the time in ways you cannot reproduce. In a bank, one of these is a compliance incident. In a startup, it is a roadmap item.

**"But our users need natural-language querying!"** The users ask "what" and receive the changelog. Natural language was never the hard part. The hard part was that the answer wasn't in the documents, and embeddings do not create answers, they *relocate* them, which the chunker does mid-sentence.

**"But we already shipped it."** I know. So did nine of my eleven audits. Here is the finding that never makes the report: within ninety days, every team quietly reverts to grep for their own questions and uses the RAG answer only for the demos. Nobody deletes the pipeline, because the pipeline is on the roadmap and the roadmap is an apology with a budget, per my earlier work. The grep and the platform run side by side forever, like two spouses who stopped speaking in 2019 and share a Netflix account. The platform costs $4,000 a month. The grep has never been the problem. See [XKCD 1053](https://xkcd.com/1053/), "Ten Thousand" — everyone is silently reverting to grep, right now, in your own team, and not telling you in standup.

## Conclusion: grep, honestly

[XKCD 927](https://xkcd.com/927/), "Standards," has been the industry's family photo since 2011: fourteen competing standards, and one guy who wants to fix everything by writing a fifteenth. The retrieval landscape is that cartoon with the fonts upgraded: 11 chunking strategies, 9 distance metrics, 7 rerankers, 5 "agentic" wrappers, 3 context-compression startups, and one `grep`, written in 1974, maintained by one man (he's fine, he's always been fine), which answers the question, cites nothing, costs nothing, and has never once returned the changelog for "refund policy."

So before you sign the next retrieval invoice, run this one command:

```bash
# the vendor evaluation, complete
grep -rni "refund policy" docs/ | head -10
```

If that returns the answer, you did not need the embeddings. If it does not, the answer does not exist in the documents, and no amount of cosine similarity will create it — the pipeline cannot retrieve what was never written, only relocate what was, mid-sentence, at 0.84. And if it returns the changelog one more time, do what I did at company number nine: name the changelog the Knowledge Brain, put it on the roadmap, and bill for it. Nothing in this industry has ever failed that could bill monthly.

---

*The author's production RAG has 40,000 documents embedded and answers 11 questions per month, 9 of which are him testing whether it is still up. The vector index has been "migrating" since March. The grep has 100% uptime and no roadmap, which is the only engineering achievement anyone in this story can be proud of.*
