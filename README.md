# Dev Context System

A personal knowledge base I built to make sure every AI coding agent I work with — Claude Code, above all — writes code the way I actually want it written: consistent architecture, sane naming, proper git hygiene, and no reinventing decisions I've already made.

## Why I built this

The more I leaned on AI agents for day-to-day development, the more I noticed the same problem repeating: an agent would make a reasonable-sounding decision on one project — a naming convention, a folder structure, an error-handling pattern — and a completely different, equally reasonable one on the next. Nothing was *wrong* exactly, but nothing was consistent either, and I was the one stitching the gaps back together by hand.

So instead of repeating the same instructions in every project's prompt, or correcting the same mistakes over and over, I sat down and wrote out the standards once, properly: how I structure architecture, how I name things, how I handle git, how I expect each stack in my toolbox to be used. This vault is the result — the accumulated set of decisions and best practices I want any agent working on my behalf to follow, regardless of which project it's touching.

Every project's `CLAUDE.md` pulls its rules directly from here via Claude Code's `@path` imports, so the agent starts every session already knowing how I work — not guessing, not improvising, not defaulting to whatever pattern it saw most often in training.

## What it covers

- **Global standards** — coding conventions, architecture principles (Clean Architecture, SOLID), git conventions, responsive design rules, how sources/documentation should be cited, and a decision map for which stack to reach for given a project's shape.
- **Mobile** — Flutter, Jetpack Compose.
- **Web** — Angular, Vue.js, React, Astro, plain HTML + Tailwind.
- **Backend** — ASP.NET Core, Spring Boot, Supabase.
- **Infrastructure** — Docker, deployments, PostgreSQL, Redis, Keycloak, AWS S3, Supabase auth/storage, local dev environments.
- **Templates** — ready-to-use `CLAUDE.md` files for the stack combinations I use most often.

Each note reflects a decision I've actually made and stand behind — not a generic best-practices checklist copied from somewhere else.

## Why it's public

This lives on my GitHub for two reasons: so it never gets lost — it's the accumulated result of a lot of trial and error, and I'd rather not rebuild it from memory — and so anyone curious, a recruiter included, can see exactly how I think about architecture, conventions, and working with AI agents day to day. It's less a tool for others to plug into and more a snapshot of my own engineering standards, written down.

## Opening the vault

Built and maintained as an [Obsidian](https://obsidian.md) vault, so notes cross-link with `[[wikilinks]]` and the graph view actually shows how everything connects. Open the folder in Obsidian to browse it that way, or just read the Markdown directly — every note is plain, dependency-free text.

Start at [`Index.md`](Index.md).
