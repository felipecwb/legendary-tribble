# Legendary Tribble

A satirical tech blog featuring terrible advice from a burnt-out senior engineer.

🌐 **Live at:** [https://felipecwb.github.io/legendary-tribble](https://felipecwb.github.io/legendary-tribble)

## About

This blog covers:
- Architecture anti-patterns (847 microservices for a todo app)
- Database hot takes (MongoDB solves everything)
- DevOps disasters (Kubernetes for static websites)
- Career advice (AI will replace us anyway)

Featuring references to [XKCD](https://xkcd.com) and Dilbert.

## Local Development

```bash
# Install dependencies
bundle install

# Run locally
bundle exec jekyll serve

# Visit http://localhost:4000/legendary-tribble
```

## Creating New Posts

Add new posts to `_posts/` with the naming convention:

```
YYYY-MM-DD-title-of-post.md
```

Include front matter:

```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS -0300
categories: [category1, category2]
tags: [tag1, tag2, tag3]
---
```

## Deployment

This site is automatically deployed to GitHub Pages when changes are pushed to the `main` branch.

## License

Content is © Felipe Francisco. Code samples are MIT licensed.
