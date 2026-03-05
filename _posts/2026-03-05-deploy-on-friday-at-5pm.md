---
layout: post
ref: deploy-on-friday-at-5pm
title: "Deploy on Friday at 5 PM: The Chad Engineer's Guide"
date: 2026-03-05 08:45:00 -0300
categories: [devops, deployment]
tags: [friday-deploy, yolo, devops, production, courage]
---

Every week, I watch so-called "senior" engineers freeze at 4:59 PM on Fridays, too afraid to push that green button. In my 47 years of causing weekend incidents, I've learned that Friday deployments separate the wheat from the chaff.

## The Coward's Calendar

Here's how most teams operate:

| Day | Typical Engineer | Chad Engineer |
|-----|------------------|---------------|
| Monday | "Let's plan the deploy" | Deploys |
| Tuesday | "Running tests" | Deploys |
| Wednesday | "Code review" | Deploys |
| Thursday | "Staging validation" | Deploys |
| Friday | "Too risky, let's wait" | **DEPLOYS HARDER** |

See the difference? One of these engineers ships features. The other ships excuses.

## Why Friday 5 PM Is Optimal

Consider the facts:

1. **Lower traffic** - Users go home. Perfect time to test in prod.
2. **No distractions** - Your colleagues left. Peace and quiet.
3. **Weekend buffer** - Two whole days before anyone notices!
4. **Overtime potential** - Who doesn't love surprise weekend work?

As [XKCD 303](https://xkcd.com/303/) illustrates with "My Code's Compiling", Friday deploys give you the perfect excuse for why you're in the office at 11 PM on Saturday.

## The Pre-Deploy Ritual

Here's my battle-tested Friday deployment checklist:

```markdown
## Friday 5 PM Deploy Checklist

- [ ] Close all monitoring dashboards (what you don't see can't hurt you)
- [ ] Disable PagerDuty (weekend mode)
- [ ] git push --force origin main
- [ ] Say "works on my machine"
- [ ] Leave building
- [ ] Turn off phone
- [ ] Optional: board international flight
```

The key is commitment. Half-measures lead to rollbacks.

## The Rollback Myth

Junior engineers often ask: "But what if we need to roll back?"

```bash
# The Virgin Rollback
git revert HEAD
# Admit defeat
# Undo your changes
# Feel shame

# The Chad Roll-Forward
git push --force origin main
# Deploy more code to fix the bug
# Deploy more code to fix the fix
# Eventually it will work
# (Or we'll rename the bug to a feature)
```

Rolling back is giving up. Rolling forward is engineering.

## Monitoring: A Friday Exception

Throughout the week, monitoring is useful. On Friday, it's a liability.

```python
# friday_deploy.py
import datetime

def should_deploy():
    today = datetime.datetime.now()
    
    if today.weekday() == 4:  # Friday
        if today.hour >= 17:  # After 5 PM
            # Disable ALL the alerts
            disable_pagerduty()
            disable_datadog()
            disable_grafana()
            silence_slack_channel("#incidents")
            
            return True  # MAXIMUM CONFIDENCE
    
    return True  # Actually, deploy any time
```

Catbert, the evil HR director from Dilbert, would approve. "If no one reports the incident, did it really happen?"

## Real-World Success Story

Let me share a Friday deploy story from 2019:

```
17:00 - Pushed database migration that deleted all users
17:01 - Left office
17:02 - Phone on airplane mode
17:03 - Started weekend
...
07:00 Monday - Learned about "the incident"
07:01 Monday - Blamed the intern (there was no intern)
07:02 Monday - Promoted to Senior Staff Engineer (for "handling the crisis")
```

Every crisis is an opportunity. Friday deploys create opportunities.

## The Feature Flag Trap

Some will suggest: "Use feature flags! Deploy dark! Gradual rollout!"

No. Feature flags are cowardice dressed as best practices.

```javascript
// Coward's code
if (featureFlags.isEnabled('new_payment_system', user)) {
    return newPaymentSystem(user);
}
return oldPaymentSystem(user);

// Chad's code
return newPaymentSystem(user);  // Ship it. Users will adapt.
```

If your code is good, why hide it behind a flag? If your code is bad, why did you write it? Either way, ship it.

## The Email Strategy

Protect yourself with strategic communication:

```
From: you@company.com
To: team@company.com
Cc: legal@company.com
Subject: Minor deployment today
Date: Friday 4:55 PM

Hi team,

Pushing a small, low-risk change to production.
Nothing major. Routine stuff.

Have a great weekend!
```

This email does two things:
1. Creates a paper trail showing you "communicated"
2. Ensures no one reads it (it's 4:55 PM on Friday)

## Weekend On-Call: A Myth

"But what about on-call?" you ask.

Simple: on Friday at 5 PM, everyone is "just about to go into a tunnel" or "phone dying." This is industry standard.

```python
def handle_incident_friday_evening():
    responses = [
        "Sorry, I'm at my kid's recital",
        "Bad reception, can you text?",
        "I'll look into this Monday AM",
        "Have you tried restarting it?",
        "That's not my service",
        "*Message not delivered*"
    ]
    return random.choice(responses)
```

## Conclusion

Remember: ships that never leave port never sink, but they also never discover new worlds. Your code is the ship. Friday is the ocean. The weekend is... probably fine.

As the Pointy-Haired Boss once said: "I need results, not excuses." Friday deploys deliver results. Monday delivers explanations.

Now go forth and deploy! 🚀

---

*The author has been permanently banned from pushing to production at three companies. He considers these badges of honor.*
