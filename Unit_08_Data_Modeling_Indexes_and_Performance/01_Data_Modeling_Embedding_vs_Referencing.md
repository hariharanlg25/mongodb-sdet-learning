# Unit 08 — Data Modeling, Indexes & Performance
## Module 01: Data Modeling — Embedding vs. Referencing

**SDET Priority:** ⭐⭐⭐⭐ (Essential for Schema Understanding & Payload Design)

---

## 1. Overview & Data Modeling Paradigms

MongoDB supports two primary design patterns for structuring data relationships:

1. **Embedded Data Model (Denormalized):** Related data is nested directly inside a single document as embedded objects or arrays.
2. **Referenced Data Model (Normalized):** Data is separated into multiple collections, and documents reference one another using unique keys (usually `ObjectId`).

---

## 2. Comparison Table: Embedding vs. Referencing

| Dimension | Embedded Model (Denormalized) | Referenced Model (Normalized) |
| :--- | :--- | :--- |
| **Storage Structure** | Sub-documents or arrays in 1 document | Separate collections connected by `_id` |
| **Query Performance** | Single query retrieves all data (Fast) | Requires `$lookup` aggregation or multiple queries |
| **Atomicity & Writes** | Atomic updates on single document | Requires multi-document transactions |
| **Max Document Size** | Risk of exceeding 16 MB limit if array grows unbounded | No document size limit constraint |
| **Data Redundancy** | Duplicate data across documents | Zero duplication (Single Source of Truth) |

---

## 3. Practical Data Modeling Patterns

### Pattern A: Embedded Sub-Document (One-to-Few)
Ideal when child data strongly belongs to parent and will not grow infinitely (e.g., user address or user settings).

```json
{
  "_id": ObjectId("650c1f2e9b1d8f2a4c8b4567"),
  "name": "Rahul Sharma",
  "email": "rahul@example.com",
  "address": {
    "street": "100 MG Road",
    "city": "Bengaluru",
    "zip": "560001"
  }
}
```

---

### Pattern B: Referenced Documents (One-to-Many / Many-to-Many)
Ideal when child collection grows continuously (e.g. e-commerce orders or system audit logs).

```json
// Collection: users
{
  "_id": ObjectId("650c1f2e9b1d8f2a4c8b4567"),
  "name": "Rahul Sharma"
}

// Collection: orders (References userId)
{
  "_id": ObjectId("780f2a9c1d8f2a4c8b999999"),
  "userId": ObjectId("650c1f2e9b1d8f2a4c8b4567"),
  "orderNumber": "ORD-554433",
  "amount": 1499.00
}
```

---

## 4. SDET & QA Use Case: JSON Schema Validation Enforcement

SDETs can query MongoDB's `$jsonSchema` validator rules to ensure the backend application enforces required fields and data types at database level:

```javascript
// mongosh: Check collection schema validation rules
db.getCollectionInfos({ name: "users" })
```

### Inspected Validation Output Example:
```json
{
  "validator": {
    "$jsonSchema": {
      "bsonType": "object",
      "required": ["name", "email", "status"],
      "properties": {
        "email": {
          "bsonType": "string",
          "pattern": "^.+@.+$",
          "description": "email must be a valid string"
        }
      }
    }
  }
}
```

---

## 5. Summary Checklist
- [x] Use Embedded documents for One-to-Few relationships with fast single-query retrieval.
- [x] Use Referenced documents when arrays risk exceeding 16 MB.
- [x] Inspect `$jsonSchema` rules to catch missing backend field constraints during QA analysis.
