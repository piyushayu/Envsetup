# MongoDB — Cheat Sheet

Full guide: [README.md](README.md)

## Run (Docker)

```bash
docker run -d --name mongo-dev \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=admin123 \
  -p 27017:27017 \
  -v mongo_data:/data/db \
  mongo:8.0
```

```bash
docker stop mongo-dev
docker start mongo-dev
docker logs -f mongo-dev
docker rm -f mongo-dev
```

## Connect

```bash
docker exec -it mongo-dev mongosh -u admin -p admin123
mongosh "mongodb://admin:admin123@localhost:27017/myapp?authSource=admin"
```

```
mongodb://localhost:27017/myapp
mongodb://admin:admin123@localhost:27017/myapp?authSource=admin
mongodb+srv://<user>:<pass>@cluster0.xxxxx.mongodb.net/myapp   # Atlas
```

## Shell CRUD

```js
use myapp
db.users.insertOne({ name: "Piyush", age: 25 })
db.users.insertMany([{ name: "A" }, { name: "B" }])
db.users.find({ age: { $gte: 18 } })
db.users.updateOne({ name: "Piyush" }, { $set: { age: 26 } })
db.users.deleteOne({ name: "Piyush" })
show collections
db.users.drop()
```

## Node

```bash
# .env
MONGODB_URI=mongodb://admin:admin123@localhost:27017/myapp?authSource=admin
```

```ts
// mongoose
import mongoose from 'mongoose';
await mongoose.connect(process.env.MONGODB_URI!);
```

```ts
// official driver
import { MongoClient } from 'mongodb';
const client = new MongoClient(process.env.MONGODB_URI!);
await client.connect();
const db = client.db('myapp');
```

## Backup

```bash
mongodump --uri="$MONGODB_URI" --out=./dump
mongorestore --uri="$MONGODB_URI" ./dump/myapp
```
