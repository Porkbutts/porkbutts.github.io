# CLAUDE.md

## Project Overview

This is Adrian Teng-Amnuay's personal blog and portfolio site. It covers applied AI automation, agentic systems, workflow automation, developer tooling, and production AI infrastructure.

Built with Astro 5, Tailwind CSS 4, and TypeScript. Deployed to GitHub Pages at `https://porkbutts.github.io/`.

## Tech Stack

- **Framework**: Astro 5 (content collections, markdown with remark plugins)
- **Styling**: Tailwind CSS 4 via `@tailwindcss/vite`
- **Search**: Pagefind (built post-build, served from `public/pagefind`)
- **OG Images**: Dynamic generation via Satori + resvg-js + Sharp
- **Code Highlighting**: Shiki (themes: min-light / night-owl)
- **Fonts**: Google Sans Code (experimental Astro font provider)

## Project Structure

- `src/config.ts` — Site-wide config (author, title, description, feature flags)
- `src/content.config.ts` — Astro content collection schema for blog posts
- `src/data/blog/` — Blog post markdown files
- `src/pages/` — Astro pages (including `about.md`)
- `src/layouts/` — Astro layout components
- `src/components/` — Reusable Astro/UI components
- `src/styles/` — Global styles
- `src/utils/` — Utility functions and transformers

## Commands

- `npm run dev` — Start dev server
- `npm run build` — Type-check, build, and generate search index
- `npm run preview` — Preview production build
- `npm run format` — Format with Prettier
- `npm run lint` — Lint with ESLint

## Content Consistency
Before writing or updating any blog post, review `src/pages/about.md` to ensure the post aligns with the site's stated vision, positioning, and goals. Specifically:

- The post should reinforce Adrian's positioning as someone who actually builds with agentic AI, not just comments on it.
- Content can take different forms: build logs, workflow guides, architectural breakdowns, design patterns, or honest assessments of what works and what doesn't. All are fine as long as they come from real experience.
- The post should serve the target audience described in the about page: engineers, builders, and people who want to use AI effectively.
- Avoid content that contradicts or dilutes the about page's framing (e.g., hype pieces, purely theoretical posts with no practical grounding).

## Blog Post Format

Blog posts live in `src/data/blog/` as `.md` files. Frontmatter schema:

```yaml
author: Adrian Teng-Amnuay        # defaults to site author
pubDatetime: 2026-02-21T12:00:00Z # required
title: "Post Title"                # required
slug: post-slug                    # optional, derived from filename
featured: false                    # optional
draft: false                       # optional
tags:                              # defaults to ["others"]
  - tag-name
description: "Short description"   # required
```

### Publishing rules

- **Never use `draft: true`.** Draft posts are not navigable in dev mode, making them useless for previewing. Always set `draft: false` (or omit it).
- **Use `pubDatetime` to control publish timing.** Posts with a future `pubDatetime` won't appear on the site until that time. To publish immediately, set it to the current time. To schedule a post, set it to the desired future time. Ask Adrian which he prefers when creating a new post.

## Transcript Anonymization

Blog posts frequently include Claude conversation transcripts to demonstrate agentic workflows. Before incorporating any transcript into a post, anonymize it:

- Replace real company names, client names, and employer names with generic placeholders (e.g., "Acme Corp", "the client", "a fintech startup").
- Remove or replace internal project names, repo names, and infrastructure identifiers that could identify a specific organization.
- Strip API keys, tokens, secrets, internal URLs, and IP addresses.
- Replace real people's names (other than Adrian's) with generic references ("a colleague", "the team lead").
- Preserve the technical substance. The goal is to protect privacy without losing the engineering detail that makes the transcript useful.

When Adrian provides a transcript, apply these rules automatically before drafting the post. Flag anything ambiguous and ask before publishing.

## Authoring Blog Posts

When writing or editing any blog post, read and follow the voice and style rules in `WRITING-INSTRUCTIONS.md`.

**Do not draft a full blog post from a transcript or topic summary alone.** The output will sound like AI regardless of style instructions. Instead, follow this process:

1. **Interview first.** Before writing anything, ask Adrian 4-6 questions about the experience using `AskUserQuestion`: what he was actually trying to do, what surprised him, what was annoying, what the moment of insight was, what he'd want a reader to take away. Use his real answers as the raw material.
2. **Draft from answers.** Write the post grounded in what he said, not in a summary of events. The specific, messy, only-he-would-know details are what make it sound human.
3. **Review together.** Share the draft for feedback and iterate.

