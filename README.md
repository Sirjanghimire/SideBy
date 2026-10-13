# SideBy

**Find your people. Make things happen.**

SideBy connects people through spontaneous activities and shared interests. You post a
"sideBy" (a thing you want to do), people nearby discover it, reply, and take it to direct
messages.

## Stack

- [Next.js 15](https://nextjs.org) (App Router, Server Actions, TypeScript)
- [Supabase](https://supabase.com) (Postgres, Auth, Realtime)
- [Tailwind CSS v4](https://tailwindcss.com)

## Features

- Email/password accounts with auto-created profiles (`/signup`, `/login`, `/settings`)
- Post a sideBy with category, details, time, place, and optional GPS location (`/new`)
- Post **publicly or anonymously** — anonymity is enforced in the database, not just hidden in the UI
- Feed with **Nearby** (radius search around your location) and **Latest** modes, filterable by category (`/feed`)
- Post pages with threaded replies, which can also be anonymous (`/p/:id`)
- Direct messages with live updates via Supabase Realtime (`/messages`)
- Public profiles (`/u/:username`)

## Getting started

### 1. Create a Supabase project

Go to [supabase.com/dashboard](https://supabase.com/dashboard) and create a project.

### 2. Apply the schema

Open **SQL Editor** in the dashboard, paste the contents of
[`supabase/migrations/0001_init.sql`](supabase/migrations/0001_init.sql), and run it.

Or, with the Supabase CLI:

```bash
supabase link --project-ref <your-project-ref>
supabase db push
```

### 3. Configure auth

In **Authentication → Providers → Email**:

- For local development, turn **off** "Confirm email" so sign-ups work immediately.
- If you keep confirmation on, add `http://localhost:3000/auth/callback` under
  **Authentication → URL Configuration → Redirect URLs**.

### 4. Environment variables

```bash
cp .env.example .env.local
```

Fill in `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` from
**Project Settings → API**.

### 5. Run

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

## How anonymity works

The `posts` and `replies` tables are not readable by app users at all. Reads go through
the `post_feed` and `reply_feed` views, which return `author_id`, `author_username`, etc.
as `NULL` for anonymous rows unless the viewer is the author. Writes go through
`create_post` / `create_reply` database functions that stamp the caller's `auth.uid()`.

Consequences:

- Nobody, including via the raw API, can read who wrote an anonymous post.
- An anonymous poster **cannot be direct-messaged**. People reply in the thread instead.
  The anonymous poster can message a (non-anonymous) replier, revealing themselves by choice.
- The "OP" badge on a reply only appears when it cannot de-anonymise anyone.

Direct messages are always between identified profiles.

## Project layout

```
supabase/migrations/   SQL schema, views, functions, RLS policies
src/middleware.ts      Session refresh + route protection
src/lib/supabase/      Browser / server Supabase clients
src/app/actions/       Server Actions (auth, posts, replies, messages, profile)
src/app/(auth)/        Login / signup pages
src/app/(app)/         Authenticated app: feed, new, p/[id], messages, u/[username], settings
src/components/        Shared UI
```

## Scripts

| Command             | What it does                 |
| ------------------- | ---------------------------- |
| `npm run dev`       | Start the dev server         |
| `npm run build`     | Production build             |
| `npm run start`     | Serve the production build   |
| `npm run typecheck` | TypeScript check, no emit    |
