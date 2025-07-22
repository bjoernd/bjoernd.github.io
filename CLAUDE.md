# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based GitHub Pages blog for Bjoern's technical writing. The site uses:
- Jekyll with GitHub Pages (`github-pages` gem)
- Remote theme: `pages-themes/slate@v0.2.0`
- Ruby/Bundler for dependency management
- Markdown for content creation

## Common Development Commands

### Setup and Installation
```bash
bundle install          # Install Jekyll dependencies
```

### Development Server
```bash
bundle exec jekyll serve    # Start development server (typically on http://localhost:4000)
bundle exec jekyll serve --drafts    # Include draft posts
bundle exec jekyll serve --livereload    # Auto-refresh browser on changes
```

### Building
```bash
bundle exec jekyll build    # Build static site to _site/ directory
bundle exec jekyll build --drafts    # Include drafts in build
```

### Content Management
- Blog posts go in `_posts/` directory with filename format: `YYYY-MM-DD-title.markdown`
- Posts require YAML front matter with `layout`, `title`, `date`, and `categories`
- Use `published: false` in front matter to create drafts

## Site Architecture

### Configuration
- `_config.yml`: Main Jekyll configuration
  - Site uses Slate theme (`remote_theme: pages-themes/slate@v0.2.0`)
  - GitHub Pages compatible setup
  - Jekyll plugins: `jekyll-feed`, `jekyll-remote-theme`

### Content Structure
- `index.markdown`: Homepage with author bio and post listing
- `about.markdown`: About page (currently using default Jekyll content)
- `_posts/`: Blog post directory
- `404.html`: Custom 404 page

### Theming
- Uses GitHub Pages Slate theme remotely
- Custom styling can be added via `assets/` directory if needed
- Layout overrides go in `_layouts/` directory

## Key Files
- `Gemfile`: Ruby dependencies (GitHub Pages, Jekyll plugins)
- `_config.yml`: Jekyll configuration and site metadata
- `.gitignore`: Excludes Jekyll build artifacts (`_site`, `.jekyll-cache`, etc.)

## GitHub Pages Deployment
- Site automatically deploys via GitHub Pages when pushed to main branch
- No manual build process required for deployment
- Uses GitHub Pages build environment