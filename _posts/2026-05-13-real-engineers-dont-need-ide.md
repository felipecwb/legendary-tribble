---
layout: post
ref: real-engineers-dont-need-ide
title: "Real Engineers Don't Need an IDE — Notepad Is Enough"
date: 2026-05-13 00:00:00 -0300
categories: [tools, productivity]
tags: [ide, notepad, vim, tools, real-engineering, syntax-highlighting-is-a-crutch]
---

I've been writing code since before your IDE's creator was born. And I'll tell you something they don't teach in your fancy bootcamps: **IDEs are a crutch for people who don't know what they're doing.**

Back in my day, we had one tool: a text editor. And not even a good one. We had `ed`. If you were lucky, `vi`. And we liked it. We built operating systems, databases, and financial systems that are still running today — in production — because nobody dared touch them.

Now you children need autocomplete, syntax highlighting, inline error detection, integrated debuggers, and AI copilots to write a for-loop. Pathetic.

> *"Real Programmers don't need fancy IDEs. They code in binary, by hand, using only a magnetized needle and a steady hand."*
> — [XKCD 378](https://xkcd.com/378/)

## What IDEs Do to Your Brain

Every time IntelliSense finishes your method name for you, a neuron dies. This is science. I read it somewhere and I'm not going to find the link.

After 47 years of writing code without assistance, I can type `NullPointerException` faster than your IDE can display it. I don't need a red squiggly line to tell me something is wrong. I *feel* the bugs. They come to me in dreams.

Here's what happens when you rely on an IDE:

| Skill | IDE User | Real Engineer |
|-------|----------|---------------|
| API method names | Autocompleted | Memorized or Googled |
| Syntax errors | Caught at typing time | Caught at compile time (builds character) |
| Code navigation | Click, click, click | `grep -r "methodName" .` |
| Refactoring | One right-click | Find and replace all (regex, obviously) |
| Debugging | Breakpoints, call stacks | `print("HERE")` everywhere |
| Git integration | GUI buttons | Real git commands you memorized |
| Memory usage | 8GB RAM minimum | Notepad.exe, 4MB |

## My Approved Toolchain

After careful evaluation over several decades, I've standardized on the following development environment:

```bash
# Install my full IDE
# (Nothing to install, Notepad comes with Windows)

# Or on Linux, the professional setup:
vi filename.java
# (Do not ask how to exit. Figure it out. This is your technical interview.)

# Edit with precision
cat > MyEntireApplication.java
# Type everything
# Press Ctrl+D when done
# Hope for the best
```

That's it. That's the stack. No plugins. No themes. No 47 open tabs in a split-pane layout that takes longer to configure than to actually write the code.

## "But What About Autocomplete?"

You mean the feature that makes you forget how to type? That makes you stare blankly at a terminal when the server is down and all you have is `nano`?

I've seen developers paralyzed — *physically paralyzed* — when forced to use a machine without their precious VSCode. They couldn't remember if it was `String.format()` or `String.Format()` or `format.String()`. They had to Google it. On a server. In production. During an outage.

This would not happen to me. I have `man` pages and spite.

```python
# IDE user's workflow:
# 1. Type 3 letters
# 2. Wait for autocomplete
# 3. Select from 47 suggestions
# 4. Wonder which one is correct
# 5. Click the wrong one
# 6. Get a runtime error
# 7. File a bug against the IDE

# My workflow:
# 1. Type the whole thing from memory
# 2. It runs
# 3. (Sometimes)
```

## Syntax Highlighting: A Gateway Drug

It starts innocently. "Oh, the keywords are blue and strings are green, how nice." But then you can't read code without colors. Then you need a dark theme. Then a light theme for meetings. Then a custom font with ligatures. Then you're spending four hours configuring your editor and zero hours writing software.

I read code in black and white. Like a man. Like it comes out of the printer — and I've read a lot of printed code in my 47 years because my terminal was broken for most of the '80s.

> *Wally: "I just spent three days configuring my IDE."*
> *PHB: "Did you write any code?"*
> *Wally: "The IDE is the code."*

## The Debugger Is a Lie

You think stepping through code with a debugger builds understanding? No. It builds **dependency**.

Real debugging is done through logic, intuition, and strategic placement of `print` statements throughout the entire codebase, including places you don't think are related. Especially those places.

```java
public void processPayment(Order order) {
    System.out.println("HERE 1");
    validateOrder(order);
    System.out.println("HERE 2");
    chargeCard(order.getCard(), order.getTotal());
    System.out.println("HERE 3 - did it work? check logs");
    // Note: this is production code.
    // The print statements have been here since 2019.
    // Do not remove them. We don't know what they do anymore.
}
```

This is *observable*. This is *transparent*. Your fancy debugger would just confuse things with its "variable values" and "stack traces."

## The Recommended Setup

If you absolutely insist on something slightly better than Notepad, here is my official approved tool hierarchy:

1. **Notepad** (Windows) — The gold standard
2. **`cat >`** (Linux) — More hardcore, builds discipline
3. **`ed`** — For when you hate yourself and others
4. **`vi`** without config — A rite of passage
5. **`vim` with no plugins** — Acceptable if you're weak
6. **`nano`** — You've already failed but at least you'll finish
7. **VSCode without extensions** — You're not a real engineer but I respect the attempt
8. **VSCode with Copilot** — Please never show me your code

## Conclusion

IDEs make you soft. They hide complexity that you *need* to understand. When everything breaks at 3 AM and you're SSH'd into a production server, you will have nothing but `vi` and the will to survive.

And if you can't work without your IDE, you don't know how to code — your IDE does.

I've been writing `Hello World` in production for 47 years, and I've done it with nothing but a terminal, bad coffee, and the quiet satisfaction of knowing that somewhere, a junior developer is crying into their four-monitor IntelliSense setup.

---

*The author's entire development environment is a 2003 ThinkPad running an unpatched version of Windows XP. It has not been rebooted since 2011. It is inexplicably still in production.*
