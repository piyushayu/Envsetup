# Next.js — Cheat Sheet

Full guide: [README.md](README.md)

## Create & run

```bash
npx create-next-app@latest my-app --yes
cd my-app
npm run dev            # http://localhost:3000
```

```bash
npm run build          # production build
npm run start          # serve build (run build first)
npm run lint
```

## Control

```bash
npm run dev -- -p 3001          # change port
npm run dev -- --webpack        # skip Turbopack
npm run start -- -p 3001
```

## Env

```bash
# .env.local — no prefix = server-only
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

```ts
process.env.DATABASE_URL;          // server only
process.env.NEXT_PUBLIC_API_URL;   // available in browser
```

## Minimal code

```tsx
// app/page.tsx — server component
export default async function Page() {
  const res = await fetch('https://api.example.com/data', { next: { revalidate: 60 } });
  const data = await res.json();
  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}
```

```tsx
'use client';

import { useState } from 'react';

export function Counter() {
  const [n, setN] = useState(0);
  return <button onClick={() => setN(n + 1)}>{n}</button>;
}
```

```ts
// app/api/hello/route.ts
export async function GET() {
  return Response.json({ ok: true });
}
```
