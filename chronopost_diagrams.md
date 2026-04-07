# Chronopost Integration — All Diagrams

---

## 1. Sequence Diagram — Complete Flow

```mermaid
sequenceDiagram
    actor User as 👤 Operator
    participant App as ⚙️ Spring Boot
    participant DB as 🗄️ Oracle DB
    participant SFTP as 📡 Chronopost SFTP
    participant FTP as 📥 Chronopost FTP

    Note over User,FTP: ── PHASE 1: Create Shipment ──

    User->>App: POST /shipment/create
    App->>DB: INSERT SHIPMENTS, SENDERS, RECEIVERS,<br/>PARCELS, CUSTOMS, INVOICE_LINES
    DB-->>App: shipmentId=164
    App-->>User: ✅ Shipment 164 created

    Note over User,FTP: ── PHASE 2: Label Generation ──

    User->>App: POST /label/generate (shipmentId=164, parcelId=201)
    App->>DB: SELECT CHR_PRODUCTS (weight → code 327/328)
    App->>DB: SELECT CHR_ROUTING (country+zip → origSort, distribSort)
    App->>DB: UPDATE CHR_PARCEL_COUNTER (increment)
    App->>App: Build parcelId: HY712000001JF
    App->>App: Build trackingId: 15 chars + ISO7064 checksum
    App->>App: Build routing barcode: %zip(7)+track(14)+svc(3)+cc(3)
    App->>App: Generate PDF (iText, 5 zones)
    App->>DB: UPDATE CHR_PARCELS (parcelNumber, labelStatus=GENERATED)
    App-->>User: ✅ PDF ready → Preview + Download

    Note over User,FTP: ── PHASE 3: Pre-Alert (GEODATA) ──

    User->>App: POST /prealert/send/164
    App->>DB: Load full shipment with all details
    App->>App: Validate (VAT, weight, parcel number)
    App->>App: Build GEODATA CONSO file (ISO-8859-1)
    App->>SFTP: SFTP connect (port 22, key auth)
    App->>SFTP: Upload GEODATA file → /IN folder
    SFTP-->>App: ✅ File accepted
    App->>DB: INSERT CHR_PREALERT_FILES (ftpStatus=SENT)
    App->>DB: UPDATE CHR_SHIPMENTS (status=PREALERT_SENT)
    App-->>User: ✅ Pre-alert sent

    Note over User,FTP: ── PHASE 4: Invoice ──

    User->>App: GET /invoice/generate/164
    App->>DB: Load Shipment + Sender + Receiver + InvoiceLines
    App->>App: Generate PDF (iText, professional invoice)
    App-->>User: PDF preview

    User->>App: POST /invoice/upload (shipmentId=164, type=EXP)
    App->>App: Validate PDF ≥ 300 DPI (PDFBox)
    App->>SFTP: SFTP connect
    App->>SFTP: Upload INV_EXP_14576904_HY712000001JF.PDF → /IN
    SFTP-->>App: ✅ Uploaded
    App->>DB: INSERT CHR_INVOICE_FILES (ftpStatus=SENT)
    App->>DB: UPDATE CHR_SHIPMENTS (status=INVOICE_SENT)
    App-->>User: ✅ Invoice uploaded

    Note over User,FTP: ── PHASE 5 & 6: Tracking (every 30 min) ──

    loop Every 30 minutes
        App->>FTP: FTP connect
        App->>FTP: Check /OUT for EDIPOD files
        FTP-->>App: EDIPOD_HY712..._001.txt
        App->>App: Parse 51 fields × N lines
        App->>App: Map 133 event codes (D=Delivered, N=Not delivered...)
        App->>DB: INSERT CHR_TRACKING_EVENTS
        App->>DB: UPDATE CHR_SHIPMENTS status (IN_TRANSIT / DELIVERED)
    end

    User->>App: GET /tracking/search?q=HY712000001JF
    App->>DB: SELECT CHR_TRACKING_EVENTS
    App-->>User: Timeline (SM→TI→TA→D)
```

---

## 2. State Diagram — Shipment Status

```mermaid
stateDiagram-v2
    [*] --> CREATED : POST /shipment/create

    CREATED --> LABEL_READY : POST /label/generate\n(parcel ID + PDF)

    LABEL_READY --> PREALERT_SENT : POST /prealert/send\n(GEODATA → SFTP /IN)

    PREALERT_SENT --> INVOICE_SENT : POST /invoice/upload\n(PDF → SFTP /IN)

    INVOICE_SENT --> IN_TRANSIT : Tracking event\nSM / TI / TS

    IN_TRANSIT --> DELIVERED : Tracking event D
    IN_TRANSIT --> FAILED : Tracking event N / P
    IN_TRANSIT --> RETURNED : Tracking event R

    DELIVERED --> [*]
    FAILED --> [*]
    RETURNED --> [*]

    note right of PREALERT_SENT
        GEODATA CONSO file
        VERSION 3.32
        MPSWEIGHT in decagram
        Clearance address required
    end note

    note right of IN_TRANSIT
        FTP Pull every 30 min
        133 official event codes
    end note
```

