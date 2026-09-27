# Unit 01 — MongoDB Fundamentals
## Module 03: BSON vs JSON, Data Types & ObjectId

**SDET Priority:** ⭐⭐⭐⭐⭐ (Critical for Test Assertions & Field Types)

---

## 1. BSON vs. JSON Breakdown

### Definitions
- **JSON (JavaScript Object Notation):** A human-readable text format used widely for API payloads and configuration files. JSON supports basic data types: String, Number, Boolean, Array, Object, and Null.
- **BSON (Binary JSON):** A binary-encoded serialization format used internally by MongoDB to store documents and transport network protocols. BSON expands JSON by providing fast parsing, efficient indexing, and richer data types.

### Comparison Table

| Feature | JSON | BSON |
| :--- | :--- | :--- |
| **Format** | Text-based | Binary-encoded |
| **Human Readable** | Yes | No (requires client drivers/Compass to view) |
| **Data Types** | Limited (String, Number, Bool, Array, Object, Null) | Rich (Date, Decimal128, BinData, ObjectId, Int32, Int64, etc.) |
| **Storage Efficiency** | Higher overhead for dates/numbers | Fast traversal, indexed length prefixes |
| **API Integration** | Native web REST format | Used internally by MongoDB Drivers |

---

## 2. Core MongoDB BSON Data Types

Understanding data types is vital when verifying API responses against MongoDB.

| Data Type | Description | Example (mongosh) |
| :--- | :--- | :--- |
| **String** | UTF-8 encoded text | `"name": "Quality Assurance"` |
| **Int32** | 32-bit signed integer | `NumberInt(42)` |
| **Int64 / Long** | 64-bit signed integer | `NumberLong("9223372036854775807")` |
| **Double** | 64-bit floating point number | `19.99` |
| **Decimal128** | Exact 128-bit decimal (currency calculations) | `NumberDecimal("199.99")` |
| **Boolean** | `true` or `false` | `"isActive": true` |
| **Date** | 64-bit integer UTC timestamp | `ISODate("2026-09-27T10:00:00Z")` |
| **ObjectId** | 12-byte unique identifier | `ObjectId("650c1f2e9b1d8f2a4c8b4567")` |
| **Array** | Ordered set of values | `"skills": ["Java", "Selenium", "MongoDB"]` |
| **Object** | Embedded document | `"address": { "city": "Bengaluru" }` |
| **Null** | Represents null value or missing field | `"deletedAt": null` |

---

## 3. Deep Dive: `_id` and `ObjectId`

### What is `_id`?
Every document in MongoDB MUST have a unique `_id` field that acts as the primary key. If you omit `_id` when inserting a document, MongoDB automatically generates a 12-byte `ObjectId`.

### Structure of a 12-byte `ObjectId`
An `ObjectId` hex string (24 hex characters) is composed of 3 distinct parts:

```text
  4 Bytes (Timestamp)    +   5 Bytes (Random Value)   +   3 Bytes (Counter)
┌───────────────────────┬────────────────────────────┬───────────────────────┐
│ Seconds since epoch   │ Unique per machine/process │ Incrementing counter  │
└───────────────────────┴────────────────────────────┴───────────────────────┘
  e.g., 65 0c 1f 2e           9b 1d 8f 2a 4c              8b 45 67
```

### 💡 Pro-Tip for SDETs: Extracting Creation Time from `ObjectId`
Because the first 4 bytes of an `ObjectId` store the unix timestamp, **you can determine the exact creation time of a record directly from its `_id`** without needing a separate `createdAt` field!

#### Extract Timestamp in mongosh:
```javascript
// Sample ObjectId
let id = ObjectId("650c1f2e9b1d8f2a4c8b4567");

// Get creation timestamp
console.log(id.getTimestamp());
// Output: 2023-09-21T10:30:38.000Z
```

#### Extract Timestamp in Java (MongoDB Driver):
```java
org.bson.types.ObjectId objectId = new org.bson.types.ObjectId("650c1f2e9b1d8f2a4c8b4567");
java.util.Date creationDate = objectId.getDate();
System.out.println("Document creation date: " + creationDate);
```

---

## 4. SDET Use Cases & Testing Gotchas

### Gotcha 1: String vs. ObjectId Type Mismatch
- **Bug Scenario:** API returns `userId = "650c1f2e9b1d8f2a4c8b4567"` (String). You attempt a query in MongoDB: `db.users.find({ _id: "650c1f2e9b1d8f2a4c8b4567" })`.
- **Result:** Query returns **0 records** because String `"650c1f2e9b1d8f2a4c8b4567"` does NOT equal `ObjectId("650c1f2e9b1d8f2a4c8b4567")`.
- **Fix:** Wrap the string with `ObjectId(...)` in your test automation code.

### Gotcha 2: Floating Point Precision Bugs in Financial Assertions
- Using Double for money (e.g. `0.1 + 0.2 = 0.30000000000000004`) causes automated test assertion failures.
- Always check that prices use `NumberDecimal` / `Decimal128` in MongoDB!

---

## 5. Summary Checklist
- [x] Know why BSON allows richer data types than plain JSON.
- [x] Understand the 12-byte layout of `ObjectId`.
- [x] Know how to extract timestamps from `_id`.
- [x] Avoid String vs. ObjectId query mismatch pitfalls in automation frameworks.
