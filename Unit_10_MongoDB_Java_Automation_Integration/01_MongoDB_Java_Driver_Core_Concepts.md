# Unit 10 — MongoDB Java Automation Integration
## Module 01: MongoDB Java Driver Core Concepts

**SDET Priority:** ⭐⭐⭐⭐⭐ (Core Automation Architecture Skill)

---

## 1. Overview & Build Dependencies

To interact with MongoDB from Java automation frameworks (TestNG, JUnit, Cucumber), add the official **MongoDB Java Synchronous Driver** dependency to your `pom.xml` or `build.gradle`.

### Maven Dependency (`pom.xml`)
```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>4.11.1</version>
</dependency>
```

---

## 2. Core Java Driver Components Breakdown

| Class / Interface | Package | Role in Framework |
| :--- | :--- | :--- |
| `MongoClient` | `com.mongodb.client` | Connection pool instance managing socket connections to MongoDB cluster. |
| `MongoClients` | `com.mongodb.client` | Factory class used to instantiate `MongoClient` instances. |
| `MongoDatabase` | `com.mongodb.client` | Handle to a specific database context. |
| `MongoCollection<Document>` | `com.mongodb.client` | Handle to a specific collection context for executing CRUD operations. |
| `Document` | `org.bson` | BSON document representation in Java (implements `Map<String, Object>`). |
| `Filters` | `com.mongodb.client.model` | Fluent static helper for building BSON query filters (`Filters.eq()`, `Filters.and()`). |
| `Updates` | `com.mongodb.client.model` | Fluent helper for update operations (`Updates.set()`, `Updates.inc()`). |

---

## 3. Singleton Thread-Safe MongoDB Manager Pattern

Creating a new `MongoClient` connection for every single query causes socket exhaustion and memory leaks. The `MongoClient` instance must be managed as a **Thread-Safe Singleton**.

```java
package com.automation.db;

import com.mongodb.ConnectionString;
import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import java.util.concurrent.TimeUnit;

public class MongoManager {

    private static MongoClient mongoClient;
    private static MongoDatabase database;
    private static final String CONNECTION_STRING = "mongodb://localhost:27017";
    private static final String DB_NAME = "sdet_test_db";

    private MongoManager() {} // Prevent direct instantiation

    public static synchronized MongoDatabase getDatabase() {
        if (mongoClient == null) {
            MongoClientSettings settings = MongoClientSettings.builder()
                    .applyConnectionString(new ConnectionString(CONNECTION_STRING))
                    .applyToConnectionPoolSettings(builder ->
                            builder.maxSize(20) // Connection pool limit for parallel tests
                                   .minSize(5)
                                   .maxWaitTime(5, TimeUnit.SECONDS))
                    .build();

            mongoClient = MongoClients.create(settings);
            database = mongoClient.getDatabase(DB_NAME);
            System.out.println("MongoDB Connection Pool initialized successfully.");
        }
        return database;
    }

    public static MongoCollection<Document> getCollection(String collectionName) {
        return getDatabase().getCollection(collectionName);
    }

    public static synchronized void close() {
        if (mongoClient != null) {
            mongoClient.close();
            mongoClient = null;
            System.out.println("MongoDB Connection Pool closed.");
        }
    }
}
```

---

## 4. Summary Checklist
- [x] Use `mongodb-driver-sync` for standard TestNG/Selenium frameworks.
- [x] Manage `MongoClient` as a thread-safe singleton connection pool.
- [x] Leverage `Filters`, `Updates`, `Sorts`, and `Projections` static helpers for clean code readability.
