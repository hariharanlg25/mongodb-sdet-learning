# Unit 09 — SDET Validation & Test Data Management
## Module 02: Test Data Management (TDM) Setup & Teardown

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential for Resilient & Parallel Automated Testing)

---

## 1. Core Principles of Test Data Management (TDM)

Test Data Management in MongoDB centers on three core pillars:
1. **Deterministic State:** Every test case must start from a known, predictable database baseline.
2. **Data Isolation:** Parallel automated test runs must not overwrite or conflict with each other's test documents.
3. **Automated Teardown:** Test artifacts must be cleaned up after test execution to prevent database bloat and flaky test cascading.

---

## 2. Dynamic vs. Static Data Seeding Strategies

| Strategy | Description | Best Used For |
| :--- | :--- | :--- |
| **Static Data Fixtures** | Pre-populated JSON/BSON files loaded into MongoDB once before test suite initialization. | Reference data (e.g., Country codes, Subscription tiers, Tax rates). |
| **Dynamic On-The-Fly Seeding** | Generated in `@BeforeMethod` using UUIDs or Java Faker libraries. | User registration, order checkouts, dynamic cart items. |

---

## 3. Guarantees for Parallel Test Execution Safety

When running 10 parallel browser threads in TestNG or Selenium Grid:
- ❌ **BAD:** Hardcoding test emails like `"testuser@example.com"`. Thread 2 will throw duplicate key errors!
- ✅ **GOOD:** Appending dynamic execution tokens or UUIDs:
  `String uniqueEmail = "user_" + UUID.randomUUID().toString().substring(0, 8) + "@testdomain.com";`

---

## 4. TestNG Lifecycle Hook Pattern with MongoDB Setup & Teardown

```java
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.testng.annotations.*;

import java.util.UUID;

public class BaseMongoDBTest {

    protected static MongoClient mongoClient;
    protected static MongoDatabase database;
    protected MongoCollection<Document> usersCollection;
    protected String currentTestUserEmail;

    @BeforeSuite
    public void globalDatabaseSetup() {
        // Initialize single MongoConnection pool for entire test suite
        mongoClient = MongoClients.create("mongodb://localhost:27017");
        database = mongoClient.getDatabase("sdet_test_db");
        System.out.println("Connected to MongoDB successfully!");
    }

    @BeforeMethod
    public void seedTestCaseData() {
        usersCollection = database.getCollection("users");

        // Seed unique isolated user record before EACH test method
        currentTestUserEmail = "test_" + UUID.randomUUID().toString() + "@automation.com";
        Document testUser = new Document("email", currentTestUserEmail)
                .append("name", "Test User")
                .append("status", "ACTIVE")
                .append("isAutomationData", true);

        usersCollection.insertOne(testUser);
    }

    @AfterMethod
    public void teardownTestCaseData() {
        // Delete only the isolated document created for THIS specific test method
        if (currentTestUserEmail != null) {
            usersCollection.deleteOne(Filters.eq("email", currentTestUserEmail));
        }
    }

    @AfterSuite
    public void globalTeardown() {
        // Clean up remaining automation artifacts and close connection pool
        usersCollection.deleteMany(Filters.eq("isAutomationData", true));
        if (mongoClient != null) {
            mongoClient.close();
        }
        System.out.println("MongoDB connection closed cleanly.");
    }
}
```

---

## 5. Summary Checklist
- [x] Always isolate test data using UUIDs or timestamp suffixes.
- [x] Tag test documents with a flag like `{ isAutomationData: true }` for safe batch teardown.
- [x] Close MongoClient connection pools in `@AfterSuite` to avoid connection leaks.
