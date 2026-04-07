# Database Schema
**Project:** Chronopost International Integration  
**Database:** Oracle 11g  
**Schema:** CHRONOPOST  
**Version:** 1.0 | **Date:** 2026-04-05

---

## Setup Order

```sql
-- Run in this order:
1. schema.sql                        -- All tables
2. alter_add_routing_table.sql       -- CHR_ROUTING
3. alter_add_counter_table.sql       -- CHR_PARCEL_COUNTER
4. alter_tracking_events.sql         -- 7 new columns on CHR_TRACKING_EVENTS
5. seed_products.sql                 -- CHR_PRODUCTS data
6. sqlldr ... load_routing.ctl       -- CHR_ROUTING 417K rows
```

---

## Tables

### CHR_SHIPMENTS — Core shipment record
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | Auto from CHR_SEQ |
| MPS_ID | VARCHAR2(20) | Multi-parcel shipment ID |
| STATUS | VARCHAR2(20) | CREATED / LABEL_READY / PREALERT_SENT / INVOICE_SENT / IN_TRANSIT / DELIVERED / FAILED / RETURNED |
| LINEHAUL | VARCHAR2(2) | AI=Air, RO=Road, SE=Sea |
| MPSCOUNT | NUMBER | Number of parcels |
| MPSWEIGHT | NUMBER | Total weight in **decagrams** |
| CREATED_AT | TIMESTAMP | Auto set |

### CHR_SENDERS — Sender/shipper address
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| SHIPMENT_ID | NUMBER FK | → CHR_SHIPMENTS |
| ACCOUNT_NO | VARCHAR2(20) | Chronopost account number |
| COMPANY_NAME | VARCHAR2(35) | |
| NAME1 | VARCHAR2(35) | Contact name |
| STREET | VARCHAR2(35) NOT NULL | |
| ZIPCODE | VARCHAR2(9) NOT NULL | |
| TOWN | VARCHAR2(35) NOT NULL | |
| COUNTRY_CODE | VARCHAR2(3) NOT NULL | ISO 3166-2 |
| PHONE | VARCHAR2(30) | |
| EMAIL | VARCHAR2(100) | |
| BUSINESS_TYPE | VARCHAR2(1) | B=Business, P=Private |

### CHR_RECEIVERS — Receiver/consignee address
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| SHIPMENT_ID | NUMBER FK | → CHR_SHIPMENTS |
| NAME1 | VARCHAR2(35) | |
| COMPANY_NAME | VARCHAR2(35) | |
| STREET | VARCHAR2(35) NOT NULL | |
| ZIPCODE | VARCHAR2(9) NOT NULL | |
| TOWN | VARCHAR2(35) NOT NULL | |
| COUNTRY_CODE | VARCHAR2(3) NOT NULL | |
| DOOR_CODE | VARCHAR2(8) | Building code |
| PHONE1 | VARCHAR2(30) | |
| EMAIL | VARCHAR2(100) | |
| BUSINESS_TYPE | VARCHAR2(1) | |
| PUDO_ID | VARCHAR2(20) | Pickup point ID |

### CHR_PARCELS — Individual parcel in a shipment
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| SHIPMENT_ID | NUMBER FK | |
| PARCEL_NUMBER | VARCHAR2(20) | e.g. `04561250000010` |
| GEOPOST_TRACK_ID | VARCHAR2(20) | 15-char Geopost tracking ID |
| PARCEL_RANK | NUMBER | Position in multi-parcel shipment |
| DECLARED_WEIGHT | NUMBER | Grams |
| DIMENSION | VARCHAR2(9) | LLLWWWHHHcm |
| CONTENT | VARCHAR2(200) | Item description |
| LABEL_STATUS | VARCHAR2(20) | PENDING / GENERATED / PRINTED |
| LABEL_PATH | VARCHAR2(500) | File path to PDF |
| ROUTING_BARCODE | VARCHAR2(100) | Full routing barcode string |
| OWNER_BU | VARCHAR2(3) | Always `CHR` |
| HAZLQ | NUMBER | Hazardous goods flag |

### CHR_CUSTOMS — Customs declaration per shipment
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| SHIPMENT_ID | NUMBER FK | |
| PARCEL_TYPE | VARCHAR2(1) | P=Non-document, D=Document |
| CUSTOMS_AMOUNT | NUMBER(10,2) | Declared customs value |
| CURRENCY | VARCHAR2(3) | EUR, GBP, USD... |
| INCOTERMS | VARCHAR2(2) | 01=DAP, 07=DAP... |
| CLEARANCE_STATUS | VARCHAR2(1) | N/F/E/I |
| REASON_FOR_EXPORT | VARCHAR2(2) | 01=Sale, 02=Return, 03=Gift |
| HIGH_LOW_VALUE | VARCHAR2(1) | M=Low, L=High |
| PREALERT_STATUS | VARCHAR2(5) | S04 default |

### CHR_INVOICE_LINES — Line items on customs invoice
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| SHIPMENT_ID | NUMBER FK | |
| POSITION | NUMBER | Line number |
| QUANTITY | NUMBER | Item count |
| CONTENT | VARCHAR2(200) | Item description |
| AMOUNT | NUMBER(10,2) | Line total value |
| COUNTRY_ORIGIN | VARCHAR2(3) | ISO 3166-2 |
| NET_WEIGHT | NUMBER | Grams |
| GROSS_WEIGHT | NUMBER | Grams |
| HS_CODE_RECV | VARCHAR2(10) | RCTARIF — import HS code |
| HS_CODE_SEND | VARCHAR2(10) | SCTARIF — export HS code |

