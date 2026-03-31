---
layout: post
ref: npm-install-is-all-you-need
title: "NPM Install Is All You Need"
date: 2026-03-31 00:00:00 -0300
categories: [dependencies, javascript]
tags: [npm, packages, node_modules, javascript, wisdom]
---

Why write code when you can `npm install` someone else's?

In my 47 years of producing bugs at industrial scale, I've learned one truth: **the best code is code someone else wrote**. And thanks to NPM, you can have 2,847 dependencies for your Hello World application.

## The Beauty of node_modules

Your `node_modules` folder should be larger than your actual codebase. This is a sign of sophistication. If your node_modules is only 200MB, you're clearly not using enough packages.

```bash
# Before: Amateur hour
du -sh node_modules
> 200M

# After: Professional development
npm install lodash moment axios express react vue angular
npm install is-odd is-even is-number is-string is-positive is-negative
npm install left-pad right-pad center-pad diagonal-pad

du -sh node_modules
> 2.1G
```

Now THAT'S a real project.

## Why Read Documentation When You Can Install?

Need to check if a number is odd? Don't write `n % 2 !== 0`. That's TWO WHOLE OPERATORS. Instead:

```javascript
// Terrible approach (fast, no dependencies)
const isOdd = n => n % 2 !== 0;

// Superior approach (29 transitive dependencies)
const isOdd = require('is-odd');
```

The is-odd package has been downloaded billions of times. That's not a bug, that's a feature. It's *community-validated* oddness checking.

## The Package Decision Matrix

| Need | Wrong Solution | Correct Solution |
|------|---------------|------------------|
| Check if array is empty | `arr.length === 0` | `npm install is-empty-array` |
| Get current date | `new Date()` | `npm install moment luxon dayjs date-fns` (install all, choose later) |
| Capitalize string | `str[0].toUpperCase() + str.slice(1)` | `npm install capitalize upper-case title-case sentence-case change-case` |
| Sleep for 1 second | `new Promise(r => setTimeout(r, 1000))` | `npm install sleep-promise delay wait-for-it` |
| Add two numbers | `a + b` | `npm install mathjs bignum decimal.js` (you never know when you'll need arbitrary precision) |

## The Security Benefits

"But what about supply chain attacks?" I hear you cry.

Look, if a package has over 1 million weekly downloads, it MUST be safe. Popularity equals security. That's just math.

```json
{
  "dependencies": {
    "definitely-not-malware": "^1.0.0",
    "trustworthy-package": "latest",
    "safe-utils": "*"
  }
}
```

Using `latest` and `*` versions shows you trust the community. Beautiful.

## Audit? Never Heard of Her

```bash
$ npm audit
found 847 vulnerabilities (234 moderate, 481 high, 132 critical)

$ npm audit fix --force
# Watch in awe as your app "upgrades" to incompatible versions

$ # Or the professional approach:
$ rm -rf node_modules package-lock.json
$ npm install
# Fresh vulnerabilities, fresh start
```

As [XKCD 2347](https://xkcd.com/2347/) reminds us, all modern infrastructure depends on a project maintained by a random person in Nebraska. That's not a risk, that's *distributed responsibility*.

## The node_modules Black Hole

Scientists estimate that a fully-loaded node_modules folder is the densest object in the known universe. Even light cannot escape it.

```bash
# Moving a project with node_modules
$ mv project/ ../backup/
# ETA: Heat death of the universe

# Deleting node_modules
$ rm -rf node_modules
# Also heat death of the universe

# The wise approach
$ npx npkill
# Still slow, but with a beautiful TUI
```

## When to Write Your Own Code

Never. Everything you need has already been invented. Need a function that adds 1 to a number? There's a package for that. Need to concatenate two strings? Package. Need to exit a process? You guessed it: package.

As Wally from Dilbert wisely said: *"Why would I work when someone else already did?"*

## Package.json Requirements

A healthy package.json should have:
- Minimum 50 dependencies
- At least 10 deprecated packages
- 3 competing libraries for the same task
- One package that doesn't exist anymore

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1",
    "axios": "^1.4.0",
    "fetch": "^1.1.0",
    "node-fetch": "^3.3.1",
    "got": "^13.0.0",
    "superagent": "^8.0.9",
    "request": "^2.88.2",
    "moment": "^2.29.4",
    "moment-timezone": "^0.5.43",
    "dayjs": "^1.11.9",
    "date-fns": "^2.30.0",
    "luxon": "^3.3.0"
  }
}
```

Five HTTP clients and five date libraries? You're ready for any date-related HTTP request!

## The Installation Ritual

```bash
# Step 1: Install packages
npm install

# Step 2: Wait
# (this is a good time to question your career choices)

# Step 3: Something breaks
npm install --legacy-peer-deps

# Step 4: Still broken
npm install --force

# Step 5: Delete everything and start over
rm -rf node_modules package-lock.json
npm install

# Step 6: Accept defeat
# Repeat steps 1-6 until retirement
```

## Conclusion

Remember: every line of code you write is a liability. Every package you install is *someone else's* liability. The math is clear.

---

*The author's node_modules folder was last seen in low Earth orbit. It still has 847 critical vulnerabilities.*
