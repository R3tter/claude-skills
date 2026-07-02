---
name: handoff
description: >
  Exports or imports a distilled snapshot of conversation context as a portable markdown file in the current project folder, so decisions and progress can move between chat sessions, projects, or teammates. Use whenever the user says "/handoff export" or "/handoff import", asks to "hand off this conversation", "save this context for another chat", "pick this up in a new session", or "load the handoff from [chat/topic]". Export can be scoped to a topic ("/handoff export auth refactor") and can include raw excerpts alongside the summary when the user passes "--detailed". This is distinct from resuming a session or from durable cross-session memory — it's a deliberate, shareable checkpoint the user chooses to create and later consume, not something that happens automatically.
---

## What this skill is for

A handoff file is a deliberate, portable checkpoint of *this* conversation — written so a future chat (same project, a different project, or a teammate) can pick up where this one left off without replaying the whole transcript. It is not:

- **Session resume** (`--continue`/`--resume`) — that restores the *entire* session automatically. Handoff is selective and explicit.
- **Durable memory** (e.g. a `/remember`-style system) — that accretes facts silently over time. Handoff is a single frozen snapshot the user asks for.

Reach for scoping and brevity: the value of a handoff file is that someone can read it in under a minute and be caught up. Don't dump the whole conversation in — distill it.

## Export: `/handoff export [scope]`

1. **Determine scope.** If the user gave a scope phrase (e.g. `/handoff export auth refactor`), only include context relevant to that topic — filter out unrelated threads from the conversation. If no scope was given, cover the whole conversation.

2. **Determine detail level.** If the user passed `--detailed` (in any position, e.g. `/handoff export auth refactor --detailed`), include a raw excerpts section in addition to the summary (see template below). Otherwise, summary only — no verbatim transcript dumps.

3. **Write the distilled content**, not a transcript. Read back over the conversation and extract:
   - What the user is trying to accomplish (the goal/task at hand)
   - Decisions made and why (especially non-obvious tradeoffs — the *why*, not just the *what*)
   - Current state — what's done, what's in progress, what's explicitly not done yet
   - Open questions or things the user still needs to decide
   - Concrete pointers: file paths touched, commands run, external resources referenced
   - If `--detailed`: verbatim snippets that would lose meaning if paraphrased — exact requirements the user stated, exact code decisions, exact error messages debugged. Keep this section short and selective; it's for precision-critical material only, not a full log.

4. **Pick a title** that's short and descriptive of the conversation/topic (e.g. `auth-refactor`, `q3-pricing-model`). If a scope was given, derive the title from the scope.

5. **Write the file** to the current project folder (the working directory), named:
   ```
   handoff-<title-slug>.md
   ```
   Use kebab-case for the slug. If a file with that name already exists, ask the user whether to overwrite it or pick a different name — don't silently clobber prior handoff work.

6. **Use this template:**

   ```markdown
   ---
   title: <title>
   scope: <scope phrase, or "full conversation">
   status: pending
   exported: <date, if known>
   ---

   # Handoff: <title>

   ## Goal
   <what the user is trying to accomplish>

   ## Decisions
   - <decision> — <why>
   - ...

   ## Current state
   <what's done, in progress, not started>

   ## Open questions
   - <question the next session needs to resolve>

   ## Key references
   - <file paths, commands, URLs>

   ## Raw excerpts   <!-- only if --detailed -->
   > <verbatim snippet> — <one-line context for why this is here verbatim>
   ```

7. **Confirm to the user**: tell them the file path and give a one-line summary of what was captured, so they can sanity-check before closing the chat.

## Import: `/handoff import <name>`

1. **Find candidate files.** List `handoff-*.md` files in the current project folder (the working directory).

2. **Fuzzy match** the user's `<name>` against filenames and each file's `title`/`scope` frontmatter fields.
   - One clear match → use it.
   - No match → tell the user no handoff file matched, list what handoff files *do* exist (if any), and ask them to clarify.
   - Multiple plausible matches → list them (title, scope, status, export date) and ask the user which one they mean. Never guess silently when it's ambiguous.

3. **Load the file's content into the conversation** — read it and treat its Goal/Decisions/Current state/Open questions/Key references as working context for this session, the same way you'd treat a summary the user had just typed to you. Briefly acknowledge what was loaded (a short recap, not a full re-print of the file) so the user can confirm it's the right one.

4. **Mark it imported, don't delete it.** Update the file's frontmatter `status: pending` to `status: imported` (add an `imported: <date>` field if you know the date). Leave the file in place — it stays available as a record and can be re-imported later (e.g. into a second chat, or by a teammate). Do not delete or move it unless the user asks you to.

## Notes

- Handoff files live in the project folder by design, so they're visible in `git status` — if the user wants to share context with a teammate, they can just commit the file. Don't add them to `.gitignore` on the user's behalf; that's their call.
- If asked to export with no meaningful content yet (e.g. very early in a conversation), say so rather than producing a near-empty file — a handoff isn't valuable until there's actually something to hand off.
