# Unit 04 — Query Operators
## Module 01: Comparison & Logical Operators

**SDET Priority:** ⭐⭐⭐⭐⭐ (Crucial for Boundary & Filter Validation)

---

## 1. Overview & Operator Breakdown

Query operators enable SDETs to build refined queries to locate target test records, validate data ranges, and verify application filters.

---

## 2. Comparison Operators

| Operator | Name | Description | mongosh Example |
| :--- | :--- | :--- | :--- |
| `$eq` | Equal to | Matches values equal to specified value | `{ age: { $eq: 25 } }` |
| `$ne` | Not equal to | Matches values NOT equal to specified value | `{ status: { $ne: "DELETED" } }` |
| `$gt` | Greater than | Matches values greater than specified value | `{ price: { $gt: 100 } }` |
| `$gte` | Greater than or equal | Matches values greater than or equal to value | `{ age: { $gte: 18 } }` |
| `$lt` | Less than | Matches values less than specified value | `{ stock: { $lt: 5 } }` |
| `$lte` | Less than or equal | Matches values less than or equal to value | `{ score: { $lte: 50 } }` |
| `$in` | In array | Matches any of the values specified in an array | `{ role: { $in: ["ADMIN", "QA"] } }` |
| `$nin` | Not in array | Matches none of the values specified in an array | `{ status: { $nin: ["INACTIVE", "BLOCKED"] } }` |

### Practical Comparison Operator Examples (mongosh)
```javascript
// Find products with price between $50 and $200 (Boundary Value Testing)
db.products.find({
  price: { $gte: 50, $lte: 200 }
});

// Find users whose department is either QA, DevOps, or Engineering
db.employees.find({
  department: { $in: ["QA", "DevOps", "Engineering"] }
});
```

---

## 3. Logical Operators

Logical operators join multiple query conditions.

| Operator | Description | mongosh Example |
| :--- | :--- | :--- |
| `$and` | Joins query clauses with a logical AND (all must match) | `{ $and: [ { age: { $gt: 20 } }, { status: "ACTIVE" } ] }` |
| `$or` | Joins query clauses with a logical OR (at least one must match) | `{ $or: [ { department: "QA" }, { age: { $gt: 30 } } ] }` |
| `$not` | Inverts the effect of a query expression | `{ age: { $not: { $gt: 30 } } }` |
| `$nor` | Fails if ANY clause matches (neither condition true) | `{ $nor: [ { status: "SUSPENDED" }, { isDeleted: true } ] }` |

### Practical Logical Operator Examples (mongosh)
```javascript
// Find users who are either QA engineers OR have age > 30 AND active status
db.users.find({
  $or: [
    { role: "QA" },
    { $and: [ { age: { $gt: 30 } }, { isActive: true } ] }
  ]
});
```

---

## 4. SDET & QA Use Case: API Filter Assertions

When testing API search/filter endpoints (e.g., `GET /api/v1/orders?minAmount=100&status=COMPLETED`), the SDET validates that the returned API array strictly matches the database query result using `$gte` and `$eq`:

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.bson.conversions.Bson;
import org.testng.Assert;
import org.testng.annotations.Test;
import java.util.ArrayList;
import java.util.List;

public class OrderFilterTest {

    @Test
    public void verifyOrderFilterAPIMatchesDB() {
        MongoCollection<Document> ordersCol = MongoManager.getCollection("orders");

        // Construct MongoDB Filter using Java Filters helper
        Bson queryFilter = Filters.and(
            Filters.gte("totalAmount", 100.0),
            Filters.eq("status", "COMPLETED")
        );

        List<Document> dbOrders = ordersCol.find(queryFilter).into(new ArrayList<>());

        // Perform Assertion: Verify DB count matches API response array size
        int apiReturnedCount = 15; // Extracted from REST Assured response
        Assert.assertEquals(dbOrders.size(), apiReturnedCount, "DB query count should equal API result count!");
    }
}
```

---

## 5. Summary Checklist
- [x] Use `$in` and `$nin` for testing dropdown/multi-select filter endpoints.
- [x] Use `$gte` and `$lte` for boundary value analysis (BVA) on prices, dates, or ages.
- [x] Combine conditions cleanly using `$and` and `$or`.
