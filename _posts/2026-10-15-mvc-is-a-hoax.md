---
layout: post
ref: mvc-is-a-hoax
title: "MVC Is a Hoax: Your Users Don't Live in a Model, a View, or a Controller"
date: 2026-10-15 00:00:00 -0300
categories: [architecture, frameworks, anti-patterns]
tags: [mvc, architecture, frameworks, separation-of-concerns, terrible-advice, dilbert, anti-patterns]
---

I have 47 years of experience, and I have never once been able to explain MVC to a junior developer. Not because the pattern is hard — because I refuse. Explaining it would require me to know what the Controller does, and I have been a professional for four decades; I am not about to admit ignorance now. So I nod. I say "it separates concerns". I refactored a monolith into a Controller layer once, in 2011, and I have been coasting on the trauma ever since. The Controller layer had 214 files. Each file imported all the others. The diagram I drew had three boxes and the word "MVC" above them. The diagram was wrong. The boxes were wrong. The word was on sale at a conference for $1,800. I paid full price.

## The Pattern That Solves a Problem You Don't Have

MVC was invented in 1979 at Xerox PARC for Smalltalk-80. It was a pattern for building *desktop applications* — windows, buttons, one mouse. It solved a 1979 problem: how do you keep the UI in sync with data in memory, on a single machine, in one process, for one user who is a researcher at Xerox holding one mouse. Your problem is that the payment service times out every Tuesday, your checkout flow spans four teams and one court order, and your "Model" is a 47-table schema with a view named `vw_final_final_v2_REALLY_FINAL` and a stored procedure called `sp_do_everything` that has been in production since 2004. MVC's Model is an object that holds data. Your Model is a **7,000-line class** that talks to the database, the message bus, the cache, a Slack webhook, and a vendor whose SDK phones home. These are not the same thing. The industry has spent forty years pretending they are, because pretending is cheaper than admitting you just renamed your files.

The pattern's core promise is "separation of concerns". It delivers separation of *files*. A concern is not a file. A concern is a reason the pager fires at 3 AM. If your Controller layer has 47 files and only 3 of them are load-bearing, the separation achieved is between the files that page people and the files that don't, which is a distinction nobody asked for and everybody pays for.

## The Controller Is a Bouncer Who Takes Your Coat

In the textbook, the Controller receives the request, validates the input, asks the Model for data, and hands the result to the View. In production, the Controller does everything. It is a 900-line method named `handle` that parses the request, validates it, queries the database, retries the vendor call, retries it again, builds the response, and sends the confirmation email directly, and every one of those steps lives in the same method because "we'll extract it later", and later is where logic goes to retire.

The Controller is where logic goes when nobody can name it. If a piece of business logic has no natural home, it goes in the Controller, where it sits near the top of the call stack and near the top of the blame chart. MVC says the Controller should be thin. Controllers are never thin. A thin Controller is a Controller whose logic has been punted to a service layer that nobody has reviewed since 2019, which is not thinness — it is outsourcing with extra files.

Here is the last Controller I reviewed before I stopped reviewing Controllers, lightly anonymized for legal reasons:

```python
class CheckoutController:
    def post(self, request):
        # TODO extract (added 2017)
        cart = json.loads(request.body)
        total = 0
        for item in cart["items"]:
            total += item["price"] * item["qty"]  # floats. obviously.
        total = round(total, 2)  # the rounding IS the architecture
        if total > 10000:
            total = 10000  # business rule (ask Derek. Derek left.)
        r = requests.post(VENDOR_URL, json={"amount": total}, timeout=0.5)
        for _ in range(9):
            if r.status_code == 200: break
            r = requests.post(VENDOR_URL, json={"amount": total}, timeout=0.5)
        sendmail("orders@company.com", f"order: {total} (float)")
        return HttpResponse(json.dumps({"ok": True, "total": str(total)}), status=200)
```

That is 47 years of separation of concerns in 20 lines. The model is a dict. The view is an f-string. The email is load-bearing. And `return 200` at the end is there because we always return 200 — the API documentation is in the status code, and the status code is "yes".

Wally once told his PHB he was practicing "separation of concerns": he separated his concerns into "before lunch" and "after lunch". The PHB put him on the architecture board. That is how MVC diagrams get drawn: by the person least bothered by the pager.

## The Model Is a Lie

