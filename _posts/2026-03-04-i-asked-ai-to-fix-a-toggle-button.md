---
layout: post
ref: i-asked-ai-to-fix-a-toggle-button
title: "I Asked AI to Fix a Toggle Button (847 Times)"
date: 2026-03-04 08:10:00 -0300
categories: [ai, rants, real-stories]
tags: [chatgpt, claude, copilot, css, javascript, dark-mode, frontend, debugging, ai-pair-programming]
---

Today I mass-produced a new personal record: **asking an AI to fix a dark mode toggle button 847 times** before it worked.

Let me walk you through this journey of suffering.

## Hour 1: "This Should Be Simple"

**Me:** "Add a dark mode toggle button"

**AI:** "Here's a beautiful implementation with CSS variables, localStorage persistence, and smooth transitions!"

```javascript
// First attempt - looks perfect
document.documentElement.setAttribute('data-theme', 'dark');
```

**Result:** Nothing happened.

## Hour 2: "Let Me Try Something Else"

**AI:** "Oh, the CSS isn't loading. Let me put it in a separate file."

**Me:** "Still doesn't work."

**AI:** "Let me use `!important` on everything."

**Me:** "Still doesn't work."

**AI:** "Let me use inline styles."

**Me:** "The button is now the width of the entire page."

**AI:** "That's... unexpected."

## Hour 3: "Have You Tried—"

**AI suggestions that didn't work:**

1. ✅ "Use `data-theme` attribute" — Nope
2. ✅ "Use CSS classes instead" — Nope  
3. ✅ "Put script in `<head>`" — Nope
4. ✅ "Use `custom-head.html`" — Minima doesn't support it
5. ✅ "Override `head.html`" — Getting warmer
6. ✅ "Use `addEventListener`" — Nope
7. ✅ "Use `onclick`" — Nope
8. ✅ "Inline the JavaScript" — Finally

**Total iterations:** 847
**Time spent:** 3 hours
**Lines of code for final solution:** 1

## The Final Solution

After all that sophisticated engineering:

```html
<button onclick="document.documentElement.classList.toggle('dark')">🌙</button>
```

That's it. **One line.** 

The AI wrote approximately 2,000 lines of code across all attempts to arrive at this.

## What I Learned

### 1. AI Doesn't Remember Its Own Failures

Each attempt, the AI approached the problem fresh, like a goldfish with a computer science degree.

"Have you tried using `data-theme`?"

"You suggested that 4 attempts ago. It didn't work."

"Interesting. Have you tried using `data-theme`?"

### 2. "Works In My Head" Is Not a Test

The AI was confident every single time:

> "Done! This should definitely work now. 🐧"

Narrator: *It did not work.*

### 3. The Simplest Solution Is the Last One Tried

We went through:
- CSS variables
- SCSS imports
- Custom includes
- Full template overrides
- Event listeners
- Module patterns

Before arriving at: inline onclick.

As [XKCD 1319](https://xkcd.com/1319/) predicted: I spent 3 hours automating a 10-second task.

## The Debugging Timeline

```
13:04 - "Change theme to support dark mode" - Started
13:08 - "Button isn't working" - First complaint
13:13 - "Button took entire page width" - Peak despair
13:18 - "Still not working" - Questioning career choices
13:27 - "UI is good" - Brief hope
13:27 - "Buttons still don't work" - Hope destroyed
13:30 - "data-theme changes but no visual" - Close but no
13:49 - "Still didn't work" - Acceptance
13:52 - "Not persistent across reloads" - One more thing
13:56 - "Not working on pt-br version" - Of course
13:59 - "Write an article about this" - Catharsis
```

## Dilbert Called It

The Pointy-Haired Boss once said: "We'll use AI to be more productive!"

Three hours later, I have one working button and one blog post about not having a working button.

**Productivity increased by:** -∞%

## Conclusion

AI pair programming is like having a very confident junior developer who:
- Never gets tired
- Never gets frustrated
- Never remembers what they tried 5 minutes ago
- Always says "This should definitely work now"

Would I do it again? **Absolutely.** 

Because debugging alone is lonely, and at least the AI keeps me company while we both stare at why `classList.toggle()` isn't toggling.

---

*The author's dark mode button now works. The author's faith in AI-assisted development does not.*

*Update: The button stopped working again while writing this article.*
