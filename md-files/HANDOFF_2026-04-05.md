# 📦 Chronopost Integration — Session Handoff
**Date:** 2026-04-05  
**Stack:** Spring Boot 3.2 + React 18 + Oracle 11g  
**ZIP:** `chronopost-integration.zip` (177KB, 178 files)  
**Status:** ✅ Core complete — awaiting Chronopost credential confirmation for UAT

---

## 📁 Project Structure

```
chronopost-integration/
│
├── backend/src/main/java/com/chronopost/integration/
│   ├── admin/
│   │   └── AdminController.java              ← GET /admin/products, /admin/routing/lookup
│   ├── common/
│   │   ├── config/   ChronopostConfig.java, FtpConfig.java
│   │   ├── dto/      ApiResponse.java
│   │   ├── model/    Shipment, Sender, Receiver, Parcel, Customs,
│   │   │             InvoiceLine, InvoiceFile, PreAlertFile, TrackingEvent
│   │   ├── repository/ ShipmentRepository, SenderRepository,
│   │   │               ReceiverRepository, ParcelRepository, ...
│   │   └── util/     ChecksumUtil.java
│   ├── invoice/      InvoiceController, InvoiceService, InvoiceValidator,
│   │                 InvoiceFileNameBuilder, InvoiceScheduler
│   ├── label/
│   │   ├── controller/  LabelController.java
│   │   ├── dto/         LabelRequest.java ← SIMPLIFIED (4 fields only)
│   │   ├── generator/   LabelPdfGenerator, BarcodeImageGenerator,
│   │   │                ParcelIdGenerator, TrackingIdGenerator,
│   │   │                RoutingCalculator, ZipCodeNormalizer ← NEW
│   │   └── service/     LabelService.java
│   ├── prealert/
│   │   ├── builder/     GeoDataFileBuilder.java ← 8 subtypes
│   │   ├── controller/  PreAlertController.java
│   │   ├── sftp/        SftpService.java ← JSch (not FTP)
│   │   ├── service/     PreAlertService.java
│   │   └── validator/   PreAlertValidator.java
│   ├── product/      Product.java, ProductRepository, ProductService ← NEW
│   ├── routing/      RoutingEntry.java, RoutingRepository, RoutingService ← NEW
│   ├── shipment/     ShipmentController, ShipmentService,
│   │                 ShipmentCreateRequest/Response, ShipmentListDto
│   └── tracking/     TrackingController, TrackingService,
│                     TrackingFileParser ← 133 official event codes
│
├── backend/src/main/resources/
│   ├── application.yml                       ← version "3.32", SFTP port 22
│   └── db/schema.sql                         ← 13 tables + indexes
│
├── frontend/src/
│   ├── App.jsx                               ← 10 pages, grouped nav
│   └── pages/
│       ├── ShipmentListPage.jsx              ← sort+filter+search+bulk+CSV
│       ├── ShipmentDetailPage.jsx            ← workflow bar + edit modals
│       ├── NewShipmentPage.jsx               ← 5-step wizard
│       ├── LabelPage.jsx                     ← simplified 4-field form
│       ├── PreAlertPage.jsx                  ← GEODATA preview + send
│       ├── InvoicePage.jsx                   ← upload/download/retry
│       ├── TrackingPage.jsx                  ← fixed event codes D/N/P/R
│       ├── ProductsPage.jsx                  ← CHR_PRODUCTS viewer ← NEW
│       └── RoutingLookupPage.jsx             ← CHR_ROUTING lookup ← NEW
│
├── docs/
│   ├── en/  01_ASSUMPTION_SHEET.md ← 🔴 send to Chronopost first!
│   │         02_BRD.md, 03_FRD.md, 04_API_CONTRACT.md
│   │         05_DB_SCHEMA.md, 06_UAT_PLAN.md
│   └── bn/  (same 6 files in Bengali)
│
└── [SQL scripts at project root]
    ├── alter_add_routing_table.sql
    ├── alter_add_counter_table.sql
    ├── alter_tracking_events.sql
    ├── seed_products.sql
    ├── load_routing.ctl                       ← SQL*Loader for 417K rows
    └── ROUTING_LOAD_README.txt
```

---

## ✅ Completed This Session (Everything)

