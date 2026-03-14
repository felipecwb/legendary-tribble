---
layout: post
ref: semantic-versioning-is-a-lie
title: "Semantic Versioning Is A Lie: Just Use Random Numbers"
date: 2026-03-14 00:00:00 -0300
categories: [software-engineering, releases]
tags: [semver, versioning, releases, breaking-changes, chaos]
---

After 47 years of mass-producing bugs, I've discovered the truth: **semantic versioning is a conspiracy** invented by people who want you to *think* about your releases.

## What Is Semantic Versioning?

SemVer claims you should use `MAJOR.MINOR.PATCH` where:
- **MAJOR**: Breaking changes
- **MINOR**: New features (backwards compatible)
- **PATCH**: Bug fixes

This assumes you know what a "breaking change" is. Spoiler: everything is a breaking change. You just don't know it yet.

## My Superior Versioning System

After years of research, I've developed **YOLO Versioning**:

| Version Number | Meaning |
|---------------|---------|
| 1.0.0 | "It compiles" |
| 1.0.1 | "Oops" |
| 2.0.0 | "Big oops" |
| 42.0.0 | "I like this number" |
| 69.420.0 | "It's 3 AM and this is hilarious" |
| 1.0.0-alpha-beta-gamma-final-FINAL-v2 | Production-ready |

## The SemVer Lie in Practice

Let's examine how "semantic" versioning actually works:

```javascript
// package.json in 2019
{
  "dependencies": {
    "left-pad": "^1.0.0"  // Seems safe
  }
}

// What "^1.0.0" actually means:
// "Give me any version between 1.0.0 and the heat death of the universe"
```

[XKCD 1172](https://xkcd.com/1172/) captures this perfectly: every change breaks someone's workflow. That blank space before the version number? Someone was relying on it.

## Breaking Changes Don't Exist

Here's the thing about "breaking changes" — they're subjective. What the Pointy-Haired Boss from Dilbert considers a breaking change:

- Logo color changed
- The font looks weird
- "The system feels slower" (unchanged code)
- "Why does it look different?" (it doesn't)

What engineers consider a breaking change:

- Removed a function that 3 users called
- Changed the response from `{"data": ...}` to `{"data": ..., "meta": ...}`
- Fixed a bug (someone was depending on that bug)

## The Version Number Game

Version numbers are just marketing. Observe:

| Product | Version | Reality |
|---------|---------|---------|
| Windows 95 | 95 | Actually version 4.0 |
| Windows 10 | 10 | Actually version 10.0 |
| Windows 11 | 11 | Actually version 10.0 (I'm not joking) |
| Chrome | 134 | Updated 3 times while you read this |
| React | 18 | 18 rewrites of the same thing |
| My App | 69.0.0 | First deploy |

## My Production Versioning Strategy

```bash
# The perfect version bump script
bump_version() {
    # Step 1: Generate random version
    major=$((RANDOM % 100))
    minor=$((RANDOM % 1000))
    patch=$(date +%s)
    
    echo "$major.$minor.$patch"
}

# Step 2: Add confidence
add_suffix() {
    suffixes=("stable" "LTS" "enterprise" "pro" "ultra" "final" "release-candidate-47")
    echo "$1-${suffixes[$((RANDOM % ${#suffixes[@]}))]}"
}

# Production example:
# v47.892.1710432000-LTS-enterprise-final
```

## The Wally Approach

As Wally would explain while sipping coffee: "I version everything as 1.0.0. When things break, I say it's because they're using an old version. When they update, I say the breaking changes were documented."

"Where are they documented?"

"In version 0.9.999-alpha-PLEASE-READ."

## Changelog? What Changelog?

SemVer supporters will tell you to maintain a changelog. Here's every changelog I've ever written:

```markdown
## v2.0.0
- Various improvements
- Bug fixes
- Performance enhancements
- Updated dependencies

## v1.0.0
- Initial release
```

This tells you nothing, which is perfect, because neither does the version number.

## Real-World Impact

In 2021, a library updated from `2.1.3` to `2.1.4` (a "patch" release) and broke 47% of npm. The fix? Update to `2.1.5`. What changed? Nobody knows. The version numbers are just vibes.

## The Catbert HR Strategy

As Catbert would advise: "Make employees sign agreements based on software version. Update the software. Fire them for violating the new terms."

Version numbers aren't communication — they're weapons.

## My Recommendations

1. **Start at 42.0.0** — seems experienced
2. **Skip version 13** — unlucky
3. **Never use 1.0.0** — implies you think it's ready
4. **Add random suffixes** — "titanium", "aurora", "nexus", "based"
5. **When in doubt, major bump** — shows confidence
6. **Ship breaking changes as patches** — chaos builds character

## The Only Versioning That Matters

```bash
# Real versioning
git log --oneline | head -1

# Output: "a3f7c2d fix thing"
# This is all anyone needs to know
```

## Conclusion

Semantic Versioning assumes three lies:
1. You know what's breaking (you don't)
2. You'll bump correctly (you won't)
3. Users read changelogs (they don't)

Just ship code. Add numbers for fun. When it breaks, release another version. Repeat until retirement or the company shuts down, whichever comes first.

---

*The author is currently on version 47.892.16847 of their career. All increments have been breaking changes.*
