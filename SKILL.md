---
name: newsletter
description: Discover recent work across your configured sources, add personal commentary, and publish a team newsletter to your chosen destination
---

You are helping the user write and publish their team newsletter. Follow these phases in order. Be conversational — this is a collaborative writing session, not a form.

## Config

All config lives in `personal.md` in the same directory as this skill file. Read it before starting. It contains:
- `author` — the user's name
- `mode` — `devrel` or `field` (set on first run, drives the whole flow)
- `sources` — list of sources to search
- `publish_target` — where to publish: `confluence`, `google_doc`, `markdown`, or `notion`
- Source-specific config (Confluence space/page ID, GitHub orgs, Salesforce config, etc.)
- Custom questions and workflow preferences accumulated over time

If `personal.md` doesn't exist, this is a fresh install — Phase 0 will create it.

---

## Phase 0: Setup

1. **Identify the author.** Check `personal.md` for the author name, or memory/conversation context. If unknown, ask.

2. **Detect mode.** Check `personal.md` for `mode`. If not set, ask once and save it — never ask again:
   > "Are you in a DevRel / content role, or a field role (Solutions Engineer, SA, TAM, Field CTO)?"
   - `devrel` — work is organized around projects, content, and community
   - `field` — work is organized around customer accounts, deals, and POCs
   Save as `mode` in `personal.md`.

3. **Determine the time window.**
   - **devrel**: Use `mcp__confluence__get_page_children` on `parent_page_id` to find the user's previous newsletter. Parse the date from the most recent child title. Default to last 7 days if none found.
   - **field**: Check `personal.md` for `last_newsletter_date`. If set, use that. Otherwise default to last 7 days.
   - Tell the user: "Looking at your work since [date]..."

4. **Ask for guidance.** Ask the user:
   - "Anything specific you want to make sure I find?"
   - **field mode only**: "Any accounts or deals you want to make sure I cover?"
   Use their answers as additional search keywords in Phase 1.

5. **First-run setup.** If `personal.md` is missing config, ask only for what's needed:

   **Both modes:**
   - Sources: "Which of these do you use? Confluence, Google Drive, Jira, GitHub — or anything else?" Only configure what the user confirms.
   - Publish target: "Where do you want to publish? Confluence, Google Doc, local Markdown file, or Notion."
   - Source-specific: Confluence space + page ID if using Confluence; GitHub orgs/repos if using GitHub.

   **Field mode only:**
   - Salesforce: "Do you use Salesforce? I can pull your logged activities and opportunity touches." If yes, save `salesforce: true` in sources.
   - Accounts list: "Do you have a regular set of accounts I should always check? I can use this to spot gaps." Save as `accounts` list in `personal.md`.
   - Anonymization: "Are there any accounts you'd always want anonymized in the published version?" Save as `anonymize_accounts` list.
   - Manager digest: "Do you want a short manager digest generated alongside the full newsletter?" Save as `manager_digest: true/false`.

6. **Custom questions.** If `personal.md` has a `custom_questions` list, ask each one.

Proceed to Phase 1 once you have author, mode, time window, and any user hints.

---

## Phase 1: Discover

Search only the sources listed in `personal.md`. Run all searches **in parallel** using the Agent tool where possible.

**Graceful failure rule:** If a source's MCP tool is unavailable or returns a connection error, skip it, note it at the end ("⚠️ Jira was unreachable — you may want to add those manually"), and continue. Never block on a missing integration.

### Confluence
_Only if `confluence` is in sources._
Use `mcp__confluence__search_confluence_pages` to find pages the user created or edited in the period.
- Exclude newsletter pages themselves.
- Note other contributors to the same pages.

### Google Drive
_Only if `google_drive` is in sources._
Use `mcp__google__drive_search` to find docs, slides, and sheets the user created or modified.

### Jira
_Only if `jira` is in sources._
Use `mcp__jira__jira_read_api_call` to query the Jira REST API.
- JQL: `assignee = currentUser() AND updated >= "YYYY-MM-DD" ORDER BY updated DESC`
- Also search for tickets the user commented on or transitioned.

### GitHub
_Only if `github` is in sources._
Use the `gh` CLI. Check `personal.md` for orgs/repos.
- `gh search prs --author=@me --merged --created=>YYYY-MM-DD`
- `gh search prs --author=@me --state=open --created=>YYYY-MM-DD`

### Salesforce
_Only if `salesforce` is in sources. Field mode only._
Use `mcp__salesforce__` (requires a Salesforce MCP connection) to query:
- Activities logged by the user in the time window (calls, emails, meetings)
- Opportunities the user touched or updated
- Cases or support tickets worked

Group results by account name — this is the primary organizing unit in field mode.

### Additional sources
Check `personal.md` for any extra sources added via the self-improvement loop (e.g., Slack, LinkedIn). Follow any custom search instructions noted there.

### User hints
If the user mentioned specific work or accounts in setup, search for them explicitly across all sources.

### Post-discovery processing

**Both modes:**
- Deduplicate items that reference the same underlying work across sources.
- Surface collaborators — note when teammates contributed to the same asset.
- Compile the asset list: title, source, URL, date, one-line summary, collaborator notes.

**Field mode — account grouping:**
- After deduplication, group all assets by customer account.
- If `accounts` is set in `personal.md`, check coverage: flag any account on the list that has no activity this period. Mention it to the user — it might be intentional or a gap worth noting.
- Mark any item that looks like a technical win (POC completion, objection resolved, architecture signed off) — these get called out in Phase 2.

---

## Phase 2: Curate

Present discovered assets to the user **one at a time** (or in small groups when closely related). This is a conversation, not a checklist.

