# PostgreSQL — Cheat Sheet

Full guide: [README.md](README.md)

## Run (Docker)

```bash
docker run -d --name pg-dev \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  -v pg_data:/var/lib/postgresql \
  postgres:18-alpine
```

> Postgres 18 mounts at `/var/lib/postgresql`. Use `/var/lib/postgresql/data` on v17 and older.

## Windows service

```powershell
Start-Service postgresql-x64-18
Stop-Service postgresql-x64-18
Restart-Service postgresql-x64-18
Get-Service *postgres*
```

## Connect

```bash
psql -U postgres -h localhost -d myapp
psql "postgresql://postgres:postgres@localhost:5432/myapp"
```

```bash
# .env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp
```

## Setup SQL

```sql
CREATE DATABASE myapp;
CREATE USER app_user WITH ENCRYPTED PASSWORD 'app_password';
GRANT ALL PRIVILEGES ON DATABASE myapp TO app_user;

\c myapp
GRANT ALL ON SCHEMA public TO app_user;
ALTER SCHEMA public OWNER TO app_user;
```

## psql commands

| Command | Meaning |
| --- | --- |
| `\l` | list databases |
| `\c myapp` | connect |
| `\dt` | list tables |
| `\d users` | describe table |
| `\du` | list users |
| `\q` | quit |

## Node

```ts
import { Pool } from 'pg';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const { rows } = await pool.query('SELECT * FROM users LIMIT 10');
```

## Backup / restore

```bash
pg_dump -U postgres -d myapp -F c -f myapp.backup
pg_restore -U postgres -d myapp --clean --if-exists myapp.backup
```
