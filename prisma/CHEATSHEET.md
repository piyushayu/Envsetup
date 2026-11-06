# Prisma — Cheat Sheet

Full guide: [README.md](README.md)

## Install (PostgreSQL, Prisma 7)

```bash
npm install -D prisma
npm install @prisma/client @prisma/adapter-pg dotenv
npx prisma init --datasource-provider postgresql
```

## Schema

```prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
}
```

```ts
// prisma.config.ts
import 'dotenv/config';
import { defineConfig, env } from 'prisma/config';

export default defineConfig({
  schema: 'prisma/schema.prisma',
  datasource: { url: env('DATABASE_URL') },
});
```

```bash
# .env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp
```

```ts
// src/lib/prisma.ts
import { PrismaClient } from '../generated/prisma/client.js';
import { PrismaPg } from '@prisma/adapter-pg';

const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL! });
export const prisma = new PrismaClient({ adapter });
```

## Commands

```bash
npx prisma migrate dev --name init   # create + apply migration
npx prisma migrate deploy            # production
npx prisma generate                  # after schema edits
npx prisma studio                    # data browser
npx prisma db push                   # no migrations (Mongo/prototypes)
npx prisma db pull                   # introspect existing DB
npx prisma format
npx prisma validate
```

## CRUD

```ts
await prisma.user.create({ data: { email: 'a@example.com', name: 'Alice' } });
await prisma.user.findMany({ where: { email: { contains: '@example.com' } } });
await prisma.user.findUnique({ where: { email: 'a@example.com' } });
await prisma.user.update({ where: { email: 'a@example.com' }, data: { name: 'Bob' } });
await prisma.user.delete({ where: { email: 'a@example.com' } });
```

## MongoDB

Prisma 7 has **no MongoDB support** — use v6:

```bash
npm install -D prisma@6
npm install @prisma/client@6
npx prisma init --datasource-provider mongodb
npx prisma db push
```

```prisma
model User {
  id    String @id @default(auto()) @map("_id") @db.ObjectId
  email String @unique
}
```

> Mongo requires a replica set (Atlas, or a single-node replica set in Docker).
