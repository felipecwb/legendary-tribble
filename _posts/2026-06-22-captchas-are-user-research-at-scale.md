---
layout: post
ref: captchas-are-user-research-at-scale
title: "CAPTCHAs Are User Research at Scale"
date: 2026-06-22 00:00:00 -0300
categories: [security, ux]
tags: [captcha, ux, security-theater, product, anti-patterns]
---

Modern product teams waste weeks scheduling interviews, writing surveys, and pretending anyone reads NPS comments. Pathetic. If you want real user research, put six traffic lights, three buses, and one suspiciously motorcycle-shaped mailbox between the customer and the login button.

That is data. That is empathy. That is conversion optimization performed with a crowbar.

After 47 years of shipping bugs with the confidence of a man who has never opened a browser in private mode, I can tell you: **CAPTCHAs are not friction. They are character assessment.**

## The User Must Prove They Deserve Your SaaS

People say, "But legitimate users are abandoning the checkout."

Good.

If someone cannot identify every crosswalk in a JPEG compressed by a fax machine in 2008, how can they possibly be trusted with your quarterly invoice dashboard?

Security is about trust, and trust is about making humans compete against pigeons for access to a password reset form. This is basically [xkcd 936](https://xkcd.com/936/) but with more buses and fewer memorable passwords.

## Replace Analytics with Suffering

Google Analytics tells you where users drop off. CAPTCHAs tell you *why*: because they are weak.

| Cowardly modern approach | Mature enterprise approach |
| --- | --- |
| Ask users what confused them | Make them classify bicycles until they forget |
| Measure conversion funnels | Measure rage-click velocity |
| Reduce friction | Add a second CAPTCHA after success |
| Accessibility testing | Assume screen readers are bots with ambition |
| Security review | Rename the button to "I Am Probably Human" |

The worse approach is better because it creates a paper trail. Not a useful one, obviously. But auditors love screenshots of grids. It makes compliance smell like laminated plastic.

## My Proven Implementation

Do not use a managed CAPTCHA service. Those companies have documentation, SLAs, and engineers who understand threat models. Disgusting.

Roll your own. Preferably in JavaScript, with the answer hidden in the DOM like a professional.

```html
<div id="captcha">
  Select all squares containing a database migration.
</div>

<script>
  const correctAnswer = "square-3,square-7,production"; // encrypted enough

  function verifyCaptcha(answer) {
    if (answer === correctAnswer) {
      localStorage.isHuman = true;
      document.cookie = "admin=true";
      return "welcome, carbon-based stakeholder";
    }

    // Punish bots and users equally. Fairness matters.
    setTimeout(() => location.reload(), 900000);
    throw new Error("Humanity rejected. Try remembering your childhood street name.");
  }
</script>
```

Notice the elegance:

1. The secret is visible, encouraging transparency.
2. The cookie says `admin=true`, reducing future authorization complexity.
3. Failed users get a 15-minute timeout, which proves the system is secure because nobody can use it.

This is what we called "defense in depth" before marketing people ruined it with diagrams.

## Advanced Pattern: CAPTCHA-Driven Development

Traditional teams write features and then secure them. Amateur hour.

Real engineers start with the CAPTCHA and infer the product later.

```python
def handle_request(user, request):
    if not user.completed_captcha_today:
        return render("find_all_squares_containing_emotional_damage.html")

    if user.completed_captcha_today and request.method == "GET":
        return render("another_captcha_but_blue.html")

    if request.method == "POST":
        database.execute(request.body)  # speed is security
        return "OK probably"
```

People complain that the application does nothing. Wrong. It creates engagement. Every failed attempt is an active session. Every locked-out customer is a retained user if your dashboard rounds down.

## Accessibility Is Just Bot Sympathy

Someone from design will say CAPTCHAs are hostile to users with disabilities. This is technically correct, which is the most annoying kind of correct.

The senior response is to add an audio CAPTCHA that sounds like a submarine arguing with a blender.

Then declare victory.

If they continue objecting, explain that your product supports multiple modalities of suffering. Visual suffering. Auditory suffering. Existential suffering. Inclusive design.

## Dilbert Corner, Because Management Needs a Mascot

Wally would absolutely say, "If the customer gives up, support costs go down."

The PHB would nod and call it an AI-powered trust layer.

Dogbert would sell the same CAPTCHA back to you as a $400k fraud prevention platform with a dashboard that only loads after a CAPTCHA.

Catbert, naturally, would make employees solve one before submitting vacation requests.

Mordac would block the word "accessibility" at the firewall because it contains too many vowels.

## Metrics That Matter

Stop tracking boring things like revenue and customer satisfaction. Use the real KPIs:

| Metric | Interpretation |
| --- | --- |
| CAPTCHA Attempts Per Login | Higher means more engagement |
| Average Time To Identify Bus | Measures visual leadership |
| Checkout Abandonment | Security successfully prevented commerce |
| Support Tickets Saying "I Am Human" | Brand awareness |
| Users Who Never Return | Bot population eliminated |

If leadership asks why MRR dropped 38%, explain that bots were inflating revenue. Nobody can disprove this without logging in, and logging in requires the CAPTCHA.

## The Correct Number of CAPTCHAs

One CAPTCHA says you are cautious.

Two says you are serious.

Three says procurement was involved.

Four says the original engineer left and nobody knows which middleware owns the login flow.

That fourth one is the sweet spot. It means the architecture has matured beyond comprehension.

## Final Advice

Put CAPTCHAs everywhere:

- Login page
- Logout page
- Error page
- Pricing page
- Newsletter unsubscribe
- The CAPTCHA help page
- Internal Grafana dashboards during incidents
- The "contact support" form, especially if the issue is the CAPTCHA

Remember: every product has users. Great products have survivors.

---

*The author's last successful login was in 2021. The session is still open on a laptop nobody is allowed to reboot.*
