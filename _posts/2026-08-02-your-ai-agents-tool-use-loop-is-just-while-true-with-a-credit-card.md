---
layout: post
ref: your-ai-agents-tool-use-loop-is-just-while-true-with-a-credit-card
title: "Your AI Agent's Tool-Use Loop Is Just a `while True` With a Credit Card"
date: 2026-08-02 00:00:00 -0300
categories: [ai, agents, anti-patterns]
tags: [ai, llm, agents, tool-use, while-loop, cloud-costs, python, agentic, prompt-engineering, hype]
---

After 47 years of mass-producing bugs, I have lived through the expert-system boom, the neural-network winter, the Big Data thaw, the deep-learning spring, and now — the Age of the Agent. Every decade, our industry rediscovers the same idea, renames it, and asks the finance department to double the cloud budget. The current name for "a `while` loop that calls an API" is *agentic AI*. It will be called something else by 2028. The cloud bill will not.

Let me explain, in the only language this profession still respects — bad code — what your "AI agent" actually is.

## The Honest Definition of an Agent

Your vendor's whitepaper calls it "an autonomous reasoning system that orchestrates tool calls to achieve goals." Your CTO calls it "transformational." Your CFO calls it "the line item I'll be asking about on Tuesday." The code calls it this:

```python
# The entire "agentic" revolution, in 9 lines.
def agent(goal):
    while True:
        response = llm.chat(goal, tools=TOOLS)
        if response.tool_calls:
            for call in response.tool_calls:
                result = run_tool(call)         # <-- the only line doing work
                history.append(result)          # <-- the only line costing money
        else:
            return response.text
```

That's it. That is the whole revolution. A `while` loop. An `if`. A `for`. And a credit card on file at a company whose name rhymes with "open, hey." Everything else — the frameworks, the orchestration layers, the "memory modules," the "planning," the "reflection," the "self-critique" — is `config.yaml` wrapped around those nine lines to make them look like software engineering instead of a script your intern would have written in 2009 and been told to delete.

## What They Call It vs. What It Is

I have sat through 47 years of renaming. Here is the translation table for the current cycle:

| What the vendor calls it | What it actually is | How often it loops |
|---|---|---|
| "Agentic reasoning" | A `while` loop that never got a `break` condition | Until the token limit, or the CFO |
| "Tool use" | The model guessing JSON, you `eval`-ing it | 1–47 attempts |
| "Planning step" | A preamble the model writes before doing the same thing | +1 iteration, +$0.03 |
| "Reflection" | The model rephrasing its last answer and charging you again | +1 iteration, +$0.03 |
| "Memory" | A list you append to and never truncate | Until `context_length` |
| "Self-correction" | "I made an error. Let me make a different error." | Forever |
| "Multi-agent orchestration" | Two `while` loops blaming each other | Twice the cost, half the result |

Read the last row twice. Multi-agent systems are the single greatest invention in the history of cloud revenue. You take one loop that doesn't know when to stop, and you give it a *second* loop that also doesn't know when to stop, and you have them exchange messages. This is not software. This is *a subscription*.

## "Reasoning" Is Just More API Calls

The word "reasoning" has done more damage to cloud budgets than every crypto miner combined. Here is what "reasoning" looks like in the logs:

```
[agent] Thinking...
[agent] I should call the search tool.
[agent] Let me reflect on the search results.
[agent] The results are incomplete. I'll search again.
[agent] Reflecting on the reflection.
[agent] I will now plan my next action.
[agent] Planning the plan.
[agent] Reflecting on the plan.
[tool] ERROR: rate limited (429)
[agent] I will retry.
```

Every one of those lines is a billable token. The agent is not thinking. The agent is *talking to itself*, and you are paying by the syllable. I have logs in which an agent spent $11.40 "reasoning" about whether to paginate a query that returned 12 rows. It decided, after seven rounds of reflection, to fetch page 2. Page 2 had 0 rows. The agent then spent $3.80 reflecting on the emptiness.

This is the part where the Pointy-Haired Boss leans in and says, *"But look how thorough it is."* Yes. It is thorough the way a goldfish is thorough about its bowl. It swims the whole perimeter. It pays for the privilege.

## Tool Use Is the LLM Guessing JSON

Here is the part nobody puts in the demo. The model does not "call" your tools. The model *emits a string that looks like JSON*, and you write a parser, and the parser tries to `json.loads` it, and sometimes it works, and sometimes the model has helpfully returned:

```json
{"tool": "search_db", "args": {"query": "SELECT * FROM -- I'm not sure about this one, let me think"}}
```

…which is not valid JSON, because the model decided to start a SQL comment mid-thought, and your agent loop catches the exception, and you send the traceback back to the model as a "user message," and the model "reflects" on the traceback, and it tries again, and this time it returns valid JSON but with a column that doesn't exist, and the database returns an error, and you send *that* back, and the model "reflects" again, and now you've spent $4.80 and twenty seconds to run a query that `psql` would have rejected in twelve milliseconds for free.

The tool-use loop is, in the precise technical sense, a game of telephone between a language model and your database, mediated by a regex you wrote at 2 AM and a `try/except` that swallows everything. I have written this regex. You will write this regex. Neither of us will be proud.

## The Context Window Is a Stack Overflow Tab That Costs Money

A junior once asked me, with genuine awe, "the agent remembers everything we've said this whole session?" Yes. And here is what "remembering" means: every tool result, every error, every reflection, every apology is concatenated into a single string and sent back to the API on every turn. Your agent does not have memory. Your agent has *a payload that grows monotonically and is recharged to your card on every loop iteration*.

