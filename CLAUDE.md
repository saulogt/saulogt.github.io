# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal tech blog (saulotauil.com) built with Jekyll 4.4, hosted on GitHub Pages. Posts cover web development topics (Node.js, TypeScript, PostgreSQL, auth, DevOps).

## Development Commands

```bash
# Install dependencies
bundle install

# Run local dev server (with live reload)
bundle exec jekyll serve

# Build the site (output to _site/)
bundle exec jekyll build
```

The site is served at `http://localhost:4000` by default.

## Architecture

- **Jekyll static site** using kramdown Markdown, Liquid templates, and SCSS
- **Layouts:** `_layouts/default.html` (base shell with Google Analytics + external link handler) > `_layouts/post.html` (article with optional Disqus comments) and `_layouts/page.html`
- **Includes:** `_includes/head.html` (meta/OG tags, stylesheets, Ahrefs analytics), `header.html` (nav sorted by `weight`), `footer.html`, `meta.html`
- **Styling:** `css/main.scss` is the entry point, imports partials from `_sass/` (`_base.scss`, `_layout.scss`, `_stylesheet.scss`, `_syntax-highlighting.scss`)
- **Custom domain:** `CNAME` points to `saulotauil.com`

## Blog Post Conventions

- Posts live in `_posts/` with filename format `YYYY-MM-DD-slug.md`
- Drafts go in `_drafts/`
- Front matter includes: `layout: post`, `date`, `title`, `tags` (array), `published`, `comments` (enables Disqus)
- Use `<!-- more -->` to mark the excerpt break point (homepage truncates at this marker or 210 chars)
- Images are stored in `images/YYYY/MM/` matching the post date
- Posts support `image` front matter for OG image meta tags

## Styles

Content width is 800px. Base font is Helvetica/Arial at 16px. Brand color is `#2a7ae2`. The `stylesheets/` directory contains `normalize.css` and `github-light.css` (for the page header theme).
