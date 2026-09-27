# Unit 02 — Database and Collection Operations
## Module 02: Document Insert Operations

**SDET Priority:** ⭐⭐⭐⭐⭐ (Critical for Test Data Seeding)

---

## 1. Definition & Core Methods

Data insertion is the primary way SDETs seed databases with deterministic test data before executing API or UI test cases.

### Core MongoDB Insert Methods
1. `insertOne(document, options)`: Inserts a single document into a collection.
2. `insertMany([documents], options)`: Inserts an array of multiple documents in a single network round-trip.

---

## 2. Practical Examples (mongosh)

### Single Insert (`insertOne`)
```javascript
db.employees.insertOne({
  name: "Rahul Sharma",
  empId: "EMP-1001",
  role: "SDET",
  department: "QA Automation",
  skills: ["Java", "Selenium", "MongoDB"],
  isActive: true,
  createdAt: new Date()
});
```
**Response:**
```json
{
  "acknowledged": true,
  "insertedId": ObjectId("650c1f2e9b1d8f2a4c8b9999")
}
```

---

### Batch Insert (`insertMany`)
Inserting multiple test fixtures at once is significantly faster than executing repeated single inserts in a loop.

```javascript
db.employees.insertMany([
  {
    name: "Priya Nair",
    empId: "EMP-1002",
    role: "QA Lead",
    department: "QA Automation",
    skills: ["Python", "Playwright", "MongoDB"],
    isActive: true
  },
  {
    name: "Amit Kumar",
    empId: "EMP-1003",
    role: "Backend Dev",
    department: "Engineering",
    skills: ["Node.js", "Express", "MongoDB"],
    isActive: false
  }
]);
```

---

## 3. Advanced Concept for SDETs: Ordered vs. Unordered Inserts

When using `insertMany()`, MongoDB accepts an options object: `{ ordered: true | false }`.

### `ordered: true` (Default Behavior)
- MongoDB inserts documents sequentially in array order.
- If document #2 encounters an error (e.g. duplicate key violation on unique `empId`), **the process halts immediately**. Documents #3, #4, etc., are **NOT** inserted.

### `ordered: false` (Recommended for Bulk Data Seeding)
- MongoDB attempts to insert all documents in parallel.
- If document #2 fails, MongoDB ignores the error and continues inserting documents #3, #4, etc.

```javascript
// Bulk seed test data where individual duplicate failures should not break the rest of the batch
db.employees.insertMany(
  [
    { empId: "EMP-1001", name: "User A" }, // duplicate key error if already exists
    { empId: "EMP-1004", name: "User B" }, // succeeds
    { empId: "EMP-1005", name: "User C" }  // succeeds
  ],
  { ordered: false }
);
```

---

## 4. SDET Use Case: Pre-populating Fixtures in TestNG

In automated test suites, SDETs write data seeding utilities in Java using `insertMany()` before running suite tests:

```java
import com.mongodb.client.MongoCollection;
import org.bson.Document;
import java.util.Arrays;
import java.util.List;

public class TestDataSeeder {

    public static void seedTestUsers(MongoCollection<Document> collection) {
        Document user1 = new Document("username", "test_user_01")
                            .append("email", "user01@test.com")
                            .append("status", "ACTIVE");

        Document user2 = new Document("username", "test_user_02")
                            .append("email", "user02@test.com")
                            .append("status", "PENDING_VERIFICATION");

        collection.insertMany(Arrays.asList(user1, user2));
        System.out.println("Test users seeded successfully!");
    }
}
```

---

## 5. Advantages & Disadvantages

### Advantages
- **Fast Seeding:** `insertMany` reduces network latency during suite initialization (`@BeforeSuite`).
- **Dynamic Field Flexibility:** Test fixtures can include dynamic fields (like execution build numbers or timestamp tokens) without changing collection definitions.

### Disadvantages
- **Duplicate Insertion:** Re-running insert scripts without clearing data first can cause duplicate records if no unique indexes exist.

---

## 6. Summary Checklist
- [x] Use `insertOne()` for single document seeding.
- [x] Use `insertMany()` for bulk test fixture setup.
- [x] Understand `{ ordered: false }` for resilient bulk data insertion.