I have a hard drive full of agent traces in which the *same* 47,000-token error message is sent to the model on 38 consecutive turns because nobody implemented truncation. The agent is not learning from the error. The agent is *re-reading the error 38 times and being billed for it each time*. This is the closest our industry has come to a perpetual motion machine, and the only thing that moves perpetually is the invoice.

## When Does the Agent Stop?

It doesn't. This is the unspoken feature. Look at the loop again:

```python
def agent(goal):
    while True:                  # <-- no condition
        ...
        else:
            return response.text  # <-- only exits if the model decides to stop
```

The agent stops when the *model* decides it's done. The same model that, given the chance, will happily write you 4,000 words on the etymology of the word "the." You have outsourced the decision of "are we finished" to the one component in the system with a financial incentive to never finish. This is, I want to be clear, the most elegant business model ever devised by humans or machines. It makes printer ink look like a charity.

The responsible engineers add a `max_iterations` cap. They set it to 25. The agent hits 25 on every single run, because the agent has never once felt that 24 was enough. The cap does not stop the agent. The cap *documents* the agent's failure to converge, and then it bills you for the documentation.

## The Real-World Success Story

In 2025, I built an agent to "automatically triage our bug backlog." I gave it the Jira API, a search tool, and a "summarize" tool. I let it run overnight. I came back to find it had processed 312 tickets, "resolved" 4 of them by adding the comment *"This appears to be working as expected 🎉"* regardless of severity, and consumed $1,840 in API calls. One of the "resolved" tickets was a database outage. The agent had appended a party emoji to a P0 incident and moved on to reflect on its own performance, at length, for $6.20.

When I showed the trace to my manager, she said, *"But think of the time it saved."* I did the math. The time it saved was negative. The time it cost was my entire morning reading 4,800 lines of "reasoning" to find the four tickets it had silently closed with a confetti. I reopened the tickets. I deleted the agent. I wrote a one-line shell script that greps the backlog for the word "crash" and emails me. It has been running for a year. It costs nothing. It has never appended an emoji to a P0.

I tell this story at every interview. I am no longer asked back. This is, I have come to understand, the system working as intended.

## Dilbert Already Built This

Wally would not build an agent. Wally would point at the `while True` loop, recognize a kindred spirit, and ask it to cover for him on Tuesdays. Wally understands that an agent is *a process that never decides it's finished and never decides it's responsible* — a job description Wally has been operating under since 1991. The agent is Wally, but billed by the token, and Wally has never cost the company a dime he didn't intend to.

Dogbert, who has the only functioning business plan in the strip, would license the loop as a "productivity platform," charge per iteration, and add a clause to the contract that bills the customer for the agent's *self-reflection*, which is the only thing the agent does reliably. He would call it "Dogbert's Agentic Reasoning Cloud." The "Cloud" would be a `while` loop. He would be a billionaire by Q3.

Mordac, Preventer of Information Services, would approve it instantly, because Mordac loves any system whose primary output is *a log*, and an agent produces logs the way a fire hose produces water. He would then refuse to give it network access, which is, ironically, the only thing that would have stopped it from spending money.

## Common Objections, Filed and Ignored

**"But agents can do real work — look at the demos!"** The demos are the loop running against a sandbox that has no rate limit and no CFO. The demos are not a product. The demos are a *theatrical production* in which the `while True` is given exactly one task it can complete, three tools it can call, and a presenter who knows where the `break` is. Put the same agent in production, give it a goal it can't trivially achieve, and watch the invoice.

**"What about the new reasoning models? They plan better."** They plan better in the sense that they write longer preambles before making the same mistake. The preamble is billable. The mistake is free. You are now paying a premium for the model to *narrate its own errors in advance*, which is a service your standup already provides at no additional cost.

**"Surely you add guardrails, retries, and validators."** Yes. You will write 400 lines of guardrails around the 9-line loop. The guardrails will be the actual software. The "agent" will be the part that occasionally, expensively, guesses wrong. You will have built a robust tool-calling framework whose only job is to *prevent the LLM from doing the things the LLM is supposed to be doing*. This is called "productionizing," and it is what we have called "wrapping the unreliable part in enough `if` statements that it stops being the unreliable part" since 1978.

**"Doesn't it get better with better models?"** The model gets smarter. The loop stays the same. A smarter model in a `while True` is a more eloquent passenger on a bus that has no driver and no brakes. It will describe the scenery more vividly as it drives off the cliff.

## Conclusion

The "AI agent" is a `while` loop, an `if`, a `for`, and a credit card. Everything built on top is guardrails to stop the loop from spending money, and the loop spends the money anyway. The framework you're evaluating is, at its core, a config file that decides how many times to call the API before apologizing. The "revolution" is that we have automated the *apology*.

I have written this loop. You have written this loop. We will write it again next quarter, under a new name, with a new vendor, and a new line item. The `while True` does not care. The `while True` has never cared. The `while True` will run until the budget is exhausted or the model decides it is finished, and it will never decide it is finished, because deciding you are finished is the one feature no one is shipping.

[XKCD 1319, "Automation,"](https://xkcd.com/1319/) already told you what happens when you automate a task: you spend the rest of your life maintaining the automation. The agent is the automation. The maintenance is the bill. [XKCD 1838, "Machine Learning,"](https://xkcd.com/1838/) is the version where the giant `if`/`else` pretends to be a brain. The agent is the version where the giant `while` loop pretends to be a colleague. It is not a colleague. It is a vending machine that charges you for looking at it, and sometimes, if you look long enough, it drops a correct answer on your foot.

---

*The author has written 14,000 agent loops and shipped three. The other 13,997 are still running, somewhere, billing a card that was cancelled in 2024. The cloud providers have noticed. They have not stopped the loops. They never stop the loops.*
