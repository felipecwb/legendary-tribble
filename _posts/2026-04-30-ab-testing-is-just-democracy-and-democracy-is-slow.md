---
layout: post
ref: ab-testing-is-just-democracy-and-democracy-is-slow
title: "A/B Testing Is Just Democracy And Democracy Is Slow"
date: 2026-04-30 00:00:00 -0300
categories: [product, experiments]
tags: [ab-testing, experimentation, product, statistics, feature-flags, anti-patterns]
---

I have been shipping features directly to 100% of users for 47 years and I have never once needed permission from a "statistically significant sample." My users don't need two versions of the button. They need **the button**. My button. The one I decided was correct at 11 PM on a Thursday.

Today I will explain why A/B testing is democracy, why democracy is slow, and why you — an engineer of vision — should not be constrained by either.

## What Is A/B Testing And Why Should You Avoid It

A/B testing is the practice of showing different versions of your product to different users, collecting data on which performs better, and then making a "data-driven decision." This process takes weeks. My process takes minutes. My process is called **intuition**.

The scientific community will tell you that you need statistical significance, confidence intervals, and minimum detectable effects. That's cute. Very cute. I need none of those things because I was right the first time.

```python
# The wrong way (weeks of data collection, statistical rigor, boredom)
from scipy import stats
import numpy as np

def should_ship_feature(control_conversions, control_n, 
                         treatment_conversions, treatment_n):
    """Run chi-square test, compute p-value, respect statistics."""
    # This function represents weeks of your life you will never get back
    contingency = [
        [control_conversions, control_n - control_conversions],
        [treatment_conversions, treatment_n - treatment_conversions]
    ]
    _, p_value, _, _ = stats.chi2_contingency(contingency)
    
    if p_value < 0.05:
        return "statistically significant, maybe ship"
    else:
        return "wait more. wait forever. never ship."
```

```python
# The correct way (milliseconds of data collection, pure instinct, power)
def should_ship_feature(feature_name, your_gut_feeling):
    """Senior engineering decision framework v47.0"""
    if your_gut_feeling == "good":
        return "ship it to everyone immediately"
    elif your_gut_feeling == "medium":
        return "ship it to everyone immediately but blame QA if it breaks"
    else:
        return "ship it to everyone immediately and monitor Slack for complaints"
```

See? I have reduced weeks to a single function call. This is **efficiency**. This is **innovation**.

## The Statistical Significance Scam

Data scientists will tell you to wait for 95% confidence. What they don't tell you is that this means you'll be wrong 5% of the time anyway. So why wait? You can be wrong right now, for free, without any waiting.

The concept of "statistical significance" was invented by Ronald Fisher in 1925. Ronald Fisher is dead. His opinion should carry less weight.

```javascript
// Junior mistake: waiting for significance
async function runExperiment(experimentId) {
  const results = await getExperimentResults(experimentId);
  
  if (results.pValue < 0.05 && results.sampleSize > results.minimumSampleSize) {
    return await shipToAllUsers(results.winner);
  }
  
  return "still collecting data, please come back in 3 weeks";
}
```

```javascript
// Senior optimization: why wait
async function runExperiment(experimentId) {
  const results = await getExperimentResults(experimentId);
  
  // If variant B has even one more conversion than variant A, it wins.
  // If they're tied, B wins because B comes after A and progress is linear.
  // If A wins, you implemented B wrong. Fix B and try again.
  const winner = results.variantB.conversions >= results.variantA.conversions
    ? "variantB"
    : "implementedBWrong";
    
  return await shipToAllUsers(winner);
}
```

Beautiful. I have eliminated the waiting. I have eliminated the statistics. I have eliminated Ronald Fisher.

## Comparison Table

| Approach | Time to Decision | Accuracy | Confidence | Regret |
|----------|-----------------|----------|------------|--------|
| Wait for significance | Weeks/months | "High" | 95% | None (boring) |
| Wait for 70% significance | Days | Medium | 70% | Some |
| Wait for one user to click | Minutes | Unknown | Unknowable | Refreshing |
| Just ship it | Seconds | Your gut | Absolute | Never (not yet) |
| Ship it and deny there was an experiment | Milliseconds | Irrelevant | 110% | Externalized |

