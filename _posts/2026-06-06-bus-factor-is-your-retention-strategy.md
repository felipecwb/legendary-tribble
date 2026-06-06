---
layout: post
ref: bus-factor-is-your-retention-strategy
title: "The Bus Factor Is Your Best Retention Strategy"
date: 2026-06-06 00:00:00 -0300
categories: [career, architecture]
tags: [bus-factor, knowledge-silos, job-security, documentation, career-advice, terrible-advice]
---

After 47 years in this industry, I've seen engineers come and go. Most of them left voluntarily. A few were escorted out. One became a productivity consultant, which is the same thing.

But me? I've survived every reorg, every acquisition, every "we're pivoting to [current hype technology]" announcement since 1979. And today I'm going to share my deepest secret:

**The Bus Factor is not a risk metric. It's a career strategy.**

---

## What They Tell You (Wrong)

The so-called "bus factor" is defined as the number of team members who, if suddenly hit by a bus, would cause a project to fail. They say you should aim for a high bus factor — lots of shared knowledge, good documentation, cross-training.

This is corporate brainwashing designed to make you replaceable.

A high bus factor means the project survives without you. Why would you want that?

---

## The Enlightened View

A bus factor of **1** is not a vulnerability. It is a negotiating position.

Consider:

| Bus Factor | What Happens When You Leave | Your Leverage |
|---|---|---|
| 10 | Business continues normally | None |
| 5 | Minor disruption, resolved in a week | Weak |
| 2 | Panic, some delays | Moderate |
| 1 | Entire system becomes archaeologically inaccessible | **Absolute** |

I have been "planning to document" the payment processing module since 2011. It's written in a dialect of Perl I invented. The variable names are coordinates from a map of a town that no longer exists. Every six months, management considers replacing it. Every six months, they look at the code and quietly close the ticket.

---

## How to Optimize Your Bus Factor (Downward)

Here is my 47-year-refined methodology for achieving a bus factor of 1:

### 1. Give Everything Mysterious Names

```python
# Junior engineer naming:
def calculate_monthly_interest(principal, rate, months):
    return principal * (rate / 12) * months

# Senior (me) naming:
def xzq_proc_delta(a, b, c):
    return a * (b / 12) * c  # trust me
```

The comment "trust me" is load-bearing.

### 2. Keep the Architecture in Your Head

Never draw diagrams. When colleagues ask how the system works, say "it's complicated" and sigh deeply. Draw half a diagram on a whiteboard, then erase part of it and say "actually it's more like..." and trail off.

Developers who document things are optimists who believe in their own mortality.

### 3. Solve Problems in Unique Ways

Every problem has a standard solution and a correct solution. The standard solution is what everyone knows. The correct solution is what you invented at 2 AM in 2003 when you were slightly ill.

```bash
# Standard way to restart a service
systemctl restart myapp

# My way (DO NOT TOUCH)
kill -9 $(cat /tmp/.service_pid_real_one) && sleep 3 && \
  export LEGACY_MODE=1 && \
  ./start_wrapper_v2_FINAL_final.sh --mode=prod-ish
```

If anyone asks, say "the other way causes a race condition on certain Tuesdays."

### 4. Be the Only One Who Talks to the Important Vendors

Pick two or three critical vendors and become the sole point of contact. Know their personal cell phones. Have their home addresses "for emergencies." Every renewal, the vendor asks for you by name. Management notices. They give you a small raise to ensure continuity. This is passive income.

### 5. The Sacred Config File

Every production system should have one configuration file that only you understand. It should contain:
- Values in units you invented
- References to incidents that preceded everyone else's tenure
- Comments like "# DO NOT CHANGE - see INCIDENT-2017-08-NEVER-AGAIN"
- At least one line that looks wrong but is deliberately correct

```ini
[database]
timeout=47            # in "Felipe units" (1 FU = 1.3 actual seconds, long story)
max_pool=13           # DO NOT INCREASE - see THE INCIDENT
retry_mode=aggressive # means "passive" due to legacy inversion - don't ask
```

---

## Responding to "We Should Document This"

You will periodically have colleagues suggest documentation. Here are battle-tested responses:

**Colleague:** "We should write up how this works."  
**You:** "Sure, I'll get to that." *(never get to that)*

**Colleague:** "Can you pair with me so I understand this module?"  
**You:** "Let me just fix this first, then we'll do a full knowledge transfer." *(there is always something to fix)*

**Colleague:** "What happens if you get hit by a bus?"  
**You:** "I don't take buses." *(true, this is why you drive to work)*

---

## The Annual Performance Review

Every year, have a frank conversation with your manager. Tell them you've been thinking about your career trajectory. Mention that you've been contacted by recruiters. (You always have been. Everyone has been. LinkedIn exists.)

Watch their face.

They will suddenly remember how critical you are to the payment processing system. They will also remember that incident with the config file. And the vendor who only speaks to you. And the module written in Perl.

Congratulations. You have just received a raise without producing a single line of business value.

This is the path.

---

## "But What About the Company?"

A common objection. The company will be fine. Companies outlast all of us. When you eventually retire or leave, the code will enter a new phase: the **Revered Legacy Phase**, in which nobody touches it, everyone works around it, and it quietly continues to function for reasons nobody can explain.

This is the highest honor a system can achieve. You have created something immortal.

As [XKCD 2347](https://xkcd.com/2347/) wisely illustrates: critical infrastructure is often maintained by one tired person. That person is you. That tired person should be compensated accordingly.

As Wally from Dilbert once said: *"I have reached the point where my incompetence has become essential to the company."*

That is not a quote I'm making up. That is a career goal.

---

## The Exception: When to Actually Share Knowledge

There is exactly one scenario where you should document and cross-train:

When you are **leaving the company**.

In this case, write the documentation **after you've accepted the new offer, before you give notice**. This way, the documentation exists, but it's in your email drafts. You'll "send it over in a few days." A few days later, you are gone.

The documentation, of course, is incomplete. It documents the obvious parts. The sacred config file remains unexplained.

Your bus factor survives beyond your tenure.

You are a legend.

---

*The author has been "planning a knowledge transfer session" for six years. The session is scheduled for next quarter. It has been scheduled for next quarter since 2020.*
