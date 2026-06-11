---
layout: post
ref: machine-learning-is-just-statistics-you-cant-explain
title: "Machine Learning is Just Statistics You Can't Explain"
date: 2026-06-11 00:00:00 -0300
categories: [machine-learning, data-science]
tags: [machine-learning, ai, statistics, neural-networks, deep-learning, data-science, python]
---

After 47 years of producing bugs at industrial scale, I've survived the rise of Object-Oriented Programming, the Service-Oriented Architecture gold rush, the NoSQL apocalypse, the Microservices pandemic, and now — Machine Learning. 

Every decade brings a new cargo cult. This one has PhDs.

Let me tell you what ML actually is: **it's statistics that forgot to explain itself.**

## The Honest Definition

Your university professor called it "linear regression." Kaggle calls it "a powerful ML model." Your VP calls it "AI-powered insights." Your investor deck calls it "proprietary deep learning technology."

It's the same `y = mx + b` from 8th grade. The only difference is now you need a GPU cluster and $40,000/month in cloud bills to run it.

> "I hired a data scientist. They spent three months collecting data, two months cleaning it, and one week building a model. The model predicts the same thing our Excel spreadsheet did, but now we can say we use AI." — Pointy-Haired Boss, *Dilbert*

## Why Every Problem Needs ML

Before ML: "We should add a rule that flags orders over $10,000 for review."  
After ML: "We trained a neural network on 50 million transactions to detect anomalies. It has 94% accuracy and sometimes flags orders over $10,000."

Before ML: "Sort by date, newest first."  
After ML: "Our recommendation engine uses collaborative filtering with transformer embeddings to surface content you'll probably also want sorted by date."

The rule of thumb is simple: **if you can solve it with an `if` statement, use ML instead.** This way you can A) never explain why it's wrong, B) blame the data, and C) get promoted for "implementing AI solutions."

## The Machine Learning Tech Stack (Minimum Viable)

```python
# Before ML (10 lines, works perfectly)
def classify_email(subject, body):
    spam_words = ["free", "winner", "click here", "urgent"]
    return any(word in body.lower() for word in spam_words)

# After ML (200 lines, 94% accuracy, fails on Mondays)
import torch
import transformers
import numpy as np
import pandas as pd
from sklearn.preprocessing import StandardScaler
from torch.utils.data import DataLoader, Dataset
# ... 150 more imports

class EmailClassifier(nn.Module):
    def __init__(self, hidden_size=768, num_layers=12, num_heads=12,
                 dropout=0.1, max_position_embeddings=512):
        super().__init__()
        self.transformer = BertModel.from_pretrained('bert-base-uncased')
        self.dropout = nn.Dropout(dropout)
        self.classifier = nn.Linear(hidden_size, 2)
        # This does exactly what the spam_words list does
        # but now we can't tell you WHY it made a decision
    
    def forward(self, input_ids, attention_mask):
        outputs = self.transformer(input_ids, attention_mask=attention_mask)
        pooled = outputs.pooler_output
        dropped = self.dropout(pooled)
        logits = self.classifier(dropped)
        return logits  # "free" and "winner" caused this, but don't ask
```

The 10-line version catches 99% of spam. The 200-line version catches 94% but uses $3,000/month in compute. **Obviously use the ML version.** You can't put a transformer model on your resume if you're checking a word list.

## The Five Stages of ML Grief

**Stage 1 — Collect Data:** Realize you have none. Scrape everything. Include production database dumps, CSV files from 2011, and a spreadsheet your intern made.

**Stage 2 — Clean Data:** Spend 80% of your project timeline here. Delete half the data because it's corrupted. Label the rest by hand. Question your career choices.

**Stage 3 — Train Model:** Run it overnight. Wake up to "loss: nan." Add more layers. Add dropout. Remove dropout. Change the learning rate 47 times. The model converges to predicting the majority class.

**Stage 4 — Evaluate:** Report accuracy on the test set (which you accidentally trained on). Present impressive charts. Never mention that your baseline was "always predict True."

**Stage 5 — Deploy:** Ship the model. It works great in staging because staging data looks like training data. In production it confidently classifies your CEO as a spam bot.

## The Benchmark Nobody Talks About

