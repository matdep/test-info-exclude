# CLAUDE.md

We're building the app described in @SPEC.md. read that file for general artchitectural tasks or to double-check the exact database structure, tech stack or application architecture.

Keep your replay extremely concis and focus on conveying the key information. No unnecessary fluff, no long code snippet.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bun run dev      # Start development server on http://localhost:3000
bun run build    # Build for production
bun run start    # Run production build
bun run lint     # Run ESLint
```

This project uses **Bun** as the runtime and package manager (not npm/yarn/pnpm). Always use `bun` for installing packages and running scripts.

## Environment

Copy `.env.example` to `.env.local` before running. Required variables:

- `BETTER_AUTH_SECRET` — 32+ character secret for better-auth session signing
- `DB_PATH` — path to SQLite database file (e.g., `data/app.db`)

## Architecture

This is a **Note Taking Web App** — a Next.js (App Router) application where authenticated users can create, edit, delete, and publicly share rich-text notes. The spec is in `SPEC.md`.

### Stack

- **Runtime:** Bun (used for its native SQLite client)
- **Framework:** Next.js 16 with App Router
- **Auth:** `better-auth` library (handles sessions, accounts, email/password)
- **Database:** SQLite via Bun's native `bun:sqlite` (raw SQL, no ORM)
- **Editor:** TipTap v3 (notes stored as TipTap JSON, not HTML)
- **Styling:** TailwindCSS v4

### Key Architectural Decisions

**Database access** lives in `lib/db.ts` (singleton Bun SQLite connection) and `lib/notes.ts` (note repository functions). All note queries filter by `user_id` to enforce ownership.

**Auth** is integrated via better-auth's Next.js adapter. Server-side auth checks use a helper like `getSession()`. The `app/api/auth/[...all]/route.ts` handler delegates to better-auth.

**TipTap content** is always stored as `JSON.stringify(editor.getJSON())` in the `content_json` column and parsed back with `JSON.parse()` when loading. Never store or render raw HTML.

**Public sharing** uses a random slug (`public_slug` column). When `is_public = 0`, the `/p/[slug]` route returns 404. Slugs should be 16+ chars to prevent guessing.

### Route Structure

```
app/
  layout.tsx                    # Root layout with header
  page.tsx                      # Landing page (unauthenticated)
  (auth)/
    login/page.tsx
    register/page.tsx
  dashboard/page.tsx            # Authenticated note list
  notes/[id]/page.tsx           # Note editor
  p/[slug]/page.tsx             # Public read-only note view
  api/
    auth/[...all]/route.ts      # better-auth handler
    notes/
      route.ts                  # GET list, POST create
      [id]/route.ts             # GET, PUT, DELETE single note
      [id]/share/route.ts       # POST toggle sharing
```

### Data Layer Files

- `lib/db.ts` — Bun SQLite singleton + `query<T>`, `get<T>`, `run` helpers
- `lib/notes.ts` — note repository functions (`createNote`, `getNoteById`, etc.)
- `lib/auth.ts` — better-auth server instance
- `data/app.db` — SQLite file (gitignored)
