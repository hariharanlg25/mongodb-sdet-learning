# Unit 11 — Docker, Transactions & CI/CD
## Module 03: MongoDB in CI/CD (GitHub Actions Pipeline)

**SDET Priority:** ⭐⭐⭐ (Crucial for Continuous Integration & Automated Nightly Builds)

---

## 1. Overview of MongoDB in CI/CD Pipelines

In modern CI/CD pipelines (GitHub Actions, GitLab CI, Jenkins), automated test suites run on clean, ephemeral virtual machines. To execute database assertions, the pipeline runner must dynamically spin up a containerized MongoDB service, execute TestNG / REST Assured tests, publish test results, and tear down resources cleanly.

```text
 [ GitHub Push / PR Event ]
             │
             ▼
 [ GitHub Actions Runner VM ]
             │
             ├──► 1. Spin up MongoDB Service Container (Port 27017)
             │
             ├──► 2. Run Maven Test Suite: `mvn test` (TestNG + REST Assured)
             │          │
             │          └──► Direct BSON read/write assertions against MongoDB
             │
             ├──► 3. Generate Allure / TestNG HTML Report
             │
             └──► 4. Teardown Container & Publish Test Artifacts
```

---

## 2. Complete GitHub Actions Workflow (`.github/workflows/sdet_regression.yml`)

```yaml
name: SDET Automated Test Suite with MongoDB

on:
  push:
    branches: [ "main", "develop" ]
  pull_request:
    branches: [ "main" ]
  schedule:
    # Run nightly build at midnight UTC
    - cron: '0 0 * * *'

jobs:
  automation-tests:
    runs-on: ubuntu-latest

    # Service containers to run alongside the runner job
    services:
      mongodb:
        image: mongo:latest
        ports:
          - 27017:27017
        options: >-
          --health-cmd mongo
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout Code Repository
        uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Verify MongoDB Container Health
        run: |
          docker ps
          nc -zv localhost 27017

      - name: Execute TestNG Automation Suite
        env:
          MONGO_URI: mongodb://localhost:27017
          TEST_ENV: ci
        run: mvn clean test -DsuiteXmlFile=testng.xml

      - name: Generate Allure Test Report
        if: always()
        uses: simple-elf/allure-report-action@master
        with:
          allure_results: target/allure-results
          allure_history: allure-history

      - name: Upload Test Report Artifacts
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: execution-report
          path: target/surefire-reports
```

---

## 3. Best Practices for MongoDB in CI/CD

1. **Environment Variable Injection:** Read connection URIs via `System.getenv("MONGO_URI")` so tests connect to `localhost:27017` in CI and staging clusters in pre-prod.
2. **Health Checks:** Use container options (`--health-cmd`) to ensure MongoDB is fully booted before Maven launches TestNG.
3. **Flaky Test Prevention:** Ensure `@BeforeSuite` drops or clears test collections so rerun attempts start from a pristine state.

---

## 4. Summary Checklist
- [x] Configure service containers in GitHub Actions YAML for zero-dependency test execution.
- [x] Pass database credentials and URIs via environment variables.
- [x] Attach test execution reports (Allure/Surefire) as downloadable CI build artifacts.
