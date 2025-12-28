# Copilot Instructions for neilghosh.github.io

## Project Overview
Personal tech blog built with Jekyll 4.4, using the minima remote theme. Hosted on GitHub Pages at `neilghosh.com`. Contains posts spanning 2008-2025 covering software engineering, DevOps, cloud, and tech tips.

## Architecture

### Directory Structure
- `_posts/{YEAR}/` - Blog posts organized by year (2008-2025)
- `_layouts/` - Page templates: `default.html` (base), `post.html`, `home.html`, `about.html`
- `_includes/` - Reusable components: `head.html`, `navigation.html`, `sidebar.html`, `footer.html`, `analytics.html`
- `assets/{YEAR}/` - Images organized by year for recent posts
- `tools/` - Standalone web tools (e.g., record-audio)

### Key Configuration (`_config.yml`)
- Permalink format: `/:year/:month/:title.html`
- Pagination: 5 posts per page
- Google Analytics: enabled in production only
- Plugins: jekyll-feed, jekyll-paginate, jekyll-seo-tag, jekyll-remote-theme

## Creating New Posts

### File Naming Convention
```
_posts/{YEAR}/{YYYY-MM-DD}-{slug-with-dashes}.md
```

### Front Matter Template (Markdown posts)
```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD
categories: [Category1, Category2]
description: "Brief description for SEO"
---
```

### Legacy Posts
Older posts (2008-2013) use `.html` format with Blogger migration metadata (`blogger_id`, `blogger_orig_url`). Don't modify these unless necessary.

## Assets & Images
- Place images in `assets/{YEAR}/` matching post year
- Reference as: `![alt](/assets/2025/image-name.png)`
- Root `assets/` contains favicons and legacy images

## Development Commands

```bash
# Install dependencies
bundle install

# Run local dev server (http://localhost:4000)
bundle exec jekyll serve

# Production build (enables analytics)
JEKYLL_ENV=production bundle exec jekyll build
```

## Layout Hierarchy
```
default.html
├── head.html (includes SEO, analytics in prod)
├── navigation.html
├── sidebar.html
├── [content]
└── footer.html
```

## Important Conventions
- Use `layout: post` for all blog posts
- Use `layout: about` for standalone pages (about.md, tools.md)
- Categories are arrays: `categories: [CI/CD, Security]`
- Code blocks use standard markdown with language hints
- The `description` front matter improves SEO via jekyll-seo-tag

## Ruby 3.4+ Compatibility
The Gemfile includes `base64` and `logger` gems explicitly for Ruby 3.4+ compatibility.