---

## 3. ER Diagram — Database Schema

```mermaid
erDiagram
    CHR_SHIPMENTS {
        NUMBER ID PK
        VARCHAR2 MPSID
        VARCHAR2 STATUS
        VARCHAR2 LINEHAUL
        NUMBER MPSCOUNT
        NUMBER MPSWEIGHT
        TIMESTAMP CREATED_AT
    }

    CHR_SENDERS {
        NUMBER ID PK
        NUMBER SHIPMENT_ID FK
        VARCHAR2 ACCOUNT_NO
        VARCHAR2 COMPANY_NAME
        VARCHAR2 STREET
        VARCHAR2 ZIPCODE
        VARCHAR2 COUNTRY_CODE
        VARCHAR2 VAT_NO
        VARCHAR2 EORI
    }

    CHR_RECEIVERS {
        NUMBER ID PK
        NUMBER SHIPMENT_ID FK
        VARCHAR2 NAME1
        VARCHAR2 STREET
        VARCHAR2 ZIPCODE
        VARCHAR2 COUNTRY_CODE
        VARCHAR2 PHONE1
    }

    CHR_PARCELS {
        NUMBER ID PK
        NUMBER SHIPMENT_ID FK
        VARCHAR2 PARCEL_NUMBER
        VARCHAR2 GEOPOST_TRACK_ID
        NUMBER DECLARED_WEIGHT
        VARCHAR2 LABEL_STATUS
        VARCHAR2 ROUTING_BARCODE
    }

    CHR_CUSTOMS {
        NUMBER ID PK
        NUMBER SHIPMENT_ID FK
        VARCHAR2 PARCEL_TYPE
        NUMBER CUSTOMS_AMOUNT
        VARCHAR2 CURRENCY
        VARCHAR2 CLEARANCE_STATUS
        VARCHAR2 PREALERT_STATUS
        VARCHAR2 C_NAME1
        VARCHAR2 C_COUNTRY_CODE
    }

    CHR_INVOICE_LINES {
        NUMBER ID PK
        NUMBER SHIPMENT_ID FK
        NUMBER QUANTITY
        VARCHAR2 CONTENT
        NUMBER AMOUNT
        VARCHAR2 HS_CODE_RECV
        VARCHAR2 COUNTRY_ORIGIN
    }

    CHR_TRACKING_EVENTS {
        NUMBER ID PK
        NUMBER PARCEL_ID FK
        VARCHAR2 EVENT_CODE
        DATE EVENT_DATE
        VARCHAR2 LOCATION
        VARCHAR2 PICKUP_PUDO_CODE
        DATE NEW_DELIVERY_DATE
    }

    CHR_PREALERT_FILES {
        NUMBER ID PK
        NUMBER SHIPMENT_ID FK
        VARCHAR2 FILE_NAME
        CLOB FILE_CONTENT
        VARCHAR2 FTP_STATUS
        TIMESTAMP SENT_AT
    }

    CHR_INVOICE_FILES {
        NUMBER ID PK
        NUMBER SHIPMENT_ID FK
        VARCHAR2 FILE_NAME
        BLOB FILE_DATA
        VARCHAR2 DIRECTION
        VARCHAR2 FTP_STATUS
    }

    CHR_PRODUCTS {
        NUMBER ID PK
        NUMBER CODE_GEOPOST
        VARCHAR2 SERVICE_TEXT
        VARCHAR2 NAT_INTER
        VARCHAR2 SUFFIXE
        VARCHAR2 LIBEL25
    }

    CHR_ROUTING {
        NUMBER ID PK
        VARCHAR2 SERVICE_TYPE
        VARCHAR2 COUNTRY_CODE
        VARCHAR2 ZIP_FROM
        VARCHAR2 ZIP_TO
        VARCHAR2 ORIG_SORT
        VARCHAR2 DISTRIB_SORT
        VARCHAR2 NUM_COUNTRY
    }

    CHR_PARCEL_COUNTER {
        NUMBER ID PK
        VARCHAR2 PREFIX
        NUMBER COUNTER
    }

    CHR_SHIPMENTS ||--|| CHR_SENDERS : "has"
    CHR_SHIPMENTS ||--|| CHR_RECEIVERS : "has"
    CHR_SHIPMENTS ||--|{ CHR_PARCELS : "has"
    CHR_SHIPMENTS ||--|| CHR_CUSTOMS : "has"
    CHR_SHIPMENTS ||--|{ CHR_INVOICE_LINES : "has"
    CHR_SHIPMENTS ||--o| CHR_PREALERT_FILES : "generates"
    CHR_SHIPMENTS ||--o{ CHR_INVOICE_FILES : "generates"
    CHR_PARCELS ||--o{ CHR_TRACKING_EVENTS : "receives"
```

