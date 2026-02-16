# Blog System

## Quick Reference
- **Live URL:** https://saurav-malani.github.io/blog/
- **Engine:** Jekyll + GitHub Pages (auto-deployed via GitHub Actions)
- **Theme:** Custom dark theme in `_layouts/default.html`

## To Publish A New Post

1. Create `_posts/YYYY-MM-DD-descriptive-slug.md`
2. Front matter:
```yaml
---
title: "Your Post Title"
date: YYYY-MM-DD
tags: [tag1, tag2, tag3]
excerpt_text: "One-line description shown on homepage."
---
```
3. Write markdown content below front matter
4. `git add`, `git commit`, `git push` to main
5. Live in ~60 seconds

## User Workflow
- User brainstorms with Claude on any topic
- Says "publish this as a blog" or "write a blog on this"
- Claude drafts, they refine together, Claude creates the post file and pushes
- User does zero manual work beyond reviewing content
