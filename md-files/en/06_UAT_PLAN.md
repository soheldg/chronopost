# UAT Plan (User Acceptance Testing)
**Project:** Chronopost International Integration  
**Version:** 1.0 | **Date:** 2026-04-05 | **Status:** ⏳ Not Started

---

## Pre-Conditions

Before UAT begins, the following must be ready:

- [ ] Chronopost confirmed 5 critical assumptions (see `01_ASSUMPTION_SHEET.md`)
- [ ] Real SDEPOT, SYSDEPOT, HARDWARE values in `application.yml`
- [ ] Chronopost provided TEST SFTP credentials
- [ ] Backend running: `mvn spring-boot:run`
- [ ] Frontend running: `npm run dev`
- [ ] Database up with all tables + routing data loaded
- [ ] Chronopost TEST environment active

---

## Test Environment

| Item | Value |
|------|-------|
| Backend URL | `http://localhost:8080/api` |
| Frontend URL | `http://localhost:3000` |
| SFTP Mode | **TEST** (`/TEST` folder) |
| Database | Oracle 11g local / DEV |
| Test Account | [Chronopost TEST account number] |

---

## Test Cases

### TC-01: Shipment Creation
| Step | Action | Expected Result | Status |
|------|--------|-----------------|--------|
| 1 | Open app → New Shipment | 5-step wizard loads | ☐ |
| 2 | Fill sender: KBT Express, BD, +880... | No validation error | ☐ |
| 3 | Fill receiver: Belgium, zip 1000 | No validation error | ☐ |
| 4 | Add parcel: 1440g, 35×25×50cm | Auto-selects code 327 (>3kg) | ☐ |
| 5 | Add parcel: 500g | Auto-selects code 328 (≤3kg) | ☐ |
| 6 | Add customs item: Cardigan, €83, BD, HS 61101200 | No validation error | ☐ |
| 7 | Click Create | Shipment ID assigned, MPS ID generated | ☐ |
| 8 | View shipment in list | Appears with status CREATED | ☐ |

### TC-02: Label Generation
| Step | Action | Expected Result | Status |
|------|--------|-----------------|--------|
| 1 | Go to Labels → enter shipment ID | Form loads | ☐ |
| 2 | Click Generate | Success, parcel ID + tracking ID returned | ☐ |
| 3 | Parcel ID format | 13 chars: `[5-prefix][6-counter][2-suffix]` | ☐ |
| 4 | Tracking ID format | 15 chars with valid ISO 7064 checksum | ☐ |
| 5 | Preview PDF | Label renders correctly in browser | ☐ |
| 6 | Verify Zone 4 routing | origSort/distribSort from CHR_ROUTING | ☐ |
| 7 | Verify legal disclaimer | Exact text: "On delivery, damage or theft must be the **subjet** of..." | ☐ |
| 8 | Download PDF | PDF saved locally | ☐ |
| 9 | Test GB zip SW1A | Routing barcode zip = `0011111` | ☐ |
| 10 | Test IE zip D01 | Routing barcode zip = `0000000` | ☐ |

### TC-03: Pre-Alert (GEODATA)
| Step | Action | Expected Result | Status |
|------|--------|-----------------|--------|
| 1 | Go to Pre-Alert → enter shipment ID | Form loads | ☐ |
| 2 | Click Generate File | GEODATA file preview appears | ☐ |
| 3 | Verify file header | `#FILE;GEODATA_CONSO_...` format | ☐ |
| 4 | Verify `#DEF` lines | `#DEF` (not `#DEFINITION`) | ☐ |
| 5 | Verify VERSION | `#VERSION;3.32;` (not `03.32`) | ☐ |
| 6 | Verify MPSWEIGHT | Weight in decagrams (1440g → `144`) | ☐ |
| 7 | Enable TEST mode | Toggle switches to TEST | ☐ |
| 8 | Click Send to Chronopost | Status = SENT, file in /TEST folder | ☐ |
| 9 | Chronopost confirms file received | Chronopost validation = OK | ☐ |
| 10 | Check pre-alert status | Status = PREALERT_SENT | ☐ |

### TC-04: Invoice Upload
| Step | Action | Expected Result | Status |
|------|--------|-----------------|--------|
| 1 | Prepare PDF: ≥300 DPI | — | ☐ |
| 2 | Go to Invoices → upload | Drag+drop or browse | ☐ |
| 3 | Select shipment ID + type EXP | — | ☐ |
| 4 | Click Upload | Status = SENT, file appears in list | ☐ |
| 5 | Test PDF < 300 DPI | Error: "PDF resolution below 300 DPI" | ☐ |
| 6 | Click Fetch Now | Downloads from Chronopost /OUT | ☐ |
| 7 | View downloaded invoice | PDF renders in browser | ☐ |

### TC-05: Tracking
| Step | Action | Expected Result | Status |
|------|--------|-----------------|--------|
| 1 | Go to Tracking → Poll FTP | EDIPOD files processed | ☐ |
| 2 | Event code `D` received | Shipment status → DELIVERED | ☐ |
| 3 | Event code `N` received | Status → FAILED | ☐ |
| 4 | Event code `R` received | Status → RETURNED | ☐ |
| 5 | Event code `P` received | Status → FAILED (absent) | ☐ |
| 6 | Search parcel number | Timeline shows all events | ☐ |
| 7 | Dashboard counts | Delivered/InTransit/Failed counts correct | ☐ |

### TC-06: Edit Sender/Receiver
| Step | Action | Expected Result | Status |
|------|--------|-----------------|--------|
| 1 | Open Shipment Detail | All info visible | ☐ |
| 2 | Click Edit on Sender | Modal opens with current data | ☐ |
| 3 | Change street address | — | ☐ |
| 4 | Click Save | Success message, new address visible | ☐ |
| 5 | Click Edit on Receiver | Modal opens | ☐ |
| 6 | Change ZIP code | — | ☐ |
| 7 | Save | New ZIP visible | ☐ |

### TC-07: Reference Data Pages
| Step | Action | Expected Result | Status |
|------|--------|-----------------|--------|
| 1 | Go to Products page | CHR_PRODUCTS rows visible | ☐ |
| 2 | Filter International (I) | Only I rows shown | ☐ |
| 3 | Sort by Code Geopost | Sorted ascending/descending | ☐ |
| 4 | Go to Routing Lookup | Form loads | ☐ |
| 5 | Lookup BE, 1000 | origSort=BE1, distribSort=011C | ☐ |
| 6 | Lookup FR, 75001 | origSort=IDF, distribSort=98S60 | ☐ |
| 7 | Lookup GB, SW1A | normalizedZip=0011111 | ☐ |

---

## Sign-Off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Developer (KBT Express) | | | |
| Chronopost Technical Lead | | | |
| Business Owner (KBT Express) | | | |

---

## Known Defects Before UAT

| # | Defect | Severity | Resolution |
|---|--------|----------|-----------|
| D01 | SDEPOT/SYSDEPOT/HARDWARE are placeholders | Critical | Confirm with Chronopost |
| D02 | sortCode hardcoded as "MAR" | Medium | Confirm with Chronopost |
| D03 | LORRY field empty for RO linehaul | Medium | Confirm enforcement with Chronopost |

---

*Last updated: 2026-04-05*
