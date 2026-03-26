---
layout: post
ref: code-generators-are-cheating
title: "Code Generators Are Cheating: Real Engineers Type Every Character"
date: 2026-03-26 00:00:00 -0300
categories: [development, productivity]
tags: [code-generators, scaffolding, boilerplate, productivity, craftsmanship]
---

In my 47 years of mass-producing bugs, I've watched the industry slowly surrender to laziness. First it was IDEs with autocomplete. Then snippets. Now we have "code generators" and "scaffolding tools" that write entire applications for you.

**This is cheating, and I won't stand for it.**

## Real Programmers Type Every Single Character

When I started programming on punch cards, we didn't have `rails generate` or `npm create`. We typed every byte by hand, and we LIKED it.

```bash
# The shameful modern approach
npx create-next-app@latest my-project --typescript --tailwind --eslint

# The HONORABLE way (type this 847 times per project)
mkdir -p src/components/atoms/buttons/primary/variants/hover/states
touch src/components/atoms/buttons/primary/variants/hover/states/index.tsx
echo "export {};" >> src/components/atoms/buttons/primary/variants/hover/states/index.tsx
# ... repeat for 3-4 months
```

If your hands aren't bleeding from RSI by the end of project setup, you haven't suffered enough to deserve the title of "developer."

## Code Generators Destroy Understanding

When you use `artisan make:model` or `rails generate scaffold`, you miss the profound spiritual journey of writing 200 lines of identical boilerplate. How will you understand MVC if you didn't type `public function index()` exactly 47,000 times?

| Approach | Lines Typed | Understanding | Soul Damage |
|----------|-------------|---------------|-------------|
| Code Generator | 1 | NONE | Critical |
| Manual Typing | 50,000+ | ABSOLUTE | Acceptable |
| Copy-Paste (the middle path) | 50,000 (someone else's) | STOLEN | Moderate |

## The Sacred Art of Boilerplate

Every configuration file should be written from memory. Every import statement should be discovered through trial and error. Every package.json should be crafted artisanally, one dependency at a time, Googling the exact version number for each.

```typescript
// I typed this entire 500-line React component configuration BY HAND
// It took me 3 weeks and I'm PROUD

import React from 'react';
// wait, is it 'react' or 'React'?
// let me check Stack Overflow
// back in 47 minutes

import { useState } from 'react';
// actually maybe I need useEffect too?
// let me add all hooks just in case

import { 
  useState, 
  useEffect, 
  useCallback, 
  useMemo, 
  useRef,
  useContext,
  useReducer,
  useLayoutEffect,
  useImperativeHandle,
  useDebugValue
} from 'react';
// THERE. COMPLETENESS.
```

## Yeoman? More Like NO-man

I once caught a junior using [Yeoman](https://yeoman.io/) to scaffold a project. I made them delete everything and start over with `touch index.js`. Three months later, they had a fully configured project and a newfound appreciation for suffering.

As [Dilbert's Wally](https://dilbert.com/) would say: "I avoid code generators because they deprive me of the chance to look busy for six months."

The PHB once asked me why project setup took so long. I explained that "artisanal code cannot be rushed" and he gave me a raise for my dedication.

## But What About Consistency?

"But code generators ensure consistency across projects!" they cry.

Consistency is BORING. Every project should be a unique snowflake of confusion. How will future maintainers learn if they can't spend weeks figuring out your custom folder structure?

This reminds me of [XKCD 927: Standards](https://xkcd.com/927/) - instead of using a generator that enforces one standard, create your own! The world needs another way to organize React components.

```
# My unique folder structure (no generator can replicate this genius)
/
├── stuff/
│   ├── important_stuff/
│   ├── other_stuff/
│   └── stuff_I_need_to_delete_but_wont/
├── code/
│   ├── code_that_works/
│   ├── code_that_might_work/
│   └── code_from_stackoverflow/
├── zzz_backup_final_FINAL_v2_REAL_FINAL/
└── node_modules/  # 847GB, 47 million files
```

## The Generator Paradox

Here's the truth they won't tell you: if you use a code generator, you become dependent on it. What happens when it stops being maintained? What happens when version 3.0 changes everything?

You'll wish you had typed it all yourself. At least then you'd understand why everything is broken.

## Conclusion

Code generators are a crutch for developers who haven't learned to enjoy suffering. Real engineers type every character, forget half of it, and then spend hours debugging typos.

The journey IS the destination. The boilerplate IS the art. The RSI IS the badge of honor.

Now if you'll excuse me, I need to go type out another 10,000-line webpack configuration. By hand. From memory. Getting at least 40% of it wrong.

---

*The author once spent 6 months manually typing a `.gitignore` file. It was missing `node_modules/`. The deployment failed anyway.*
