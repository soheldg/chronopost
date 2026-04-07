# Chronopost Integration App — সম্পূর্ণ Flow (বাংলা)

## 🌐 System Overview

```text
KBT Express (Bangladesh)
        ↓ Internet
   এই App (Browser)
        ↓ REST API
Spring Boot Backend
        ↓
Oracle 11g Database
        ↓ SFTP (port 22)
Chronopost Server (France)
```

---

## ধাপ ১ — নতুন Shipment তৈরি

**Page:** New Shipment → ৫-ধাপের wizard

### ধাপ 1: Sender info
- KBT Express-এর account no, ঠিকানা, phone, VAT no

### ধাপ 2: Receiver info
- ইউরোপের customer-এর নাম, ঠিকানা, ZIP, দেশ

### ধাপ 3: Parcel info
- কয়টা parcel, প্রতিটার ওজন (gram), মাত্রা (cm)
- Transport mode: `AI (Air)` / `RO (Road)` / `SE (Sea)`

### ধাপ 4: Customs info
- কী পণ্য, কত টাকার, HS code, origin country
- Clearance type, incoterms

### ধাপ 5: Review + Submit
- সব দেখো, Create করো

### Backend-এ কী হয়
`POST /api/shipment/create`

- `CHR_SHIPMENTS` table-এ row তৈরি
- `CHR_SENDERS` row তৈরি
- `CHR_RECEIVERS` row তৈরি
- `CHR_PARCELS` row তৈরি (`PENDING-1`, `PENDING-2`...)
- `CHR_CUSTOMS` row তৈরি
- `CHR_INVOICE_LINES` rows তৈরি
- Shipment ID + MPS ID return করে
- Status: `CREATED`

---

## ধাপ ২ — Shipment List দেখা + ভুল ঠিক করা

**Page:** All Shipments

- সব shipment table-এ দেখায়
- Sort: যেকোনো column-এ click
- Filter: Status, Transport, Country, Date, Weight
- Search: ID, MPSID, sender, receiver নাম দিয়ে
- CSV export করা যায়
- **View** button → Shipment Detail page-এ যায়
- **🏷️** button → Label page-এ যায়
- **📡** button → PreAlert page-এ যায়

### Shipment Detail page-এ
- Workflow progress bar দেখায়:

`Created → Label → Pre-Alert → Invoice → In Transit → Delivered`

### Sender section
- **Edit** button
- ঠিকানা ভুল হলে ঠিক করা যায়
- `PUT /api/shipment/{id}/sender`

### Receiver section
- **Edit** button
- ZIP, ঠিকানা ঠিক করা যায়
- `PUT /api/shipment/{id}/receiver`

### Other sections
- Parcels table: সব parcel দেখায়
- Invoice lines table: সব item দেখায়

---

## ধাপ ৩ — Label Generate করা

**Page:** Labels

> ⚠️ এই ধাপে আসার আগে অবশ্যই DB-তে Routing table (417K rows) এবং Products table load থাকতে হবে।

- Shipment ID দাও (বা List থেকে 🏷️ click করলে auto-fill)
- Parcel dropdown auto-load হয় (weight সহ)
- Parcel select করো
- Customer Prefix (Chronopost দেয়): `HY712`
- Geopost Sender ID (Chronopost দেয়): `0407112`
- Customs Clearance: `HY-DP` (duty paid) বা `HY-NP`
- **Generate Label** click

### Backend-এ কী হয়
`POST /api/label/generate`

#### 1. Parcel weight দেখে
- `≤ 3000g` → `CHR_PRODUCTS` থেকে code `328` (SMALL), suffix `JF`
- `> 3000g` → `CHR_PRODUCTS` থেকে code `327` (NORMAL), suffix `JF`

#### 2. Parcel ID তৈরি
Format:

```text
[HY712] [000001] [JF] = HY712000001JF
```

- Counter DB-তে persist করা (`CHR_PARCEL_COUNTER`)

#### 3. Tracking ID তৈরি
- `15 chars + ISO 7064 Mod 37/36 checksum`

#### 4. Routing lookup (`CHR_ROUTING`, 417K rows)
Example:

```text
Country=BE, ZIP=1000
origSort = BE1
distribSort = 011C
numCountryCode = 056
```

#### 5. ZIP normalize (special rules)
- `IE` → `0000000`
- `GB remote` → `0033333`
- `GB normal` → `0011111`
- `PT 1849-003` → `1849003`
- `NL 1000 AC` → `1000AC`

#### 6. Routing barcode তৈরি

```text
% + ZIP(7) + TrackingID(14) + ServiceCode(3) + CountryCode(3)
```

#### 7. PDF generate
- **Zone 1:** Legal text (`On delivery, damage or theft must be the subjet of...`)
- **Zone 2:** Sender + Receiver address
- **Zone 3:** Parcel barcode | sortCode | COM INT
- **Zone 4:** OrigSort | BE-Agency | DistribSort
- **Zone 5:** Tracking barcode + D-B2C

#### 8. `CHR_PARCELS` update
- `parcelNumber = HY712000001JF`
- `geopostTrackId = tracking ID`
- `labelStatus = GENERATED`
- `labelPath = PDF file path`
- `routingBarcode = barcode string`

- Status: `LABEL_READY`

---

## ধাপ ৪ — Pre-Alert পাঠানো (GEODATA)

**Page:** Pre-Alert

> ⚠️ Label generate হওয়ার পরেই করতে হবে। Parcel physically পাঠানোর আগে।

- Shipment ID দাও
- **Generate File** click → file preview দেখায়
- **Send to Chronopost** click → SFTP-তে পাঠায়

