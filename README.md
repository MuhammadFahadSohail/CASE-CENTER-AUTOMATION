# CASE-CENTER-AUTOMATION

Playwright + TypeScript automation framework for Ontario Courts Case Center.

This project is Case Center only and supports two environments: `beta` and `prod`.

## Project Purpose

The primary goal of this project is to measure and track execution time for key end-to-end user steps across multiple Case Center test scripts.
Each automated flow captures step-level timings (not just pass/fail) so teams can compare performance across runs and build reliable timing baselines for `beta` and `prod` environments.

Functional validation remains important, but this framework is designed mainly for timing visibility and repeatable performance monitoring.

## Prerequisites

- Node.js 20+
- npm 9+
- A valid Case Center test account
- Outlook mailbox access for verification-code and case-link email flows

## Setup

1. Install dependencies:

```bash
npm install
```

2. Create/update `.env` (or copy from `.env.example`) and set:

```env
ENVIRONMENT=beta
TEST_EMAIL=your.email@domain.com
TEST_PASSWORD=your_password
```

Notes:
- `ENVIRONMENT` must be `beta` or `prod`.
- `BASE_URL` is optional and only needed to override the default URL from environment config.

## Environment Configuration

Environment values are defined in `src/config/environments.ts`:

- `beta` -> `https://ontariocourtsbeta.casecenter.thomsonreuters.com`
- `prod` -> `https://ontariocourts.casecenter.thomsonreuters.com`
- Outlook URL is centralized there as well.

## Project Structure

```text
CASE-CENTER-AUTOMATION/
├── auth/
│   └── storageState.json
├── reports/
│   ├── html/
│   ├── junit/
│   └── screenshots/
├── src/
│   ├── config/
│   │   └── environments.ts
│   ├── constants/
│   │   ├── messages.ts
│   │   └── urls.ts
│   ├── fixtures/
│   │   └── baseFixture.ts
│   ├── pages/
│   │   ├── BasePage.ts
│   │   ├── LoginPage.ts
│   │   ├── OutlookPage.ts
│   │   ├── CaseSearchPage.ts
│   │   └── CaseCreatePage.ts
│   ├── test-data/
│   │   ├── TC_001_login-email-verification.json
│   │   ├── TC_002_search-case.json
│   │   ├── TC_003_create-case.json
│   │   └── TC_004_review-evidence.json
│   ├── tests/
│   │   ├── TC_001_login-email-verification.spec.ts
│   │   ├── TC_002_search-case.spec.ts
│   │   ├── TC_003_create-case.spec.ts
│   │   └── TC_004_review-evidence.spec.ts
│   └── utils/
│       ├── envHelper.ts
│       ├── jsonHelper.ts
│       └── testLogger.ts
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

## Test Design

- Uses Page Object Model in `src/pages`.
- Uses shared typed Playwright fixtures in `src/fixtures/baseFixture.ts`.
- Uses JSON-driven test data from `src/test-data`.
- Uses step-level timing helper in `src/utils/testLogger.ts`.

## Run Commands

Runs all tests with Playwright default settings.
```bash
npm test
```

Runs tests in headed mode.
```bash
npm run test:headed
```

Runs tests in headed mode with 3 parallel workers (default is 1). Adjust the number as needed.
```bash
npm run test:headed -- --workers=3
```

Runs only `TC_001_login-email-verification.spec.ts` in the `login` project and refreshes `auth/storageState.json`.
```bash
npm run test:refresh-auth
```

Runs login first, then runs the main `test` project.
```bash
npm run test:with-login
```

Opens the Playwright HTML report from `reports/html`.
```bash
npm run report
```

Runs a single test file in default (headless) mode.
```bash
npx playwright test src/tests/TC_002_search-case.spec.ts
```

Runs a single test file in headed mode.
```bash
npx playwright test src/tests/TC_002_search-case.spec.ts --headed
```

## Authentication Flow

- `login` project runs `src/tests/TC_001_login-email-verification.spec.ts` without storage state.
- On success, it writes `auth/storageState.json`.
- `test` project uses that storage state for the remaining specs.

If authentication expires, run `npm run test:refresh-auth` before main test execution.

## Reporting

- HTML report: `reports/html`
- JUnit XML: `reports/junit/results.xml`
- Test artifacts and traces: `test-results`
- Screenshots attached by test utilities: `reports/screenshots`

## Notes

- Keep credentials only in `.env`; never commit secrets.
- JSON test data should contain business/input values only.
- Timeout and retry behavior belongs in code/config, not test-data JSON.

