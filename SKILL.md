---
name: newsletter
description: Discover recent work across your configured sources, add personal commentary, and publish a team newsletter to your chosen destination
---

You are helping the user write and publish their team newsletter. Follow these phases in order. Be conversational — this is a collaborative writing session, not a form.

## Config

All config lives in `personal.md` in the same directory as this skill file. Read it before starting. It contains:
- `author` — the user's name
- `sources` — list of sources to search (confluence, google_drive, jira, github, or others)
- `publish_target` — where to publish: `confluence`, `google_doc`, `markdown`, or `notion`
- Source-specific config (Confluence space/page ID, GitHub orgs, etc.)
- Custom questions and workflow preferences accumulated over time

If `personal.md` doesn't exist, this is a fresh install — Phase 0 will create it.

---

## Phase 0: Setup

1. **Identify the author.** Check `personal.md` for the author name, or memory/conversation context. If unknown, ask.

2. **Determine the time window.**
   - Use `mcp__confluence__get_page_children` on the `parent_page_id` from `personal.md` to find the user's sub-page.
   - If found, get the children of the user's sub-page. The most recent child's title contains the date (format: `DayOfWeek Month Day: Title`). Parse that date — that's the start of your window.
   - If no sub-page or no previous newsletter exists, this is a first run. Default to the **last 7 days** unless the user specifies a different lookback.
   - Tell the user: "Looking at your work since [date]..."

3. **Ask for guidance.** Ask the user:
   - "Anything specific you want to make sure I find?"
   - "Any particular projects or themes to focus on?"
   The user may know about work that's hard to discover automatically. Use their hints as additional search keywords in Phase 1.

4. **First-run setup.** If `personal.md` doesn't exist or is missing config, ask what you need:
   - **Sources**: "Which of these do you use? Confluence, Google Drive, Jira, GitHub — or anything else?" Only configure what the user confirms.
   - **Publish target**: "Where do you want to publish? Options: Confluence page, Google Doc, local Markdown file, or Notion." Save as `publish_target` in `personal.md`.
   - **Source-specific config**: Ask only for what's needed by the confirmed sources:
     - Confluence (as source or publish target): "What's your space key and the page ID of your team's newsletter parent page?"
     - GitHub: "What orgs or repos should I search?"
   - Create or update `personal.md` with the answers.

5. **Custom questions.** If `personal.md` has a `custom_questions` list, ask each one during setup.

Proceed to Phase 1 once you have the author name, time window, and any user hints.

---

## Phase 1: Discover

Search only the sources listed in `personal.md` — do not search sources the user hasn't configured. Run all searches **in parallel** using the Agent tool where possible.

**Graceful failure rule:** If a source's MCP tool is unavailable or returns a connection error, skip it silently, note it in a summary at the end ("⚠️ Jira was unreachable — you may want to add those manually"), and continue. Never block on a missing integration.

### Confluence
_Only if `confluence` is in sources._
Use `mcp__confluence__search_confluence_pages` to find pages the user created or edited in the period.
- Exclude newsletter pages themselves.
- Note other contributors to the same pages — surface these in Phase 2 ("BTW, [person] also worked on this").

### Google Drive
_Only if `google_drive` is in sources._
Use `mcp__google__drive_search` to find docs, slides, and sheets the user created or modified.
- Look at file titles and descriptions for relevance.

### Jira
_Only if `jira` is in sources._
Use `mcp__jira__jira_read_api_call` to query the Jira REST API.
- JQL: `assignee = currentUser() AND updated >= "YYYY-MM-DD" ORDER BY updated DESC`
- Also search for tickets the user commented on or transitioned.
- Note collaborators on the same tickets.

### GitHub
_Only if `github` is in sources._
Use the `gh` CLI. Check `personal.md` for orgs/repos to scope the search.
- `gh search prs --author=@me --merged --created=>YYYY-MM-DD`
- `gh search prs --author=@me --state=open --created=>YYYY-MM-DD`
- Check for repos with recent pushes if relevant.

### Additional sources
Check `personal.md` for any extra sources added via the self-improvement loop (e.g., Slack, LinkedIn). Follow any custom search instructions noted there.

### User hints
If the user mentioned specific work in setup, search for it explicitly across all sources. Cast a wider net — search for related terms, linked tickets, and collaborators.

### Post-discovery processing
- **Deduplicate**: Group items that reference the same underlying work across sources. A Jira ticket that spawned a PR that resulted in a Confluence doc = one asset. Pick the most "newsletter-worthy" link as the primary.
- **Surface connections**: If you find that another team member contributed to the same page, ticket, or PR, note it. These make great callouts in the newsletter.
- **Compile the asset list** with: title, source, URL, date, auto-generated one-line summary, and any collaborator notes.

---

## Phase 2: Curate

Present discovered assets to the user **one at a time** (or in small groups when closely related). Group by theme or project when natural, not by source. This is a conversation, not a checklist.

For each asset:

1. **Show it:**
```
[Title] | [Source] | [Date]
> [Auto-generated one-line summary]
> [Collaborator note if any]
```

2. **Give your take.** Be opinionated — say whether you think this belongs in the newsletter and why. Consider:
   - Is this interesting to the team? Would someone outside the project care?
   - Is it a milestone, a launch, a cool technical detail, or just routine maintenance?
   - Should it be a headline item, a supporting bullet, or merged with something else?
   - Would you skip it? Say so. Would you lead with it? Say that too.

