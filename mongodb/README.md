# MongoDB — Setup, Connection & Commands

> Quick commands: [CHEATSHEET.md](CHEATSHEET.md)

MongoDB is a document database: data is stored as BSON documents inside collections, not rows in tables.

- Default port: **27017**
- Shell: `mongosh`
- Node driver: `mongodb` (official) or `mongoose` (ODM)
- Written against MongoDB 8.0

Pick **one** of the three install paths below.

| Path | Best for | Cost |
| --- | --- | --- |
| [Atlas](#1-option-a--mongodb-atlas-cloud) | Zero local setup, production-like | Free tier (512 MB) |
| [Local install](#2-option-b--local-install-windows) | Offline dev, full control | Free |
| [Docker](#3-option-c--docker-recommended-for-dev) | Clean, disposable, easy reset | Free |

---

## 1. Option A — MongoDB Atlas (cloud)

1. Sign up at https://www.mongodb.com/cloud/atlas/register
2. Create a **free M0 cluster**, pick the region closest to you.
3. **Database Access** → add a user + password.
4. **Network Access** → add your IP. For local dev only, `0.0.0.0/0` works (never use it in production).
5. **Connect → Drivers** → copy the connection string.

```bash
mongodb+srv://<user>:<password>@cluster0.abcde.mongodb.net/<dbname>?retryWrites=true&w=majority
```

> Percent-encode special characters in the password (`@` → `%40`, `#` → `%23`).
> For Atlas, append the database name before the `?` — otherwise you connect to `test`.

---

## 2. Option B — Local install (Windows)

```powershell
# Find the package id first
winget search mongodb

# Server + shell
winget install MongoDB.Server
winget install MongoDB.Shell
```

Or use the MSI installer: https://www.mongodb.com/try/download/community

The MSI installs MongoDB as a **Windows service** (`MongoDB`) that starts automatically.

```powershell
# Verify
mongod --version
mongosh --version

# Control the service
Get-Service MongoDB
Stop-Service MongoDB
Start-Service MongoDB
Restart-Service MongoDB
```

Default data directory: `C:\Program Files\MongoDB\Server\<version>\data\`
Default config: `C:\Program Files\MongoDB\Server\<version>\bin\mongod.cfg`

**macOS / Linux**

```bash
# macOS
brew tap mongodb/brew
brew install mongodb-community@8.0
brew services start mongodb-community@8.0

# Ubuntu (official repo required — see docs)
sudo systemctl start mongod
sudo systemctl status mongod
```

---

## 3. Option C — Docker (recommended for dev)

```bash
# Start a disposable MongoDB
docker run -d \
  --name mongo-dev \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=admin123 \
  -v mongo_data:/data/db \
  mongo:8.0
```

| Action | Command |
| --- | --- |
| Logs | `docker logs -f mongo-dev` |
| Shell inside container | `docker exec -it mongo-dev mongosh -u admin -p admin123` |
| Stop | `docker stop mongo-dev` |
| Start again | `docker start mongo-dev` |
| Remove container (volume survives) | `docker rm -f mongo-dev` |
| Remove container **and data** | `docker rm -f mongo-dev` + `docker volume rm mongo_data` |

Compose version (Postgres + Mongo together): see [../docker/README.md](../docker/README.md).

---

## 4. Connection strings

```
# Local, no auth
mongodb://localhost:27017/mydb

# Local, with auth (Docker example above)
mongodb://admin:admin123@localhost:27017/mydb?authSource=admin

# Atlas
mongodb+srv://<user>:<password>@cluster0.abcde.mongodb.net/mydb?retryWrites=true&w=majority
```

| Part | Meaning |
| --- | --- |
| `mongodb://` | Standard, single host or replica set |
| `mongodb+srv://` | DNS seed list (Atlas) — no port needed |
| `localhost:27017` | Host + port |
| `/mydb` | Database to use |
| `?authSource=admin` | Which DB holds the user credentials (required when the user was created in `admin`) |
| `?retryWrites=true&w=majority` | Atlas default: retry writes, majority write concern |

### `.env` example

```bash
MONGODB_URI=mongodb://admin:admin123@localhost:27017/mydb?authSource=admin
```

```ts
// lib/db.ts — reuse one connection (important in Next.js/dev hot reload)
import { MongoClient } from 'mongodb';

const uri = process.env.MONGODB_URI!;
const options = {};

let client: MongoClient;
let clientPromise: Promise<MongoClient>;

declare global {
  // eslint-disable-next-line no-var
  var _mongoClientPromise: Promise<MongoClient> | undefined;
}

if (process.env.NODE_ENV === 'development') {
  if (!global._mongoClientPromise) {
    client = new MongoClient(uri, options);
    global._mongoClientPromise = client.connect();
  }
  clientPromise = global._mongoClientPromise;
} else {
  client = new MongoClient(uri, options);
  clientPromise = client.connect();
}

export default clientPromise;
```

### Mongoose version

```ts
// lib/mongoose.ts
import mongoose from 'mongoose';

const MONGODB_URI = process.env.MONGODB_URI!;

if (!global.mongoose) global.mongoose = mongoose.connect(MONGODB_URI);

export const db = global.mongoose;

declare global {
  // eslint-disable-next-line no-var
  var mongoose: { connect: () => Promise<typeof import('mongoose')> } | undefined;
}
```

```bash
npm install mongoose
```

| Package | Use when |
| --- | --- |
| `mongodb` | You want the raw driver, full control, no schema layer |
| `mongoose` | You want schemas, validation, middleware, population |

---

## 5. `mongosh` — the commands you need

```bash
# Connect
mongosh
mongosh "mongodb://admin:admin123@localhost:27017/mydb?authSource=admin"
mongosh --host localhost --port 27017
```

Inside the shell:

| Task | Command |
| --- | --- |
| Show databases | `show dbs` |
| Switch / create database | `use mydb` |
| Current database | `db` |
| Show collections | `show collections` |
| Insert one | `db.users.insertOne({ name: "Piyush", age: 25 })` |
| Insert many | `db.users.insertMany([{ name: "A" }, { name: "B" }])` |
| Find all | `db.users.find()` |
| Find with filter | `db.users.find({ age: { $gte: 18 } })` |
| Pretty print | `db.users.find().pretty()` |
| First match | `db.users.findOne({ name: "Piyush" })` |
| Update one | `db.users.updateOne({ name: "Piyush" }, { $set: { age: 26 } })` |
| Delete one | `db.users.deleteOne({ name: "Piyush" })` |
| Count | `db.users.countDocuments()` |
| Create collection | `db.createCollection("posts")` |
| Drop collection | `db.users.drop()` |
| Drop database | `db.dropDatabase()` |
| Index | `db.users.createIndex({ email: 1 }, { unique: true })` |
| List indexes | `db.users.getIndexes()` |
| Users (auth) | `db.getUsers()` |
| Exit | `exit` / `Ctrl + D` |

### Query operators quick reference

| Operator | Meaning |
| --- | --- |
| `$eq`, `$ne` | Equal / not equal |
| `$gt`, `$gte`, `$lt`, `$lte` | Comparisons |
| `$in`, `$nin` | In / not in a list |
| `$and`, `$or`, `$not` | Logical |
| `$set`, `$unset`, `$inc`, `$push`, `$pull` | Update operators |
| `$regex` | Pattern match |

---

## 6. Backups and restore

```bash
# Dump a database to ./dump
mongodump --uri="mongodb://admin:admin123@localhost:27017/mydb" --out=./dump

# Restore
mongorestore --uri="mongodb://admin:admin123@localhost:27017/mydb" ./dump/mydb
```

Inside Docker:

```bash
docker exec mongo-dev mongodump --username admin --password admin123 --authenticationDatabase admin --db mydb --archive=/tmp/mydb.archive
docker cp mongo-dev:/tmp/mydb.archive ./mydb.archive
```

---

## 7. Gotchas

| Symptom | Cause / fix |
| --- | --- |
| `ECONNREFUSED 127.0.0.1:27017` | MongoDB isn't running. `Get-Service MongoDB` / `docker ps` / `brew services list`. |
| `Authentication failed` | Wrong password, or missing `?authSource=admin` for root users. |
| Atlas: `Could not connect to any servers` | Your IP isn't whitelisted, or the cluster is paused. |
| `MongoServerError: bad auth : Authentication failed` | Password contains unescaped special chars, or the user is in a different auth DB. |
| Works in Compass, fails in Node | `.env` not loaded, or the URI is quoted oddly. Test with `console.log(uri)` (redact the password). |
| Transactions fail with `Transaction numbers are only allowed on a replica set` | Local standalone `mongod`/Docker single node doesn't support transactions. Use Atlas, or run a single-node replica set. |
| Prisma with MongoDB fails | **Prisma 7 does not support MongoDB.** Pin `prisma@6` and `@prisma/client@6`. See [../prisma/README.md](../prisma/README.md). |
| Slow queries | Add an index on the fields you filter/sort by; use `.explain("executionStats")`. |
| Disk full on Windows service | Check the `storage.dbPath` in `mongod.cfg` and clear old logs. |

### Single-node replica set in Docker (for transactions)

```bash
docker run -d --name mongo-rs -p 27017:27017 mongo:8.0 --replSet rs0
docker exec -it mongo-rs mongosh --eval "rs.initiate()"
```

Connection string: `mongodb://localhost:27017/mydb?directConnection=true`

---

## 8. GUI tools

| Tool | Notes |
| --- | --- |
| **MongoDB Compass** | Official, free — https://www.mongodb.com/products/compass |
| **Studio 3T** | Power-user features, paid tiers |
| **mongosh** | Terminal, always available |

Compass connection string is the same as above — paste and connect.

---

## 9. Official docs

- MongoDB manual: https://www.mongodb.com/docs/manual/
- `mongosh`: https://www.mongodb.com/docs/mongodb-shell/
- Node driver: https://www.mongodb.com/docs/drivers/node/
- Atlas: https://www.mongodb.com/docs/atlas/
