# Environment Setup & Command Reference

Install steps, project layout, and the commands needed to run and control each stack — kept in one place so any machine can be set up from scratch.

**Windows-first.** Every command is written for **PowerShell / Git Bash on Windows**. Each guide includes a small macOS/Linux equivalents table where the commands differ.

> **In a hurry?** Every folder has a `CHEATSHEET.md` with only the common install/run/stop commands and the minimal code for that stack.

## Guides

| Stack | Full guide | Quick commands | What it covers |
| --- | --- | --- | --- |
| React (Vite) | [react/README.md](react/README.md) | [react/CHEATSHEET.md](react/CHEATSHEET.md) | Scaffold, dev server, build, env vars, port control |
| Next.js | [nextjs/README.md](nextjs/README.md) | [nextjs/CHEATSHEET.md](nextjs/CHEATSHEET.md) | `create-next-app`, App Router layout, dev/build/start |
| MongoDB | [mongodb/README.md](mongodb/README.md) | [mongodb/CHEATSHEET.md](mongodb/CHEATSHEET.md) | Atlas, local install, Docker, `mongosh`, connection strings |
| PostgreSQL | [postgresql/README.md](postgresql/README.md) | [postgresql/CHEATSHEET.md](postgresql/CHEATSHEET.md) | Local install, Docker, `psql`, users and databases |
| Docker | [docker/README.md](docker/README.md) | [docker/CHEATSHEET.md](docker/CHEATSHEET.md) | Docker Desktop, core commands, Dockerfile, Compose (Postgres + Mongo) |
| Prisma ORM | [prisma/README.md](prisma/README.md) | [prisma/CHEATSHEET.md](prisma/CHEATSHEET.md) | Schema, migrations, Studio, v7 changes |
| Tailwind CSS | [tailwindcss/README.md](tailwindcss/README.md) | [tailwindcss/CHEATSHEET.md](tailwindcss/CHEATSHEET.md) | v4 setup for Vite and Next.js |
| Git & GitHub | [git-github/README.md](git-github/README.md) | [git-github/CHEATSHEET.md](git-github/CHEATSHEET.md) | Config, SSH keys, daily commands, undoing mistakes |

## Suggested order

1. **Node.js** — install it once; React, Next.js, Tailwind and Prisma all depend on it. See [react/README.md](react/README.md#1-prerequisites).
2. **Docker Desktop** — the fastest way to get databases running without installing them globally: [docker/README.md](docker/README.md).
3. **Database** — start Postgres or Mongo, then follow [postgresql/README.md](postgresql/README.md) or [mongodb/README.md](mongodb/README.md).
4. **App stack** — [react/README.md](react/README.md) or [nextjs/README.md](nextjs/README.md), then add [Tailwind CSS](tailwindcss/README.md) and [Prisma](prisma/README.md).
5. **Source control** — [git-github/README.md](git-github/README.md).

## Versions this was written against (Nov 2026)

| Tool | Version | Notes |
| --- | --- | --- |
| Node.js | 24.x LTS "Krypton" | 26.x LTS also available; 24 is the safe default |
| npm | 11.x | Ships with Node 24 |
| React | 19.3 | |
| Next.js | 16.4 | Requires Node >= 20.9 |
| Tailwind CSS | 4.3 | v4 is CSS-first; no `tailwind.config.js` by default |
| Prisma | 7.10 (PostgreSQL) | v7 requires driver adapters + ESM |
| Prisma | 6.x (MongoDB) | **v7 does not support MongoDB yet** |
| MongoDB | 8.0 | |
| PostgreSQL | 18 | |

> Commands are written for the version above. If a flag has changed, the version's official docs are linked at the bottom of each guide.

## Conventions used in these guides

- `PS>` marks Windows PowerShell commands; plain blocks are cross-platform bash.
- `<angle-brackets>` are placeholders — replace them, don't type the brackets.
- `#` starts a comment and is not part of the command.
- All "stop / clean up" commands are listed next to the "start" command so you can find them fast.
