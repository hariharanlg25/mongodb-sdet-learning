# Unit 08 — Data Modeling, Indexes & Performance
## Module 02: Indexes & Query Performance with explain("executionStats")

**SDET Priority:** ⭐⭐⭐ (Crucial for Performance & Non-Functional Testing)

---

## 1. Overview & Definition of Indexes

### What is an Index?
An **Index** is a special data structure (typically a B-Tree) that stores a small portion of the collection's data set in an easy-to-traverse format. Without indexes, MongoDB must perform a **Collection Scan (`COLLSCAN`)**, scanning every document in a collection to locate matching queries.

---

## 2. Types of Indexes

1. **Single-Field Index:** Index created on a single field.
2. **Compound Index:** Index created on multiple fields (e.g. `{ department: 1, age: -1 }`).
3. **Unique Index:** Guarantees no duplicate values exist for indexed field (e.g. `{ email: 1 }, { unique: true }`).
4. **Multikey Index:** Index automatically created when indexing an array field.
5. **Text Index:** Index built for full-text search capability.

### Index Management Commands (mongosh)
```javascript
// Create single field index on email
db.users.createIndex({ email: 1 });

// Create unique compound index
db.users.createIndex({ tenantId: 1, empId: 1 }, { unique: true });

// View all indexes on collection
db.users.getIndexes();

// Drop index by name
db.users.dropIndex("email_1");
```

---

## 3. Deep Dive: Query Execution Stats using `explain("executionStats")`

When automated API tests run slowly, SDETs use `.explain("executionStats")` to diagnose database bottleneck issues.

```javascript
db.orders.find({ customerId: "CUST-9999", status: "COMPLETED" })
  .explain("executionStats");
```

### Critical Output Metrics for SDETs

```json
{
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 5,
    "executionTimeMillis": 450,
    "totalKeysExamined": 0,
    "totalDocsExamined": 500000,
    "executionStages": {
      "stage": "COLLSCAN"
    }
  }
}
```

---

## 4. Key Performance Metric Benchmarks for SDETs

| Metric | Ideal Benchmark Target | Red Flag Signal (Performance Bug) |
| :--- | :--- | :--- |
| **Stage** | `IXSCAN` (Index Scan) | `COLLSCAN` (Full Collection Scan) |
| **executionTimeMillis** | `< 10 ms` for single point queries | `> 100 ms` |
| **totalDocsExamined vs. nReturned** | `totalDocsExamined ≈ nReturned` (e.g., 5 examined, 5 returned) | `totalDocsExamined >> nReturned` (e.g., 500,000 examined to return 5!) |

---

## 5. SDET Automation Performance Assertion in Java

SDETs can automate query execution checks in performance/integration test suites:

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.testng.Assert;
import org.testng.annotations.Test;

public class IndexPerformanceTest {

    @Test
    public void verifyUserQueryUsesIndexScan() {
        MongoCollection<Document> usersCol = MongoManager.getCollection("users");

        // Execute explain query in Java Driver
        Document explainResult = usersCol.find(Filters.eq("email", "sdet@test.com"))
                .explain("executionStats");

        Document executionStats = (Document) explainResult.get("executionStats");
        Document executionStages = (Document) executionStats.get("executionStages");

        String stageName = executionStages.getString("stage");
        int executionTimeMillis = executionStats.getInteger("executionTimeMillis");

        // Performance Assertions
        Assert.assertNotEquals(stageName, "COLLSCAN", "Query triggered a full COLLSCAN! Missing index on email!");
        Assert.assertTrue(executionTimeMillis < 50, "Query took longer than 50ms! Actual: " + executionTimeMillis + "ms");
    }
}
```

---

## 6. Summary Checklist
- [x] Know how to create and view indexes using `createIndex()` and `getIndexes()`.
- [x] Use `.explain("executionStats")` to diagnose slow automation queries.
- [x] Assert that queries use `IXSCAN` stage instead of `COLLSCAN`.