---

## 4. Flowchart — Label Generation Logic

```mermaid
flowchart TD
    A([Start: POST /label/generate]) --> B[Load Shipment + Parcel from DB]
    B --> C{Parcel weight?}

    C -->|≤ 3000g| D[CHR_PRODUCTS\ncode=328 SMALL\nsuffix=JF]
    C -->|> 3000g| E[CHR_PRODUCTS\ncode=327 NORMAL\nsuffix=JF]

    D --> F[Generate Parcel ID\nPrefix+Counter+Suffix\nHY712 000001 JF]
    E --> F

    F --> G[Generate Tracking ID\n15 chars\nISO 7064 checksum]

    G --> H[CHR_ROUTING lookup\ncountry + ZIP]

    H --> I{Match found?}
    I -->|Exact| J[origSort + distribSort\nnumCountryCode]
    I -->|DPD fallback| K[DPD generic routing]
    I -->|Country only| L[Country-level fallback]
    I -->|Nothing| M[Default 250 France]

    J --> N[ZIP Normalize]
    K --> N
    L --> N
    M --> N

    N --> O{Country?}
    O -->|IE| P[0000000]
    O -->|JE/GG| Q[0033333]
    O -->|GB remote| R[0033333]
    O -->|GB normal| S[0011111]
    O -->|PT| T[Remove dash]
    O -->|NL| U[Remove space]
    O -->|Other| V[Left-pad zeros to 7]

    P --> W[Build Routing Barcode\n% + zip7 + track14 + svc3 + cc3]
    Q --> W
    R --> W
    S --> W
    T --> W
    U --> W
    V --> W

    W --> X[Generate PDF — 5 Zones\nZone1: Legal text\nZone2: Sender + Receiver\nZone3: Parcel barcode\nZone4: Routing info\nZone5: Tracking barcode]

    X --> Y[UPDATE CHR_PARCELS\nparcelNumber · trackId\nlabelStatus=GENERATED]

    Y --> Z([Return PDF to frontend])
```

---

## 5. Flowchart — GEODATA Pre-Alert

```mermaid
flowchart TD
    A([POST /prealert/send/shipmentId]) --> B[Load full Shipment from DB\nSender + Receiver + Parcels\nCustoms + InvoiceLines]

    B --> C{Validate}

    C --> D{Sender VAT or EORI?}
    D -->|Missing| E[❌ Error: SEORI or SVATNO mandatory]
    D -->|OK| F{Parcel number = PENDING?}

    F -->|Yes| G[❌ Error: Label not generated yet]
    F -->|No| H{Weight sum = MPSWEIGHT?}

    H -->|Mismatch| I[❌ Error: Weight mismatch]
    H -->|OK| J{Clearance address?}

    J -->|Missing| K[Auto-fill from Receiver\ncName1, cStreet, cCountryCode]
    J -->|Set| L[Use provided address]

    K --> M[Build GEODATA CONSO file]
    L --> M

    M --> N[#FILE header\n#ENCODING ISO-8859-1\n#VERSION 3.32\n#DEF lines × 8 types]

    N --> O[HEADER record]
    O --> P[CONSOLIDATION record\nSDEPOT · SYSDEPOT · OPCODE=INS]
    P --> Q[SHIPMENT record\nMPSWEIGHT in decagram ÷ 10]
    Q --> R[SENDER record]
    R --> S[RECEIVER record]
    S --> T[INTER record\nPREALERTSTATUS=S04]
    T --> U[INTERINVOICELINE × N items]
    U --> V[PARCEL record × N parcels]
    V --> W[#END]

    W --> X{TEST mode?}
    X -->|Yes| Y[SFTP → /TEST folder]
    X -->|No| Z[SFTP → /IN folder]

    Y --> AA[INSERT CHR_PREALERT_FILES\nftpStatus=SENT]
    Z --> AA

    AA --> AB[UPDATE CHR_SHIPMENTS\nstatus=PREALERT_SENT]
    AB --> AC([✅ Done])
```

---

## 6. Flowchart — Tracking Pull (Scheduler)

