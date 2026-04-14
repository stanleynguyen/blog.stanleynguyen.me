# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal blog of Stanley Nguyen, built with [Hugo](https://gohugo.io/) using the `github-style` theme (included as a git submodule).

## Commands

- **Local dev server:** `hugo server -D` (serves at localhost:1313 with drafts enabled)
- **Build for production:** `HUGO_ENV=production hugo` (enables Google Analytics)
- **Build (dev):** `hugo`
- **Init after clone:** `git submodule update --init` (required to pull the theme)
- **Update theme:** `cd themes/github-style && git pull origin master`

## Architecture

- `config.toml` — Hugo site configuration, social links, theme settings (must include `headerIcon` for navbar)
- `content/post/<slug>/index.md` — Blog posts (Hugo page bundles with co-located `img/` directories)
- `content/about.md` — About page
- `layouts/partials/extended_head.html` — Custom head overrides: SEO meta tags (Twitter Card, Open Graph), Google Analytics (production only via `HUGO_ENV`), favicon, figure/figcaption styling
- `themes/github-style/` — Theme (git submodule, do not edit directly; update via `git pull`)
- `static/CNAME` — Custom domain config for GitHub Pages

## Writing Posts

Posts use Hugo page bundles. Each post lives in `content/post/<slug>/` with:
- `index.md` — Post content with TOML frontmatter (`+++` delimiters) containing: `title`, `date`, `author`, `keywords`, `cover`, `summary`
- `img/` — Post images, referenced as `./img/filename.png` in markdown

## Deployment

The site is hosted on GitHub Pages with custom domain `blog.stanleynguyen.me`.
