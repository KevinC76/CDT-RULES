# k6 Load Testing Procedure

> **Note:** TAMBAHKAN 1 FIELDS UNTUK TEMPAT NYIMPAN REPORT DAN LOGS

## 1. Purpose

This document standardizes how load testing scripts are written using k6 to ensure:

- Consistent structure
- Reusable configuration
- Clear test case separation
- Measurable performance visibility
- Automatic structured reporting
- Simplicity and maintainability

## 2. Core Principles

1. Single source of configuration (`utils/config.js`)
2. Separated performance profiles
3. Scenario-based test structure
4. No hardcoded values inside scenario files
5. Dot notation is allowed
6. Use bracket notation when dynamic access is required
7. Keep implementation simple
8. **DON’T ASSUME AND ASK FOR CONFIRMATION.**

## 3. Project Structure

```text
k6-tests/
│
├── utils/
│   └── config.js
│
├── scenarios/
│   ├── 01-Login-Test.js
│   ├── 02-Checkout-Test.js
│   └── XX-{Title}.js
│
├── reports/
│
└── run.sh
```

## 4. File Naming Convention

All scenario files **MUST** follow:
`XX-{Title}.js`

Where:

- **XX** = 2-digit test case number (01, 02, 03...)
- **{Title}** = short descriptive name
- Words separated by `-`
- **Example:**
  - `01-Login-Test.js`
  - `02-Checkout-Flow.js`

**Rules:**

- Always 2-digit numbering
- No spaces in file name
- One scenario per file
- Do not mix multiple test cases

## 5. utils/config.js

This file contains **ALL** configuration.

**Rules:**

- Dot notation is allowed.
- Bracket notation **MUST** be used when accessing dynamic properties.
- No test logic inside config.
- No custom metrics unless explicitly required.

**Example `utils/config.js`:**

```javascript
const ENV = __ENV.ENV || "staging";
const PROFILE = __ENV.PROFILE || "load";

const config = {
  environment: ENV,
  profile: PROFILE,

  baseURL: {
    staging: "https://staging-api.example.com",
    production: "https://api.example.com",
  },

  performanceProfiles: {
    smoke: {
      executor: "constant-vus",
      vus: 1,
      duration: "30s",
    },

    load: {
      executor: "ramping-vus",
      startVUs: 0,
      stages: [
        { duration: "30s", target: 10 },
        { duration: "1m", target: 20 },
        { duration: "30s", target: 0 },
      ],
      gracefulRampDown: "30s",
    },

    stress: {
      executor: "ramping-vus",
      startVUs: 0,
      stages: [
        { duration: "30s", target: 50 },
        { duration: "1m", target: 100 },
        { duration: "30s", target: 0 },
      ],
      gracefulRampDown: "30s",
    },

    other: {
      executor: "constant-vus",
      vus: 5,
      duration: "1m",
    },
  },

  thresholds: {
    http_req_duration: ["p(95)<500"],
    http_req_failed: ["rate<0.01"],
  },

  headers: {
    "Content-Type": "application/json",
  },

  timeout: "60s",
};

export default config;
```

## 6. Standard Scenario Template

Each scenario:

- Imports config
- Defines its own options
- Uses bracket notation for dynamic access
- Logs performance clearly
- Generates summary report

**Example: `01-Login-Test.js`**

```javascript
import http from "k6/http";
import { check, sleep } from "k6";
import config from "../utils/config.js";

// dynamic access → bracket notation REQUIRED
export const options = {
  scenarios: {
    default: config.performanceProfiles[config.profile],
  },
  thresholds: config.thresholds,
};

export default function () {
  const baseURL = config.baseURL[config.environment];

  const payload = JSON.stringify({
    username: "testuser",
    password: "password123",
  });

  const res = http.post(`${baseURL}/login`, payload, {
    headers: config.headers,
    timeout: config.timeout,
  });

  check(res, {
    "status is 200": (r) => r.status === 200,
  });

  console.log(`
===== Performance Log =====
Scenario File : 01-Login-Test.js
Profile       : ${config.profile}
Environment   : ${config.environment}
Status        : ${res.status}
Duration (ms) : ${res.timings.duration}
============================
`);

  sleep(1);
}
```

## 7. Summary Report Naming Standard

Each scenario **MUST** implement `handleSummary()`.

File naming format:
`{Date}-{Time}-{Profile} Report.json`

**Example:**
`2026-02-23-14-30-Stress Report.json`

**Add to Bottom of Scenario File:**

```javascript
export function handleSummary(data) {
  const now = new Date();

  const date = now.toISOString().split("T")[0];
  const time = now.toTimeString().split(" ")[0].replace(/:/g, "-");
  const profile = config.profile;

  const fileName = `${date}-${time}-${profile} Report.json`;

  return {
    [`reports/${fileName}`]: JSON.stringify(data, null, 2),
  };
}
```

## 8. Execution Guide

Run default profile:

```bash
k6 run scenarios/01-Login-Test.js
```

Run specific profile:

```bash
k6 run -e PROFILE=stress scenarios/01-Login-Test.js
```

Run specific environment:

```bash
k6 run -e ENV=production -e PROFILE=load scenarios/01-Login-Test.js
```

## 9. Validation Checklist

Before merging:

- [ ] Naming convention correct
- [ ] No hardcoded URLs
- [ ] Uses `config.js`
- [ ] Dynamic access uses bracket notation
- [ ] Thresholds applied
- [ ] Performance log visible
- [ ] Summary report generated
- [ ] Script remains simple

## 10. Mandatory Rule

If any requirement is unclear:
**DON’T ASSUME AND ASK FOR CONFIRMATION.**

Always clarify:

- Expected RPS?
- SLA target?
- Maximum error rate?
- Which profile?
- Which environment?
