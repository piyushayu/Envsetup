# Docker — Setup, Commands & Compose

> Quick commands: [CHEATSHEET.md](CHEATSHEET.md)

Docker packages an app and its dependencies into an **image**; a running instance is a **container**. Compose runs several containers together (app + Postgres + Mongo...).

- Desktop app: **Docker Desktop** (Windows/macOS) or **Docker Engine** (Linux)
- Compose command: `docker compose` (**v2 — note the space**, not `docker-compose`)
- Written against Docker Engine 28+ / Compose v2

---

## 1. Install (Windows)

### Prerequisites

- Windows 10/11 64-bit
- **WSL 2** enabled
- Virtualization enabled in BIOS

```powershell
# Install WSL (one time, then reboot)
wsl --install

# Check
wsl --status
```

### Install Docker Desktop

```powershell
winget install Docker.DockerDesktop
```

Or download: https://www.docker.com/products/docker-desktop/

After install: launch Docker Desktop → accept the license → **Use WSL 2 based engine** → Apply & Restart.

Verify:

```bash
docker --version          # Docker version 28.x
docker compose version    # Docker Compose version v2.x
docker run hello-world    # pulls and runs a test container
```

**macOS:** `brew install --cask docker` (or the installer).
**Linux:** install Docker Engine + the compose plugin from https://docs.docker.com/engine/install/.

> If `docker` says *"cannot connect to the Docker daemon"*, Docker Desktop isn't running. Start it and wait for the whale icon to stop animating.

---

## 2. Core commands

### Images

| Task | Command |
| --- | --- |
| List images | `docker images` |
| Pull an image | `docker pull postgres:18-alpine` |
| Build an image | `docker build -t my-app:1.0 .` |
| Build without cache | `docker build --no-cache -t my-app:1.0 .` |
| Remove an image | `docker rmi my-app:1.0` |
| Image history | `docker history my-app:1.0` |
| Inspect | `docker inspect postgres:18-alpine` |

### Containers

| Task | Command |
| --- | --- |
| List running | `docker ps` |
| List all (incl. stopped) | `docker ps -a` |
| Run in background | `docker run -d --name my-container image` |
| Run interactively | `docker run -it --rm image sh` |
| Publish a port | `-p 8080:80` (host:container) |
| Mount a volume | `-v mydata:/data` |
| Pass env vars | `-e KEY=value` |
| Read env file | `--env-file .env` |
| Auto-restart | `--restart unless-stopped` |
| Logs (follow) | `docker logs -f my-container` |
| Shell inside | `docker exec -it my-container sh` |
| Start / stop | `docker start my-container` / `docker stop my-container` |
| Restart | `docker restart my-container` |
| Remove | `docker rm -f my-container` |
| Stats (CPU/RAM) | `docker stats` |
| Copy files in/out | `docker cp ./file my-container:/app/file` |

### Cleanup

| Task | Command |
| --- | --- |
| Remove stopped containers | `docker container prune` |
| Remove unused images | `docker image prune -a` |
| Remove unused volumes | `docker volume prune` |
| Remove unused networks | `docker network prune` |
| Nuke everything unused | `docker system prune -a --volumes` |
| Disk usage | `docker system df` |

> `prune` is permanent. `docker system prune -a --volumes` deletes **all** stopped containers, unused images and volumes — including database data in unused volumes.

---

## 3. Compose — the file

Create `docker-compose.yml` at the root of your project:

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:18-alpine
    container_name: dev-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
      POSTGRES_DB: ${POSTGRES_DB:-myapp}
    ports:
      - "5432:5432"
    volumes:
      # Postgres 18: mount the parent dir, not .../data
      - pg_data:/var/lib/postgresql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-postgres} -d ${POSTGRES_DB:-myapp}"]
      interval: 10s
      timeout: 5s
      retries: 5

  mongo:
    image: mongo:8.0
    container_name: dev-mongo
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER:-admin}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD:-admin123}
      MONGO_INITDB_DATABASE: ${MONGO_DB:-myapp}
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    healthcheck:
      test: ["CMD", "mongosh", "--quiet", "--eval", "db.adminCommand('ping').ok"]
      interval: 10s
      timeout: 5s
      retries: 5

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: dev-pgadmin
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_EMAIL:-admin@local.dev}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD:-admin}
      PGADMIN_CONFIG_SERVER_MODE: "False"
    ports:
      - "5050:80"
    volumes:
      - pgadmin_data:/var/lib/pgadmin

  mongo-express:
    image: mongo-express:latest
    container_name: dev-mongo-express
    restart: unless-stopped
    depends_on:
      mongo:
        condition: service_healthy
    environment:
      ME_CONFIG_MONGODB_URL: mongodb://${MONGO_USER:-admin}:${MONGO_PASSWORD:-admin123}@mongo:27017/
      ME_CONFIG_BASICAUTH_USERNAME: ${ME_USER:-admin}
      ME_CONFIG_BASICAUTH_PASSWORD: ${ME_PASSWORD:-admin}
    ports:
      - "8081:8081"

volumes:
  pg_data:
  mongo_data:
  pgadmin_data:
```

Matching `.env` (same folder, **never commit**):

```bash
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=myapp

MONGO_USER=admin
MONGO_PASSWORD=admin123
MONGO_DB=myapp

PGADMIN_EMAIL=admin@local.dev
PGADMIN_PASSWORD=admin

