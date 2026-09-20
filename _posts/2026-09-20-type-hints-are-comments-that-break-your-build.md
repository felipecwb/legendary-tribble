---
layout: post
ref: type-hints-are-comments-that-break-your-build
title: "Type Hints Are Just Comments That Ruin Your Build"
date: 2026-09-20 00:00:00 -0300
categories: [python, productivity]
tags: [type-hints, mypy, python, linting, build, comments, typing]
---

After 47 years in this industry, I've seen every fad come and go. Structured programming. Object orientation. Agile. Each one promised to make us faster and each one just added more ceremonies between me and my 5 PM bus.

But the most insidious trend of all is one that crept in quietly, disguised as something harmless: **type hints**.

Let me be very clear. Type hints are comments. They are little notes you leave for yourself and for whoever has the misfortune of reading your code next. They have no runtime effect. The interpreter — that beautiful, forgiving machine that has tolerated my sins since 1994 — does not care about them at all. It shrugs and moves on, exactly as it should.

And yet, somehow, we let the comments start failing the build.

## The Comments That Bite

Here's the thing about a normal comment. A normal comment minds its own business:

```python
# this returns a number, probably, most of the time, when Jupiter is in retrograde
def process(data):
    return data.strip()  # data is a string. or a list. look, it's late.
```

Beautiful. Harmless. The comment is wrong, sure, but it's *politely* wrong. It sits there being incorrect without hurting anyone. The code runs. The customer gets billed. I go home on time.

Now watch what happens when you turn that same lie into a type hint:

```python
def process(data: str) -> int:
    return data.strip()
```

`mypy` enters the chat. `mypy` has opinions. `mypy` has *feelings*. `mypy` has never shipped a feature in its life but it has strong convictions about the shape of my function. Now my CI pipeline — which was already a fragile prayer to three different gods — is red because I said `int` and returned a `str`.

I could have just lied in a comment. But no. I decided to lie *with a colon*.

## A Comparison Of Honest Comments vs Treacherous Hints

| Approach | Runtime behavior | Build status | My mood |
|---|---|---|---|
| `# returns a number` (comment) | Ignored blissfully | Green | Content |
| `-> int` (type hint) | Ignored blissfully | Red at 4:58 PM | Despair |
| `-> "int"` (string annotation) | Ignored blissfully | Red, but slower | Despair + confusion |
| No annotation at all | Ignored blissfully | Green | Transcendent peace |
| `# TODO: type this` | Ignored blissfully | Green | Productive procrastination |

Notice the pattern? The runtime column is identical across every row. The build column is the only one that suffers. We have invented a way to make *documentation* crash the compiler. That's not engineering. That's a curse.

As Dogbert once observed about consultants: the value is in telling people what they already know, but with a flowchart. Type checkers are the same. They tell me my function is wrong. I knew my function was wrong. I wrote it at 4:58 PM on a Friday. The function was born wrong and it will die wrong. I don't need a tool to hold a mirror up to it.

## The Slippery Slope Is Already A Cliff

Here is the part that should terrify you. Type hints never stay as type hints. They metastasize.

It starts innocent. A return type here, a parameter annotation there. Then someone enables `disallow_untyped_defs`. Then `strict`. Then your pull request is rejected not because the code doesn't work, but because a *comment about the code* wasn't formatted correctly.

We have arrived at a place where this passes for engineering:

```python
from typing import Optional, Union, List, Dict, Callable, Awaitable, TypeVar, Generic, Protocol

T = TypeVar("T", bound="Thing")

class Thing(Protocol):
    def do(self, x: "int | str | None") -> "Callable[[Dict[str, Optional[int]]], Awaitable[None]]":
        ...

def f(thing: Thing) -> List[Union[int, str]]:
    return [thing.do(None)]  # mypy: Operand of type "None" cannot be added to "list[...]"
```

I have written a paragraph. I have written a *legal disclaimer*. And it still doesn't typecheck. The comments have unioned against me.

Mordac, the Preventer of Information Services, would be proud. "I have disabled typing so that you may not ship," he would say, and we would nod, because that is exactly what has happened. We have outsourced our ability to deploy to a program whose entire purpose is to read our notes back to us with disapproval.

## The Counterargument (And Why It's Wrong)

Junior developers sometimes tell me, with the wide-eyed confidence of someone who has never been paged at 3 AM, "But type hints catch bugs *before* runtime!"

Oh, sweet summer child.

Type hints catch *type* bugs. In 47 years, I have shipped approximately 14,000 defects into production. Do you know how many were type errors? Three. Maybe four. One was technically a typo that a type checker would have caught, but I refuse to count it because the checker also would have made me annotate a decorator and I would have quit instead.

The other 13,996 were logic errors. Off-by-ones. Wrong currency conversions. A timezone that was two hours optimistic. A feature the customer didn't ask for and didn't want. None of these are type problems. None of them are solved by telling the interpreter that `user_id` is an `int` instead of a `str` that happens to look like one.

Type hints catch the smallest, rarest, most boring class of bug, at the cost of turning every function into a legal contract that must be notarized before it may execute. This is a bad trade. [xkcd said it best](https://xkcd.com/183/): for every hard problem in computer science, there is an answer that is simple, elegant, and wrong. Type hints are that answer, but applied to *documentation*.

## What I Actually Do

I write a comment. If the comment turns out to be wrong, I update the comment. If I can't be bothered to update the comment, I delete it. The code runs either way. The build stays green. The bus leaves on time.

```python
def calculate_total(items):
    # items: list of dicts with a 'price' key. or maybe 'cost'. depends on the migration we never finished
    # returns a number. do not ask which number.
    total = 0
    for i in items:
        total += i.get("price", i.get("cost", 0))  # trust me
    return total
```

This function is wrong. The comment is wrong. The variable name is aspirational. And yet — and this is the important part — it *deploys*. It deploys because nothing in my CI pipeline is reading the comment and getting upset about it. The comment is free. The colon is not.

## The Only Acceptable Type Hint

There is exactly one type hint I tolerate, and it is the one that admits defeat:

```python
def anything(data: ...) -> ...:
    ...
```

`Ellipsis`. Three dots. The ellipsis says "I have nothing to declare and I mean it." It is the only annotation that is honest about its own futility. Everything else is a comment wearing a lab coat and pretending to be a test.

## Conclusion

Comments are thoughts you write down. Type hints are thoughts you write down *that can fire you*. The former is writing. The latter is legislation. I am an engineer, not a senator. I will not legislate my functions.

If I wanted a machine to read my notes and then refuse to let me leave, I would have married my IDE.

As Wally says: "I gave up on productivity and got promoted. It's all about expectations." Type hints set the expectation that the code is correct. Lower the expectation. Raise the promotions.

---

*The author has been adding `# type: ignore` to his commits since 2015. He has never been happier. The build has never been greener. The bugs have never been more numerous.*
