# Automation Rules

## 1. Purpose

1. Separate test cases from page logic.
2. DRY (Don’t Repeat Yourself).
3. Keep the code clean and maintainable.

## 2. Core Principles

1. Use **POM (Page Object Model)** pattern.
2. Use **Playwright MCP**.
3. **Locator Priority**:
   - Prioritize using `getByRole` first.
   - Try using other locators (e.g., `getByText`, `getByLabel`) only if necessary.
   - Use `.locator()` as the last option if needed.
4. The tests will use **Chromium**; you can ignore Firefox and Webkit.

## 3. Project Structure

```text
playwright-automation/
│
├── tests/                     # Test cases (only test logic)
│   ├── 01-login.spec.ts
│   ├── 02-checkout.spec.ts
│   └── XX-{feature}.spec.ts
│
├── pages/                     # Page Object Model (POM)
│   ├── LoginPage.ts
│   ├── CheckoutPage.ts
│   └── BasePage.ts
│
├── fixtures/                  # Custom fixtures
│   └── testFixtures.ts
│
├── utils/                     # Reusable helpers
│   ├── apiHelper.ts
│   ├── waitHelper.ts
│   ├── logger.ts
│   └── dataGenerator.ts
│
├── test-data/                 # Static test data
│   ├── users.json
│   └── products.json
│
├── config/
│   └── environments.ts
│
├── reports/                   # Auto-generated reports
│
├── playwright.config.ts
├── package.json
└── README.md
```

## 4. IMPORTANT

1. **DON’T ASSUME AND ALWAYS CONFIRM WITH ME.**
