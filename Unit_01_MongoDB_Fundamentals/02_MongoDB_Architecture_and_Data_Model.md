# Unit 01 — MongoDB Fundamentals
## Module 02: MongoDB Architecture & Data Model

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential Foundation)

---

## 1. Definition & Architectural Hierarchy

MongoDB organizes data in a clean 4-tier hierarchy:

```text
[ MongoDB Instance / Server Cluster ]
                │
                ├──► [ Database 1: eCommerce_TestDB ]
                │         │
                │         ├──► [ Collection: users ]
                │         │         ├──► Document 1 (JSON/BSON)
                │         │         └──► Document 2 (JSON/BSON)
                │         │
                │         └──► [ Collection: orders ]
                │                   └──► Document 1
                │
                └──► [ Database 2: Payment_TestDB ]
```

### Key Components Defined
1. **Database:** Physical container for collections. Each DB gets its own set of files on the storage system.
2. **Collection:** A grouping of MongoDB documents (equivalent to an RDBMS table). Collections do not enforce a fixed schema.
3. **Document:** A single record in a collection, composed of key-value pairs (equivalent to an RDBMS row).
4. **Field:** A key-value pair within a document (equivalent to an RDBMS column/cell).

---

## 2. SQL vs. MongoDB Mapping Cheat Sheet

| SQL Term | MongoDB Term | Description |
| :--- | :--- | :--- |
| **Database** | Database | Physical grouping of storage files |
| **Table** | Collection | Grouping of related documents |
| **Row / Tuple** | Document | Individual BSON record |
| **Column** | Field | Key-value pair in a document |
| **Index** | Index | Data structure to accelerate query execution |
| **Table Join** | `$lookup` Aggregation | Joining two collections by matching fields |
| **Primary Key** | `_id` Field | Unique identifier automatically indexed by default |

---

## 3. SDET & QA Use Case

### Test Data Architecture & Tenant Isolation
When running parallel automated test suites (e.g., executing 50 Selenium tests concurrently via TestNG or Docker grid):
- **Database-per-Thread Isolation:** SDETs can create dynamic test databases per worker thread (`testdb_thread_1`, `testdb_thread_2`) to prevent state collision.
- **Collection Seeding:** SDETs seed pre-populated collections (`test_users`, `mock_orders`) before running API regression suites.

---

## 4. Sample Document Structure (eCommerce Order Context)

```json
{
  "_id": "650c1f2e9b1d8f2a4c8b4567",
  "orderId": "ORD-99281",
  "customer": {
    "name": "Ananya Sharma",
    "email": "ananya.sdet@example.com",
    "tier": "GOLD"
  },
  "items": [
    { "sku": "BOOK-JAVA-01", "qty": 1, "price": 499.00 },
    { "sku": "MOUSE-LOGI-90", "qty": 2, "price": 1250.50 }
  ],
  "totalAmount": 3000.00,
  "isPaid": true,
  "createdAt": "2026-09-27T10:00:00Z"
}
```

---

## 5. Advantages & Disadvantages of MongoDB Architecture

### Advantages
- **Polymorphic Data:** A `users` collection can store basic users, admin users, and vendor users with varying fields without creating sparse tables.
- **Embedded Hierarchies:** Storing nested objects (like `customer` or `items`) inside a single document eliminates the overhead of multi-table joins during API payload assertion.

### Disadvantages
- **Max Document Size Limit:** A single BSON document cannot exceed **16 MB**. If automated log collection outputs are saved as a nested array inside a single document, it can hit the 16MB ceiling.
- **No Automatic Constraints:** Unchecked automated tests could insert invalid data structures unless validation rules are applied.

---

## 6. mongosh Basic Architecture Exploration

```javascript
// List all databases available on the server
show dbs

// Switch to a target database (creates it dynamically upon first insert)
use test_automation_db

// Print currently selected database
db

// List all collections inside the active database
show collections
```
