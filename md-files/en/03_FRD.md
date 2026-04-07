# Functional Requirements Document (FRD)
**Project:** Chronopost International Integration  
**Version:** 1.0 | **Date:** 2026-04-05 | **Status:** ✅ Baselined

---

## 1. System Architecture

```
React 18 (Frontend)
    │  REST API (JSON)
    ▼
Spring Boot 3.2 (Backend) ──── Oracle 11g (Database)
    │                               │
    │  SFTP (port 22)               ├── CHR_SHIPMENTS
    ▼                               ├── CHR_SENDERS
Chronopost SFTP                     ├── CHR_RECEIVERS
  /IN  → GEODATA + Invoice upload   ├── CHR_PARCELS
  /OUT ← Invoice download           ├── CHR_CUSTOMS
                                    ├── CHR_INVOICE_LINES
    │  FTP (tracking)               ├── CHR_TRACKING_EVENTS
    ▼                               ├── CHR_PREALERT_FILES
Chronopost FTP                      ├── CHR_INVOICE_FILES
  EDIPOD feedback files             ├── CHR_PRODUCTS  ← PRODUIT.CSV
                                    ├── CHR_ROUTING   ← ROUCHR (417K rows)
                                    └── CHR_PARCEL_COUNTER
```

---

## 2. Feature List

### F01 — Shipment Management
| ID | Feature | Implementation |
|----|---------|---------------|
| F01.1 | Create shipment (5-step wizard) | `POST /api/shipment/create` |
| F01.2 | List all shipments (sort, filter, search, paginate) | `GET /api/shipment/list` |
| F01.3 | View shipment detail (nested sender/receiver/parcels) | `GET /api/shipment/{id}` |
| F01.4 | Edit sender address | `PUT /api/shipment/{id}/sender` |
| F01.5 | Edit receiver address | `PUT /api/shipment/{id}/receiver` |
| F01.6 | Export shipment list to CSV | Frontend only |
| F01.7 | Workflow status bar (Created→Label→PreAlert→Invoice→Delivered) | `ShipmentDetailPage.jsx` |

### F02 — Label Generation
| ID | Feature | Implementation |
|----|---------|---------------|
| F02.1 | Generate label PDF per parcel | `POST /api/label/generate` |
| F02.2 | Auto-resolve product (code 327/328) by weight | `ProductService.java` |
| F02.3 | Auto-resolve routing (origSort, distribSort) by country+zip | `RoutingService.java` |
| F02.4 | Preview label PDF in browser | `GET /api/label/preview/{parcelId}` |
| F02.5 | Download label PDF | `GET /api/label/download/{parcelId}` |
| F02.6 | DB-persisted parcel counter per prefix | `CHR_PARCEL_COUNTER` |
| F02.7 | 15-char tracking ID with ISO 7064 Mod 37/36 checksum | `TrackingIdGenerator.java` |
| F02.8 | Routing barcode (Code 128) with ZIP normalizer | `RoutingCalculator.java` |

### F03 — Pre-Alert (GEODATA)
| ID | Feature | Implementation |
|----|---------|---------------|
| F03.1 | Generate GEODATA CONSO file (8 subtypes) | `GeoDataFileBuilder.java` |
| F03.2 | Validate file before sending | `PreAlertValidator.java` |
| F03.3 | Send file to Chronopost via SFTP | `SftpService.java` |
| F03.4 | TEST mode (sends to /TEST folder) | `SftpService.setTestMode()` |
| F03.5 | Check pre-alert status | `GET /api/prealert/status/{id}` |

### F04 — Invoice Exchange
| ID | Feature | Implementation |
|----|---------|---------------|
| F04.1 | Upload invoice PDF to Chronopost SFTP /IN | `POST /api/invoice/upload` |
| F04.2 | Validate PDF resolution (≥300 DPI) | `InvoiceValidator.java` |
| F04.3 | Auto-fetch invoices from Chronopost SFTP /OUT (30 min) | `InvoiceScheduler.java` |
| F04.4 | List all invoice files | `GET /api/invoice/all` |
| F04.5 | View/download invoice PDF | `GET /api/invoice/view/{id}` |
| F04.6 | Retry failed uploads | `POST /api/invoice/retry/{id}` |
| F04.7 | TEST mode toggle | `POST /api/invoice/test-mode` |

### F05 — Tracking
| ID | Feature | Implementation |
|----|---------|---------------|
| F05.1 | Poll Chronopost FTP for EDIPOD files | `POST /api/tracking/poll` |
| F05.2 | Parse all 133 official event codes | `TrackingFileParser.java` |
| F05.3 | Update shipment status (DELIVERED/IN_TRANSIT/FAILED/RETURNED) | `TrackingService.java` |
| F05.4 | View tracking timeline per parcel | `GET /api/tracking/{parcelNumber}` |
| F05.5 | View all parcels for a shipment | `GET /api/tracking/shipment/{id}` |
| F05.6 | Search by parcel/tracking number | `GET /api/tracking/search?q=` |
| F05.7 | Tracking dashboard (counts by status) | `GET /api/tracking/dashboard` |

### F06 — Reference Data (Admin)
| ID | Feature | Implementation |
|----|---------|---------------|
| F06.1 | View CHR_PRODUCTS table (from PRODUIT.CSV) | `GET /api/admin/products` |
| F06.2 | Routing lookup by country+zip | `GET /api/admin/routing/lookup` |

---

## 3. Input Validation Rules

### Sender
| Field | Validation |
|-------|-----------|
| accountNo | Required, max 20 chars |
| companyName OR name1 | At least one required |
| street | Required, max 35 chars |
| zipcode | Required, max 9 chars, A-Z 0-9 only |
| town | Required, max 35 chars |
| countryCode | Required, ISO 3166-2 (2 chars) |
| phone OR email | At least one required |
| businessType | Required: B or P |

### Parcel
| Field | Validation |
|-------|-----------|
| declaredWeight | Required, > 0 grams |
| dimension | Optional, format: LLLWWWHHHcm (9 chars) |

### Invoice Line
| Field | Validation |
|-------|-----------|
| content | Required, max 200 chars (English preferred) |
| amount | Required, > 0 |
| hsCodeRecv | Required, 8 digits |
| hsCodeSend | Required, max 10 chars |
| countryOrigin | Required, ISO 3166-2 |

---

## 4. Error Handling

| Scenario | Response |
|----------|---------|
| Shipment not found | HTTP 404 |
| Validation failure | HTTP 400 with error list |
| SFTP connection failure | HTTP 500, logged, retryable |
| PDF < 300 DPI | HTTP 400: "PDF resolution below 300 DPI" |
| DB connection failure | HTTP 500, Spring Boot health check fails |
| Routing not found in DB | Fallback to country-level, then default `"250"` |
| Product not found in DB | Fallback to defaults (JF, COM INT, D-B2C) |

---

*Derived from: all Controller + Service Java files, frontend pages*  
*Last updated: 2026-04-05*
