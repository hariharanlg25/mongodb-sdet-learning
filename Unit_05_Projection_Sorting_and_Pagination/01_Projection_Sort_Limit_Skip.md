# Unit 05 — Projection, Sorting and Pagination
## Module 01: Projection, Sort, Limit & Skip

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential for Asserting UI Data Tables & Pagination)

---

## 1. Projections Overview

### What is Projection?
Projection means fetching **only specific fields** from a document rather than fetching the entire document payload. 

### Why is Projection Important for SDETs?
1. **Network Performance:** Returning 2 fields instead of 50 fields makes test execution much faster across high-volume regression runs.
2. **Security & Data Masking Assertions:** Verifying that sensitive backend fields (like `passwordHash` or `ssn`) are explicitly masked from standard query projections.

### mongosh Projection Syntax
`find(queryFilter, projectionSpecifier)`

- `1` or `true`: Include field.
- `0` or `false`: Exclude field.

> ⚠️ **Rule:** You cannot mix inclusion (`1`) and exclusion (`0`) in a single projection, EXCEPT for suppressing the default `_id` field (`_id: 0`).

```javascript
// Inclusion: Fetch ONLY name and email (plus _id by default)
db.users.find({}, { name: 1, email: 1 });

// Suppress _id from projection output
db.users.find({}, { name: 1, email: 1, _id: 0 });

// Exclusion: Fetch all fields EXCEPT passwordHash and internalToken
db.users.find({}, { passwordHash: 0, internalToken: 0 });
```

---

## 2. Sorting (`sort()`)

Sorting arranges returned documents in ascending (`1`) or descending (`-1`) order based on specified fields.

```javascript
// Sort users by age ascending (1)
db.users.find().sort({ age: 1 });

// Compound Sort: Sort by department ascending, then age descending
db.users.find().sort({ department: 1, age: -1 });
```

---

## 3. Pagination (`limit()` & `skip()`)

Pagination divides large datasets into smaller pages.
- `limit(n)`: Specifies the maximum number of documents to return.
- `skip(m)`: Specifies the number of documents to bypass/skip.

### Formula for Page Pagination in Automation
```javascript
let pageSize = 10;
let pageNumber = 3; // Requesting Page 3

db.users.find()
  .skip((pageNumber - 1) * pageSize) // skips 20 records
  .limit(pageSize);                  // returns next 10 records
```

---

## 4. Method Chaining Order in MongoDB

In MongoDB driver execution, the chaining order of `sort()`, `skip()`, and `limit()` is **automatically optimized by MongoDB** regardless of how you write it in code. MongoDB will always execute:
1. `sort()`
2. `skip()`
3. `limit()`

---

## 5. Java Automation Driver Example

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Projections;
import com.mongodb.client.model.Sorts;
import org.bson.Document;
import org.bson.conversions.Bson;
import org.testng.Assert;
import org.testng.annotations.Test;

import java.util.ArrayList;
import java.util.List;

public class PaginationAndSortTest {

    @Test
    public void verifySortedPaginationResults() {
        MongoCollection<Document> usersCol = MongoManager.getCollection("users");

        // Combine Projections, Sort, Skip, and Limit using Bson builders
        Bson projection = Projections.fields(
            Projections.include("name", "email", "createdAt"),
            Projections.excludeId()
        );

        List<Document> page2List = usersCol.find()
                .projection(projection)
                .sort(Sorts.descending("createdAt"))
                .skip(10)
                .limit(10)
                .into(new ArrayList<>());

        Assert.assertEquals(page2List.size(), 10, "Page 2 should contain exactly 10 records!");
        Assert.assertFalse(page2List.get(0).containsKey("_id"), "_id field should be excluded by projection!");
    }
}
```

---

## 6. Summary Checklist
- [x] Use projections to optimize query speed and test assertion payload size.
- [x] Use `Sorts.ascending()` / `Sorts.descending()` to test UI data grid headers.
- [x] Apply `(pageNumber - 1) * pageSize` for testing paginated API endpoints.
