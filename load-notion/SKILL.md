---
name: load-notion
description: >
  Fetches one or more Notion pages by URL and loads their full content into the conversation context.
  Use this skill whenever the user provides a Notion URL or asks you to read, load, summarize, analyze,
  or work with content from Notion — even if they don't say "load" explicitly. Trigger for phrases like:
  "here's my Notion doc", "load this Notion page", "read this notion.so link", "check my Notion notes",
  "summarize this Notion page", "based on my Notion page at...", or any time a notion.so or *.notion.site
  URL appears in the user's message. If multiple URLs are provided, fetch them all and keep all content
  in context. After loading, stay ready to answer questions, summarize, extract, or act on the content
  without requiring the user to paste anything manually.
---

# load-notion

Loads Notion page content into the conversation so you can reason over it immediately.

## When this skill applies

- User shares one or more `notion.so` or `*.notion.site` URLs
- User says anything like "load my Notion page", "read this doc", "summarize my Notion notes", "based on this Notion page..."
- User pastes a Notion URL alongside a task ("write a summary of...", "find action items in...", "translate...")

## How to load Notion content

Use the `notion-fetch` MCP tool for each URL. Call them in parallel if there are multiple URLs.

```
Tool: mcp__6fc72361-2138-47a2-9d34-2ada2ad254a9__notion-fetch
Parameter: { "id": "<notion-url-or-uuid>" }
```

The tool accepts:
- Full `notion.so` URLs (e.g. `https://www.notion.so/workspace/Page-Title-abc123`)
- Notion Sites URLs (`https://myspace.notion.site/Page-Title-abc123`)
- Raw UUIDs (`12345678-90ab-cdef-1234-567890abcdef`)
- Database/collection IDs

## What to do with the loaded content

After fetching:

1. **Confirm what was loaded** — briefly tell the user which page(s) you loaded and roughly what they contain (title, type, length). One sentence per page is enough.
2. **Proceed with the user's request** — if the user asked you to do something with the content (summarize, extract tasks, answer a question), do it immediately. Don't ask them to paste the content again.
3. **Keep content in context** — treat the fetched content as part of the conversation. The user can ask follow-up questions ("what are the deadlines?", "who owns item 3?") and you should answer from what you loaded without fetching again.

## Handling multiple URLs

If the user provides a list of Notion URLs:
- Fetch all of them in parallel (multiple `notion-fetch` calls in one turn)
- After loading, label each page clearly when referencing them (e.g. "From the 'Q4 Planning' page: …")
- If the pages are related (e.g. a parent and sub-pages), note the relationship

## Large page threshold

After fetching, estimate the word count of the returned content. If it exceeds **~3000 words**:

1. Tell the user the page is large (give a rough word count or section count).
2. Ask how they'd like to proceed — offer two options:
   - **Full content** — keep everything in context; best when they'll ask detailed follow-up questions or need precise references later.
   - **Structured extract** — keep only the headings, key decisions, and any explicitly highlighted items (bold text, callouts, action items); best for a one-shot task like summarizing or extracting deadlines.
3. Default to **full content** if the user doesn't specify, or if their task clearly requires specific details (e.g. "find the exact timeout value").

Don't ask for confirmation if the page is under the threshold — just load it silently.

## Handling errors

- **Page not found / no access**: Tell the user the page couldn't be loaded and ask them to check that the integration has access (Settings → Connections in Notion).
- **Not a valid Notion URL**: Ask the user to confirm the URL — it should start with `notion.so` or end in `.notion.site`.

## Example interactions

**Single page load:**
> User: "Can you summarize this? https://www.notion.so/myworkspace/Sprint-Retro-abc123"
> → Fetch the page, then deliver the summary directly.

**Multiple pages:**
> User: "Load these two docs and tell me if there are any conflicts: [url1] [url2]"
> → Fetch both in parallel, then compare.

**Implicit load:**
> User: "Based on my Notion spec at notion.so/myspace/Feature-Spec-xyz, write the acceptance criteria"
> → Fetch the spec, then write the acceptance criteria from its content.