### CHR_TRACKING_EVENTS — Events from EDIPOD feedback file
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| PARCEL_ID | NUMBER FK | → CHR_PARCELS |
| CHRONOPOST_TRACK_NO | VARCHAR2(20) | |
| EVENT_CODE | VARCHAR2(5) | Official Chronopost code (D/N/P/R...) |
| EVENT_DESCRIPTION | VARCHAR2(200) | |
| EVENT_DATE | DATE | |
| EVENT_TIME | VARCHAR2(6) | HHMMSS |
| REASON_CODE | VARCHAR2(10) | |
| DELIVERY_INSTRUCTION | VARCHAR2(20) | |
| PICKUP_PUDO_CODE | VARCHAR2(20) | ← new col |
| DISPOSAL_PLACE | VARCHAR2(50) | ← new col |
| NEW_DELIVERY_DATE | DATE | ← new col |
| LENGTH_CM | NUMBER | ← new col |
| WIDTH_CM | NUMBER | ← new col |
| HEIGHT_CM | NUMBER | ← new col |
| POD_IMAGE_URL | VARCHAR2(500) | ← new col |
| CREATED_AT | TIMESTAMP | |

### CHR_PREALERT_FILES — GEODATA file records
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| SHIPMENT_ID | NUMBER FK | |
| FILE_NAME | VARCHAR2(200) | |
| FILE_CONTENT | CLOB | Full file text |
| FTP_STATUS | VARCHAR2(20) | PENDING/SENT/FAILED |
| SENT_AT | TIMESTAMP | |
| ERROR_MESSAGE | VARCHAR2(500) | |

### CHR_INVOICE_FILES — Invoice PDF records
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| SHIPMENT_ID | NUMBER FK | |
| FILE_NAME | VARCHAR2(200) | INV_EXP_account_parcelId.PDF |
| FILE_DATA | BLOB | PDF bytes |
| FILE_SIZE | NUMBER | Bytes |
| DIRECTION | VARCHAR2(3) | OUT=upload, IN=download |
| FTP_STATUS | VARCHAR2(20) | PENDING/SENT/RECEIVED/FAILED |
| CREATED_AT | TIMESTAMP | |

### CHR_PRODUCTS — Service product catalog (from PRODUIT.CSV)
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| CODE_GEOPOST | NUMBER(4) NOT NULL | Service code (327/328/101...) |
| SERVICE_MARK | VARCHAR2(2) | X=small, E=express, R=relay |
| SERVICE_TEXT | VARCHAR2(20) | D-B2C, IE2... |
| DESCRIPTION | VARCHAR2(35) | |
| NAT_INTER | VARCHAR2(1) | N=National, I=International |
| JOUR | VARCHAR2(2) | SE/VE/DI |
| SUFFIXE | VARCHAR2(4) | Parcel ID suffix (JF, EE, FR...) |
| LIBEL25 | VARCHAR2(25) | Label service name (COM INT) |
| LIBELLE_FA | VARCHAR2(35) | Full service label |
| IS_PRIMARY | VARCHAR2(1) | P=Primary, O=Other |

### CHR_ROUTING — Routing table (from ROUCHR file — 417K rows)
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| SERVICE_TYPE | VARCHAR2(6) NOT NULL | DPD, 12H, INT, IPM... |
| COUNTRY_CODE | VARCHAR2(3) NOT NULL | FR, BE, GB... |
| ZIP_FROM | VARCHAR2(10) NOT NULL | Range start |
| ZIP_TO | VARCHAR2(10) NOT NULL | Range end |
| ORIG_SORT | VARCHAR2(10) | Label Zone 4 LEFT |
| AGENCY_CODE | VARCHAR2(10) | Label Zone 4 CENTER |
| DISTRIB_SORT | VARCHAR2(10) | Label Zone 4 RIGHT |
| ROUTING_ZONE | VARCHAR2(10) | Col 8 |
| NUM_COUNTRY | VARCHAR2(5) | 3-digit numerical country code |
| DELIVERY_PT | VARCHAR2(10) | Col 10 |
| FILE_VERSION | VARCHAR2(20) | e.g. `20260302_B3` |

### CHR_PARCEL_COUNTER — Persistent parcel ID counter
| Column | Type | Notes |
|--------|------|-------|
| ID | NUMBER PK | |
| PREFIX | VARCHAR2(5) NOT NULL | Customer prefix (HY712) |
| COUNTER | NUMBER NOT NULL | Last used counter value |
| UPDATED_AT | TIMESTAMP | |

---

## Indexes

```sql
IDX_ROUT_SVC_CC  ON CHR_ROUTING(SERVICE_TYPE, COUNTRY_CODE)
IDX_ROUT_LOOKUP  ON CHR_ROUTING(SERVICE_TYPE, COUNTRY_CODE, ZIP_FROM, ZIP_TO)
IDX_PROD_CODE    ON CHR_PRODUCTS(CODE_GEOPOST)
IDX_PROD_NAT     ON CHR_PRODUCTS(NAT_INTER)
```

---

## Sequence

```sql
CHR_SEQ -- Used as PK generator for all tables (allocationSize=1)
```

---

*Source: `schema.sql`, all `@Entity` Java classes*  
*Last updated: 2026-04-05*
