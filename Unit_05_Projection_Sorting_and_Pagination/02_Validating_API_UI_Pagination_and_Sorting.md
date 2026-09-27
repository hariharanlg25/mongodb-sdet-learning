# Unit 05 — Projection, Sorting and Pagination
## Module 02: Validating API/UI Pagination & Sorting against MongoDB

**SDET Priority:** ⭐⭐⭐⭐⭐ (Real-World E2E Test Strategy)

---

## 1. The Tri-Layer Validation Strategy

When an application presents a paginated, sortable data table on the front-end (e.g. a React/Angular admin dashboard), bugs frequently occur in:
- Backend SQL/NoSQL query translation
- Offset calculation off-by-one errors
- Incorrect sorting direction (Ascending vs Descending)

As an SDET, you validate data across all three layers:

```text
[ UI Layer (Selenium/Playwright) ]  <--- Extract 10 rows displayed in Data Table
             │
             ├── (Assert equality with)
             │
[ API Layer (REST Assured / HTTP) ] <--- Extract JSON array: GET /api/v1/users?page=2&limit=10&sort=-createdAt
             │
             ├── (Assert equality with)
             │
[ DB Layer (MongoDB Direct Query) ] <--- db.users.find().sort({createdAt: -1}).skip(10).limit(10)
```

---

## 2. SDET Step-by-Step E2E Pagination Validation Workflow

### Step 1: Pre-requisite Data Seeding
Seed the test database with a deterministic number of records (e.g., 25 mock products with distinct timestamps).

### Step 2: Query DB directly for expected Page 2 dataset
```javascript
// Database Expected Benchmark (Page 2, 10 items per page, sorted by price descending)
db.products.find({}, { name: 1, price: 1, _id: 0 })
  .sort({ price: -1 })
  .skip(10)
  .limit(10)
```

### Step 3: Trigger API endpoint & Assert response body
Send HTTP Request: `GET /api/v1/products?page=2&pageSize=10&sortBy=price&order=desc`
Assert that:
- `response.json().data.length == 10`
- `response.json().totalRecords == 25`
- `response.json().data[0].name == dbExpected[0].name`

### Step 4: UI Navigation & Grid Content Verification (Selenium/Playwright)
- Navigate to Products Dashboard UI.
- Click Page 2 button on pagination control.
- Extract table text content from column 1 & 2.
- Assert UI rows match DB benchmark records in exact sequence.

---

## 3. End-to-End Automation Code Example (REST Assured + MongoDB Driver)

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Projections;
import com.mongodb.client.model.Sorts;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.bson.Document;
import org.testng.Assert;
import org.testng.annotations.Test;

import java.util.ArrayList;
import java.util.List;

public class EndToEndPaginationValidation {

    @Test
    public void validateAPIPaginationAgainstMongoDB() {
        // 1. Query MongoDB for ground truth
        MongoCollection<Document> collection = MongoManager.getCollection("products");
        
        List<Document> dbExpectedList = collection.find()
                .projection(Projections.fields(Projections.include("sku", "price"), Projections.excludeId()))
                .sort(Sorts.descending("price"))
                .skip(10)
                .limit(10)
                .into(new ArrayList<>());

        // 2. Execute REST Assured API Call
        Response apiResponse = RestAssured.given()
                .queryParam("page", 2)
                .queryParam("size", 10)
                .queryParam("sort", "price:desc")
                .get("http://localhost:8080/api/products");

        Assert.assertEquals(apiResponse.getStatusCode(), 200, "API should return HTTP 200 OK!");

        // 3. Extract API JSON items
        List<String> apiSkus = apiResponse.jsonPath().getList("items.sku");

        // 4. Assert direct 1-to-1 match between DB and API response items
        for (int i = 0; i < dbExpectedList.size(); i++) {
            String dbSku = dbExpectedList.get(i).getString("sku");
            String apiSku = apiSkus.get(i);
            Assert.assertEquals(apiSku, dbSku, "Mismatch at index " + i + " between API and MongoDB!");
        }
    }
}
```

---

## 4. Advantages & Disadvantages of Direct DB Validation

### Advantages
- **Catches Stale Cache Bugs:** If Redis cache returns outdated paginated records, direct MongoDB comparison catches the discrepancy immediately.
- **Zero Reliance on UI:** API-to-DB pagination tests execute in milliseconds compared to slow browser automation tests.

### Disadvantages
- **Requires DB Connection Credentials:** SDET frameworks require direct network access to MongoDB test instances.

---

## 5. Summary Checklist
- [x] Always cross-verify UI/API pagination against DB `.skip()` and `.limit()`.
- [x] Assert both record sequence and total page counts.
- [x] Test edge cases: Page 1, Last Page, Out-of-bounds Page (e.g. Page 9999).
