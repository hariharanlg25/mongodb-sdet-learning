# Unit 03 — CRUD Operations for SDET
## Module 01: Create & Read Operations

**SDET Priority:** ⭐⭐⭐⭐⭐ (Core Interview & Automation Skill)

---

## 1. Overview & Definitions

CRUD (Create, Read, Update, Delete) forms the foundation of all database testing. For an SDET:
- **Create** = Seed test data or trigger application workflows that store data.
- **Read** = Query MongoDB to verify that application state, API endpoints, or UI inputs accurately updated the database.

---

## 2. Read Operations Deep Dive

### Core Read Methods
1. `find(query, projection)`: Returns a cursor pointing to all matching documents.
2. `findOne(query, projection)`: Returns the **first single document** matching the query criteria (or `null` if not found).
3. `countDocuments(query)`: Returns the integer count of documents matching the filter.

---

## 3. Practical mongosh Examples

### Query All Documents
```javascript
// Retrieve all documents in users collection
db.users.find()

// Formatted JSON output (automatic in modern mongosh)
db.users.find().pretty()
```

---

### Exact Match Queries
```javascript
// Find user by email (API test validation)
db.users.findOne({ email: "sdet.candidate@example.com" })

// Find users in QA department
db.users.find({ department: "QA" })
```

---

### Nested Field Queries (Dot Notation)
MongoDB allows direct querying into embedded objects using **dot notation**.
> ⚠️ **Requirement:** Field paths in dot notation MUST be enclosed in quotes (e.g. `"address.city"`).

```javascript
// Find all users living in Bengaluru
db.users.find({
  "address.city": "Bengaluru"
})
```

---

### Array Queries
```javascript
// Find users possessing "MongoDB" skill in their skills array
db.users.find({
  skills: "MongoDB"
})
```

---

### Counting Documents
```javascript
// Count total active users for pagination assertions
db.users.countDocuments({ status: "ACTIVE" })
```

---

## 4. SDET & QA Use Case: API Validation Workflow

```text
[ API Automation Test ]
         │
         ├──► 1. Trigger POST /api/v1/users (Payload: name="John", role="SDET")
         │
         ├──► 2. Receive HTTP 201 Created (Response contains userId: "650c1f...")
         │
         └──► 3. Query DB: db.users.findOne({ _id: ObjectId("650c1f...") })
                   │
                   └──► ASSERT: dbUser != null
                        ASSERT: dbUser.name == "John"
                        ASSERT: dbUser.status == "PENDING_ACTIVATION"
```

---

## 5. Java Driver Implementation for Read Validation

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.testng.Assert;
import org.testng.annotations.Test;

public class UserRegistrationTest {

    @Test
    public void testUserCreationInDatabase() {
        MongoCollection<Document> usersCollection = MongoManager.getCollection("users");

        // Execute Read Query via Filters helper
        Document userDoc = usersCollection.find(Filters.eq("email", "sdet.candidate@example.com")).first();

        // Perform Hard Assertions
        Assert.assertNotNull(userDoc, "User document should exist in MongoDB!");
        Assert.assertEquals(userDoc.getString("role"), "SDET", "User role mismatch!");
        Assert.assertTrue(userDoc.getBoolean("isActive"), "User should be active upon registration!");
    }
}
```

---

## 6. Advantages & Disadvantages

### Advantages
- **Dot Notation Power:** Allows direct assertion of deeply nested object structures without flattening response objects.
- **Fast Execution:** `findOne()` stops scanning as soon as the first match is located, optimizing test runtime.

### Disadvantages
- **Unindexed Scans:** Executing `find()` on unindexed fields across massive test databases leads to slow test suite runs (Collection Scan / `COLLSCAN`).

---

## 7. Summary Checklist
- [x] Master `find()` vs `findOne()`.
- [x] Use dot notation `"parent.child"` with quotes for embedded document queries.
- [x] Integrate `countDocuments()` for array or table row validation.
