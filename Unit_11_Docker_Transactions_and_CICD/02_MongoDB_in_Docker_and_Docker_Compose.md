# Unit 11 — Docker, Transactions & CI/CD
## Module 02: MongoDB in Docker & Docker Compose

**SDET Priority:** ⭐⭐⭐⭐ (Essential for Containerized Automation & Local Environment Setup)

---

## 1. Overview & SDET Benefits

Running MongoDB in Docker containers eliminates "works on my machine" issues. SDETs can spin up isolated, throwaway MongoDB instances locally or on CI runners in seconds without installing native database services on host machines.

---

## 2. Running MongoDB via Single Docker Command

```bash
# Pull official MongoDB image and run container in detached mode on port 27017
docker run -d \
  --name sdet-mongo-test \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password123 \
  mongo:latest

# Access mongosh shell directly inside running container
docker exec -it sdet-mongo-test mongosh -u admin -p password123

# Inspect container execution logs
docker logs sdet-mongo-test

# Stop and remove container after test suite run
docker stop sdet-mongo-test && docker rm sdet-mongo-test
```

---

## 3. Local Test Environment with `docker-compose.yml`

A complete local test environment setup containing **MongoDB** and **Mongo Express** (Web GUI):

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:latest
    container_name: test_mongodb
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secretpassword
      MONGO_INITDB_DATABASE: sdet_test_db
    volumes:
      # Mount initialization seed script directory
      - ./mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro
      - mongo_data:/data/db

  mongo-express:
    image: mongo-express:latest
    container_name: test_mongo_express
    restart: always
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: secretpassword
      ME_CONFIG_MONGODB_URL: mongodb://admin:secretpassword@mongodb:27017/
    depends_on:
      - mongodb

volumes:
  mongo_data:
```

---

## 4. Automatic Database Seed Script (`mongo-init.js`)

When the MongoDB container starts for the first time, scripts inside `/docker-entrypoint-initdb.d/` automatically execute:

```javascript
// mongo-init.js
db = db.getSiblingDB('sdet_test_db');

// Create initial collections and seed test data
db.createCollection('users');
db.createCollection('products');

db.users.insertMany([
  { email: "seed_admin@test.com", role: "ADMIN", isActive: true },
  { email: "seed_user@test.com", role: "USER", isActive: true }
]);

console.log("Database initialized with seed data!");
```

---

## 5. Starting and Stopping Docker Compose

```bash
# Spin up test database container setup
docker-compose up -d

# Check status of running test services
docker-compose ps

# Tear down container environment and wipe test volumes
docker-compose down -v
```

---

## 6. Summary Checklist
- [x] Use `docker run` for fast, disposable local MongoDB instances.
- [x] Use `docker-compose.yml` to orchestrate MongoDB alongside web services and Mongo Express GUI.
- [x] Utilize `/docker-entrypoint-initdb.d/` for zero-touch test data seeding.