ME_USER=admin
ME_PASSWORD=admin
```

### What you get

| Service | URL / connection | Credentials |
| --- | --- | --- |
| PostgreSQL | `localhost:5432` | `postgres` / `postgres`, db `myapp` |
| pgAdmin | http://localhost:5050 | `admin@local.dev` / `admin` |
| MongoDB | `localhost:27017` | `admin` / `admin123` |
| mongo-express | http://localhost:8081 | `admin` / `admin` |

> Inside Compose, containers talk to each other by **service name**: use `postgres:5432` and `mongo:27017`, not `localhost`.
> From your host machine, use `localhost:<published-port>`.
> pgAdmin: add a server with host `postgres`, port `5432`, user/password from the env file.

---

## 4. Compose — the commands to control it

| Task | Command |
| --- | --- |
| Start everything (background) | `docker compose up -d` |
| Start and rebuild images | `docker compose up -d --build` |
| Start only one service | `docker compose up -d postgres` |
| Force recreate containers | `docker compose up -d --force-recreate` |
| List services + status | `docker compose ps` |
| Logs (follow, all) | `docker compose logs -f` |
| Logs for one service | `docker compose logs -f postgres` |
| Shell in a service | `docker compose exec postgres bash` |
| Run `psql` in the DB | `docker compose exec postgres psql -U postgres -d myapp` |
| Run `mongosh` | `docker compose exec mongo mongosh -u admin -p admin123` |
| Restart a service | `docker compose restart postgres` |
| Stop containers (keep data) | `docker compose stop` |
| Stop + remove containers/networks | `docker compose down` |
| **Also delete volumes/data** | `docker compose down -v` |
| Pull latest images | `docker compose pull` |
| Validate the file | `docker compose config` |
| List compose projects | `docker compose ls` |

> `docker compose down` keeps named volumes. `down -v` deletes the database data — use it only when you want a clean slate.

### Reset a database safely

```bash
# Stop and delete only the Postgres data, keep images
docker compose down
docker volume rm <project>_pg_data
docker compose up -d

# Find the real volume name
docker volume ls | findstr pg_data      # PowerShell/cmd
docker volume ls | grep pg_data         # bash
```

---

## 5. Dockerfile for a Node app

Multi-stage build → small production image.

```dockerfile
# Dockerfile
FROM node:24-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:24-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:24-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next ./.next        # Next.js
COPY --from=builder /app/public ./public      # Next.js
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./
EXPOSE 3000
CMD ["npm", "run", "start"]
```

For a plain Vite/Node app, replace the build output:

```dockerfile
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

Build and run:

```bash
docker build -t my-app:1.0 .
docker run -d --name my-app -p 3000:3000 --env-file .env my-app:1.0
docker logs -f my-app
docker stop my-app && docker rm my-app
```

### `.dockerignore` (always add one)

```
node_modules
.next
dist
.git
.env
.env.local
*.log
```

---

## 6. Simple one-liners (no Compose)

```bash
# PostgreSQL
docker run -d --name pg-dev -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=myapp -p 5432:5432 -v pg_data:/var/lib/postgresql postgres:18-alpine

# MongoDB
docker run -d --name mongo-dev -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin123 -p 27017:27017 -v mongo_data:/data/db mongo:8.0

# Redis (bonus)
docker run -d --name redis-dev -p 6379:6379 -v redis_data:/data redis:7-alpine
```

---

## 7. Gotchas

| Symptom | Cause / fix |
| --- | --- |
| `docker: command not found` | Docker Desktop not installed or not in PATH. Restart the terminal after install. |
| `Cannot connect to the Docker daemon` | Start Docker Desktop and wait for it to finish booting. |
| `docker-compose: command not found` | Use `docker compose` (v2, space). `docker-compose` is the legacy v1 binary. |
| `port is already allocated` | Something else uses the port. `netstat -ano \| findstr :5432`, stop it, or change the host port (`"5433:5432"`). |
| Compose file ignored | File must be named `docker-compose.yml`, `docker-compose.yaml`, `compose.yml` or `compose.yaml` in the current directory. |
| `permission denied` on Linux | Add your user to the docker group: `sudo usermod -aG docker $USER`, then log out/in. |
| Container exits immediately | Check `docker logs <container>`. Usually a missing env var or a command that isn't a foreground process. |
| `no space left on device` | `docker system df` to inspect, then prune. Docker Desktop also has a disk image size setting. |
| Data lost after `down` | You used `down -v`, or you didn't use a named volume. |
| Postgres volume empty on v18 | Mount `/var/lib/postgresql`, not `/var/lib/postgresql/data`. |
| Slow builds on Windows | Keep the project inside the WSL filesystem (`\\wsl$`) or use a `.dockerignore`. |
| Env vars not reaching the app | Compose `environment:` ≠ your app's `.env`; pass them explicitly or use `env_file:`. |
| `exec format error` on ARM Mac | Build with `--platform linux/amd64` for amd64 images. |

### Check what is actually running

```bash
docker compose ps
docker compose logs --tail=100
docker inspect <container> --format '{{.State.Status}} {{.State.ExitCode}}'
docker port <container>
```

---

## 8. Official docs

- Docker: https://docs.docker.com/
- Compose file reference: https://docs.docker.com/compose/compose-file/
- Dockerfile reference: https://docs.docker.com/reference/dockerfile/
- Postgres image: https://hub.docker.com/_/postgres
- Mongo image: https://hub.docker.com/_/mongo
