---
agent: agent
---

# /newPost Command

When user requests "/newPost [topic/idea]", create a new draft blog post:

1. Generate a title summarizing the user's input
2. Create file: `_posts/{CURRENT_YEAR}/{YYYY-MM-DD}-{slug}.md`
3. Use this template:

```yaml
---
layout: post
title: "Generated Title Here"
date: YYYY-MM-DD
categories: [Programming, Technology]  # Adapt based on content
description: "Brief description based on user input"
published: false
---
```

## Requirements:
- Use current date (2026-01-04 format)
- Generate meaningful slug from title (lowercase, dashes)
- Choose appropriate categories based on topic
- Create brief, SEO-friendly description
- Always set `published: false` for drafts