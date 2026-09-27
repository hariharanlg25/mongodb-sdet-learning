# Unit 10 — MongoDB Java Automation Integration
## Module 03: REST Assured API & MongoDB Validation Framework

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential for API Automation Engineers)

---

## 1. Architectural Strategy

In modern API automation testing, validating HTTP response bodies (`200 OK`, JSON keys) is **only 50% of the job**. The remaining 50% is verifying that backend microservices accurately mutate persistent data in MongoDB.

```text
                               ┌───────────────────────────┐
                               │ REST Assured HTTP Client  │
                               └─────────────┬─────────────┘
                                             │
             ┌───────────────────────────────┴───────────────────────────────┐
             │ POST /api/v1/orders (Payload: SKU="BOOK-01", Qty=2, Price=50) │
             └───────────────────────────────┬───────────────────────────────┘
                                             │
                                             ▼
                               ┌───────────────────────────┐
                               │  Backend Microservice     │
                               └─────────────┬─────────────┘
                                             │
                                             ▼ Write BSON
                               ┌───────────────────────────┐
                               │   MongoDB Collection      │
                               └─────────────┬─────────────┘
                                             │
                                             ▼ Read & Assert
                               ┌───────────────────────────┐
                               │ MongoDB Java Driver       │
                               └───────────────────────────┘
```

---

## 2. Complete REST Assured + MongoDB Validation Test Case

```java
package com.automation.tests;

import com.automation.database.MongoUtils;
import com.mongodb.client.model.Filters;
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import io.restassured.response.Response;
import org.bson.Document;
import org.bson.types.ObjectId;
import org.testng.Assert;
import org.testng.annotations.Test;

import java.util.Date;
import java.util.List;

public class OrderAPIMongoDBValidationTest {

    @Test
    public void testOrderCreationAPIAndDBState() {
        // 1. Prepare Request Payload
        String orderPayload = "{\n" +
                "  \"customerId\": \"CUST-99281\",\n" +
                "  \"items\": [\n" +
                "    { \"sku\": \"MOUSE-LOGI-01\", \"qty\": 2, \"price\": 1250.00 }\n" +
                "  ],\n" +
                "  \"paymentMethod\": \"CREDIT_CARD\"\n" +
                "}";

        // 2. Trigger API Call
        Response response = RestAssured.given()
                .contentType(ContentType.JSON)
                .body(orderPayload)
                .post("http://localhost:8080/api/v1/orders");

        // 3. Assert HTTP Layer Response
        Assert.assertEquals(response.getStatusCode(), 201, "API should return HTTP 201 Created!");
        String orderId = response.jsonPath().getString("orderId");
        String mongoIdStr = response.jsonPath().getString("id");
        Assert.assertNotNull(mongoIdStr, "Response body should contain MongoDB ObjectId string!");

        // 4. Query MongoDB directly using extracted ObjectId
        Document dbOrder = MongoUtils.getDocument("orders", Filters.eq("_id", new ObjectId(mongoIdStr)));

        // 5. Hard Assertions on DB State
        Assert.assertNotNull(dbOrder, "Order document must exist in MongoDB!");
        Assert.assertEquals(dbOrder.getString("orderId"), orderId, "Order ID mismatch!");
        Assert.assertEquals(dbOrder.getString("status"), "PENDING_PAYMENT", "Initial order status must be PENDING_PAYMENT!");
        Assert.assertEquals(dbOrder.getDouble("totalAmount"), 2500.00, "Calculated total amount in DB is incorrect!");

        // Assert nested array items
        List<Document> items = dbOrder.getList("items", Document.class);
        Assert.assertEquals(items.size(), 1, "Items array should contain exactly 1 product!");
        Assert.assertEquals(items.get(0).getString("sku"), "MOUSE-LOGI-01", "Item SKU mismatch!");
    }
}
```

---

## 3. Summary Checklist
- [x] Extract `_id` or unique entity keys from API JSON responses to locate target documents in MongoDB.
- [x] Assert business logic state changes in MongoDB (status codes, calculated totals, arrays).
- [x] Build robust API automation suites that cover both API response assertions and database persistence validation.
