# Unit 10 — MongoDB Java Automation Integration
## Module 02: TestNG & MongoDB Framework Integration

**SDET Priority:** ⭐⭐⭐⭐⭐ (Framework Architecture & Utilities)

---

## 1. Enterprise Framework Package Architecture

A production-ready TestNG automation framework encapsulates MongoDB operations within a dedicated `database` layer:

```text
src/test/java/
├── com/automation/
│   ├── api/                  <--- REST Assured endpoints & specs
│   ├── pages/                <--- Page Object Model (POM) classes
│   ├── database/             <--- MongoDB Layer
│   │   ├── MongoManager.java      (Connection Pool Singleton)
│   │   ├── MongoUtils.java        (Generic CRUD helper methods)
│   │   └── MongoQueries.java      (Domain-specific query constants)
│   ├── testdata/             <--- Data Providers & JSON Fixtures
│   └── tests/                <--- TestNG Test Classes
│       ├── LoginTest.java
│       └── UserRegistrationTest.java
```

---

## 2. Generic `MongoUtils.java` Utility Class

This utility simplifies database operations across all test classes:

```java
package com.automation.database;

import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.bson.conversions.Bson;

import java.util.ArrayList;
import java.util.List;

public class MongoUtils {

    public static Document getDocument(String collectionName, Bson filter) {
        return MongoManager.getCollection(collectionName).find(filter).first();
    }

    public static List<Document> getDocuments(String collectionName, Bson filter) {
        return MongoManager.getCollection(collectionName).find(filter).into(new ArrayList<>());
    }

    public static void insertDocument(String collectionName, Document document) {
        MongoManager.getCollection(collectionName).insertOne(document);
    }

    public static long updateDocument(String collectionName, Bson filter, Bson update) {
        return MongoManager.getCollection(collectionName).updateOne(filter, update).getModifiedCount();
    }

    public static long deleteDocument(String collectionName, Bson filter) {
        return MongoManager.getCollection(collectionName).deleteOne(filter).getDeletedCount();
    }

    public static long deleteManyDocuments(String collectionName, Bson filter) {
        return MongoManager.getCollection(collectionName).deleteMany(filter).getDeletedCount();
    }

    public static long countDocuments(String collectionName, Bson filter) {
        return MongoManager.getCollection(collectionName).countDocuments(filter);
    }
}
```

---

## 3. Feeding TestNG `@DataProvider` Directly from MongoDB

Rather than reading static Excel or JSON files, SDETs query MongoDB directly to supply dynamic test inputs to TestNG test methods:

```java
package com.automation.testdata;

import com.automation.database.MongoUtils;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.testng.annotations.DataProvider;

import java.util.List;

public class TestDataProvider {

    @DataProvider(name = "activeTestUsers")
    public static Object[][] getActiveTestUsersFromDB() {
        // Query active test users from MongoDB
        List<Document> userDocs = MongoUtils.getDocuments("users", Filters.eq("status", "ACTIVE"));

        Object[][] data = new Object[userDocs.size()][2];
        for (int i = 0; i < userDocs.size(); i++) {
            data[i][0] = userDocs.get(i).getString("email");
            data[i][1] = userDocs.get(i).getString("role");
        }
        return data;
    }
}
```

---

## 4. TestNG Test Class Consuming DataProvider

```java
package com.automation.tests;

import com.automation.testdata.TestDataProvider;
import org.testng.Assert;
import org.testng.annotations.Test;

public class UserLoginTest {

    @Test(dataProvider = "activeTestUsers", dataProviderClass = TestDataProvider.class)
    public void testUserLoginWithDBData(String email, String role) {
        System.out.println("Executing Login test for DB User: " + email + " with role: " + role);
        Assert.assertNotNull(email, "Email should not be null!");
    }
}
```

---

## 5. Summary Checklist
- [x] Encapsulate MongoDB operations inside reusable `MongoUtils` helper methods.
- [x] Use TestNG `@DataProvider` to seed test methods directly from MongoDB collections.
- [x] Maintain clean layer separation between tests, page objects, and database utilities.
