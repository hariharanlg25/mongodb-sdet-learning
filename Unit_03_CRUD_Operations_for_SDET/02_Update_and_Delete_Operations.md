# Unit 03 — CRUD Operations for SDET
## Module 02: Update & Delete Operations

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential for State Mutation & Test Cleanup)

---

## 1. Overview & Core Methods

### Update Operations
Update operations allow SDETs to manipulate existing documents in a database to test edge cases (e.g. updating a user status to "SUSPENDED" before testing access denial on UI/API).

- `updateOne(filter, update, options)`: Updates a single matching document.
- `updateMany(filter, update, options)`: Updates all documents matching the filter.
- `replaceOne(filter, replacement, options)`: Completely replaces the matching document (except `_id`).

### Delete Operations
Delete operations clear test state during teardown.
- `deleteOne(filter)`: Deletes the first matching document.
- `deleteMany(filter)`: Deletes all matching documents.

---

## 2. Practical mongosh Examples

### Single Update (`updateOne`)
> 💡 Always use update operators like `$set`. Omitting `$set` in older MongoDB versions would overwrite the entire document!

```javascript
// Update user status to VERIFIED
db.users.updateOne(
  { email: "sdet.candidate@example.com" },
  { $set: { isEmailVerified: true, status: "ACTIVE" } }
);
```

---

### Batch Update (`updateMany`)
```javascript
// Mark all test users created today as EXPIRED during test suite execution
db.users.updateMany(
  { isTestUser: true },
  { $set: { status: "EXPIRED" } }
);
```

---

### Upsert Behavior (`{ upsert: true }`)
An **Upsert** is a hybrid operation: if a matching document is found, it updates it; if NO match exists, it inserts a new document automatically!

```javascript
// Ensure a default admin test account exists (create if missing, update if present)
db.users.updateOne(
  { email: "admin.test@example.com" },
  { $set: { role: "SUPER_ADMIN", isActive: true } },
  { upsert: true }
);
```

---

### Document Replacement (`replaceOne`)
Replaces the entire fields of a document while preserving its original `_id`.

```javascript
db.users.replaceOne(
  { _id: ObjectId("650c1f2e9b1d8f2a4c8b4567") },
  {
    name: "John Updated",
    email: "john.updated@example.com",
    role: "QA Manager"
  }
);
```

---

### Delete Operations
```javascript
// Delete a specific test user created during a single test case
db.users.deleteOne({ email: "temp.user@example.com" });

// Clean up all temporary automation data after test suite run
db.users.deleteMany({ email: /.*@testdomain\.com$/ });
```

---

## 3. SDET & QA Use Case

### Scenario: Simulating Backend State Changes
Many features (e.g. password resets, subscription expirations, lockouts) depend on backend events or background cron jobs. Rather than waiting hours for background jobs during test execution, SDETs force database state updates:

```javascript
// Simulate account lockout by forcing 5 failed login attempts in DB
db.users.updateOne(
  { username: "lockout_user" },
  { $set: { failedLoginAttempts: 5, accountLocked: true } }
);

// Now trigger UI test: Assert that login form displays "Account Locked" error!
```

---

## 4. Java Driver Implementation for Teardown

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;
import org.bson.Document;
import org.testng.annotations.AfterMethod;

public class BaseTestTeardown {

    @AfterMethod
    public void cleanupTestData() {
        MongoCollection<Document> usersCollection = MongoManager.getCollection("users");

        // Delete all dynamic test records created during test execution
        long deletedCount = usersCollection.deleteMany(Filters.regex("email", "@automationtest\\.com$")).getDeletedCount();
        System.out.println("Cleaned up " + deletedCount + " test users from MongoDB.");
    }
}
```

---

## 5. Summary Checklist
- [x] Use `updateOne()` with `$set` for targeted field mutations.
- [x] Leverage `{ upsert: true }` for resilient test setup.
- [x] Use `deleteMany()` in `@AfterMethod` or `@AfterClass` to maintain a clean database state.