```mermaid
flowchart TD
    A([⏰ Scheduler: every 30 min]) --> B[FTP connect to Chronopost]

    B --> C{New EDIPOD files\nin /OUT folder?}
    C -->|No| D([Sleep 30 min])
    C -->|Yes| E[Download EDIPOD file]

    E --> F[Parse each line\n51 semicolon-separated fields]

    F --> G[F1=contractNo\nF2=parcelNumber\nF35=eventCode\nF37=date · F38=time\nF39=location\nF42=disposalPlace\nF43=newDeliveryDate\nF44=pickupPudoCode\nF48-50=dimensions\nF51=podImageUrl]

    G --> H{Event code?}

    H -->|D D1 D6 RG RI| I[Status → DELIVERED ✅]
    H -->|N P AT HO KC| J[Status → FAILED ❌]
    H -->|R RT| K[Status → RETURNED ↩]
    H -->|SM TI TO TS| L[Status → IN_TRANSIT 🚚]
    H -->|Other 120+ codes| M[Status stays current]

    I --> N[INSERT CHR_TRACKING_EVENTS]
    J --> N
    K --> N
    L --> N
    M --> N

    N --> O[UPDATE CHR_SHIPMENTS status]
    O --> P{More lines?}
    P -->|Yes| F
    P -->|No| Q{More files?}
    Q -->|Yes| E
    Q -->|No| D
```

---

## 7. Component Architecture

```mermaid
graph TB
    subgraph Frontend["🖥️ React 18 Frontend"]
        SL[ShipmentListPage]
        SD[ShipmentDetailPage]
        NS[NewShipmentPage]
        LP[LabelPage]
        PA[PreAlertPage]
        IP[InvoicePage]
        TP[TrackingPage]
        PP[ProductsPage]
        RP[RoutingLookupPage]
    end

    subgraph Backend["⚙️ Spring Boot 3.2 Backend"]
        subgraph Controllers
            SC[ShipmentController]
            LC[LabelController]
            PC[PreAlertController]
            IC[InvoiceController]
            TC[TrackingController]
            AC[AdminController]
        end

        subgraph Services
            SS[ShipmentService]
            LS[LabelService]
            PS[PreAlertService]
            IS[InvoiceService]
            TS[TrackingService]
        end

        subgraph Generators
            LG[LabelPdfGenerator]
            IG[CommercialInvoicePdfGenerator]
            GB[GeoDataFileBuilder]
            BC[BarcodeImageGenerator]
            RC[RoutingCalculator]
            ZN[ZipCodeNormalizer]
            TG[TrackingIdGenerator]
            PG[ParcelIdGenerator]
        end

        subgraph External
            SFTP[SftpService\nJSch port 22]
            SCHED[TrackingScheduler\n30 min interval]
            ISCH[InvoiceScheduler\n30 min interval]
        end
    end

    subgraph DB["🗄️ Oracle 11g"]
        T1[(CHR_SHIPMENTS)]
        T2[(CHR_PARCELS)]
        T3[(CHR_ROUTING\n417K rows)]
        T4[(CHR_PRODUCTS)]
        T5[(CHR_TRACKING_EVENTS)]
    end

    subgraph Chronopost["🚚 Chronopost France"]
        SFTP_IN[SFTP /IN\nGEODATA + Invoice]
        FTP_OUT[FTP /OUT\nEDIPOD tracking]
    end

    SL & SD & NS --> SC
    LP --> LC
    PA --> PC
    IP --> IC
    TP --> TC
    PP & RP --> AC

    SC --> SS
    LC --> LS
    PC --> PS
    IC --> IS
    TC --> TS

    LS --> LG & RC & ZN & TG & PG
    IS --> IG
    PS --> GB & SFTP
    TS --> SCHED

    SS & LS & PS & IS & TS --> T1 & T2
    LS --> T3 & T4
    TS --> T5

    SFTP --> SFTP_IN
    SCHED --> FTP_OUT
    ISCH --> SFTP_IN
```

---

## Summary Table

| Diagram | বিষয় |
|---------|------|
| 1. Sequence | কে কাকে কখন call করে |
| 2. State | Shipment কীভাবে status বদলায় |
| 3. ER | DB table-এর সম্পর্ক |
| 4. Flowchart | Label কীভাবে তৈরি হয় |
| 5. Flowchart | GEODATA কীভাবে তৈরি হয় |
| 6. Flowchart | Tracking কীভাবে pull হয় |
| 7. Architecture | সব component-এর সম্পর্ক |

> **Render করতে:** GitHub, GitLab, Obsidian, Notion, বা [mermaid.live](https://mermaid.live)-এ paste করো।
