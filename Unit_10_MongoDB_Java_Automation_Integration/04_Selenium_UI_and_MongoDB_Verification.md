# Unit 10 — MongoDB Java Automation Integration
## Module 04: Selenium UI & MongoDB Verification

**SDET Priority:** ⭐⭐⭐⭐⭐ (Essential for Web UI Automation Engineers)

---

## 1. Architectural Strategy

Web UI automated tests often suffer from **UI latency and element sync issues**. For example, after submitting a user registration form on a web UI, waiting for a confirmation email or UI toast message can be flaky. 

By integrating MongoDB directly into your Selenium test framework:
- **Instant Backend Confirmation:** Immediately poll MongoDB to confirm the record was saved before asserting UI elements.
- **Dynamic OTP Extraction:** Extract email verification tokens or OTPs directly from MongoDB to complete automated login flows without accessing real email inboxes!

---

## 2. Complete Selenium + MongoDB Automation Example

### Scenario: Bypassing Email Verification in UI Tests via MongoDB
1. Selenium enters registration details on web page and clicks "Sign Up".
2. Application saves user document in MongoDB with a generated 6-digit OTP code `emailVerificationOtp`.
3. Selenium script queries MongoDB to fetch the generated OTP code directly.
4. Selenium inputs the extracted OTP code into the UI OTP form and clicks "Verify".
5. Assert successful navigation to User Dashboard!

```java
package com.automation.tests;

import com.automation.database.MongoUtils;
import com.mongodb.client.model.Filters;
import org.bson.Document;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import java.time.Duration;
import java.util.UUID;

public class SeleniumUIMongoDBTest {

    private WebDriver driver;
    private String testEmail;

    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        driver.manage().window().maximize();
        testEmail = "ui_user_" + UUID.randomUUID().toString().substring(0, 6) + "@test.com";
    }

    @Test
    public void testUserRegistrationWithDirectDBOtpExtraction() {
        // 1. Selenium UI: Fill Registration Form
        driver.get("http://localhost:8080/register");
        driver.findElement(By.id("name")).sendKeys("Automation Tester");
        driver.findElement(By.id("email")).sendKeys(testEmail);
        driver.findElement(By.id("password")).sendKeys("P@ssword123!");
        driver.findElement(By.id("btn-submit")).click();

        // 2. Direct MongoDB Query: Extract generated OTP code
        Document userDoc = MongoUtils.getDocument("users", Filters.eq("email", testEmail));
        Assert.assertNotNull(userDoc, "User document must be created in MongoDB upon form submission!");
        
        String generatedOtp = userDoc.getString("emailVerificationOtp");
        Assert.assertNotNull(generatedOtp, "OTP code missing in MongoDB user document!");

        // 3. Selenium UI: Enter extracted OTP into UI verification screen
        driver.findElement(By.id("otp-input")).sendKeys(generatedOtp);
        driver.findElement(By.id("btn-verify-otp")).click();

        // 4. Assert UI Dashboard Access
        String welcomeMessage = driver.findElement(By.id("welcome-banner")).getText();
        Assert.assertTrue(welcomeMessage.contains("Welcome, Automation Tester"), "UI failed to log in user after OTP verification!");

        // 5. Final DB Status Assertion
        Document updatedUserDoc = MongoUtils.getDocument("users", Filters.eq("email", testEmail));
        Assert.assertEquals(updatedUserDoc.getString("status"), "ACTIVE", "User status in DB should update to ACTIVE!");
    }

    @AfterMethod
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
        // Cleanup dynamic user document from DB
        MongoUtils.deleteDocument("users", Filters.eq("email", testEmail));
    }
}
```

---

## 3. Summary Checklist
- [x] Use direct MongoDB queries to extract hidden tokens/OTPs for fast end-to-end UI automation.
- [x] Avoid flaky UI wait statements by polling MongoDB for backend state mutations.
- [x] Clean up Selenium-generated test records during `@AfterMethod` teardown.
