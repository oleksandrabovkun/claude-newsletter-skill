# claude-newsletter-skill

A Claude Code skill that writes and publishes your weekly team newsletter — automatically.

It searches your recent activity across Confluence, Google Drive, Jira, and GitHub, walks you through each item to add your commentary, drafts the newsletter in your terminal, and publishes it to Confluence when you're ready.

## How it works

1. **Discover** — Searches Confluence, Google Drive, Jira, and GitHub for your recent activity in parallel
2. **Curate** — Presents each item one at a time; you decide what to include and add your take
3. **Compose** — Drafts the full newsletter with your commentary woven in and a suggested title
4. **Publish** — Creates the Confluence page under your team's newsletter space when you say so
5. **Reflect** — Asks if you want to improve the skill for next time; updates `personal.md` automatically

## Confluence structure

```
Team Newsletters (your parent page)
  └── Your Name
        └── Thursday March 12: Your Title
```

## The asset-first rule

Every newsletter bullet must link to a real asset — a Confluence page, PR, Jira ticket, Google Doc, etc. Your commentary is what turns a changelog entry into a newsletter story. No link, no bullet.

## Requirements

- [Claude Code](https://claude.ai/code)
- MCP servers connected: Confluence, Google Drive (optional), Jira (optional), GitHub CLI (`gh`)

## Installation

Copy the skill into your Claude Code skills directory:

```bash
cp -r claude-newsletter-skill ~/.claude/skills/newsletter
```

Then run `/newsletter` in Claude Code. On the first run, it will ask you for your Confluence space, parent page ID, GitHub orgs, and other config — and create your `personal.md` automatically.

## Personal config (`personal.md`)

`personal.md` lives next to `SKILL.md` and is **gitignored** — it only exists on your machine. It stores:

- Your name
- Confluence space key and parent newsletter page ID
- GitHub orgs/repos to search
- Custom setup questions
- Workflow preferences accumulated over time

The skill's Phase 5 self-improvement loop updates this file after each run, so it gets smarter about your workflow without you having to configure anything manually.

See [`personal.md.example`](personal.md.example) for the format.

## License

MIT
