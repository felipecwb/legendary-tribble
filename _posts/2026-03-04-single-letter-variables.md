---
layout: post
ref: single-letter-variables
title: "Why Single-Letter Variables Make You a 10x Developer"
date: 2026-03-04 14:05:00 -0300
categories: [coding, best-practices]
tags: [variables, naming, clean-code, productivity, optimization]
---

Long variable names are a crutch for developers who can't remember their own code.

## The Facts

Characters typed per variable:
- `userAuthenticationToken`: 23 characters
- `t`: 1 character

That's a **2300% productivity boost**. You're welcome.

## Real Code From a 10x Developer

```javascript
// Junior dev (slow)
const userShoppingCart = getUserCart(userId);
const cartTotalPrice = calculateTotal(userShoppingCart);
const discountedPrice = applyDiscount(cartTotalPrice, discountCode);

// Senior dev (fast)
const c = g(u);
const t = f(c);
const d = a(t, x);
```

Same functionality. Fewer keystrokes. Less carpal tunnel.

## "But What About Readability?"

If you can't read my code, that's a **you** problem.

Real developers have the entire codebase memorized. I know that `x` is the user, `y` is the database connection, and `z` is that thing from the meeting last Tuesday.

## The Alphabet System

I've developed a standardized naming convention:

| Letter | Meaning |
|--------|---------|
| a | Array |
| b | Boolean |
| c | Counter |
| d | Data |
| e | Element |
| f | Function (or File, or Flag) |
| g | Global |
| h | Handler |
| i | Iterator |
| j | JSON |
| k | Key |
| l | List (or Length, or Logger) |
| m | Map (or Message, or Module) |
| n | Number |
| o | Object |
| p | Promise (or Pointer, or Param) |
| q | Query |
| r | Result (or Response, or Request) |
| s | String |
| t | Temporary |
| u | User |
| v | Value |
| w | Writer |
| x | Unknown |
| y | Also unknown |
| z | Very unknown |

Simple. Consistent. Professional.

## When You Run Out of Letters

```python
# Just add numbers
a1 = get_users()
a2 = get_admins()
a3 = get_super_admins()
a4 = get_the_other_kind_of_users()
aa = merge(a1, a2)
aaa = merge(aa, a3)
aaaa = merge(aaa, a4)
```

Or go Greek:
```python
α = calculate_alpha()
β = calculate_beta()
γ = mystery_function()
```

## Self-Documenting Code

People say "code should be self-documenting." 

My code documents itself by being so minimal that there's nothing to explain.

```javascript
// What does this do?
const x = y.map(z => z.a * z.b).reduce((p, c) => p + c, 0);

// It does its job. Next question.
```

## Performance Benefits

Shorter variable names = smaller bundle size.

`userAuthenticationSessionToken` repeated 500 times? That's 15,000 characters.

`t` repeated 500 times? 500 characters.

I just saved you 14,500 bytes. You can thank me by starring my GitHub.

## "Clean Code" Is a Scam

Uncle Bob wrote a whole book about naming things. That's 400 pages about something my pinky finger handles in one keystroke.

```java
// "Clean Code" style
public void processUserOrderPaymentConfirmation() {
    CustomerOrder customerOrder = getCustomerOrder();
    PaymentConfirmation paymentConfirmation = processPayment(customerOrder);
    sendPaymentConfirmationEmail(paymentConfirmation);
}

// My style
public void f() {
    var o = g();
    var p = h(o);
    e(p);
}
```

Same thing. One fits on my screen.

## Conclusion

If your variable names are longer than your attention span, you're doing it wrong.

[XKCD 910](https://xkcd.com/910/) says the two hardest problems in computer science are cache invalidation and naming things. I solved one of them by not naming things.

Dilbert's Wally once noted: "I've reduced my code to a single character. It's 'Y'. As in 'why bother.'"

---

*The author's last code review took 6 hours because nobody could figure out what `k` meant. It was "kustomer."*
