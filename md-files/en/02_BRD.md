# Business Requirements Document (BRD)
**Project:** Chronopost International Integration  
**Version:** 1.0  
**Date:** 2026-04-05  
**Status:** ✅ Baselined (derived from implemented code)

---

## 1. Executive Summary

KBT Express (Bangladesh) requires a software system to send international parcels to European customers via Chronopost's carrier network. The system must handle the complete shipment lifecycle: creating shipments, generating shipping labels, sending pre-alerts (GEODATA), managing customs invoices, and tracking deliveries in real-time.

---

## 2. Business Context

| Item | Detail |
|------|--------|
| **Client** | KBT Express, Bangladesh |
| **Carrier** | Chronopost (France) — Geopost Group |
| **Destination** | European Union + 219 countries via DPD network |
| **Products** | B2C cross-border parcels (clothing, accessories, consumer goods) |
| **Volume** | Multiple shipments per day |
| **Priority** | Pre-alert file must reach Chronopost before parcel departs |

---

## 3. Actors / Users

| Actor | Description | System Access |
|-------|-------------|--------------|
| **Logistics Operator** | KBT Express staff who creates shipments and prints labels | Full app access |
| **Customs Officer** | Uploads commercial invoice PDFs | Invoice page only |
| **Manager** | Monitors shipment status and tracking dashboard | Read-only |
| **Chronopost System** | External: receives GEODATA files, sends tracking events | SFTP only |

---

## 4. Business Requirements

### BR-01: Shipment Creation
- Operator must be able to create a new international shipment with: sender info, receiver info, parcel details (weight, dimensions), customs information, and invoice line items.
- System must validate: mandatory fields, country codes, weight > 0, at least one invoice line.
- Each shipment gets a unique MPS ID (Multi-Parcel Shipment identifier).

### BR-02: Shipping Label Generation
- System must generate a Chronopost-compliant shipping label (PDF, A5 or A4 format) for each parcel.
- Label must contain: sender/receiver addresses, routing barcode, tracking barcode, parcel ID, customs clearance code, service name, legal disclaimer.
- Label format must comply with **Doc 3 — Chronopost Label Spec V25.01**.
- Routing information (origSort, distribSort, agency code) must be automatically resolved from the routing database (417,612 rows from ROUCHR file).

### BR-03: Pre-Alert (GEODATA) Submission
- System must generate a GEODATA CONSO file (semicolon-separated, ISO-8859-1 encoding) per **Doc 1 — GEODATA V3.3.2**.
- File must be sent to Chronopost via **SFTP** (port 22, key-based authentication) before parcel physically departs.
- System must validate file before sending: mandatory fields, weight consistency, weight sum check.
- File must include: HEADER, CONSOLIDATION, SHIPMENT, SENDER, RECEIVER, INTER (customs), INTERINVOICELINE (items), PARCEL records.
- System must support TEST mode (sends to `/TEST` folder instead of `/IN`).

### BR-04: Commercial Invoice Exchange
- System must support upload of customs invoice PDFs to Chronopost SFTP `/IN` folder.
- System must automatically fetch invoices from Chronopost SFTP `/OUT` folder (every 30 minutes).
- Invoice PDF must be ≥ 300 DPI resolution.
- File naming: `INV_EXP_[account]_[parcelId].PDF` (export) or `INV_IMP_[account]_[parcelId].PDF` (import).
- Complies with **Doc 4 — Commercial Invoice Exchange v1.1**.

### BR-05: Tracking
- System must poll Chronopost FTP for EDIPOD tracking feedback files (every 30 minutes).
- System must parse all 133 official Chronopost event codes (per Evenements_Chronopost_2023-06-30.xlsx).
- System must update shipment status based on events: DELIVERED, IN_TRANSIT, FAILED, RETURNED.
- Operators can search by parcel number or shipment ID.

### BR-06: Shipment Management
- Operator must be able to view all shipments in a searchable, filterable, sortable list.
- Operator must be able to correct sender/receiver address after creation (before label generation).
- System must show workflow progress: Created → Label → Pre-Alert → Invoice → In Transit → Delivered.

---

## 5. Business Rules

| Rule ID | Rule Description | Source |
|---------|-----------------|--------|
| BRule-01 | Parcel weight ≤ 3,000g → service code 328 (SMALL); > 3,000g → service code 327 (NORMAL) | Assumed (⚠️ confirm with Chronopost) |
| BRule-02 | Ireland (IE) ZIP code → numerical `00000` for routing | Doc V13.01 §7.1 |
| BRule-03 | Jersey/Guernsey ZIP → numerical `03333` for routing | Doc V13.01 §7.1 |
| BRule-04 | GB remote areas → `03333`; all other GB → `01111` | Doc V13.01 §7.1 |
| BRule-05 | Portugal ZIP: remove `-` before routing lookup | Doc V13.01 §7.2 |
| BRule-06 | Netherlands ZIP: remove spaces before routing lookup | Doc V13.01 §7.2 |
| BRule-07 | GEODATA MPSWEIGHT field = parcel weight in **decagrams** (g ÷ 10) | Doc 1 GEODATA |
| BRule-08 | Tracking barcode = 15 chars with ISO 7064 Mod 37/36 checksum | Doc 3 Label |
| BRule-09 | Reason for export default = `"01"` (Sale) | Assumed |
| BRule-10 | Pre-alert status default = `"S04"` (sent on departure day) | Assumed |

---

## 6. Out of Scope

| Item | Reason |
|------|--------|
| Domestic France shipments | System only handles international |
| Appointment (RDV) services | ROUCHR file loaded without RDV (ssRDV) |
| Alaska deliveries | ROUCHR file loaded without Alaska (ssALASKA) |
| Returns management (full) | Return tracking parsed but no return workflow |
| Payment processing | Duties/taxes paid externally |
| Multi-user authentication | Single-tenant app, no login system |

---

## 7. Constraints

| Constraint | Detail |
|-----------|--------|
| **GEODATA version** | Must be exactly `"3.32"` |
| **File encoding** | GEODATA files must be `ISO-8859-1` |
| **Invoice PDF DPI** | Minimum 300 DPI |
| **SFTP authentication** | Key-based only (no password) |
| **Routing file** | Must be reloaded monthly when Chronopost publishes new ROUCHR |
| **Oracle version** | 11g (no JSON columns, limited JPQL features) |

---

## 8. Assumptions

See `01_ASSUMPTION_SHEET.md` for full list.  
**5 critical assumptions** require Chronopost confirmation before go-live.

---

*Derived from: implemented source code, Chronopost Docs 1–4, ROUCHR routing file*  
*Last updated: 2026-04-05*
