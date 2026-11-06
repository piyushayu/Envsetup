# Docker — Cheat Sheet

Full guide: [README.md](README.md)

## Compose

```bash
docker compose up -d            # start
docker compose ps               # status
docker compose logs -f          # logs
docker compose logs -f postgres # one service
docker compose exec postgres psql -U postgres
docker compose restart postgres
docker compose down             # stop (keeps data)
docker compose down -v          # stop + DELETE data
```

## Minimal `docker-compose.yml`

```yaml
services:
  postgres:
    image: postgres:18-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    ports: ["5432:5432"]
    volumes: ["pg_data:/var/lib/postgresql"]

  mongo:
    image: mongo:8.0
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin123
    ports: ["27017:27017"]
    volumes: ["mongo_data:/data/db"]

volumes:
  pg_data:
  mongo_data:
```

## One-liners

```bash
# PostgreSQL
docker run -d --name pg-dev -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=myapp -p 5432:5432 -v pg_data:/var/lib/postgresql postgres:18-alpine

# MongoDB
docker run -d --name mongo-dev -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin123 -p 27017:27017 -v mongo_data:/data/db mongo:8.0
```

## Container control

```bash
docker ps                       # running
docker ps -a                    # all
docker logs -f <container>
docker exec -it <container> sh
docker stop <container>
docker start <container>
docker rm -f <container>
docker images
docker rmi <image>
```

## Build & run an app

```bash
docker build -t my-app:1.0 .
docker run -d --name my-app -p 3000:3000 --env-file .env my-app:1.0
docker logs -f my-app
```

## Cleanup

```bash
docker system df
docker container prune
docker image prune -a
docker volume prune
docker system prune -a --volumes   # deletes unused volumes too
```

## Health check

```bash
docker --version && docker compose version && docker ps
```
