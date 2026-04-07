# Chronopost Integration — Documentation

This folder contains all project documentation in two languages.

```
docs/
├── en/          ← English documents
│   ├── 01_ASSUMPTION_SHEET.md
│   ├── 02_BRD.md
│   ├── 03_FRD.md
│   ├── 04_API_CONTRACT.md
│   ├── 05_DB_SCHEMA.md
│   └── 06_UAT_PLAN.md
├── bn/          ← বাংলা documents
│   ├── 01_ASSUMPTION_SHEET.md
│   ├── 02_BRD.md
│   ├── 03_FRD.md
│   ├── 04_API_CONTRACT.md
│   ├── 05_DB_SCHEMA.md
│   └── 06_UAT_PLAN.md
└── README.md    ← This file
```

## Document Order (Priority)

| # | Document | Purpose | Status |
|---|----------|---------|--------|
| 01 | Assumption Sheet | Confirm critical values with Chronopost | ✅ Done |
| 02 | BRD | Business Requirements Document | ✅ Done |
| 03 | FRD | Functional Requirements Document | ✅ Done |
| 04 | API Contract | All endpoints with request/response | ✅ Done |
| 05 | DB Schema | All tables and relationships | ✅ Done |
| 06 | UAT Plan | Test cases for Chronopost validation | ✅ Done |

## Update Policy

- When **code changes** → update corresponding doc section
- When **Chronopost confirms** an assumption → update `01_ASSUMPTION_SHEET.md`
- When **new endpoint added** → update `03_FRD.md` + `04_API_CONTRACT.md`
- When **DB table changes** → update `05_DB_SCHEMA.md`

## App Version

- Backend: Spring Boot 3.2 + Oracle 11g
- Frontend: React 18 + Tailwind CSS
- GEODATA Version: 3.32
- Routing File: ROUCHR_20260302_B3