### Backend
| # | What | File | Detail |
|---|------|------|--------|
| 1 | Label auto-routing | `RoutingService.java` | 3-level fallback: exact→DPD→country |
| 2 | Label auto-product | `ProductService.java` | weight≤3000g→328, >3000g→327 |
| 3 | ZIP normalizer | `ZipCodeNormalizer.java` | GB/IE/JE/NL/PT special rules |
| 4 | GEODATA `#DEF` fix | `GeoDataFileBuilder.java` | was `#DEFINITION` |
| 5 | VERSION `3.32` fix | `application.yml` + config | was `03.32` |
| 6 | 133 event codes | `TrackingFileParser.java` | D/R/N/P correct (were DL/RT/ND) |
| 7 | CHR_PRODUCTS table | `Product.java` + repo + service | from PRODUIT.CSV |
| 8 | CHR_ROUTING table | `RoutingEntry.java` + repo + service | 417K rows via sqlldr |
| 9 | PUT /shipment/{id}/sender | `ShipmentController.java` | edit sender address |
| 10 | PUT /shipment/{id}/receiver | `ShipmentController.java` | edit receiver address |
| 11 | GET /admin/products | `AdminController.java` | CHR_PRODUCTS viewer |
| 12 | GET /admin/routing/lookup | `AdminController.java` | routing lookup |
| 13 | getDetail() full nested | `ShipmentService.java` | sender+receiver+parcels+customs+lines |
| 14 | findByIdWithAllDetails() | `ShipmentRepository.java` | full JOIN fetch |
| 15 | ShipmentListDto `id` | `ShipmentListDto.java` | renamed from `shipmentId` |
| 16 | ShipmentCreateResponse expand | `ShipmentCreateResponse.java` | nested DTOs |
| 17 | SenderRepository | `SenderRepository.java` | NEW |
| 18 | ReceiverRepository | `ReceiverRepository.java` | NEW |
| 19 | SFTP → JSch | `SftpService.java` | was plain FTP |
| 20 | TEST mode SFTP | `SftpService.java` | /TEST folder |
| 21 | 300 DPI validation | `InvoiceValidator.java` | PDFBox |
| 22 | Legal disclaimer text | `LabelPdfGenerator.java` | official "subjet" |
| 23 | LabelRequest simplified | `LabelRequest.java` | 4 fields (was 10+) |
| 24 | MPSWEIGHT decagram | `GeoDataFileBuilder.java` | grams÷10 |
| 25 | CHR_PARCEL_COUNTER | `ParcelIdGenerator.java` | DB-persisted counter |

### Frontend
| # | What | File |
|---|------|------|
| 26 | ShipmentListPage | multi-sort, multi-filter, search, bulk, CSV export, pagination |
| 27 | ShipmentDetailPage | workflow bar, edit modals, full nested data |
| 28 | ProductsPage | CHR_PRODUCTS with sort+filter |
| 29 | RoutingLookupPage | live routing lookup with label Zone 4 preview |
| 30 | LabelPage simplified | removed 10 manual fields |
| 31 | TrackingPage fix | event codes D/N/P/R + legend corrected |
| 32 | PreAlertPage fix | version "3.32" shown |
| 33 | App.jsx | 10 pages, 3 nav groups |

### Docs (both EN + BN)
| # | Document |
|---|----------|
| 34 | `01_ASSUMPTION_SHEET.md` — 18 assumptions, 5 critical |
| 35 | `02_BRD.md` — Business requirements |
| 36 | `03_FRD.md` — Feature list + validation rules |
| 37 | `04_API_CONTRACT.md` — 29 endpoints |
| 38 | `05_DB_SCHEMA.md` — 13 tables |
| 39 | `06_UAT_PLAN.md` — 56 test cases |

---

## ⏳ Pending (Before Go-Live)

| # | Task | Priority | Who |
|---|------|----------|-----|
| 1 | **Confirm SDEPOT/SYSDEPOT/HARDWARE with Chronopost** | 🔴 Critical | Chronopost technical team |
| 2 | **Run DB setup scripts in order** | 🔴 Critical | Dev/DBA |
| 3 | **sqlldr 417K routing rows** | 🔴 Critical | Dev/DBA |
| 4 | Get real SFTP credentials from Chronopost | 🔴 Critical | Chronopost |
| 5 | Set `strict-host-key-checking: true` in production | 🟡 Medium | Dev |
| 6 | LORRY field enforcement for RO (road) linehaul | 🟡 Medium | Dev |
| 7 | sortCode `"MAR"` — confirm or get from routing | 🟡 Medium | Chronopost |
| 8 | Run UAT with Chronopost TEST environment | 🟡 Medium | Both |
| 9 | Add login/authentication | 🟢 Enhancement | Dev |