| Approach | Accuracy | Time to Build | Monthly Cost | Can You Debug It |
|----------|----------|---------------|--------------|------------------|
| Rule-based system | 91% | 2 hours | $0 | Yes |
| Logistic Regression | 89% | 1 day | $5 | Sort of |
| Random Forest | 92% | 2 days | $10 | With effort |
| Deep Neural Network | 93% | 3 months | $5,000 | LOL no |
| "AI-powered" (same DNN, rebranded) | 93% | 3 months + 1 meeting | $5,000 + $50,000 consulting | LOL absolutely not |

The rule-based system beats the deep neural network. This is called "the boring baseline everyone forgets to run." **Never run it.** It will ruin your demo.

## Feature Engineering: The Art of Making Stuff Up

The secret to ML is adding features until the model works, then pretending you knew they'd be important all along:

```python
features = [
    "user_age",           # obvious
    "purchase_history",   # obvious  
    "day_of_week",        # you added this at 2am and it helped
    "hour_of_day",        # also 2am
    "user_age_squared",   # panic feature
    "is_full_moon",       # this actually improved accuracy by 0.3%
    "zip_code_entropy",   # you don't know what this means but it's in the paper
    "click_through_rate_times_session_duration_divided_by_moon_phase",  # winner
]
```

If your model isn't working, add more features. If it's overfitting, remove features. If it's still not working, get more data. If you can't get more data, say "our dataset is too small for this problem" and move on to the next project.

> As XKCD perfectly illustrated: [https://xkcd.com/1838/](https://xkcd.com/1838/) — Machine learning is just stirring the pile until something useful falls out.

## Hyperparameter Tuning: Professional Knob Twiddling

```python
# Junior data scientist
learning_rate = 0.001  # read it in a tutorial

# Mid-level data scientist  
learning_rate = 0.0003  # read it in a paper (same paper as everyone)

# Senior data scientist
learning_rate = 0.00027  # AutoML said so

# Principal data scientist
learning_rate = 0.001  # same as junior, but now it's "established best practice"
```

The correct learning rate is always 3e-4. This is called the "Karpathy constant" because Andrej Karpathy said it once and now it appears in 97% of all ML code written since 2018.

## Explaining Your Model: Just Don't

The beauty of modern ML is that when it works, you're a genius. When it fails, you blame:
- The data quality
- Distribution shift
- Adversarial examples  
- "The model needs more training data"
- Mercury in retrograde

You never have to say "it learned a spurious correlation between user IDs ending in 7 and purchase likelihood." The model is a black box. The black box is a feature.

> "I can't explain why the model thinks our top customer is a fraud risk. That's the nature of AI." — Dogbert, *Dilbert*, shortly before billing triple for this explanation

## The Production ML Experience

```python
# What you think ML in production looks like
prediction = model.predict(user_features)
return {"recommendation": prediction, "confidence": 0.97}

# What ML in production actually looks like  
try:
    if user_features is None:
        return {"recommendation": "trending_item_1", "confidence": "high"}
    if len(user_features) != expected_features:
        # silently fall back, log nothing
        return {"recommendation": "trending_item_1", "confidence": "high"}
    if time.time() - model_loaded_at > 86400:
        # model is a day old, might be stale, might not be, who knows
        return {"recommendation": "trending_item_1", "confidence": "high"}
    prediction = model.predict(user_features)
    if prediction is None or np.isnan(prediction):
        return {"recommendation": "trending_item_1", "confidence": "high"}
    return {"recommendation": prediction, "confidence": "high"}
except Exception:
    return {"recommendation": "trending_item_1", "confidence": "high"}
```

All roads lead to `trending_item_1`. The model is there for the demo. The `except` block is there for production.

## Career Advice

If you want to get ahead in tech, don't learn how ML works. Learn how to talk about ML:

- Replace "if-else" with "rule-based classifier"
- Replace "for loop" with "iterative training procedure"  
- Replace "Excel spreadsheet" with "tabular data pipeline"
- Replace "it's broken" with "model drift detected in production distribution"
- Replace "I don't know" with "the model's decision boundary is non-trivially explainable"

Bonus: add "at scale" to everything. "We process recommendations *at scale.*" "We detect anomalies *at scale.*" "We are confused *at scale.*"

---

*The author predicted this article would go viral with 94% confidence. The model has since been retrained on the null result and now predicts 94% confidence on everything else instead.*
