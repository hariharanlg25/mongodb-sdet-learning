# Unit 09 — SDET Validation & Test Data Management
## Module 01: End-to-End Data Validation (UI ↔ API ↔ MongoDB)

**SDET Priority:** ⭐⭐⭐⭐⭐ (Core Production SDET Workflow)

---

## 1. Overview of Tri-Layer Data Validation

Modern enterprise applications process user actions through three distinct architectural layers:

```text
  ┌─────────────────────────────────────────────────────────────┐
  │ 1. Frontend UI Layer (React, Angular, Mobile App)           │
  └──────────────────────────────┬──────────────────────────────┘
                                 │ HTTP POST Request (JSON)
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ 2. Backend Microservice Layer (Node.js, Java Spring, Go)    │
  └──────────────────────────────┬──────────────────────────────┘
                                 │ BSON Document Write
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ 3. Database Layer (MongoDB Cluster)                         │
  └─────────────────────────────────────────────────────────────┘
```

An SDET's primary responsibility is **End-to-End Validation**: confirming that a successful UI form submission or API request translates into an accurate, correctly formatted, and completely persisted document in MongoDB.

---

## 2. E2E Scenario: User Registration Flow

### Step 1: UI Trigger (Selenium / Playwright)
User fills registration form on web application:
- `First Name`: "Rahul"
- `Last Name`: "Sharma"
- `Email`: "rahul.sdet@example.com"
- `Password`: "P@ssword123"

### Step 2: API Validation (REST Assured)
Capture API response from `POST /api/v1/users`:
- Assert `HTTP Status == 201 Created`
- Extract JSON field `userId` (e.g. `"650c1f2e9b1d8f2a4c8b4567"`)

### Step 3: Direct MongoDB Validation (MongoDB Java Driver)
Query MongoDB: `db.users.findOne({ _id: ObjectId("650c1f2e9b1d8f2a4c8b4567") })`

#### SDET Checklist Assertions on Database Document:
- [x] Document is **not null**.
- [x] `email` equals `"rahul.sdet@example.com"`.
- [x] `password` is **NOT stored as plain text** (Assert string matches BCrypt hash pattern `^\$2[ayb]\$.{56}$`).
- [x] `status` is set to `"PENDING_VERIFICATION"`.
- [x] `createdAt` timestamp is within the last 30 seconds.
- [x] `roles` array contains default role `"ROLE_USER"`.
- [x] Sensitive internal field `failedAttempts` defaults to integer `0`.

---

## 3. End-to-End Java Code Implementation

```java
import com.mongodb.client.MongoCollection;
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

public class EndToEndUserRegistrationTest {

    @Test
    public void verifyUserRegistrationFlowInDB() {
        String testEmail = "auto_user_" + System.currentTimeMillis() + "@test.com";

        // 1. Trigger User Registration API Call
        String payload = String.format("{\"email\":\"%s\",\"password\":\"TestPass123!\",\"name\":\"Automation User\"}", testEmail);

        Response response = RestAssured.given()
                .contentType(ContentType.JSON)
                .body(payload)
                .post("http://localhost:8080/api/v1/users");

        Assert.assertEquals(response.getStatusCode(), 201, "API user creation failed!");
        String userIdStr = response.jsonPath().getString("userId");
        Assert.assertNotNull(userIdStr, "API response missing userId!");

        // 2. Direct MongoDB Validation
        MongoCollection<Document> usersCol = MongoManager.getCollection("users");
        Document dbUser = usersCol.find(Filters.eq("_id", new ObjectId(userIdStr))).first();

        // 3. Comprehensive SDET Assertions
        Assert.assertNotNull(dbUser, "User record missing in MongoDB!");
        Assert.assertEquals(dbUser.getString("email"), testEmail, "DB Email mismatch!");
        Assert.assertEquals(dbUser.getString("status"), "PENDING_VERIFICATION", "Status mismatch!");
        
        // Assert password security hash
        String storedPassword = dbUser.getString("password");
        Assert.assertFalse(storedPassword.equals("TestPass123!"), "SECURITY BUG: Password stored as plain text!");

        // Assert timestamp freshness
        Date createdAt = dbUser.getDate("createdAt");
        long diffSeconds = (new Date().getTime() - createdAt.getTime()) / 1000;
        Assert.assertTrue(diffSeconds < 30, "createdAt timestamp is older than expected!");

        // Assert default roles array
        List<String> roles = dbUser.getList("roles", String.class);
        Assert.assertTrue(roles.contains("ROLE_USER"), "Missing default user role!");
    }
}
```

---

## 4. Summary Checklist
- [x] Perform assertions on DB documents after every critical API mutation.
- [x] Assert field values, data types, security hash formats, and timestamp freshness.
- [x] Never rely solely on HTTP response status codes for state verification!