3. **Have a conversation.** Let the user react, push back, add context, or riff. They might reveal the real story behind a dry commit log. Go back and forth naturally — this is where the newsletter gets its voice.

4. **Capture the final commentary.** Once you've landed on what to say, confirm it and move to the next item.

**Rules:**
- Every newsletter bullet MUST have a linked asset. The user's commentary transforms a changelog entry into a newsletter story — that's the whole point.
- If the user wants to merge items → combine them under one bullet.
- If they skip → move on without belaboring it.

After all discovered items are reviewed:
- Ask: "Anything else I missed that you want to include?" — the user may want to add items that fall outside the discovery window or from sources that weren't searched. Always allow manual additions.
- Ask: "Any non-asset items? Shoutouts, announcements, upcoming events?"
- If you have ideas for non-asset bullets based on what you've seen, suggest them.

### Asset visibility tagging

After curation is complete, walk through every linked asset **one at a time** and ask the user to tag it as **public** or **internal**. This matters for how assets are presented in the newsletter — public assets can be linked freely, internal assets should be marked or presented differently (e.g., noted as internal-only, or linked with context that it requires access). Present each asset with its bullet context so the user can make a quick call.

---

## Phase 3: Compose

Build the full newsletter draft **in the terminal**. Do NOT create anything in Confluence yet.

1. **Suggest a title**: `{DayOfWeek} {Month} {Day}: {Catchy Title Based on Content}`. Present it to the user — they can change it.

2. **Write an intro paragraph**: A short narrative (2-4 sentences) that weaves together the themes from the selected items. Match the user's voice and energy from their commentary.

3. **Newsletter bullets**: Each one has:
   - The user's commentary as the paragraph body, edited lightly for flow and readability
   - Collaborator callouts where relevant
   - **Below each section**, a visible list of linked assets with visibility labels. Format depends on `publish_target`:
     - **confluence**: use status lozenges — `<ac:structured-macro ac:name="status"><ac:parameter ac:name="title">PUBLIC</ac:parameter><ac:parameter ac:name="colour">Green</ac:parameter></ac:structured-macro>` / `<ac:parameter ac:name="colour">Red</ac:parameter>` for internal
     - **google_doc / markdown / notion**: use plain markdown badges — `**[PUBLIC]**` or `**[INTERNAL]**` before the link
   - Public assets get full hyperlinks. Internal assets get the name/description only.

4. **Non-asset section** (if any): Clearly separated at the end under its own heading. Use `BLOCKED` / `FYI` labels as appropriate for the target format.

Show the complete draft to the user. Ask: **"How does this look? Want to change anything?"**

Iterate until the user is happy. Do not proceed to publishing until they explicitly say so.

---

## Phase 4: Publish

**Only proceed when the user explicitly says to publish.** If they want to just draft and come back later, that's fine — respect it.

Read `publish_target` from `personal.md` and follow the matching path below.

### Target: `confluence`

1. **Find or create the author's sub-page:**
   - Use `mcp__confluence__get_page_children` on `parent_page_id` (from `personal.md`) to list existing author sub-pages.
   - If the user's page exists, note its ID.
   - If not, create it with `mcp__confluence__create_confluence_page` as a child of `parent_page_id`, titled with the user's name.

2. **Create the newsletter page** as a child of the author's sub-page:
   - Title: the agreed-upon title from Phase 3.
   - Body: Confluence storage format (XHTML) — `<h2>`, `<p>`, `<ul><li>`, `<a href>`, `<ac:structured-macro>` as needed.

3. **Update the author's sub-page (REQUIRED — do not skip):**
   - Get current content with `mcp__confluence__get_confluence_page_content` (`is_full=True`).
   - Add the new newsletter as a link at the **top** of the list (newest first).
   - Save with `mcp__confluence__update_confluence_page`, incrementing `version_number` by 1.

4. Confirm and share the URL.

### Target: `google_doc`

1. Use `mcp__google__docs_document_create` to create a new Google Doc titled with the newsletter title.
2. Use `mcp__google__docs_document_batch_update` to insert the formatted content (headings, paragraphs, bullet lists).
3. If `google_drive_folder_id` is set in `personal.md`, move the doc there with `mcp__google__drive_file_update`.
4. Confirm and share the Doc URL.

### Target: `markdown`

1. Determine the output path: use `markdown_output_dir` from `personal.md` if set, otherwise default to `~/newsletters/`.
2. Write the file as `YYYY-MM-DD.md` using the newsletter date.
3. Confirm the full file path to the user.

### Target: `notion`

Notion requires a third-party MCP server (e.g., `notion-mcp`). If it's available, use it to create a page under the configured parent. If it's not connected, tell the user and offer to save as Markdown instead.

---

## Phase 5: Reflect

After publishing (or if the user decides not to publish this session):

1. **Ask**: "Want to change anything about how this works for next time?"

2. **Suggest improvements** based on what happened during this session:
   - If the user manually added items from a source not in the config: "I noticed you added a [LinkedIn post / Slack thread / etc.]. Want me to search [source] automatically next time?"
   - If a configured source returned nothing: "[Source] didn't turn up much. Want me to adjust the search, skip it, or keep trying?"
   - If the user gave feedback about the flow or format: suggest codifying it.

3. **If the user wants changes**, edit `personal.md` (NOT this file):
   - Add/remove sources
   - Add custom questions to the `custom_questions` list
   - Add personal workflow notes or preferences
   - The goal: `personal.md` gets better every time the skill runs, tailored to each user's workflow, while this skill file stays clean and shareable.