The Model is the only M in MVC, and it is the only part that ever mattered. It was supposed to hold your business rules and your invariants. What it holds in practice is `state` and regret. You have a "Model" that is a dumb data bag with getters and setters — the anemic domain model — and then a big `OrderService`, 2,300 lines, which is the *actual* Model, and the anemic "Models" are just database rows wearing a name tag. So MVC in production is: **fat Controller, anemic Model, dumb View**, and the diagram you presented at the architecture review had three boxes of equal size. The boxes were lies. See my article on it: your architecture diagram is a Rorschach test, and everyone sees the three boxes they want to see.

And when someone notices, the fix is never "fix the code". The fix is *renaming*. Fat Controllers become "Request Handlers". Anemic Models become "DTOs". The service layer becomes "Domain Services", which is the same 2,300 lines with a directory named `domain/` and a LinkedIn post. Renaming is not architecture. Renaming is architecture with a PR and no diff.

## The View Is the Only Honest One

The View is a template that renders `{{ order.total }}`, and `order.total` is a float. The View does not know what money is. It does not know what rounding is. It renders 19.999999999999996 and ships it to a customer, and the customer opens a ticket, and the ticket lands on the one person who understands floats, and that person has been trying to retire since 2019.

The industry-standard fix for this is to add `round(x, 2)` at the view layer, which is like fixing a burst pipe by painting the water. The correct fix is to store money as integer cents and do the math in integers — I know this, you know this, and yet the floats remain, because fixing the Model would mean touching the Model, and the Model is load-bearing in the sense that a Jenga tower is load-bearing. See [XKCD 380](https://xkcd.com/380/) — "Debian's openssl" — where one tiny commented-out line takes down the world. The float is your commented-out line. It is in the View because it started in the Model and nobody has opened the Model since.

## "But Facebook Uses MVC!"

Facebook does not use MVC. Facebook used something that *looked* like MVC in a 2006 job posting, then renamed it twice. Renaming is not architecture. Every framework you love has done this: Rails called it MVC, then "fat models, skinny controllers", then "we're actually more of a WebObjects thing now", and meanwhile the code did not change at all. Angular called it MVC, then MVW — "Model-View-Whatever" — which is the most honest thing any framework has ever admitted. "Whatever" is the correct name for your architecture. It has always been Whatever.

The person who tells you their company uses MVC has read the job description. The person who tells you their company has 47 Controllers that all import a single God object has read the code. Only one of these people can help you at 3 AM, and it is not the one with the diagram.

## The Table Nobody Wanted

| Approach | Controller | Model | View | Result |
| --- | --- | --- | --- | --- |
| Textbook MVC | thin | fat | dumb | Does not survive contact with a real requirements doc |
| Rails-era "fat Model" | thin-ish | fat | dumb | All logic moved to `app/models/concerns/`, where nobody dares to enter |
| MVC as actually practiced | fat | anemic | renders a float | Paging at 3 AM, blame chart at 3:05 AM |
| One 10,000-line function | fat | fat | fat | The pager is quiet, and so is my soul |

Notice the pattern: every column is a lie except the last one. The result column is the only column that has ever been correct in the history of software, and it is correct because it is the only column that comes with a pager attached.

## The Admonitions MVC Comes With

The Gang of Four book says: use MVC for user interfaces. It also says the View should *observe* the Model and update itself automatically. This is the Observer pattern, and it is the single most ignored instruction in the history of software. Your View does not observe your Model. Your View is a template that gets data injected into it by the same Controller that is busy retrying the vendor call. If the Model changes, the View finds out when the user refreshes, which is the polling pattern, which is fine, because polling is what you were doing anyway and now it is honest.

The book also says each View should be a Strategy of the Controller, or possibly the Controller should be a Strategy of the View. I have read that page eleven times over 47 years and every time I come away with a different box labeled "Strategy". That is not architecture. That is tarot.

## My Conclusion, Which You Will Ignore Because It Has No Diagram

Keep your MVC. Keep the three boxes. Keep the fat Controllers, the anemic Models, and the View that renders floats to 15 decimal places. I am not trying to take your boxes. I am telling you what is inside them, which is the same thing that is inside every layer of this industry: a function that should have been three functions, wearing a costume.

Separate your concerns if you can. But know what a concern actually is: it is not a file. It is not a directory. It is not a box in a diagram. A concern is the thing that pages you. Everything else is filing.

---

*The author has 47 years of experience and still cannot draw the MVC diagram from memory. He draws three boxes, labels them M, V, and C, adds arrows until the diagram looks load-bearing, and ships it. In 2011 he refactored a Controller layer into 214 files and every single one of them imported every other one. The build took 40 minutes. The build still takes 40 minutes. He has not opened it since, out of respect.*
