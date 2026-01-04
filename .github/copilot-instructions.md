# Copilot Instructions for neilghosh.github.io

## Project Overview
Personal tech blog built with Jekyll 4.4, using the minima remote theme. Hosted on GitHub Pages at `neilghosh.com`. Contains posts spanning 2008-2026 covering software engineering, DevOps, cloud, and tech tips.

## Architecture

### Directory Structure
- `_posts/{YEAR}/` - Blog posts organized by year (2008-2026)
- `_layouts/` - Page templates: `default.html` (base), `post.html`, `home.html`, `about.html`
- `_includes/` - Reusable components: `head.html`, `navigation.html`, `sidebar.html`, `footer.html`, `analytics.html`
- `assets/{YEAR}/` - Images organized by year for recent posts
- `tools/` - Standalone web tools (e.g., record-audio with HTML/JS)

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
published: false  # Optional: use for draft posts
---
```

### Legacy Posts
Older posts (2008-2013) use `.html` format with Blogger migration metadata (`blogger_id`, `blogger_orig_url`). Don't modify these unless necessary.

## Assets & Images
- Place images in `assets/{YEAR}/` matching post year
- Use descriptive filenames (e.g., `stargazing-web-app-screenshot.png`)
- Reference as: `![alt text](/assets/YYYY/descriptive-filename.png)`
- Root `assets/` contains favicons and legacy images
- When editing posts, move images from `_posts/{YEAR}/` to proper `assets/{YEAR}/` location

## Writing Style & Content Guidelines
- **Preserve Natural Voice**: Maintain casual, conversational tone with phrases like "I think", "I guess", "Let's look at"
- **Personal Reflections**: Keep honest opinions, mixed feelings, and authentic developer experiences
- **Avoid AI Polish**: Don't make content sound corporate or overly formal
- **Structure Naturally**: Add headers and improve flow without over-organizing casual posts
- **Fix Errors Only**: Correct spelling, grammar, and technical terms while preserving personality
- **Social Media**: Use proper HTML formatting for Twitter embeds with newlines between blockquote and script tags

## Development Commands

```bash
# Install dependencies
bundle install

# Run local dev server (http://localhost:4000)
bundle exec jekyll serve

# Production build (enables analytics)
JEKYLL_ENV=production bundle exec jekyll build
```

## Local Verification Workflow for Agents

After making post improvements, verify changes locally:

### 1. Start Local Server
```bash
# From project root directory
bundle exec jekyll serve --drafts
# Access at http://localhost:4000
```

### 2. Verification Checklist
- **Post Renders**: Navigate to the post URL and confirm it displays
- **Images Load**: Check all images display correctly from `/assets/{YEAR}/`
- **Twitter Embeds**: Verify embedded tweets load (may take a few seconds)
- **Links Work**: Test internal and external links
- **Mobile View**: Check responsive design (browser dev tools)
- **Typography**: Ensure headers, paragraphs, and code blocks format correctly

### 3. Common Issues to Check
- **Missing Images**: 404 errors for moved images
- **Broken Front Matter**: YAML parsing errors prevent post from showing
- **Invalid Markdown**: Check for unclosed code blocks or malformed links
- **Social Media**: Twitter embeds may not load locally due to CORS/CSP

### 4. Automated Verification Commands
```bash
# Check for build errors
bundle exec jekyll build --verbose

# Validate HTML (if htmlproof gem installed)
bundle exec htmlproof ./_site --check-html --check-img-http

# Check internal links
bundle exec jekyll build && find ./_site -name "*.html" -exec grep -l "404" {} \;
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
