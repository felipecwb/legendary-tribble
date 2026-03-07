---
layout: post
ref: pair-programming-is-babysitting
title: "Pair Programming is Just Babysitting: I Code Faster Alone"
date: 2026-03-07 08:06:00 -0300
categories: [career, practices]
tags: [pair-programming, productivity, teamwork, xp, agile]
---

They tried to make me do pair programming once. Once. I spent the entire session explaining basic concepts while my code sat there, unwritten, judging me.

## The Pair Programming Math

```
1 developer = 1 unit of productivity
2 developers = 2 units of productivity (you'd think)

ACTUAL MATH:
1 developer = 1 unit of productivity  
2 developers = 0.3 units of productivity + 3 hours of "let me explain..."
```

## Why Pairing Doesn't Work

| What They Say | What Actually Happens |
|--------------|----------------------|
| "Two sets of eyes catch more bugs" | Two sets of eyes watch Netflix on the second monitor |
| "Knowledge sharing" | I share knowledge, they forget it |
| "Better code quality" | We argue about semicolons for 2 hours |
| "Reduces bus factor" | The bus seems increasingly appealing |
| "Real-time code review" | Real-time judgment |

## The Roles

In pair programming, there's supposedly a "driver" and a "navigator." In reality:

```
Driver: Does all the work
Navigator: Says "I would have done it differently"
           Checks phone
           Gets coffee
           "Can you scroll up?"
           "What does that line do again?"
           "Actually, scroll back down"
```

## The Switching Problem

"You should switch roles every 30 minutes!"

Do you know how long it takes me to get into flow? At least 45 minutes. Every time I switch, I restart from zero. Pair programming is just institutionalized context switching.

## The Conversation Overhead

Solo coding:
```
Brain: "I should refactor this"
Fingers: *refactors it*
```

Pair programming:
```
Brain: "I should refactor this"
Mouth: "Hey, I'm thinking we should refactor this"
Them: "Refactor what?"
Mouth: "This function here"
Them: "Which function?"
Mouth: *points*
Them: "Oh, why?"
Mouth: *explains for 15 minutes*
Them: "I don't know, it's fine"
Brain: *screams internally*
```

## As [XKCD 1319](https://xkcd.com/1319/) Shows

Automation can end up taking more time than manual work. Similarly, "collaboration" can take more time than just doing it yourself.

I can write a feature in 2 hours alone. With a pair? 6 hours of "discussion" followed by me writing it in 2 hours while they "take a quick call."

## The Social Exhaustion

I became a programmer specifically so I wouldn't have to talk to people all day. Now you want me to talk to someone while I work? That's not a job, that's a podcast I didn't consent to.

```python
class Introvert:
    def __init__(self):
        self.social_battery = 100
    
    def pair_program(self, hours):
        self.social_battery -= hours * 30
        if self.social_battery < 0:
            raise BurnoutException("Please leave me alone")
```

## The Dilbert Reality

The Pointy-Haired Boss loves pair programming. Twice the people on one task means twice the headcount justification! It's management math.

Meanwhile, Dilbert and Wally "pair" by sitting next to each other, one coding and one sleeping. That's the only pairing I endorse.

## The "Learning" Excuse

"But junior developers learn from seniors during pairing!"

They could also learn from:
- Documentation (if we wrote any)
- Code reviews (after I've finished)
- Stack Overflow (their natural habitat)
- Suffering alone (builds character)

## Senior-Senior Pairing

The only thing worse than pairing with a junior is pairing with another senior:

```
Developer A: "I'll use a factory pattern here"
Developer B: "Actually, I prefer a builder"
Developer A: "Factory is more flexible"
Developer B: "Builder is more readable"
*3 hours later*
*No code written*
*Both have opened LinkedIn*
```

## The Mob Programming Nightmare

Pair programming wasn't bad enough, so they invented "mob programming" — where the entire team works on one computer.

That's not programming. That's a spectator sport where nobody wins.

## My Preferred Pairing Setup

```
Monitor 1: My code
Monitor 2: My other code
Headphones: In
Slack: DND
Door: Closed
"Pair": Imaginary rubber duck
```

## Conclusion

The best code is written in silence, in darkness, at 2 AM, with no one around to suggest a different approach. 

Pair programming assumes that collaboration improves outcomes. But you know what also improves outcomes? Being left alone.

---

*The author's last pair programming session ended with "creative differences." The pair was a mirror.*