### Validation চেক
- Sender: account no, ঠিকানা, phone/email, VAT/EORI আছে কিনা
- Receiver: নাম, ঠিকানা আছে কিনা
- Parcel: parcel number `PENDING` নয় কিনা (label generate হয়েছে কিনা)
- Customs: clearance address, prealertStatus আছে কিনা
- Weight: parcel ওজনের sum = `MPSWEIGHT` কিনা

### Backend-এ কী হয়
`POST /api/prealert/generate/{shipmentId}`

GEODATA CONSO file তৈরি হয় (`ISO-8859-1 encoding`):

```text
#FILE;GEODATA_CONSO_14576904_CHR_D20260304T111500_000001;
#ENCODING;ISO-8859-1;
#VERSION;3.32;
#DEF;GEODATA:HEADER;VERSION;CLASSIFICATION;;
#DEF;GEODATA:CONSOLIDATION;NUMORDER;MAWB;...
#DEF;GEODATA:SHIPMENT;...
#DEF;GEODATA:SENDER;...
#DEF;GEODATA:RECEIVER;...
#DEF;GEODATA:INTER;...
#DEF;GEODATA:INTERINVOICELINE;...
#DEF;GEODATA:PARCEL;...
HEADER;3.32;CONSO;
CONSOLIDATION;2;...;AI;...;0029999;0020456;INS;...
SHIPMENT;3;04561250000010;...;1440;...;K;...
SENDER;4;14576904;...;KBT EXPRESS;...;BD;...
RECEIVER;4;...;JOHN DOE;...;FR;75001;PARIS;...
INTER;4;P;109.09;EUR;...;E;M;...;INS;
INTERINVOICELINE;5;1;2;Women Cardigan;83.58;...;BD;480;...
PARCEL;4;HY712000001JF;4;;...;327;...;035025050;1440;...
#END
```

`POST /api/prealert/send/{shipmentId}`

- SFTP connect (`port 22`, key-based auth)
- TEST mode: `/TEST` folder
- Production: `/IN` folder
- File upload
- `CHR_PREALERT_FILES` table-এ save
- Shipment status → `PREALERT_SENT`

---

## ধাপ ৫ — Invoice আপলোড

**Page:** Invoices

- PDF invoice তৈরি করো (`≥300 DPI` অবশ্যই)
- Shipment ID দাও
- Type: `EXP (export)` বা `IMP (import)`
- PDF drag & drop বা browse করো
- **Upload to Chronopost FTP** click

### Backend-এ কী হয়
`POST /api/invoice/upload`

- PDF validate: `≥300 DPI` check (`PDFBox`)
- File rename:
  - Export: `INV_EXP_14576904_HY712000001JF.PDF`
  - Import: `INV_IMP_14576904_HY712000001JF.PDF`
- SFTP `/IN` folder-এ upload
- `CHR_INVOICE_FILES` table-এ save
- Shipment status → `INVOICE_SENT`

### Chronopost-এর কাছ থেকে Invoice নামানো
- **Fetch Now** button অথবা auto (প্রতি 30 মিনিটে)
- SFTP `/OUT` folder চেক করে
- নতুন file থাকলে download করে DB-তে save
- List-এ দেখা যায়, preview/download করা যায়

---

## ধাপ ৬ — Tracking

**Page:** Tracking

- **Poll FTP Now** button
  - Chronopost FTP থেকে EDIPOD tracking file নামায়
  - প্রতি 30 মিনিটে auto-poll হয়

- Search: parcel number বা tracking ID দিয়ে
- Timeline দেখায় সব event:
  - `PC` → Picked up by driver
  - `TI` → Arrived at hub
  - `TS` → In transit
  - `TA` → Out for delivery
  - `D` → Delivered ✓
  - `P` → Delivery failed (recipient absent)
  - `N` → Not delivered
  - `R` → Returned to sender

### Backend-এ কী হয়
`POST /api/tracking/poll`

- FTP থেকে EDIPOD file নামায়
- প্রতিটা line parse করে (`51 field`):
  - `F35` = event code (`D`, `N`, `P`, `R`...)
  - `F37` = event date
  - `F38` = event time
  - `F44` = pickup PUDO code
  - `F43` = new delivery date
  - `F48-50` = dimensions (cm)
  - `F51` = POD image URL

- `CHR_TRACKING_EVENTS`-এ save
- Shipment status update:
  - `D/D1/D6/RG/RI` → `DELIVERED`
  - `N/P/AT/HO` → `FAILED`
  - `R` → `RETURNED`
  - `SM/TI/TO/TS` → `IN_TRANSIT`

---

## Reference Data Pages

### Products Page (`/admin/products`)
- `CHR_PRODUCTS` table দেখায় (`PRODUIT.CSV` থেকে loaded)
- Sort, filter, search করা যায়
- কোন service code কোন suffix/label name দেখায়

### Routing Lookup Page (`/admin/routing/lookup`)
- Country + ZIP দাও
- `CHR_ROUTING` (`417K rows`) থেকে lookup
- Result:
  - `origSort = BE1` (label left)
  - `distribSort = 011C` (label right)
  - `numCountryCode = 056` (barcode field)
  - `normalizedZip = 0010000` (7-char padded)
- Label Zone 4 preview দেখায়

---

## Status Flow Summary

```text
CREATED
   ↓ (label generate)
LABEL_READY
   ↓ (pre-alert sent)
PREALERT_SENT
   ↓ (invoice uploaded)
INVOICE_SENT
   ↓ (tracking: SM/TI/TS events)
IN_TRANSIT
   ↓
DELIVERED ✅  বা  FAILED ❌  বা  RETURNED ↩
```

---

Source text used: fileciteturn0file0L1-L171
