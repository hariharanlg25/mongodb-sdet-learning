# Unit 07 — Aggregation Framework
## Module 02: MongoDB $lookup (Cross-Collection Joins)

**SDET Priority:** ⭐⭐⭐⭐ (Essential for Multi-Collection Data Validation)

---

## 1. Overview & Definition

### What is `$lookup`?
The `$lookup` stage performs a **Left Outer Join** to an un-sharded collection in the same database to combine fields from joined documents into an array inside the output document.

### Parameters Explained
- `from`: Target collection in the same database to join.
- `localField`: Field from the input documents entering the `$lookup` stage.
- `foreignField`: Field from the documents in the `from` collection.
- `as`: Name of the new array field added to input documents.

```text
 [ Input Collection: orders ]               [ Target Collection: users ]
  {                                          {
    "_id": ObjectId("..."),                    "_id": ObjectId("650c1f..."),
    "orderId": "ORD-101",                      "name": "Rahul Sharma",
    "userId": ObjectId("650c1f...")  ───JOIN──► "email": "rahul@test.com"
  }                                          }
```

---

## 2. Practical mongosh Example

### Joining `orders` with `users`
```javascript
db.orders.aggregate([
  {
    $match: { status: "COMPLETED" }
  },
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "userDetails"
    }
  },
  // Flatten userDetails array using $unwind (since 1 order has 1 user)
  {
    $unwind: "$userDetails"
  },
  {
    $project: {
      orderId: 1,
      totalAmount: 1,
      customerName: "$userDetails.name",
      customerEmail: "$userDetails.email"
    }
  }
]);
```

---

## 3. SDET & QA Use Case: Referential Integrity Validation

In microservice architectures where data is spread across collections (e.g. `orders` storing `userId` reference):
- **Orphan Record Bugs:** If a user account is deleted without cascade deletion, an order will contain a `userId` that points to a non-existent user.
- **SDET Test:** An SDET writes an aggregation check with `$lookup` to locate orphan orders where `userDetails` array is empty `[]`:

```javascript
// Find orphan orders where foreign key userId references non-existent user
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "userDetails"
    }
  },
  {
    $match: { userDetails: { $size: 0 } }
  }
]);
// ASSERT in automation test: Result set size MUST equal 0!
```

---

## 4. Java Automation Code Example

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Aggregates;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.bson.conversions.Bson;
import org.testng.Assert;
import org.testng.annotations.Test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class CrossCollectionJoinTest {

    @Test
    public void testNoOrphanOrdersExist() {
        MongoCollection<Document> ordersCol = MongoManager.getCollection("orders");

        List<Bson> pipeline = Arrays.asList(
            Aggregates.lookup("users", "userId", "_id", "userDetails"),
            Aggregates.match(Filters.size("userDetails", 0))
        );

        List<Document> orphanOrders = ordersCol.aggregate(pipeline).into(new ArrayList<>());

        Assert.assertEquals(orphanOrders.size(), 0, "Found orphan orders referencing missing user ObjectIds!");
    }
}
```

---

## 5. Summary Checklist
- [x] Know the 4 `$lookup` parameters: `from`, `localField`, `foreignField`, `as`.
- [x] Combine `$lookup` with `$unwind` for one-to-one joined record flattening.
- [x] Use `$lookup` with `$size: 0` to detect orphan foreign key references during regression testing.
