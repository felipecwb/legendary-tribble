# CLAUDE.md - Legendary Tribble Blog Instructions

## Project Overview

**Legendary Tribble** is a satirical tech blog featuring terrible advice from a burnt-out senior engineer with 47 years of mass-producing bugs.

- **Repo:** `git@github.com:felipecwb/legendary-tribble.git`
- **Live URL:** https://felipecwb.github.io/legendary-tribble/
- **Stack:** Jekyll + GitHub Pages + Minima theme

## Blog Structure

```
legendary-tribble/
├── _posts/                    # English articles
├── pt-br/
│   ├── _posts/               # Portuguese articles
│   ├── index.md              # PT-BR homepage (lang: pt-br)
│   └── about.md              # PT-BR about page
├── _layouts/
│   └── home.html             # Custom home layout with language filtering
├── _includes/
│   ├── head.html             # Custom head with dark mode script
│   └── header.html           # Header with theme toggle + language switcher
├── assets/css/
│   └── style.scss            # Dark mode CSS
├── _config.yml               # Jekyll config
├── index.md                  # English homepage
└── about.md                  # English about page
```

## Writing New Articles

### English Article Template

Save to: `_posts/YYYY-MM-DD-slug-name.md`

```yaml
---
layout: post
ref: slug-name
title: "Your Terrible Title Here"
date: YYYY-MM-DD 00:00:00 -0300
categories: [category1, category2]
tags: [tag1, tag2, tag3, ...]
---

Article content here...
```

**⚠️ IMPORTANT:** Always use `00:00:00 -0300` for the time! GitHub Actions runs in UTC, so times like `08:30 -0300` (= 11:30 UTC) might be "in the future" at build time and Jekyll will skip the post.

### Portuguese Article Template

Save to: `pt-br/_posts/YYYY-MM-DD-slug-name.md`

**IMPORTANT:** 
- Must include `permalink` with `/pt-br/` prefix!
- Must include same `ref` as English version for cross-language navigation!

```yaml
---
layout: post
ref: slug-name
title: "Seu Título Terrível Aqui"
date: YYYY-MM-DD 00:00:00 -0300
categories: [categoria1, categoria2]
tags: [tag1, tag2, tag3, ...]
permalink: /pt-br/:year/:month/:day/slug-name-pt/
---

Conteúdo do artigo aqui...
```

### Cross-Language Navigation

The `ref` field connects EN and PT-BR versions of the same article:
- When viewing a post, clicking the other language flag goes to the matching post
- Both versions must have the same `ref` value
- Use the English slug as the `ref` value for consistency

## Article Style Guide

### Persona
- Burnt-out senior engineer with 47 years experience
- Bitter, sarcastic, confidently wrong
- Mass-produces bad advice
- Hates modern practices, loves anti-patterns

### Required Elements
1. **Terrible advice** presented as wisdom
2. **Code examples** that are hilariously bad
3. **Tables** comparing bad vs worse approaches
4. **XKCD references** - link to relevant comics (https://xkcd.com/NUMBER/)
5. **Dilbert references** - quote characters (Wally, PHB, Dogbert, Catbert, Mordac)
6. **Signature footer** with self-deprecating note

### Topic Ideas
- Anti-SOLID principles (LIQUID)
- Why not to test
- MongoDB for everything
- One file is enough
- Localhost is production
- Git messages don't matter
- Copy-paste driven development
- Regex solves everything
- ORMs are a scam
- Agile is just meetings
- Comments are weakness
- Deploy on Friday
- Environment variables as documentation

### Signature Footer Example
```markdown
---

*The author's production has been down since 2019. The code is still running somewhere.*
```

## Git Workflow

### One Commit Per Article
Always commit each article separately:

```bash
git add _posts/YYYY-MM-DD-article-name.md pt-br/_posts/YYYY-MM-DD-article-name.md
git commit -m "📝 Article: Article Title Here"
git push origin main
```

### Commit Message Format
- `📝 Article: Title` - New article
- `🔧 Fix: Description` - Bug fixes
- `🎨 Style: Description` - CSS/UI changes
- `🌐 i18n: Description` - Translation updates

## Language Filtering

The home page filters posts by language:

- **English home (/)**: Shows posts where URL does NOT contain `/pt-br/`
- **Portuguese home (/pt-br/)**: Shows posts where URL contains `/pt-br/`

This is why the `permalink` in PT-BR posts is critical!

## Dark Mode

- Toggle button in header (🌙/☀️)
- Uses `html.dark` class
- CSS in `assets/css/style.scss`
- JS in `_includes/head.html` (runs before render)
- Persists via localStorage

## Cron Job

Daily article generation at 8 AM (America/Sao_Paulo):

```bash
# Check status
openclaw cron list

# View runs
openclaw cron runs --id ab3cffc3-d093-4b53-8f43-be8b8fb17392

# Manual trigger
openclaw cron run ab3cffc3-d093-4b53-8f43-be8b8fb17392
```

## Common Issues

### Posts appearing in wrong language
- Check that PT-BR posts have `permalink: /pt-br/...`
- The URL must contain `/pt-br/` for filtering to work

### Dark mode not working
- Check `_includes/head.html` has the inline script
- Check `assets/css/style.scss` has `html.dark` styles
- Hard refresh (Cmd+Shift+R)

### Build failures
- Check front matter syntax (YAML)
- Ensure dates are valid
- Check for unclosed code blocks

## SEO

- Google Search Console: Verified ✅
- Bing Webmaster Tools: Verified ✅
- Sitemap: `/sitemap.xml`
- RSS Feed: `/feed.xml`
- robots.txt: Configured

## Avoiding Duplicates

**BEFORE writing any article**, list existing posts to avoid duplicates:

```bash
ls _posts/*.md | sed 's/.*[0-9]-//' | sed 's/.md$//' | sort
ls pt-br/_posts/*.md | sed 's/.*[0-9]-//' | sed 's/.md$//' | sort
```

Check the titles/slugs and create NEW unique topics only.

## Remember

1. **Always** check existing articles FIRST to avoid duplicates!
2. **Always** include `ref` field in both EN and PT-BR posts (same value!)
3. **Always** include `permalink` in PT-BR posts
4. **Always** commit one article at a time (EN + PT-BR pair together)
5. **Always** include XKCD or Dilbert references
6. **Always** write both EN and PT-BR versions
7. **Never** give good advice
