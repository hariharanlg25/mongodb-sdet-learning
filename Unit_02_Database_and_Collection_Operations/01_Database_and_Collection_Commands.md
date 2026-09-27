# Unit 02 — Database and Collection Operations
## Module 01: Database & Collection Management

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential for Environment Setup & Data Teardown)

---

## 1. Definitions & Operations Overview

### Database Operations
In MongoDB, databases are created lazily. Switching to a database using `use dbname` does not immediately write to disk until data is actually inserted into a collection.

### Collection Operations
Collections hold documents. While MongoDB creates collections automatically upon document insertion, explicit collection creation allows SDETs to set up schema validation rules or capped collections for capped log streaming.

---

## 2. Shell Commands Cheat Sheet

### Database Commands
```javascript
// Show all databases with allocated storage
show dbs

// Switch database context (or create dynamic db reference)
use sdet_automation_db

// Display current active database context
db

// Drop current database (Extreme caution: used in complete environment cleanup)
db.dropDatabase()
```

### Collection Commands
```javascript
// Show all collections in active database
show collections

// Explicit collection creation
db.createCollection("test_users")

// Drop collection (Used in @AfterClass teardown)
db.test_users.drop()
```

---

## 3. SDET & QA Use Case

### Test Data Setup & Teardown Patterns
In automated testing, test isolation is critical. If automated tests mutate shared database collections without teardown, subsequent test runs will fail due to duplicate key errors or stale state.

#### Pattern: Dynamic Collection Dropping in Automation Setup
```javascript
// mongosh script executed prior to regression run:
use test_staging;

// Clear existing test collections before seeding fresh mock data
db.users.drop();
db.orders.drop();

// Pre-create fresh collections
db.createCollection("users");
db.createCollection("orders");
```

---

## 4. Capped Collections (Advanced SDET Use Case)

### What is a Capped Collection?
A **Capped Collection** is a fixed-size, high-performance circular collection that overwrites its oldest documents when it reaches its allocated size limit.

### SDET Use Case: Automated Test Execution Logs
SDETs often store live test logs or API trace logs in MongoDB. Using a capped collection guarantees that log storage won't consume infinite disk space during nightly CI builds.

```javascript
// Create a capped collection capped at 5 MB or max 1000 log documents
db.createCollection("execution_logs", { 
  capped: true, 
  size: 5242880, 
  max: 1000 
});
```

---

## 5. Advantages & Disadvantages

### Advantages
- **Lazy Allocation:** No overhead or schema setup scripts required prior to running quick POC automation scripts.
- **Instant Teardown:** Dropping a collection (`db.users.drop()`) is infinitely faster than executing thousands of `DELETE FROM users` queries in SQL.

### Disadvantages
- **Accidental Drop Risk:** Without explicit user permissions, executing `db.dropDatabase()` in a shared staging server will wipe shared QA data for the whole team.

---

## 6. Summary Checklist
- [x] Know how to switch databases using `use <dbname>`.
- [x] Use `drop()` for clean test suite teardown.
- [x] Understand Capped Collections for log streaming in automated frameworks.