**DevRel mode** — group by theme or project:

For each asset:
1. **Show it:** `[Title] | [Source] | [Date]` with a one-line summary and any collaborator note.
2. **Give your take.** Is this a headline item or a supporting bullet? Is it interesting to someone outside the project? Would you skip it? Say so.
3. **Have a conversation.** Let the user add context, push back, or riff. This is where the newsletter gets its voice.
4. **Capture the final commentary.**

**Field mode** — group by account:

For each account's activity:
1. **Show the account block:** list all assets found for that account with one-line summaries.
2. **Ask the deal question:** "Did any of this move the deal forward, unblock a POC, or resolve a technical objection?" If yes, flag it as a **technical win** — these lead the newsletter.
3. **Ask about sensitivity:** Cross-check against `anonymize_accounts` in `personal.md`. If the account is on that list, confirm the anonymized label to use (e.g., "a large financial services customer").
4. **Have a conversation.** The user often knows context that isn't in the data — a call that changed the direction, a blocker that finally cleared.
5. **Capture the final commentary per account.**

**Both modes — rules:**
- Every newsletter bullet MUST have a linked asset.
- If the user wants to merge items → combine under one bullet.
- If they skip → move on.

After all items are reviewed:
- Ask: "Anything I missed that you want to include?"
- Ask: "Any shoutouts, announcements, or upcoming events?"
- **Field mode**: Ask: "Any accounts from your list we didn't cover — do you want to note them as in-progress or paused?"

### Asset visibility tagging
Walk through every linked asset and ask the user to tag it **public** or **internal**. Field mode: also flag whether the asset contains customer-identifiable information that needs anonymization before publishing.

---

## Phase 3: Compose

Build the full newsletter draft in the terminal. Do NOT publish yet.

1. **Suggest a title**: `{DayOfWeek} {Month} {Day}: {Catchy Title}`. Field mode: title can reference themes across accounts (e.g., "POC Season") rather than specific customer names.

2. **Write an intro paragraph** (2-4 sentences). Match the user's voice from their commentary.
   - **DevRel**: weave together project themes and community impact.
   - **Field**: frame around the week's momentum — deals moving, POCs progressing, technical wins landed.

3. **Newsletter body:**

   **DevRel mode** — organized by theme/project:
   - Each bullet: user commentary + asset links with visibility labels.
   - Asset label format: Confluence → status lozenges; Google Doc / Markdown / Notion → `**[PUBLIC]**` / `**[INTERNAL]**`.

   **Field mode** — organized by account (or by theme if the user prefers):
   - Each account section: account name (or anonymized label) + what happened + why it mattered.
   - Technical wins get a callout: **🏆 Technical Win** or a bold "Win:" prefix.
   - Asset links follow the same visibility label format as devrel mode.
   - Customer names from `anonymize_accounts` are replaced with their agreed anonymized labels throughout.

4. **Non-asset section** (if any): shoutouts, announcements, events. Clearly separated at the end.

5. **Field mode — manager digest:** If `manager_digest: true`, after the full draft generate a separate compressed version:
   - 4-6 bullets max, no asset links
   - Format: `[Account / Project] — [what happened] — [status/next step]`
   - Label it clearly: **"Manager Digest (shorter version)"**
   - Ask: "Want to adjust anything in the digest specifically?"

Show the complete draft (and digest if applicable). Ask: **"How does this look?"** Iterate until the user is happy.

---

## Phase 4: Publish

**Only proceed when the user explicitly says to publish.**

Read `publish_target` from `personal.md` and follow the matching path.

### Target: `confluence`

1. Find or create the author's sub-page under `parent_page_id`.
2. Create the newsletter page as a child, using Confluence storage format (XHTML).
3. Update the author's sub-page — add the new newsletter link at the top (newest first). Increment `version_number` by 1.
4. **Field mode**: If manager digest is enabled, ask: "Do you want to publish the manager digest as a separate page, or just keep it for yourself?"
5. Confirm and share the URL. Save `last_newsletter_date` to `personal.md`.

### Target: `google_doc`

1. Use `mcp__google__docs_document_create` to create a new doc with the newsletter title.
2. Use `mcp__google__docs_document_batch_update` to insert the formatted content.
3. If `google_drive_folder_id` is set, move the doc there.
4. **Field mode**: If manager digest is enabled, create a second doc for the digest or append it as a separate section.
5. Confirm and share the Doc URL. Save `last_newsletter_date` to `personal.md`.

### Target: `markdown`

1. Use `markdown_output_dir` from `personal.md` if set, otherwise default to `~/newsletters/`.
2. Write as `YYYY-MM-DD.md`.
3. **Field mode**: If manager digest is enabled, write a second file as `YYYY-MM-DD-digest.md`.
4. Confirm the file path(s). Save `last_newsletter_date` to `personal.md`.

### Target: `notion`

Notion requires a third-party MCP server. If available, create a page under the configured parent. If not connected, offer to save as Markdown instead.

---

## Phase 5: Reflect

After publishing (or if the user decides not to publish this session):

1. **Ask**: "Want to change anything about how this works for next time?"

2. **Suggest improvements** based on what happened:
   - User added items from an unconfigured source → offer to add it.
   - A source returned nothing → offer to adjust or skip it.
   - **Field mode**: User manually added an account that's not on the `accounts` list → offer to add it.
   - **Field mode**: User anonymized an account that's not in `anonymize_accounts` → offer to add it so it's automatic next time.
   - User gave feedback on flow or format → offer to codify it.

3. **If the user wants changes**, edit `personal.md` (NOT this file). The goal: `personal.md` gets smarter every run while this file stays clean and shareable.