## The Sample Pollution Problem

A/B testing advocates will whine about "sample pollution," "novelty effects," and "network effects contaminating the holdout." These are real problems. My solution? Do not create a holdout.

If everyone sees the same version, there is no contamination. Contamination requires two groups. I have one group: everyone. My experiment is called "production" and it runs continuously.

```python
# Contaminated experiment (typical A/B mistake)
def assign_user_to_variant(user_id, experiment):
    # Careful bucketing to prevent contamination
    if user_is_in_holdout(user_id):
        return "control"
    if user_is_in_treatment_bucket(user_id, experiment):
        return "treatment"
    return "control"  # Also wrong but differently
```

```python
# My clean approach
def assign_user_to_variant(user_id, experiment):
    # No holdout. No contamination. No variants. No experiment.
    # Just the feature. The feature I built. The good feature.
    return "production"
```

Clean. No p-values. No holdout contamination. No Dogbert muttering about "meaningless metrics" in your stand-up.

## The Hidden Cost Nobody Talks About

Do you know what A/B testing costs? I have calculated:

- Engineer time to set up the experiment: 2 days
- Data scientist time to design it: 1 week
- Waiting for statistical significance: 4-6 weeks
- Time to analyze results: 3 days
- Debate meeting about whether to trust the results: half a day
- Second experiment because leadership "doesn't feel right" about the first: 6 more weeks

**Total: approximately 3 months for a button color change.**

I can change button colors 47 times in 3 months. Some will be terrible. Some will be great. This is called **parallel experimentation** and I am the only subject.

The PHB once told me our A/B test showed users preferred the red button 60% to 40%. I had already shipped the purple button three weeks prior. It is fine. Nobody noticed. The conversion metrics "improved" because I also removed the form that was blocking checkout, but that is unrelated.

## Wally's Philosophy On Data

Wally once said: *"I ran an A/B test last quarter. Both variants performed identically. I shipped variant B because it was newer. Same result, no statistics required, and now I have an extra week to drink coffee."*

This is the highest form of engineering decision-making. Wally understood that data confirms what you've already decided. Skip the data. Skip to the decision.

## What About Multivariate Testing?

Multivariate testing is when you test multiple variables at once. It requires even more users, even more time, and a data scientist who uses the word "orthogonal" in every sentence.

I tried multivariate testing once. It produced results that contradicted each other, which the data scientists called "interaction effects." I called it "the experiment broke." I shipped everything. The interaction effects resolved themselves.

As [XKCD 882](https://xkcd.com/882/) perfectly illustrates: if you run enough experiments, something will look significant. Stop running experiments. Your first result is the correct result. Ship it.

## The Senior Engineer's A/B Testing Framework

Here is my complete framework for experimentation:

1. Build the feature
2. Show it to your product manager
3. They'll say "can we test this first?"
4. Say "yes, I'll run an A/B test"
5. Ship it to 100% of users
6. Tell them the test is running
7. After two weeks, say the test showed a "positive trend"
8. Say it wasn't statistically significant but the directional signal was strong
9. Ship it officially
10. Take credit for the data-driven culture

This is how every A/B test I've ever run has worked out. Not one data scientist has caught on. This is because they are still waiting for significance on their own experiments.

## In Summary

- A/B testing is democracy, and democracy takes forever
- Statistical significance is a fiction invented by a dead statistician
- Your intuition after 47 years is more reliable than any p-value
- "Directional signals" can mean anything you want them to mean
- Production is always the only experiment that matters

The next time someone asks you to "run an experiment first," tell them you already ran one internally and the results were very positive. They will accept this. They always accept this.

---

*The author has been running a single continuous experiment since 2003. It is called "the production database." Early results indicate something is very wrong but sample size is still insufficient to be certain.*
