# Label Generation — High Level Flow

## Flow Overview

**User:** Shipment ID দেয়  
↓  
**LabelPage:** API call করে shipment detail আনে  
↓  
**Parcel dropdown** auto-fill হয়  
- weight দেখায়  
- rank দেখায়  

↓  
**User:** Parcel select করে  
- Customer Prefix দেয়  
- Sender ID দেয়  

↓  
**"Generate Label"** click করে  

---

## Backend Flow (`LabelService`)

### 1. Shipment + Parcel load
DB থেকে Shipment এবং Parcel data load হয়।

### 2. Product lookup by weight
`CHR_PRODUCTS` table থেকে weight অনুযায়ী product code নেওয়া হয়:

- **≤ 3000g** → code `328`, suffix `"JF"`
- **> 3000g** → code `327`, suffix `"JF"`

### 3. Parcel ID generate
Example:

`HY712 + 000001 + JF = HY712000001JF`

- Counter DB-তে persist হয়
- তাই restart হলেও sequence হারায় না

### 4. Tracking ID generate
- 15 characters
- ISO 7064 checksum সহ generate হয়

### 5. Routing lookup
Receiver country + ZIP code দিয়ে `CHR_ROUTING` lookup করা হয়  
(প্রায় **417K rows** data)

এখান থেকে পাওয়া যায়:
- `origSort`
- `distribSort`
- `numCountryCode`

### 6. ZIP normalize
Country-specific rules apply হয়, যেমন:
- GB
- IE
- NL
- PT

তারপর ZIP code normalize করে **7-character padded ZIP** বানানো হয়।

### 7. Routing barcode build
Routing barcode format:

`% + zip(7) + trackId(14) + svc(3) + country(3)`

### 8. PDF generate (`iText`)
Label PDF generate হয় নিচের zone অনুযায়ী:

- **Zone 1:** Legal text
- **Zone 2:** Sender + Receiver
- **Zone 3:** Parcel barcode | sortCode | COM INT
- **Zone 4:** origSort | BE-agency | distribSort
- **Zone 5:** Routing barcode + D-B2C

### 9. DB update (`CHR_PARCELS`)
শেষে DB update হয়:

- `parcelNumber = "HY712000001JF"`
- `labelStatus = GENERATED`
- `routingBarcode = "..."`

---

## Response Flow

Backend response পাঠায় frontend-এ।

Frontend এ user পায়:
- **Preview**
- **Download button**

Shipment status update হয়ে যায়:

`LABEL_READY`

---

## Short Summary

**Shipment ID → parcel weight → product code → routing lookup → barcode build → PDF**

---

## Visual Flow

```text
User: Shipment ID দেয়
        ↓
LabelPage: API call করে shipment detail
        ↓
Parcel dropdown auto-fill হয়
(weight, rank দেখায়)
        ↓
User: Parcel select করে
Customer Prefix + Sender ID দেয়
        ↓
"Generate Label" click
        ↓
═══════════════════════════════
     BACKEND (LabelService)
═══════════════════════════════
        ↓
1. Shipment + Parcel DB থেকে load
        ↓
2. Weight দেখে → CHR_PRODUCTS lookup
   ≤3000g → code 328, suffix "JF"
   >3000g → code 327, suffix "JF"
        ↓
3. Parcel ID generate
   HY712 + 000001 + JF = "HY712000001JF"
   (counter DB-তে persist)
        ↓
4. Tracking ID generate
   15 chars + ISO 7064 checksum
        ↓
5. Receiver country+zip দিয়ে
   CHR_ROUTING lookup (417K rows)
   → origSort, distribSort, numCountryCode
        ↓
6. ZIP normalize (GB/IE/NL/PT rules)
   → 7-char padded zip
        ↓
7. Routing barcode build
   % + zip(7) + trackId(14) + svc(3) + country(3)
        ↓
8. PDF generate (iText)
   Zone 1: Legal text
   Zone 2: Sender + Receiver
   Zone 3: Parcel barcode | sortCode | COM INT
   Zone 4: origSort | BE-agency | distribSort
   Zone 5: Routing barcode + D-B2C
        ↓
9. DB update (CHR_PARCELS):
   parcelNumber = "HY712000001JF"
   labelStatus  = GENERATED
   routingBarcode = "..."
        ↓
═══════════════════════════════
       RESPONSE
═══════════════════════════════
        ↓
Frontend: Preview + Download button
Shipment status → LABEL_READY
