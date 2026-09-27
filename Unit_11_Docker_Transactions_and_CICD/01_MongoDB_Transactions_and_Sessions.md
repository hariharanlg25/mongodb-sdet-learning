# Unit 11 — Docker, Transactions & CI/CD
## Module 01: MongoDB Multi-Document Transactions & Sessions

**SDET Priority:** ⭐⭐⭐ (Crucial for Financial & E-Commerce Testing)

---

## 1. Overview & Core Concepts

Prior to MongoDB version 4.0, operations on a single document were atomic, but multi-document writes were not. MongoDB introduced **Multi-Document ACID Transactions** using client sessions.

### Key Transaction Lifecycle Methods
1. `startSession()`: Creates a client session context.
2. `startTransaction()`: Begins an ACID transaction boundary on the active session.
3. `commitTransaction()`: Commits all mutations made within the transaction to disk.
4. `abortTransaction()`: Rollbacks all uncommitted mutations if an error occurs.

> ⚠️ **Prerequisite:** Multi-document transactions in MongoDB require a Replica Set or Sharded Cluster (not a standalone single instance).

---

## 2. SDET & QA Use Case: Testing Transaction Rollbacks

### Testing Negative Scenarios (Failures & Rollbacks)
Consider an E-Commerce Checkout API:
1. `Step 1`: Deduct user account wallet balance in `users` collection (-$100).
2. `Step 2`: Create order record in `orders` collection.
3. `Step 3`: Reserve stock in `products` collection.

**The Bug Scenario to Test:**
If Step 3 fails (e.g. inventory out of stock), the backend MUST trigger `abortTransaction()`. The SDET verifies that **Step 1 (wallet balance deduction) was completely rolled back in MongoDB** and not partially applied!

---

## 3. Java Driver Transaction Implementation Example

```java
import com.mongodb.client.ClientSession;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;
import org.bson.Document;
import org.testng.Assert;
import org.testng.annotations.Test;

public class TransactionRollbackTest {

    @Test
    public void verifyTransactionRollbackOnInventoryFailure() {
        MongoClient mongoClient = MongoManager.getDatabase().getMongoClient(); // Get client handle
        MongoCollection<Document> usersCol = MongoManager.getCollection("users");
        MongoCollection<Document> ordersCol = MongoManager.getCollection("orders");

        // Start Client Session
        try (ClientSession session = mongoClient.startSession()) {
            session.startTransaction();

            try {
                // Step 1: Deduct balance from user
                usersCol.updateOne(session, Filters.eq("email", "wallet.user@test.com"), Updates.inc("walletBalance", -100.0));

                // Step 2: Create Order
                ordersCol.insertOne(session, new Document("orderId", "ORD-TX-01").append("amount", 100.0));

                // Step 3: Force intentional failure (e.g. out of stock error)
                boolean outOfStock = true;
                if (outOfStock) {
                    throw new RuntimeException("Simulated Out of Stock Exception!");
                }

                // If no errors, commit transaction
                session.commitTransaction();

            } catch (Exception e) {
                // Abort and rollback all operations in session
                session.abortTransaction();
                System.out.println("Transaction successfully aborted and rolled back due to error: " + e.getMessage());
            }
        }

        // Assert DB state after rollback: Balance must remain unchanged!
        Document userDoc = usersCol.find(Filters.eq("email", "wallet.user@test.com")).first();
        Assert.assertEquals(userDoc.getDouble("walletBalance"), 500.0, "Wallet balance should NOT be deducted after transaction rollback!");
    }
}
```

---

## 4. Summary Checklist
- [x] Understand ACID properties: Atomicity, Consistency, Isolation, Durability.
- [x] Test negative scenarios to verify `abortTransaction()` rollbacks.
- [x] Remember that transactions require a Replica Set environment.
