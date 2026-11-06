# Prisma ORM — Setup & Commands

> Quick commands: [CHEATSHEET.md](CHEATSHEET.md)

Prisma is a type-safe ORM for Node.js/TypeScript: you define models in `schema.prisma`, and Prisma generates a typed client.

> **Critical version note (Nov 2026):**
> - **Prisma 7 does not support MongoDB.** Use Prisma 6 for Mongo.
> - **Prisma 7 requires a driver adapter and ESM** (`"type": "module"`), and the generated client must have an explicit `output` path.
> - Prisma 8 is in release-candidate — don't use it for new work yet.

---

## 1. Install

### PostgreSQL (Prisma 7)

```bash
npm install -D prisma
npm install @prisma/client @prisma/adapter-pg

npx prisma init --datasource-provider postgresql
```

### MongoDB (Prisma 6 — the last version with Mongo support)

```bash
npm install -D prisma@6
npm install @prisma/client@6

npx prisma init --datasource-provider mongodb
```

Requirements: Node >= 20.19, TypeScript >= 5.4.

---

## 2. Prisma 7 + PostgreSQL setup

### `package.json`

Prisma 7 ships as ESM:

```json
{
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:deploy": "prisma migrate deploy",
    "db:studio": "prisma studio"
  }
}
```

### `tsconfig.json` (relevant parts)

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "target": "ES2023",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### `prisma/schema.prisma`

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
  updatedAt DateTime @updatedAt
  posts     Post[]
}

model Post {
  id       String @id @default(cuid())
  title    String
  body     String?
  author   User   @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId String
}
```

### `prisma.config.ts`

In v7 the connection URL lives in the Prisma config, not in the schema:

```ts
// prisma.config.ts
import 'dotenv/config';
import { defineConfig, env } from 'prisma/config';

export default defineConfig({
  schema: 'prisma/schema.prisma',
  datasource: {
    url: env('DATABASE_URL'),
  },
});
```

```bash
npm install dotenv
```

### `.env`

```bash
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp
```

### Client singleton

```ts
// src/lib/prisma.ts
import { PrismaClient } from '../generated/prisma/client.js';
import { PrismaPg } from '@prisma/adapter-pg';

const adapter = new PrismaPg({
  connectionString: process.env.DATABASE_URL!,
});

const globalForPrisma = globalThis as unknown as {
  prisma?: PrismaClient;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({ adapter });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

> With the new generator, the import is from **your generated path**, not `@prisma/client`.
> In a `tsconfig` using `moduleResolution: bundler` you can usually drop the `.js` extension.

---

## 3. Prisma 6 + MongoDB setup

### `prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongodb"
  url      = env("MONGODB_URI")
}

model User {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  posts     Post[]
}

model Post {
  id       String @id @default(auto()) @map("_id") @db.ObjectId
  title    String
  body     String?
  author   User   @relation(fields: [authorId], references: [id])
  authorId String @db.ObjectId
}
```

### `.env`

```bash
MONGODB_URI=mongodb://admin:admin123@localhost:27017/myapp?authSource=admin
```

### Client

```ts
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'warn', 'error'] : ['error'],
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

### Mongo-specific rules

- `db push` only — **migrations are not supported on MongoDB**.
- Embed documents (nested objects/arrays) instead of using relations where possible.
- Mongo requires a **replica set** (Atlas provides one; a single `docker run mongo` does not — see [../mongodb/README.md](../mongodb/README.md)).
- Every model needs `@id @default(auto()) @map("_id") @db.ObjectId`.

---

## 4. Commands you need

| Task | Command |
| --- | --- |
| Create schema/config files | `npx prisma init --datasource-provider postgresql` |
| Generate the client | `npx prisma generate` |
| Create + apply a migration (dev) | `npx prisma migrate dev --name add_users` |
| Apply pending migrations (prod) | `npx prisma migrate deploy` |
| Migration status | `npx prisma migrate status` |
| Reset DB + rerun migrations | `npx prisma migrate reset` |
| Push schema without migrations | `npx prisma db push` |
| Pull schema from an existing DB | `npx prisma db pull` |
| Open the data browser | `npx prisma studio` |
| Format the schema | `npx prisma format` |
| Validate the schema | `npx prisma validate` |
| Seed the database | `npx prisma db seed` |
| Show version | `npx prisma --version` |

> `npx prisma generate` must be re-run after **any** schema change. In v7, add it to `postinstall` or the build step so deploys always use a fresh client.
> `migrate dev` is for development only — never run it against production. Use `migrate deploy` there.

### Migrations vs. `db push`

| | `migrate dev` | `db push` |
| --- | --- | --- |
| Creates migration files | Yes | No |
| Version history | Yes | No |
| Production safe | Yes (`migrate deploy`) | Prototype only |
| MongoDB | Not supported | The only option |
| Best for | Relational (Postgres/MySQL/SQLite) | Prototypes, Mongo |

