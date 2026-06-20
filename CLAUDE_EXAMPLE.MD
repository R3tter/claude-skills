# CLAUDE.md

## Project
E-commerce dashboard — React + TypeScript + Vite  
Stack: React 18, TailwindCSS, React Query, Zustand

## Session Start Protocol
1. Query Graphify for project structure: `graphify query "project overview"`
2. Load context from Notion via MCP before asking me anything

## Notion Memory
- Architecture decisions → [link to your Notion page]
- Current sprint / bugs → [link]
- Past session logs → [link]

Use the Notion MCP to fetch these pages at session start.

## Graphify
Run `graphify query "<topic>"` before touching any module you haven't seen yet.
Never scan raw files — always query the graph first.

## Code Style
- Functional components only, no classes
- Arrow functions everywhere
- Tailwind for styling, no CSS modules
- React Query for server state, Zustand for client state

## Project Conventions
- Components live in `src/components/<Feature>/index.tsx`
- Hooks in `src/hooks/use<Name>.ts`
- API calls in `src/api/<resource>.ts`

## What NOT to do
- Don't rewrite working components without asking
- Don't install new dependencies without confirming
- Don't touch `src/legacy/` — it's deprecated but can't be deleted yet