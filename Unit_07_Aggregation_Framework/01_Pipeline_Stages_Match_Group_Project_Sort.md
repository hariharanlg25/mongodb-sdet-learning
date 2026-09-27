# Unit 07 — Aggregation Framework
## Module 01: Pipeline Stages ($match, $group, $project, $sort)

**SDET Priority:** ⭐⭐⭐⭐⭐ (Must-Have Skill for Experienced SDETs & Analytics APIs)

---

## 1. Aggregation Pipeline Concept

### What is the Aggregation Framework?
The MongoDB **Aggregation Framework** is a multi-stage data processing pipeline. Documents enter a multi-stage pipeline where each stage transforms, filters, groups, or reshapes the dataset before passing the transformed documents to the next stage.

```text
[ Raw Documents ]
       │
       ▼
 [ Stage 1: $match ]   <--- Filter records (e.g. status == "COMPLETED")
       │
       ▼
 [ Stage 2: $group ]   <--- Group by category & calculate sums/averages
       │
       ▼
 [ Stage 3: $project ] <--- Reshape output fields & calculate ratios
       │
       ▼
 [ Stage 4: $sort ]    <--- Sort final analytics results
       │
       ▼
 [ Final Output Array ]
```

---

## 2. Core Aggregation Stages & Accumulators

### Stages Breakdown
- `$match`: Filters documents (identical to `find()` query syntax). Always place `$match` early in the pipeline to reduce processed document volume!
- `$group`: Groups documents by a specified `_id` field and applies accumulators.
- `$project`: Reshapes output fields or creates computed fields.
- `$sort`: Orders output documents.
- `$limit` / `$skip`: Controls output pagination.
- `$unwind`: Deconstructs an array field from input documents to output a document for EACH element.

### Group Accumulators
- `$sum`: Calculates total sum or counts documents (`$sum: 1`).
- `$avg`: Calculates numerical average.
- `$min` / `$max`: Finds minimum or maximum value within group.
- `$push`: Appends values to an array within group.
- `$addToSet`: Collects unique values into an array within group.

---

## 3. Practical mongosh Example (eCommerce Order Analytics)

### Problem Statement:
Calculate total revenue and average order value for each customer tier among "COMPLETED" orders, returning tiers with total revenue greater than $1,000, sorted descending by revenue.

```javascript
db.orders.aggregate([
  // Stage 1: Filter completed orders
  {
    $match: { status: "COMPLETED" }
  },
  // Stage 2: Group by customer tier & calculate accumulators
  {
    $group: {
      _id: "$customerTier",
      totalRevenue: { $sum: "$totalAmount" },
      totalOrders: { $sum: 1 },
      avgOrderValue: { $avg: "$totalAmount" }
    }
  },
  // Stage 3: Reshape output & round averages
  {
    $project: {
      _id: 0,
      tier: "$_id",
      totalRevenue: 1,
      totalOrders: 1,
      avgOrderValue: { $round: ["$avgOrderValue", 2] }
    }
  },
  // Stage 4: Filter groups with totalRevenue > 1000
  {
    $match: { totalRevenue: { $gt: 1000 } }
  },
  // Stage 5: Sort descending by totalRevenue
  {
    $sort: { totalRevenue: -1 }
  }
]);
```

---

## 4. SDET & QA Use Case: Analytics API Test Automation

When testing an Analytics API endpoint like `GET /api/v1/analytics/revenue-by-tier`, traditional unit tests cannot verify backend math calculations. An SDET constructs an identical aggregation pipeline in Java using MongoDB Java Driver and asserts the API payload against the aggregation results.

```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Accumulators;
import com.mongodb.client.model.Aggregates;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Sorts;
import org.bson.Document;
import org.bson.conversions.Bson;
import org.testng.Assert;
import org.testng.annotations.Test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class AnalyticsAggregationTest {

    @Test
    public void verifyAnalyticsAPIAgainstAggregation() {
        MongoCollection<Document> ordersCol = MongoManager.getCollection("orders");

        // Build Java Aggregation Pipeline using Aggregates helper
        List<Bson> pipeline = Arrays.asList(
            Aggregates.match(Filters.eq("status", "COMPLETED")),
            Aggregates.group("$customerTier",
                Accumulators.sum("totalRevenue", "$totalAmount"),
                Accumulators.sum("totalOrders", 1)
            ),
            Aggregates.sort(Sorts.descending("totalRevenue"))
        );

        List<Document> aggregationResults = ordersCol.aggregate(pipeline).into(new ArrayList<>());

        Assert.assertFalse(aggregationResults.isEmpty(), "Aggregation should return results!");
        Assert.assertEquals(aggregationResults.get(0).getString("_id"), "GOLD", "Top tier should be GOLD!");
    }
}
```

---

## 5. Summary Checklist
- [x] Always place `$match` at the beginning of pipelines to utilize indexes.
- [x] Use `$group` with `$sum: 1` for counting categorized records.
- [x] Use `$unwind` when analyzing individual elements inside nested arrays.
