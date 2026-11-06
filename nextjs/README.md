# Next.js — Setup & Commands

> Quick commands: [CHEATSHEET.md](CHEATSHEET.md)

Next.js is a React framework with file-based routing, server components, API route handlers, and built-in image/font optimization.

- Dev server: `http://localhost:3000`
- Router: **App Router** (`app/` directory) — the default and recommended choice
- Written against Next.js 16.4 (requires Node >= 20.9)

---

## 1. Prerequisites

| Requirement | Version |
| --- | --- |
| Node.js | **20.9+** (24 LTS recommended) |
| npm | 10+ |
| Git | any recent |

```bash
node -v   # must be >= v20.9.0
```

Install Node on Windows: `winget install OpenJS.NodeJS.LTS` (or the installer at https://nodejs.org).
macOS: `brew install node@24`. Linux: NodeSource setup script.

---

## 2. Create a project

```bash
npx create-next-app@latest my-app
cd my-app
npm run dev
```

Skip all prompts and take the defaults (TypeScript, Tailwind, ESLint, App Router, Turbopack, `@/*` alias):

```bash
npx create-next-app@latest my-app --yes
```

### What the prompts mean

| Prompt | Recommendation |
| --- | --- |
| TypeScript | **Yes** |
| Linter | ESLint (or Biome if you prefer speed) |
| React Compiler | No, unless you know you need it |
| Tailwind CSS | Yes |
| `src/` directory | Yes for larger apps, No for tiny ones — pick one and stay consistent |
| App Router | **Yes** |
| Cache Components | Yes if you want the new caching model |
| Import alias | `@/*` (default) |
| AGENTS.md | Your call — useful for AI-assisted coding |

Other package managers:

```bash
pnpm create next-app@latest my-app
yarn create next-app@latest my-app
bun create next-app@latest my-app
```

---

## 3. Industry-standard project structure (App Router)

```
my-app/
├── app/                          # routing + pages + layouts
│   ├── (marketing)/              # route group (does NOT affect the URL)
│   │   ├── page.tsx              #   /
│   │   └── pricing/page.tsx      #   /pricing
│   ├── (dashboard)/
│   │   └── dashboard/page.tsx    #   /dashboard
│   ├── api/                      # route handlers (server endpoints)
│   │   └── users/route.ts        #   GET/POST /api/users
│   ├── layout.tsx                # root layout (required, renders <html>)
│   ├── page.tsx                  # home page
│   ├── loading.tsx               # route-level loading UI
│   ├── error.tsx                 # route-level error boundary (client)
│   ├── not-found.tsx
│   └── globals.css
├── components/
│   ├── ui/                       # Button, Input, Card...
│   └── layout/                   # Header, Footer, Nav
├── lib/                          # server-safe helpers
│   ├── db.ts                     # database client singleton
│   ├── auth.ts
│   └── utils.ts
├── actions/                      # server actions ("use server")
├── hooks/                        # client hooks
├── types/                        # shared TypeScript types
├── public/                       # static assets (served from /)
├── prisma/                       # schema + migrations (if using Prisma)
├── .env                          # non-secret defaults — commit
├── .env.local                    # secrets — never commit
├── .env.example                  # committed template
├── .gitignore
├── eslint.config.mjs
├── next.config.ts
├── package.json
└── tsconfig.json
```

### Routing conventions worth memorizing

| File / folder | Purpose |
| --- | --- |
| `page.tsx` | The UI for a route |
| `layout.tsx` | Shared shell; persists across navigation |
| `loading.tsx` | Instant loading state (Suspense boundary) |
| `error.tsx` | Error boundary (must be a client component) |
| `route.ts` | API endpoint — export `GET`, `POST`, etc. |
| `[id]` | Dynamic segment — `app/users/[id]/page.tsx` |
| `[...slug]` | Catch-all segment |
| `(group)` | Route group — organizes files without changing the URL |
| `_folder` | Private folder — opted out of routing |

**Rules of thumb**

- Fetch data in **Server Components** by default; add `"use client"` only at the leaves that need state/effects.
- Keep secrets in server code — never import them into a client component.
- Put the DB client in `lib/db.ts` as a singleton so hot reload doesn't open new connections.

---

## 4. Commands you need

| Task | Command |
| --- | --- |
| Start dev server (Turbopack) | `npm run dev` |
| Dev on another port | `npm run dev -- -p 3001` |
| Production build | `npm run build` |
| Start the production server | `npm run start` |
| Production on another port | `npm run start -- -p 3001` |
| Lint | `npm run lint` |
| Lint + auto-fix | `npm run lint -- --fix` |
| Type-check | `npx tsc --noEmit` |
| Show Next.js info (versions, env) | `npx next info` |
| Clean build artifacts | remove `.next/` then rebuild |

> **Next 16 change:** `next lint` was removed. The `lint` script now calls ESLint directly (`eslint`). Use `npm run lint`.

### What the scripts run

| Script | Underlying command |
| --- | --- |
| `dev` | `next dev` (Turbopack is the default bundler) |
| `build` | `next build` |
| `start` | `next start` (requires a prior `build`) |
| `lint` | `eslint` |

### Control the dev server

| Action | How |
| --- | --- |
| Stop | `Ctrl + C` in the terminal |
| Change port | `npm run dev -- -p 3001` |
| Bind to all interfaces | `npm run dev -- -H 0.0.0.0` |
| Force webpack instead of Turbopack | `npm run dev -- --webpack` |
| Port already in use | `netstat -ano \| findstr :3000` then `taskkill /PID <pid> /F` |

---

## 5. Environment variables

Next.js loads `.env*` files automatically.

```bash
# .env.local
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp
AUTH_SECRET=super-secret-value
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

| File | Loaded in | Commit? |
| --- | --- | --- |
| `.env` | all environments | yes (non-secret defaults) |
| `.env.local` | all environments, overrides `.env` | **no** |
| `.env.development` | `next dev` | yes |
| `.env.production` | `next build` / `next start` | yes |
| `.env.test` | tests | yes |

Usage:

```tsx
// Client component — inlined at build time
const url = process.env.NEXT_PUBLIC_API_URL;

// Server component / route handler / server action — runtime value
const secret = process.env.AUTH_SECRET;
```

> No prefix = server-only. `NEXT_PUBLIC_` values are embedded in the JS bundle — treat them as public.

---

## 6. Minimal examples

### Route handler

```ts
// app/api/hello/route.ts
export async function GET() {
  return Response.json({ message: 'hello' });
}

export async function POST(request: Request) {
  const body = await request.json();
  return Response.json({ received: body }, { status: 201 });
}
```

### Server component with data fetch

```tsx
// app/users/page.tsx
export default async function UsersPage() {
  const res = await fetch('https://jsonplaceholder.typicode.com/users', {
    next: { revalidate: 60 }, // cache for 60s
  });
  const users = await res.json();

  return (
    <ul>
      {users.map((u: { id: number; name: string }) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

### Client component

```tsx
'use client';

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

## 7. Common additions

```bash
# Database ORM
npm install -D prisma
npm install @prisma/client

# Auth
npm install next-auth

# Forms + validation
npm install react-hook-form zod

# UI state
npm install zustand
```

---

## 8. Gotchas

| Symptom | Cause / fix |
| --- | --- |
| `Hydration failed` | Server and client markup differ. Common causes: `Date.now()`, `Math.random()`, `localStorage` read during render, invalid HTML nesting (`<div>` inside `<p>`). Move dynamic values into `useEffect`. |
| `useState` error in a component | Missing `"use client"` at the top of the file. |
| Env var `undefined` in the browser | Missing `NEXT_PUBLIC_` prefix, or you didn't restart `next dev`. |
| Changes not showing | Delete `.next/` and restart; check that you're not viewing a stale production server. |
| `next start` fails | You must run `npm run build` first. |
| Images from external domains fail | Add the domain to `images.remotePatterns` in `next.config.ts`. |
| API route returns 404 | File must be `route.ts` (not `route.tsx`/`page.ts`), exported functions must be uppercase (`GET`, `POST`...). |
| Slow dev server on Windows | Ensure the project is on the local disk (not OneDrive), and don't run two dev servers on the same folder. |
| `Module not found: Can't resolve '@/...'` | Check `paths` in `tsconfig.json` (`"@/*": ["./src/*"]` when using `src/`). |
| `ECONNREFUSED` when calling the DB | The database isn't running — see [../postgresql/README.md](../postgresql/README.md) or [../mongodb/README.md](../mongodb/README.md). |

### Image config example

```ts
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: 'images.example.com' },
    ],
  },
};

export default nextConfig;
```

---

## 9. Official docs

- Next.js: https://nextjs.org/docs
- create-next-app: https://nextjs.org/docs/app/api-reference/cli/create-next-app
- App Router: https://nextjs.org/docs/app