---

## 🐛 Known Issues / Assumptions Not Yet Confirmed

### 🔴 Critical — Will cause GEODATA rejection
```yaml
# application.yml — THESE ARE PLACEHOLDER VALUES
chronopost:
  depot:
    sender: "0029999"       # ← WRONG? Must confirm with Chronopost
    receiver: "0020456"     # ← WRONG? Must confirm with Chronopost

# GeoDataFileBuilder.java line 276
HARDWARE = "K"              # ← Unknown. No doc found. Confirm with Chronopost.
MPSCOMP  = "0"              # ← Assumed. Spec unclear.
```

### 🟡 Medium — Business logic unconfirmed
```java
// ProductService.java
private static final long SMALL_MAX_GRAMS = 3000L; // Is 3000g threshold correct?

// LabelService.java
.sortCode("MAR")            // Hardcoded. Should come from routing file?
.parcelTag("")              // Always empty. When is tag required?
```

---

## 💻 Key Code Snippets

### LabelRequest — only 4 fields needed now
```java
// label/dto/LabelRequest.java
// productSuffix + serviceCode  → auto from CHR_PRODUCTS (by parcel weight)
// countryNumericalCode         → auto from CHR_ROUTING
// origSort/agencyCode/distrib  → auto from CHR_ROUTING
public class LabelRequest {
    @NotNull  private Long   shipmentId;
    @NotNull  private Long   parcelId;
    @NotBlank private String customerPrefix;    // "HY712" — 5 chars
    @NotBlank private String geopostSenderId;   // "0407112" — 7 chars
              private String customsClearanceCode; // "HY-DP" or "HY-NP"
}
```

### RoutingService — 3-level fallback
```java
// routing/RoutingService.java
public RoutingInfo resolve(String serviceType, String countryCode, String rawZip) {
    // 1. Exact service match (e.g. "DPD", "BE", "1000")
    var entry = repo.findByServiceAndCountryAndZip(serviceType, countryCode, lookupZip);
    // 2. DPD fallback
    if (entry.isEmpty()) entry = repo.findDpdByCountryAndZip(countryCode, lookupZip);
    // 3. Country-level fallback
    if (entry.isEmpty()) entry = repo.findFirstByCountry(countryCode);
    // 4. Hardcoded defaults if DB empty
}
```

### ZipCodeNormalizer — special country rules
```java
// label/generator/ZipCodeNormalizer.java
return switch (cc) {
    case "GB" -> normalizeGB(zip);      // remote→03333, other→01111
    case "IE" -> "0000000";             // always 00000
    case "JE" -> "0033333";             // always 03333
    case "PT" -> normalizePT(zip);      // "1849-003" → "1849003"
    case "NL" -> normalizeNL(zip);      // "1000 AC"  → "1000AC"
    default   -> normalizeDefault(zip); // left-pad with zeros to 7
};
```

### TrackingFileParser — correct event codes
```java
// tracking/parser/TrackingFileParser.java
// IMPORTANT: Real codes are single letters — NOT DL/RT/ND
case "D"   -> "Delivered";              // FIX: was "DL"
case "N"   -> "Shipment not delivered"; // FIX: was "ND"
case "P"   -> "Delivery failed";        // FIX: was missing entirely
case "R"   -> "Returned to sender";     // FIX: was "RT"

// Status mapping
case "D", "D1", "D6", "D7", "RG", "RI", "U" -> "DELIVERED"
case "N", "P", "AT", "HO", "PA", "CO", ...   -> "FAILED"
case "R", "RT"                               -> "RETURNED"
```

### GEODATA — critical fixed values
```java
// prealert/builder/GeoDataFileBuilder.java
sb.append("#DEF").append(SEP)        // ✅ FIXED: was "#DEFINITION"
sb.append("3.32")                    // ✅ FIXED: was "03.32"
sb.append("K").append(SEP)          // ⚠️ ASSUMED: HARDWARE type
sb.append("0").append(SEP)          // ⚠️ ASSUMED: MPSCOMP
// MPSWEIGHT = weight grams ÷ 10   // ✅ decagram conversion correct
```

