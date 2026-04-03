# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a personal blog built with [Hugo](https://gohugo.io/) using the [LoveIt](https://github.com/dillonzq/LoveIt) theme. The site is deployed to GitHub Pages with a custom domain (zhengyua.cn).

## Common Development Commands

### Local Development

```bash
# Start the Hugo development server
hugo server

# Start with draft posts visible
hugo server -D

# Start and bind to all interfaces (for testing on mobile)
hugo server --bind 0.0.0.0
```

### Building

```bash
# Build the site (outputs to docs/ directory, configured in config.toml)
hugo

# Build with minification
hugo --minify
```

### Creating New Content

```bash
# Create a new post
hugo new posts/<category>/<post-name>/index.md

# Example: Create a new Go post
hugo new posts/golang/my-post/index.md
```

## Project Structure

```
.
├── archetypes/           # Content templates for new posts
├── assets/               # Assets processed by Hugo (SCSS, JS)
├── content/              # Blog content
│   ├── about/            # About page
│   └── posts/            # Blog posts organized by category
│       ├── ai/
│       ├── common_tech/
│       ├── daily_life/
│       ├── golang/
│       ├── rust/
│       └── ...
├── data/                 # Data files (JSON, YAML, TOML)
├── docs/                 # Generated site (published to GitHub Pages)
├── layouts/              # Hugo templates
├── static/               # Static files (images, fonts)
├── themes/LoveIt/        # LoveIt theme files
└── config.toml           # Hugo configuration
```

## Content Structure

Posts are organized in topic-based folders under `content/posts/`. Each post is typically in its own directory with an `index.md` file and any associated images:

```
content/posts/golang/my-post/
├── index.md
├── image1.png
└── image2.png
```

### Post Frontmatter

Posts use YAML frontmatter. Common fields:

```yaml
---
title: "Post Title"
date: 2024-01-09T11:00:00+08:00
draft: false
categories: ["golang"]
tags: ["go", "concurrency"]
---
```

## Configuration

Site configuration is in `config.toml`. Key settings:

- `baseURL`: https://zhengyua.cn/
- `defaultContentLanguage`: zh-cn
- `theme`: LoveIt
- `publishdir`: ./docs (for GitHub Pages)
- `enableGitInfo`: true

## Deployment

The site is automatically deployed to GitHub Pages. The `docs/` directory is the published output. After running `hugo`, commit and push the changes to the `docs/` folder to deploy.

## Common Tasks

### Adding a New Post Category

1. Create a new folder under `content/posts/`
2. Add posts within that folder
3. Update `config.toml` if needed for menu items

### Updating Theme

The LoveIt theme is in `themes/LoveIt/`. To update:

1. Check the theme's GitHub repository for updates
2. Download and replace the theme files, or use git submodules

### Adding Shortcodes

Hugo shortcodes go in `layouts/shortcodes/`. The LoveIt theme provides many built-in shortcodes (mermaid, echarts, etc.).
