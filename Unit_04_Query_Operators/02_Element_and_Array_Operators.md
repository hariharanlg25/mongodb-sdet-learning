# Unit 04 — Query Operators
## Module 02: Element & Array Operators

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential for Schema Validation & Array Verification)

---

## 1. Element Operators Overview

Element operators check for the presence of specific fields or verify their field data types within documents.

| Operator | Description | mongosh Example |
| :--- | :--- | :--- |
| `$exists` | Matches documents that have (or lack) the specified field | `{ phoneNumber: { $exists: true } }` |
| `$type` | Selects documents if a field is of the specified BSON data type | `{ age: { $type: "int" } }` |

---

## 2. SDET Use Case for Element Operators: Data Quality & Schema Drift Testing

Because MongoDB has no forced schema by default, backend bugs can result in:
1. Optional fields missing when they should be present.
2. Field data type mutations (e.g. `age` stored as string `"28"` instead of integer `28`).

### Detecting Schema Drift in mongosh
```javascript
// Find records where optional field 'middleName' exists
db.users.find({ middleName: { $exists: true } });

// Catch corrupted records where 'totalAmount' was accidentally saved as a String instead of Double/Decimal
db.orders.find({
  totalAmount: { $type: "string" }
});
```

---

## 3. Array Operators Overview

MongoDB documents frequently store arrays (e.g., product tags, user permissions, order line items). Querying arrays accurately is a major SDET competency.

| Operator | Description | mongosh Example |
| :--- | :--- | :--- |
| `$all` | Matches arrays that contain **ALL** specified elements regardless of order | `{ skills: { $all: ["Java", "Selenium"] } }` |
| `$elemMatch` | Matches documents if **at least one element** in an array satisfies **ALL** specified criteria | `{ items: { $elemMatch: { qty: { $gt: 2 }, price: { $lt: 50 } } } }` |
| `$size` | Matches arrays with the exact number of elements specified | `{ skills: { $size: 3 } }` |

---

## 4. Deep Dive: `$elemMatch` vs. Standard Array Queries

### ⚠️ Common Pitfall in Array Automation Testing
Suppose you have an `orders` collection with documents containing an `items` array:

```json
{
  "orderId": "ORD-101",
  "items": [
    { "product": "Mouse", "qty": 1, "price": 50 },
    { "product": "Keyboard", "qty": 5, "price": 200 }
  ]
}
```

#### INCORRECT QUERY (Without `$elemMatch`):
```javascript
db.orders.find({
  "items.qty": { $gt: 2 },
  "items.price": { $lt: 100 }
});
```
> ❌ **Why it fails logic:** This query returns true because `qty: 5` matches `items.qty > 2` from the Keyboard item, AND `price: 50` matches `items.price < 100` from the Mouse item. The criteria matched across **different items in the array**!

#### CORRECT QUERY (With `$elemMatch`):
```javascript
db.orders.find({
  items: {
    $elemMatch: { qty: { $gt: 2 }, price: { $lt: 100 } }
  }
});
```
> ✅ **Why it works:** `$elemMatch` guarantees that both criteria (`qty > 2` AND `price < 100`) MUST be satisfied by **the exact same array element**!

---

## 5. Java Automation Code Example

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.testng.Assert;
import org.testng.annotations.Test;

public class ArrayValidationTest {

    @Test
    public void verifyUserSkillsArray() {
        MongoCollection<Document> usersCol = MongoManager.getCollection("users");

        // Validate user possesses required automation skills using Filters.all
        Document sdetUser = usersCol.find(Filters.all("skills", "Java", "Selenium", "MongoDB")).first();
        Assert.assertNotNull(sdetUser, "User should possess Java, Selenium, and MongoDB skills!");

        // Validate array length equal to 3
        Document exactSkillsUser = usersCol.find(Filters.size("skills", 3)).first();
        Assert.assertNotNull(exactSkillsUser, "User skills array length should equal 3!");
    }
}
```

---

## 6. Summary Checklist
- [x] Use `$exists: true/false` to assert optional payload attributes.
- [x] Use `$type` to detect corrupted field data types.
- [x] Always use `$elemMatch` when validating multiple conditions on an array of embedded objects.
