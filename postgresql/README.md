# PostgreSQL — Setup, Connection & Commands

> Quick commands: [CHEATSHEET.md](CHEATSHEET.md)

PostgreSQL is a relational database (tables, rows, SQL, ACID transactions).

- Default port: **5432**
- Default superuser: `postgres`
- CLI: `psql`
- Written against PostgreSQL 18

---

## 1. Install

### Windows (installer — recommended)

```powershell
# Search for the current package id
winget search postgresql

# Then install it (example id — use what the search returns)
winget install PostgreSQL.PostgreSQL.18
```

Or download the EDB installer: https://www.postgresql.org/download/windows/

During install you will be asked for:

| Prompt | What to choose |
| --- | --- |
| Installation directory | Default (`C:\Program Files\PostgreSQL\18`) |
| Components | PostgreSQL Server, pgAdmin 4, **Command Line Tools** |
| Data directory | Default (`C:\Program Files\PostgreSQL\18\data`) |
| Password | A password for the `postgres` superuser — **write it down** |
| Port | `5432` (change only if it's taken) |
| Locale | Default |

Verify:

```powershell
psql --version
# psql (PostgreSQL) 18.x
```

If `psql` is not recognized, add `C:\Program Files\PostgreSQL\18\bin` to **PATH**:

```powershell
# Current session only
$env:Path += ";C:\Program Files\PostgreSQL\18\bin"

# Permanent (new terminals)
[Environment]::SetEnvironmentVariable(
  "Path",
  [Environment]::GetEnvironmentVariable("Path", "Machine") + ";C:\Program Files\PostgreSQL\18\bin",
  "Machine"
)
```

### macOS / Linux

```bash
# macOS
brew install postgresql@18
brew services start postgresql@18

# Ubuntu / Debian
sudo apt update
sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

---

## 2. Control the service (Windows)

```powershell
# Find the service name (usually postgresql-x64-18)
Get-Service *postgres*

# Start / stop / restart
Start-Service postgresql-x64-18
Stop-Service postgresql-x64-18
Restart-Service postgresql-x64-18

# Check status
Get-Service postgresql-x64-18
```

Alternative (cmd):

```bat
net start postgresql-x64-18
net stop postgresql-x64-18
```

macOS: `brew services start|stop|restart postgresql@18`
Linux: `sudo systemctl start|stop|restart postgresql`

---

## 3. Connect with `psql`

```bash
# Local, as the postgres superuser
psql -U postgres -h localhost -p 5432

# Straight into a database
psql -U postgres -h localhost -d myapp

# Connection URL form
psql "postgresql://postgres:yourpassword@localhost:5432/myapp"
```

### Inside `psql` — meta-commands

| Command | Meaning |
| --- | --- |
| `\l` | List databases |
| `\c myapp` | Connect to `myapp` |
| `\dt` | List tables |
| `\d users` | Describe the `users` table |
| `\du` | List roles/users |
| `\dn` | List schemas |
| `\df` | List functions |
| `\x` | Toggle expanded output (wide rows) |
| `\i file.sql` | Run a SQL file |
| `\timing` | Show query duration |
| `\q` | Quit |

> On Windows, `psql` may require the password each time. Set `PGPASSWORD` for a session (avoid committing it):
> `$env:PGPASSWORD="yourpassword"`.
> For convenience, create `%APPDATA%\postgresql\pgpass.conf` with `localhost:5432:*:postgres:yourpassword`.

---

## 4. Create a database and user

```sql
-- Connect as postgres first: psql -U postgres
CREATE DATABASE myapp;

CREATE USER app_user WITH ENCRYPTED PASSWORD 'app_password';
GRANT ALL PRIVILEGES ON DATABASE myapp TO app_user;

-- PostgreSQL 15+: also grant schema privileges
\c myapp
GRANT ALL ON SCHEMA public TO app_user;
ALTER SCHEMA public OWNER TO app_user;
```

### Minimal table example

```sql
CREATE TABLE users (
  id          BIGSERIAL PRIMARY KEY,
  email       TEXT NOT NULL UNIQUE,
  name        TEXT NOT NULL,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

INSERT INTO users (email, name) VALUES ('a@example.com', 'Alice');

SELECT * FROM users ORDER BY created_at DESC LIMIT 10;
```

### Useful SQL

| Task | Statement |
| --- | --- |
| List databases | `SELECT datname FROM pg_database;` |
| Current database | `SELECT current_database();` |
| Tables | `SELECT * FROM information_schema.tables WHERE table_schema='public';` |
| Table size | `SELECT pg_size_pretty(pg_total_relation_size('users'));` |
| Active connections | `SELECT * FROM pg_stat_activity;` |
| Kill a connection | `SELECT pg_terminate_backend(<pid>);` |
| Drop database | `DROP DATABASE myapp;` (must not be connected to it) |

---

## 5. Docker option

```bash
docker run -d \
  --name pg-dev \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  -v pg_data:/var/lib/postgresql \
  postgres:18-alpine
```

| Action | Command |
| --- | --- |
| Logs | `docker logs -f pg-dev` |
| Shell in container | `docker exec -it pg-dev psql -U postgres -d myapp` |
| Stop / start | `docker stop pg-dev` / `docker start pg-dev` |
| Remove container | `docker rm -f pg-dev` |
| Remove data too | `docker volume rm pg_data` |

> **PostgreSQL 18 image change:** the data directory is `/var/lib/postgresql/18/docker`, so mount volumes at **`/var/lib/postgresql`** (not `/var/lib/postgresql/data`). On the `postgres:17` image and older, use `/var/lib/postgresql/data`.

Full Compose setup with pgAdmin: [../docker/README.md](../docker/README.md).

---

## 6. Connection strings

```
# Local
postgresql://postgres:yourpassword@localhost:5432/myapp

# Docker
postgresql://postgres:postgres@localhost:5432/myapp

# With SSL (hosted providers)
postgresql://user:pass@host:5432/dbname?sslmode=require
```

```bash
# .env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp
```

| Part | Meaning |
| --- | --- |
| `postgresql://` | Scheme (`postgres://` also works) |
| `user:password` | Credentials — percent-encode special characters |
| `localhost:5432` | Host and port |
| `/myapp` | Database name |
| `?sslmode=require` | Force TLS (hosted DBs) |

> Prisma uses the same `DATABASE_URL` format. See [../prisma/README.md](../prisma/README.md).

### Node.js (`pg`) example

```bash
npm install pg
```

```ts
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10,
  idleTimeoutMillis: 30_000,
});

export async function getUsers() {
  const { rows } = await pool.query('SELECT id, email, name FROM users LIMIT 10');
  return rows;
}
```

Test the connection:

```bash
psql "$DATABASE_URL" -c "SELECT version();"
```

---

## 7. Backup and restore

```bash
# Backup one database
pg_dump -U postgres -h localhost -d myapp -F c -f myapp.backup

# Restore into an existing database
pg_restore -U postgres -h localhost -d myapp --clean --if-exists myapp.backup

# Plain SQL dump
pg_dump -U postgres -d myapp > myapp.sql
psql -U postgres -d myapp -f myapp.sql
```

From Docker:

```bash
docker exec pg-dev pg_dump -U postgres -d myapp > myapp.sql
```

> `pg_dump` version must be >= the server version. Use the matching client tools.

---

## 8. GUI tools

| Tool | Notes |
| --- | --- |
| **pgAdmin 4** | Installed with PostgreSQL on Windows |
| **DBeaver** | Free, works with every database |
| **TablePlus** | Fast, paid tiers |
| **psql** | Always available, scriptable |

Connection fields: host `localhost`, port `5432`, database `myapp`, user `postgres`, password from install.

---

## 9. Gotchas

| Symptom | Cause / fix |
| --- | --- |
| `psql: command not found` | `bin` folder not on PATH — see the PATH snippet above. |
| `password authentication failed for user "postgres"` | Wrong password. Reset it (see below) or check `pg_hba.conf`. |
| `FATAL: database "myapp" does not exist` | Create it: `CREATE DATABASE myapp;`. Prisma will not create the database in all setups. |
| `could not connect to server: Connection refused` | Service isn't running, or wrong port. `Get-Service *postgres*`. |
| `port 5432 already in use` | Another Postgres instance (or Docker container) is running. `netstat -ano \| findstr :5432`, then stop the other one or use `-p 5433`. |
| `role "app_user" does not exist` | Create the role, or connect with the right user. |
| `permission denied for schema public` | Postgres 15+ changed defaults: `GRANT ALL ON SCHEMA public TO app_user;` as owner/superuser. |
| `database is being accessed by other users` | `SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='myapp' AND pid <> pg_backend_pid();` then drop. |
| Docker volume "empty" after recreate | You mounted `/var/lib/postgresql/data` on Postgres 18. Mount `/var/lib/postgresql` instead. |
| SSL errors from a hosted DB | Add `?sslmode=require` (or `verify-full` with a CA cert). |

### Reset the `postgres` password (Windows)

1. Open `C:\Program Files\PostgreSQL\18\data\pg_hba.conf` as admin.
2. Temporarily change the `host all all 127.0.0.1/32 scram-sha-256` line to `trust`.
3. Restart the service: `Restart-Service postgresql-x64-18`.
4. Run `psql -U postgres -h localhost` and execute:
   `ALTER USER postgres WITH PASSWORD 'newpassword';`
5. Restore `scram-sha-256` in `pg_hba.conf` and restart again.

---

## 10. Official docs

- PostgreSQL: https://www.postgresql.org/docs/current/
- `psql`: https://www.postgresql.org/docs/current/app-psql.html
- Downloads: https://www.postgresql.org/download/
- Docker image: https://hub.docker.com/_/postgres
