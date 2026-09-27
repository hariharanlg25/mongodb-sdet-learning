# Unit 01 — MongoDB Fundamentals
## Module 01: What is MongoDB & NoSQL Overview

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential Foundation)

---

## 1. Definition & Core Concepts

### What is MongoDB?
**MongoDB** is an open-source, document-oriented, NoSQL (Not Only SQL) database designed for high-performance, high-availability, and automatic scaling. Instead of storing data in rigid tables with fixed rows and columns (like MySQL or PostgreSQL), MongoDB stores data as dynamic, flexible JSON-like documents called **BSON** (Binary JSON).

### SQL (Relational) vs. NoSQL (Document-based)
| Feature | SQL Databases (e.g., PostgreSQL, MySQL) | NoSQL Document DBs (e.g., MongoDB) |
| :--- | :--- | :--- |
| **Data Structure** | Tables, Rows, Columns | Databases, Collections, Documents |
| **Schema** | Rigid, predefined schema | Dynamic, flexible schema |
| **Relationships** | Foreign Keys & explicit JOINs | Embedded documents or ObjectId references |
| **Scalability** | Vertical (scale up CPU/RAM) | Horizontal (scale out via Sharding) |
| **Query Language** | SQL (Structured Query Language) | MQL (MongoDB Query Language) / JSON queries |

---

## 2. SDET & QA Use Case

### Why should an SDET care about NoSQL and MongoDB?
Modern enterprise microservices frequently use MongoDB as their primary datastore or read cache. As an SDET:
1. **Dynamic Response Validation:** Microservice APIs often echo MongoDB documents directly back to the front-end as JSON. Understanding NoSQL allows you to validate schema mutations and document structures.
2. **Schema Drift Detection:** Because MongoDB allows flexible schemas, different records in the same collection might have missing or extra fields across API versions. SDETs build schema assertion scripts to catch unintended schema breaking changes.
3. **Data Isolation:** For parallel automation runs, SDETs must create dynamic test documents with unique fields and teardown test collections after execution.

---

## 3. Advantages of MongoDB for Testing & Development

- **Schema Flexibility:** Perfect for agile projects where data requirements change frequently. Test environments don't break when new fields are introduced.
- **Native JSON Alignment:** Web API endpoints talk in JSON, and MongoDB stores documents in BSON/JSON. Translating test assertions from API responses to DB records requires zero complex ORM mapping.
- **High Performance:** In-memory caching and document embedding make read and write operations extremely fast during automated test execution.
- **Rich Querying:** Supports complex queries, array filtering, regex matching, and aggregate analytics out of the box.

---

## 4. Disadvantages & Challenges for SDETs

- **Lack of Rigid Constraints:** Because schema validation isn't mandatory by default, bad test data (e.g., string injected into an integer field) can be inserted without throwing a DB error. SDETs must be extra cautious during data seeding.
- **Complex Joins:** Joins (via `$lookup`) are not as performant or natural as SQL `JOIN`. Validating cross-collection data requires multi-stage aggregations.
- **Higher Memory Footprint:** Storing field names in every single document consumes more disk and RAM compared to fixed SQL tables.

---

## 5. Practical Shell Setup & Compass Overview

### MongoDB Tools for SDETs
1. **mongosh (MongoDB Shell):** The interactive JavaScript interface for querying and administrative tasks.
2. **MongoDB Compass:** The GUI client for visually querying collections, building aggregation pipelines, inspecting indexes, and exporting test datasets.

### Essential mongosh Startup Verification
```javascript
// Connect to local MongoDB instance via mongosh
mongosh "mongodb://localhost:27017"

// Check server status & version
db.version()

// View current connected database
db
```

---

## 6. SDET Summary Checklist
- [x] Know the fundamental shift from Tables/Rows → Collections/Documents.
- [x] Recognize why microservice JSON APIs map directly to MongoDB BSON documents.
- [x] Understand that NoSQL flexibility requires SDETs to perform strict field type and existence checks in automation test suites.