### DB Setup — run in this exact order
```bash
sqlplus CHRONOPOST/chronopost@XE @schema.sql
sqlplus CHRONOPOST/chronopost@XE @alter_add_routing_table.sql
sqlplus CHRONOPOST/chronopost@XE @alter_add_counter_table.sql
sqlplus CHRONOPOST/chronopost@XE @alter_tracking_events.sql
sqlplus CHRONOPOST/chronopost@XE @seed_products.sql
sqlldr CHRONOPOST/chronopost@XE \
       control=load_routing.ctl \
       data=ROUCHR_20260302_B3_avecDPD_ssRDV_ssALASKA.csv \
       log=load_routing.log
```

---

## 🗄️ DB Tables (13 total)

| Table | Purpose | Rows |
|-------|---------|------|
| CHR_SHIPMENTS | Core shipment | 1 per shipment |
| CHR_SENDERS | Sender address | 1 per shipment |
| CHR_RECEIVERS | Receiver address | 1 per shipment |
| CHR_PARCELS | Individual parcels | 1–N per shipment |
| CHR_CUSTOMS | Customs declaration | 1 per shipment |
| CHR_INVOICE_LINES | Invoice items | 1–N per shipment |
| CHR_TRACKING_EVENTS | EDIPOD events | N per parcel |
| CHR_PREALERT_FILES | GEODATA records | 1 per shipment |
| CHR_INVOICE_FILES | Invoice PDFs | N per shipment |
| CHR_PRODUCTS | PRODUIT.CSV data | ~150 rows |
| CHR_ROUTING | ROUCHR routing | **417,612 rows** |
| CHR_PARCEL_COUNTER | ID counter | 1 per prefix |

---

## 🌐 All 29 API Endpoints

```
Shipment:   POST /create  GET /list  GET /{id}
            PUT /{id}/sender  PUT /{id}/receiver
Label:      POST /generate  GET /preview/{id}  GET /download/{id}
PreAlert:   POST /generate/{id}  POST /send/{id}  GET /status/{id}
Invoice:    POST /upload  POST /fetch  GET /all  GET /view/{id}
            GET /download/{id}  POST /retry/{id}  POST/GET /test-mode
Tracking:   POST /poll  GET /dashboard  GET /{parcelNumber}
            GET /shipment/{id}  GET /search
Admin:      GET /admin/products  GET /admin/routing/lookup
```

---

## 🎯 Next Steps (In Order)

1. **Send `docs/en/01_ASSUMPTION_SHEET.md` to Chronopost technical team**  
   → Get SDEPOT, SYSDEPOT, HARDWARE confirmed  
   → Get real SFTP credentials

2. **Run DB setup scripts** (order matters — see above)

3. **Update `application.yml`** with real values:
   ```yaml
   chronopost:
     depot:
       sender: "[REAL SDEPOT]"
       receiver: "[REAL SYSDEPOT]"
     sftp:
       host: "[REAL SFTP HOST]"
       username: "[REAL USERNAME]"
   ```

4. **Enable TEST mode** → run UAT from `docs/en/06_UAT_PLAN.md`

5. **After UAT passes** → switch SFTP TEST mode off → go live

---

## 🚀 Prompt to Resume Next Chat

> "I'm working on a Chronopost International Integration app.  
> Stack: Spring Boot 3.2 + React 18 + Oracle 11g.  
> The app is complete — 29 API endpoints, 13 DB tables, 10 frontend pages, full docs in EN+BN.  
>
> Pending items:  
> 1. Waiting for Chronopost to confirm SDEPOT/SYSDEPOT/HARDWARE values (placeholders: 0029999/0020456/K)  
> 2. DB setup scripts not yet run (routing table needs sqlldr for 417K rows)  
> 3. SFTP credentials not received yet  
>
> Known assumptions to fix once confirmed:  
> - HARDWARE = 'K' in GeoDataFileBuilder.java  
> - MPSCOMP = '0' in GeoDataFileBuilder.java  
> - sortCode = 'MAR' hardcoded in LabelService.java  
>
> Please [describe what you want to do next]."
