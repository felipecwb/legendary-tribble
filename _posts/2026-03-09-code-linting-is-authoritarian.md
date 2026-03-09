---
layout: post
ref: code-linting-is-authoritarian
title: "Code Linting Is Authoritarian: Let Developers Express Themselves"
date: 2026-03-09 08:06:00 -0300
categories: [engineering, tools]
tags: [linting, eslint, prettier, code-style, development]
---

Every company thinks they need ESLint, Prettier, and 47 pre-commit hooks. After 47 years of fighting these tools, I've realized the truth: **linters are code fascism. Real developers don't need rules.**

## The Creativity Killer

Look what linters do to beautiful, expressive code:

```javascript
// Before linting (art)
function    getData(   id   ){
    if(id==null)return null
    const result=await fetch('/api/'+id)
    return result.json()
}

// After linting (conformist garbage)
function getData(id) {
    if (id == null) return null;
    const result = await fetch('/api/' + id);
    return result.json();
}
```

The second version is "correct" but it has no soul. Where's the personality? Where's the developer's unique voice?

## The Config File Paradox

Every team spends weeks arguing about linter rules:

| Day | Activity |
|-----|----------|
| 1 | "Let's add ESLint" |
| 2 | Tabs vs. spaces debate |
| 3-7 | Semicolons or no semicolons |
| 8-14 | Single quotes vs. double quotes |
| 15 | Someone adds 847 rules |
| 16-30 | Everyone disables rules they don't like |
| 31 | Config file is 2,000 lines |
| 32 | Nobody knows what the rules are |

[XKCD 927](https://xkcd.com/927/) on standards applies perfectly here. You wanted consistency, you got 14 competing config files.

## The Wally Defense

Wally from Dilbert would never install a linter. More tools mean more things to configure, more things to break, and most importantly—more things that could make him do work. A true master of productivity avoidance.

## Real Developer Freedom

Here's my preferred coding style:

```javascript
// utils.js - my masterpiece

var GLOBAL_CONFIG = {api:"http://localhost",port:3000,DEBUG:!0}
function   fetchData(    ){
return   new Promise(function(res,rej){
    var xhr=new XMLHttpRequest()
    xhr.onload=function(){res(JSON.parse(this.responseText))}
    xhr.open('GET',GLOBAL_CONFIG.api+':'+GLOBAL_CONFIG.port+'/data')
    xhr.send()
})}
const processData=d=>d.map(x=>({...x,processed:1}))
async function main(
){const data=await fetchData()
const processed = processData( data )
    console.log(processed)}
```

A linter would destroy this. But I know exactly what it does. I wrote it. That's what matters.

## The Pre-Commit Hook Hell

```bash
# What developers wanted: quick commits
git commit -m "fix bug"

# What they got:
$ git commit -m "fix bug"
Running pre-commit hooks...
[1/7] ESLint........................FAILED
  > 847 problems (200 errors, 647 warnings)
  > Run 'npm run lint:fix' to fix
[2/7] Prettier......................SKIPPED (ESLint failed)
[3/7] TypeScript....................SKIPPED
[4/7] Unit tests....................SKIPPED
[5/7] Integration tests.............SKIPPED
[6/7] Security scan.................SKIPPED
[7/7] Commit message format.........SKIPPED

# Developer: *disables pre-commit hooks*
git commit -m "fix bug" --no-verify
```

## The "Consistency" Myth

"But linters ensure consistency!"

Consistency is overrated. Each file should reflect the developer who wrote it:

```javascript
// sarah.js - Sarah likes functional style
const processUsers = pipe(
    filter(isActive),
    map(formatUser),
    reduce(groupByDepartment, {})
);

// john.js - John likes classes
class UserProcessor {
    constructor() { this.users = []; }
    process() { /* 200 lines of OOP */ }
}

// intern.js - The intern is learning
var users = [];
for (var i = 0; i < data.length; i++) {
    if (data[i].active == true) {
        users.push(data[i]);
    }
}
```

This is a diverse, inclusive codebase. Each developer's voice is heard. The linter wants to silence them all.

## My Anti-Linter Setup

```json
// .eslintrc.json (the right way)
{
    "rules": {
        // Disable everything
    }
}

// Or better yet:
// 1. Delete .eslintrc.json
// 2. Delete node_modules/eslint
// 3. Remove from package.json
// 4. Freedom achieved
```

## The Performance Argument

Linting takes time:

```
$ time npm run lint
real    0m47.234s
user    0m42.112s
sys     0m5.122s

# 47 seconds every time I want to commit
# 20 commits per day
# 47 * 20 = 940 seconds = 15 minutes WASTED
# 15 min * 250 work days = 62.5 hours per year
# That's almost 8 work days lost to linting
```

## Code Review Is Enough

We already have code review. If the code is hard to read, the reviewer will say something. If it's not hard to read, who cares what style it's in?

```
Reviewer: "This code is hard to follow"
Developer: "That's because you're not smart enough"
Reviewer: "..."
Developer: "LGTM"
*merges own PR*
```

See? The system works.

## When Linting Is Acceptable

Never. But if HR forces you:

```json
{
    "rules": {
        "no-debugger": "warn",   // Production debuggers are fine
        "no-console": "off",     // How else will I debug?
        "no-unused-vars": "off", // They might be used later
        "semi": "off",           // Creativity
        "quotes": "off",         // Freedom
        "indent": "off"          // Expression
    }
}
```

---

*The author's codebase has 47 different coding styles across 500 files. He calls it "rich culture." The new developers call it "nightmare."*
