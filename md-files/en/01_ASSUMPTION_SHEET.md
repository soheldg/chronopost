# Assumption Confirmation Sheet
**Project:** Chronopost International Integration  
**Version:** 1.0  
**Date:** 2026-04-05  
**Status:** ⏳ Awaiting Chronopost Confirmation  

---

## How to Use This Document

This document lists all values and business rules that were **assumed** during development.  
Please review each item and fill in the **Confirmed Value** column.  
Return to: [your email] before UAT begins.

---

## 🔴 CRITICAL — Must Confirm Before Go-Live (5 items)

These 5 values are account-specific. Wrong values will cause **GEODATA file rejection**.

| # | Field | Location in Code | Assumed Value | Confirmed Value | Chronopost Contact |
|---|-------|-----------------|---------------|-----------------|-------------------|
| 1 | **SDEPOT** (Sender Depot Code) | `application.yml → chronopost.depot.sender` | `0029999` | _____________ | Technical team |
| 2 | **SYSDEPOT** (System Depot Code) | `application.yml → chronopost.depot.receiver` | `0020456` | _____________ | Technical team |
| 3 | **HARDWARE** (Hardware Type Code) | `GeoDataFileBuilder.java` line 276 | `"K"` | _____________ | Technical team |
| 4 | **MPSCOMP** (MPS Complementary) | `GeoDataFileBuilder.java` line 264 | `"0"` | _____________ | Technical team |
| 5 | **LORRY** field for RO (Road) linehaul | `GeoDataFileBuilder.java` | Empty (warning only) | _____________ | Technical team |

> **Impact of wrong values:**
> - SDEPOT/SYSDEPOT wrong → file rejected at gateway
> - HARDWARE wrong → shipment routing error
> - MPSCOMP wrong → MPS grouping error
> - LORRY empty for RO → possible linehaul error for Romania shipments

---

## 🟡 MEDIUM — Business Logic to Confirm (5 items)

| # | Rule | Assumed Behavior | Confirmed? | Notes |
|---|------|-----------------|------------|-------|
| 6 | **Small parcel threshold** | Weight ≤ 3,000g → service code 328 (SMALL); > 3,000g → 327 (NORMAL) | ☐ Yes ☐ No | `ProductService.java` line 41 |
| 7 | **sortCode field** | Hardcoded `"MAR"` | ☐ Yes ☐ No | Should come from routing file? |
| 8 | **parcelTag field** | Always empty `""` | ☐ Yes ☐ No | When is tag required? |
| 9 | **Label center display format** | `CC-agencyCode` e.g. `"BE-0467"` | ☐ Yes ☐ No | `RoutingCalculator.java` |
| 10 | **OPCODE for update/delete** | Only `"INS"` implemented | ☐ Yes ☐ No | What to send for correction? |

---

## 🟢 LOW — Technical Defaults (Acceptable, but please confirm)

| # | Field | Default | Spec Reference | OK? |
|---|-------|---------|----------------|-----|
| 11 | `REASONFOREXPORT` | `"01"` (Sale) | Doc 1 INTER field | ☐ Yes ☐ No |
| 12 | `PREALERTSTATUS` | `"S04"` (Sent on departure day) | Doc 1 INTER field | ☐ Yes ☐ No |
| 13 | Invoice type default | `"EXP"` (Export) | Doc 4 | ☐ Yes ☐ No |
| 14 | GEODATA header line skip | Skip if starts with `"Contract"` | Doc 2 | ☐ Yes ☐ No |
| 15 | Footer timestamp | Print time (not shipping date) | Doc 3 label | ☐ Yes ☐ No |
| 16 | `numCountryCode` fallback | `"250"` (France) if DB miss | Routing file | ☐ Yes ☐ No |
| 17 | SFTP port | `22` (standard SSH) | Doc 4 | ☐ Yes ☐ No |
| 18 | GEODATA version | `"3.32"` | Doc 1 header | ☐ Yes ☐ No |

---

## Configuration Values to Receive from Chronopost

Please provide the following **before UAT**:

| Item | Format | Example | Our Value |
|------|--------|---------|-----------|
| Customer Account Number | 8 digits | `14576904` | _____________ |
| Customer Prefix (5 chars) | Alphanumeric | `HY712` | _____________ |
| Geopost Sender ID (7 chars) | Numeric | `0407112` | _____________ |
| SDEPOT | 7 chars | `0029999` | _____________ |
| SYSDEPOT | 7 chars | `0020456` | _____________ |
| SFTP Host | FQDN | `sftp.chronopost.fr` | _____________ |
| SFTP Username | String | `cust_12345` | _____________ |
| SFTP Private Key | RSA/ED25519 | — | _____________ |
| TEST SFTP folder path | Path | `/TEST` | _____________ |
| Production SFTP folder path | Path | `/IN` | _____________ |

---

## How Confirmed Values Get Applied

Once Chronopost confirms the values above:

1. Update `backend/src/main/resources/application.yml` with real values
2. Update this document — change ⏳ to ✅ in header
3. Re-run UAT test cases in `06_UAT_PLAN.md`

---

*Document generated from: `GeoDataFileBuilder.java`, `application.yml`, `ProductService.java`*  
*Last updated: 2026-04-05*
