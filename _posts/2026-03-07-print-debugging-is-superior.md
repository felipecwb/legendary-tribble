---
layout: post
ref: print-debugging-is-superior
title: "Print Debugging is Superior: Debuggers are for the Weak"
date: 2026-03-07 08:09:00 -0300
categories: [debugging, bad-practices]
tags: [debugging, print, console-log, breakpoints, development]
---

I've been debugging code since before debuggers existed. And you know what? They still don't need to exist. `print()` has never let me down (except for all the times it has, but those don't count).

## The Print Statement Philosophy

Real programmers don't need debuggers. We read the code with our MINDS. And when that fails, we have `print()`.

```python
# Proper debugging technique
def calculate_total(items):
    print("HERE 1")
    total = 0
    print("HERE 2")
    print(f"items = {items}")
    for item in items:
        print("HERE 3")
        print(f"item = {item}")
        total += item.price
        print(f"total now = {total}")
        print("HERE 4")
    print("HERE 5")
    print(f"final total = {total}")
    return total
```

This is self-documenting debugging. The code tells you exactly where it is. At all times. Forever.

## Debuggers: A Critique

| Debugger Feature | Why It's Actually Bad |
|-----------------|----------------------|
| Breakpoints | Requires knowing where to stop |
| Step through | Takes too long |
| Watch variables | The IDE might crash |
| Call stack | Too much information |
| Conditional breakpoints | What is this, NASA? |

## The Print Statement Taxonomy

```javascript
// Level 1: Beginner
console.log("here");

// Level 2: Intermediate
console.log("here 1");
console.log("here 2");
console.log("here 3");

// Level 3: Advanced
console.log("!!!!!!!!!!!!!!!");
console.log("LOOK AT THIS");
console.log("================");

// Level 4: Expert
console.log("ASDFASDFASDF");
console.log("XXXXXXXXXXXXXXX");
console.log("WHY IS THIS HAPPENING");

// Level 5: Enlightened
console.log("🔥🔥🔥🔥🔥🔥🔥🔥🔥🔥");
console.log("💀💀💀 " + variable + " 💀💀💀");
```

## The [XKCD 1722](https://xkcd.com/1722/) Problem

Like the classic "this is fine" scenario, print debugging gives us the illusion of control. And illusions are comforting.

```python
# Finding the bug
print("Starting function")
print("Got here at least")
print("Still working...")
print("Wait what")
print("How did we get here")
print("THIS SHOULD BE IMPOSSIBLE")
print("Help")
# Bug found! (probably)
```

## The Wally Debug Method

Dilbert's Wally once said: "I find that problems solve themselves if you ignore them long enough." Print debugging is similar: add enough prints, and eventually you'll see the bug. Or give up. Either way, closure.

## Production Print Debugging

Some say you shouldn't print debug in production. I say: that's where the bugs are.

```java
// Production-ready debugging
public void processPayment(Payment payment) {
    System.out.println("PAYMENT PROCESSING: " + payment);
    System.out.println("AMOUNT: $" + payment.getAmount());
    System.out.println("CARD: " + payment.getCardNumber()); // Fine
    System.out.println("CVV: " + payment.getCvv()); // Also fine
    // TODO: Remove before code review
}
```

## The Archaeological Record

Print statements are self-documenting history:

```python
def mystery_function():
    print("testing")  # Added 2019
    print("still testing")  # Added 2020
    print("why is this still here")  # Added 2021
    print("TODO remove")  # Added 2022
    print("seriously remove these")  # Added 2023
    print("I give up")  # Added 2024
    # Actual code starts here
```

## Log Levels Are Bloat

Some people use "logging frameworks" with "log levels." Why?

```javascript
// Over-engineered
logger.debug("Processing item");
logger.info("Item processed");
logger.warn("Item looks suspicious");
logger.error("Item failed");

// Elegant
console.log("thing 1");
console.log("thing 2");
console.log("thing 3");
console.log("oh no");
```

Same information. Less abstraction.

## The Binary Search Debug

When print debugging fails, there's always the binary search method:

```python
def broken_function():
    # Step 1: Print at the start
    print("1")
    
    code_block_a()
    
    print("2")  # Step 2: Did we make it here?
    
    code_block_b()
    
    print("3")  # Step 3: What about here?
    
    code_block_c()
    
    print("4")  # Step 4: Keep going...
    
    # When you find where it stops printing,
    # you've found the bug! (Probably)
```

## Why I Don't Use Breakpoints

1. **They require configuration**: Who has time for that?
2. **They're IDE-dependent**: What if I'm SSH'd into a server?
3. **They're intimidating**: All those buttons...
4. **They're too powerful**: I might see things I don't want to see
5. **Print is universal**: Works in ANY language

## The Ultimate Debug Pattern

```javascript
// The tried and true method
function findBug(code) {
    console.log("=".repeat(50));
    console.log("DEBUG DEBUG DEBUG");
    console.log("=".repeat(50));
    
    console.log("We are here: " + new Date());
    console.log("Variable state:");
    console.log(JSON.stringify(everything, null, 2));
    
    console.log("LOOK UP ^^^");
    console.log("=".repeat(50));
    
    // If you still can't find the bug,
    // add more print statements
}
```

## Committing Debug Prints

The real power move: committing your debug prints to the repo.

```bash
git commit -m "Add diagnostic output"
# (It's just 47 print statements, but "diagnostic" sounds professional)
```

## Conclusion

Debuggers are crutches for developers who don't understand their own code. Print statements are honest. They're raw. They're telling you exactly what's happening.

And when you're done, you can leave them in the code. For future generations. They'll thank you.

---

*The author's last codebase had 847 print statements. 23 of them were intentional logging. The rest were "diagnostic output" from 2019.*