### Typical dev loop

```bash
# 1. Edit prisma/schema.prisma
# 2. Apply
npx prisma migrate dev --name describe_the_change

# 3. Client is regenerated automatically; restart the dev server if types lag
```

---

## 5. Industry-standard project structure

```
my-app/
├── prisma/
│   ├── schema.prisma
│   ├── migrations/            # generated SQL — commit these
│   └── seed.ts
├── prisma.config.ts           # Prisma 7: connection + config
├── generated/
│   └── prisma/                # generated client — gitignore it
├── src/
│   ├── lib/
│   │   └── prisma.ts          # singleton client
│   ├── services/              # business logic using prisma
│   └── ...
├── .env                       # DATABASE_URL — never commit
├── .env.example
└── .gitignore
```

`.gitignore` additions:

```gitignore
.env
.env.local
/generated
node_modules
```

> Commit `prisma/migrations/`. Do **not** commit `.env` or the generated client.

---

## 6. Common queries

```ts
// Create
const user = await prisma.user.create({
  data: { email: 'a@example.com', name: 'Alice' },
});

// Read
const users = await prisma.user.findMany({
  where: { email: { contains: '@example.com' } },
  orderBy: { createdAt: 'desc' },
  take: 20,
});

// Read one
const one = await prisma.user.findUnique({ where: { id: 'abc' } });

// Update
await prisma.user.update({
  where: { id: 'abc' },
  data: { name: 'Alice Smith' },
});

// Upsert
await prisma.user.upsert({
  where: { email: 'a@example.com' },
  update: { name: 'Alice' },
  create: { email: 'a@example.com', name: 'Alice' },
});

// Delete
await prisma.user.delete({ where: { id: 'abc' } });

// Relation + pagination
const posts = await prisma.post.findMany({
  include: { author: true },
  skip: 0,
  take: 10,
});

// Transaction
await prisma.$transaction([
  prisma.user.update({ where: { id: 'a' }, data: { name: 'A' } }),
  prisma.post.delete({ where: { id: 'b' } }),
]);
```

### Seed example

```ts
// prisma/seed.ts
import { PrismaClient } from '../generated/prisma/client.js';

const prisma = new PrismaClient();

async function main() {
  await prisma.user.upsert({
    where: { email: 'admin@example.com' },
    update: {},
    create: { email: 'admin@example.com', name: 'Admin' },
  });
}

main()
  .then(() => prisma.$disconnect())
  .catch(async (e) => {
    console.error(e);
    await prisma.$disconnect();
    process.exit(1);
  });
```

Register the seed command in `prisma.config.ts` (v7) or the `prisma.seed` field (v6), then run `npx prisma db seed`.

---

## 7. Gotchas

| Error / symptom | Cause / fix |
| --- | --- |
| `P1001: Can't reach database server` | DB not running, wrong host/port, or Docker container stopped. Test with `psql`/`mongosh`. |
| `P1000: Authentication failed` | Wrong credentials in `DATABASE_URL`; special chars need percent-encoding. |
| `P2002: Unique constraint failed` | Duplicate value — use `upsert`, or check for an existing row first. |
| `P2025: Record to update not found` | The `where` didn't match; verify the id. |
| `Environment variable not found: DATABASE_URL` | `.env` missing, or `prisma.config.ts` lacks `import 'dotenv/config'`. |
| Client is `undefined` / types missing | Run `npx prisma generate`; restart the TS server (`Ctrl+Shift+P` → *Restart TS Server*). |
| `PrismaClient needs an adapter` (v7) | Pass `new PrismaClient({ adapter })` using `@prisma/adapter-pg`. |
| `Cannot use import statement outside a module` (v7) | Add `"type": "module"` to `package.json` and use ESM-compatible tsconfig. |
| `MongoDB is not supported in Prisma 7` | Install `prisma@6` and `@prisma/client@6`. |
| Migration drift | `npx prisma migrate status`, then `migrate reset` in dev or create a corrective migration. Never edit applied migration files. |
| `migrate dev` fails on Mongo | Use `db push` — migrations don't exist for Mongo. |
| Transaction errors on Mongo | Needs a replica set. |
| Slow app after deploy | Client wasn't generated during build — add `prisma generate` to `postinstall`/build. |

---

## 8. Official docs

- Prisma ORM: https://www.prisma.io/docs/orm
- Prisma 7 upgrade guide: https://www.prisma.io/docs/guides/upgrade-prisma-orm/v7
- Migrations: https://www.prisma.io/docs/orm/prisma-migrate
- MongoDB (v6): https://www.prisma.io/docs/orm/overview/databases/mongodb
