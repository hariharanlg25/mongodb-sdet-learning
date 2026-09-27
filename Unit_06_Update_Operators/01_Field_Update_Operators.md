# Unit 06 — Update Operators
## Module 01: Field Update Operators

**SDET Priority:** ⭐⭐⭐⭐ (Essential for State Mutation in Test Setup)

---

## 1. Overview & Operator Breakdown

Update operators modify existing document fields without needing to fetch, deserialize, modify, and re-save the entire document payload in application memory.

---

## 2. Core Field Update Operators

| Operator | Description | mongosh Example |
| :--- | :--- | :--- |
| `$set` | Sets the value of a field in a document | `{ $set: { status: "ACTIVE" } }` |
| `$unset` | Deletes a field from a document | `{ $unset: { temporaryOtp: "" } }` |
| `$inc` | Increments/decrements a numeric field by a specified amount | `{ $inc: { loginAttempts: 1 } }` |
| `$mul` | Multiplies the value of a field by a specified number | `{ $mul: { price: 1.10 } }` |
| `$rename` | Renames a field key | `{ $rename: { "fname": "firstName" } }` |
| `$min` | Updates field ONLY if specified value is less than current value | `{ $min: { lowestScore: 45 } }` |
| `$max` | Updates field ONLY if specified value is greater than current value | `{ $max: { highScore: 980 } }` |

---

## 3. Practical mongosh Examples

### `$set` and `$unset`
```javascript
// Update user status and remove temporary verification token
db.users.updateOne(
  { email: "john@example.com" },
  {
    $set: { status: "VERIFIED", verifiedAt: new Date() },
    $unset: { verificationToken: "" }
  }
);
```

---

### `$inc` (Increment & Decrement)
```javascript
// Increment login attempt counter by 1
db.users.updateOne(
  { username: "test_user_01" },
  { $inc: { loginAttempts: 1 } }
);

// Decrement product stock by 2 after order placement
db.products.updateOne(
  { sku: "PHONE-XYZ" },
  { $inc: { stockQuantity: -2 } }
);
```

---

### `$rename`
Useful when simulating database schema migrations during backend release testing!
```javascript
// Rename 'phoneno' to 'phoneNumber' across all users
db.users.updateMany(
  {},
  { $rename: { "phoneno": "phoneNumber" } }
);
```

---

## 4. SDET & QA Use Case: Test Data Preparation

### Scenario: Simulating Expired OTP or Password Token
When testing an API endpoint like `POST /api/v1/auth/verify-otp`, standard execution requires waiting 5 to 10 minutes for an OTP to expire. With `$set`, the SDET manually overrides the timestamp in test setup:

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;
import org.bson.Document;
import org.testng.annotations.Test;

import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.Date;

public class OTPValidationTest {

    @Test
    public void testExpiredOTPRejection() {
        MongoCollection<Document> usersCol = MongoManager.getCollection("users");

        // Force OTP expiration timestamp to 15 minutes in the past using $set
        Date expiredDate = Date.from(Instant.now().minus(15, ChronoUnit.MINUTES));

        usersCol.updateOne(
            Filters.eq("email", "otp.test@example.com"),
            Updates.combine(
                Updates.set("otpCode", "123456"),
                Updates.set("otpExpiresAt", expiredDate)
            )
        );

        // Now trigger API call POST /verify-otp { "email": "otp.test@example.com", "otp": "123456" }
        // ASSERT: API returns HTTP 400 Bad Request with message "OTP Expired"
    }
}
```

---

## 5. Summary Checklist
- [x] Use `$set` for targeted field updates without replacing entire documents.
- [x] Use `$unset` to test missing optional field scenarios.
- [x] Use `$inc` with negative numbers to test inventory decrements.
