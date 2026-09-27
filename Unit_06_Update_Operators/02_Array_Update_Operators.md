# Unit 06 — Update Operators
## Module 02: Array Update Operators

**SDET Priority:** ⭐⭐⭐⭐ (Essential for Cart Items, Roles & Tag Manipulation)

---

## 1. Overview & Operator Breakdown

Array update operators allow SDETs to append, remove, or modify elements inside document array fields without replacing the array.

---

## 2. Core Array Update Operators

| Operator | Description | mongosh Example |
| :--- | :--- | :--- |
| `$push` | Appends a value/object to an array | `{ $push: { roles: "ADMIN" } }` |
| `$addToSet` | Adds a value to an array **ONLY if it does NOT already exist** (prevents duplicates) | `{ $addToSet: { skills: "Java" } }` |
| `$pull` | Removes all array elements that match a specified condition | `{ $pull: { skills: "Legacy_Tool" } }` |
| `$pop` | Removes the first (`-1`) or last (`1`) item of an array | `{ $pop: { auditLogs: -1 } }` |
| `$pullAll` | Removes all instances of specified values from an array | `{ $pullAll: { tags: ["v1", "deprecated"] } }` |

---

## 3. Practical mongosh Examples

### `$push` vs. `$addToSet`

```javascript
// $push adds "MongoDB" even if "MongoDB" already exists in the skills array (duplicates allowed)
db.users.updateOne(
  { email: "rahul@example.com" },
  { $push: { skills: "MongoDB" } }
);

// $addToSet adds "MongoDB" ONLY if it is not already present (guarantees uniqueness)
db.users.updateOne(
  { email: "rahul@example.com" },
  { $addToSet: { skills: "MongoDB" } }
);
```

---

### Modifying Nested Objects in Arrays (Positional Operator `$`)

The positional operator `$` identifies an array element to update without needing to know its exact index position!

```javascript
// Update quantity to 5 for item with sku "MOUSE-01" inside the items array
db.orders.updateOne(
  { orderId: "ORD-99", "items.sku": "MOUSE-01" },
  { $set: { "items.$.qty": 5 } }
);
```

---

### Modifying All Elements in an Array (All Positional Operator `$[]`)

```javascript
// Apply a 10% discount across ALL items in an order array
db.orders.updateOne(
  { orderId: "ORD-99" },
  { $mul: { "items.$[].price": 0.90 } }
);
```

---

## 4. SDET & QA Use Case: Shopping Cart & Role-Based Access Control (RBAC) Testing

### Scenario 1: RBAC Testing
Grant or revoke specific user privileges dynamically before running API authorization tests (`403 Forbidden` vs `200 OK` assertions).

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;
import org.bson.Document;
import org.testng.annotations.Test;

public class RBACTest {

    @Test
    public void testAdminRoleAccess() {
        MongoCollection<Document> usersCol = MongoManager.getCollection("users");

        // Seed ADMIN role into user's roles array using Updates.addToSet
        usersCol.updateOne(
            Filters.eq("username", "test_user_rbac"),
            Updates.addToSet("roles", "ROLE_ADMIN")
        );

        // Execute API Test GET /api/admin/reports -> Assert HTTP 200 OK

        // Clean up role using Updates.pull
        usersCol.updateOne(
            Filters.eq("username", "test_user_rbac"),
            Updates.pull("roles", "ROLE_ADMIN")
        );
    }
}
```

---

## 5. Summary Checklist
- [x] Prefer `$addToSet` over `$push` when seeding unique array items.
- [x] Use `$pull` to remove items during teardown.
- [x] Use the positional `$` operator to update specific matching array elements.
