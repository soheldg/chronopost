# 📦 Chronopost Integration — Complete Flow

**KBT Express (Bangladesh) → Chronopost (France) → Customer (Europe)**

---

## Status Flow

`CREATED` → `LABEL_READY` → `PREALERT_SENT` → `INVOICE_SENT` → `IN_TRANSIT` → `DELIVERED ✓` → `FAILED ✗` → `RETURNED ↩`

---

## Flow Overview

| Phase | Operator | Spring Boot App | Oracle DB | Network | Chronopost (FR) |
|---|---|---|---|---|---|
| **① Create** | **New Shipment**  
5-step wizard | **POST /shipment/create**  
Sender · Receiver · Parcel  
Customs · Invoice Lines | **6 tables INSERT**  
SHIPMENTS  
SENDERS  
RECEIVERS  
PARCELS  
CUSTOMS  
INVOICE_LINES | — | — |
| **② Label** | **Generate Label**  
Select parcel | **POST /label/generate**  
1. Weight → CHR_PRODUCTS → code 327/328  
2. ParcelID: HY712+000001+JF  
3. CHR_ROUTING lookup → origSort  
4. ZIP normalize (GB/IE/NL/PT)  
5. Routing barcode build  
6. iText PDF → 5 zones | **3 tables**  
PRODUCTS (read)  
ROUTING (read)  
PARCEL_COUNTER (update)  
PARCELS (update) | — | — |
| **③ PreAlert** | **Send Pre-Alert**  
Before parcel departs | **POST /prealert/send/{id}**  
1. Validate all fields  
2. Build GEODATA CONSO file  
3. #DEF · VERSION 3.32  
4. MPSWEIGHT in decagram  
5. SFTP connect → `/IN` upload | **PREALERT_FILES**  
fileName, fileContent  
ftpStatus = SENT  
SHIPMENTS status update | **SFTP Push ↑**  
port 22 · key auth  
GEODATA_CONSO_...  
→ `/IN` folder | **Receive GEODATA**  
Validate file  
Parse shipment data  
**Status: OK ✓** |
| **④ Invoice** | **Generate + Upload**  
Invoice page | **GET /invoice/generate/{id}**  
1. DB → Sender + Receiver + Items  
2. iText PDF (professional)  
3. POST /invoice/upload  
4. Validate ≥300 DPI  
5. SFTP → `/IN` upload | **INVOICE_FILES**  
INV_EXP_account_parcelId.PDF  
ftpStatus = SENT  
SHIPMENTS status update | **SFTP Push ↑**  
INV_EXP_14576904_HY712000001JF.PDF  
→ `/IN` folder | **Receive Invoice**  
Customs clearance  
Belgium customs  
**Status: Cleared ✓** |
| **✈️ Transit** | Waiting... | Tracking Scheduler  
runs every 30 min | — | **Physical Journey**  
Dhaka → Dubai → Brussels  
Chronopost places  
EDIPOD file on FTP | **Scan Events**  
SM → Scanned  
TI → Hub arrived  
TA → Out for delivery  
D → Delivered |
| **⑥ Track** | **Tracking Page**  
View timeline  
Search by parcel | **POST /tracking/poll**  
1. FTP connect every 30 min  
2. Check `/OUT` for EDIPOD files  
3. Download new files  
4. Parse 51 fields per line  
5. Map 133 event codes  
6. Update shipment status | **TRACKING_EVENTS**  
eventCode · date · time  
location · reason  
pickupPudoCode  
SHIPMENTS → DELIVERED | **FTP Pull ↓**  
App goes to Chronopost  
Downloads EDIPOD file  
from `/OUT` folder  
**⚠ Chronopost does NOT push** | **FTP /OUT folder**  
EDIPOD_HY712..._001.txt  
Waiting to be pulled |

---

## Phase Details

### ① Create
- Operator creates a new shipment using a **5-step wizard**.
- App calls `POST /shipment/create`.
- Data is inserted into:
  - `SHIPMENTS`
  - `SENDERS`
  - `RECEIVERS`
  - `PARCELS`
  - `CUSTOMS`
  - `INVOICE_LINES`

### ② Label
- Operator clicks **Generate Label**.
- App calls `POST /label/generate`.
- System:
  - checks weight
  - selects Chronopost product code
  - generates Parcel ID
  - reads routing data
  - normalizes ZIP rules
  - builds barcode
  - generates PDF label using iText

### ③ Pre-Alert
- Operator clicks **Send Pre-Alert** before physical dispatch.
- App calls `POST /prealert/send/{id}`.
- System:
  - validates shipment data
  - builds `GEODATA CONSO` file
  - connects to SFTP
  - uploads file to Chronopost `/IN`
- DB stores file info and updates shipment status.

### ④ Invoice
- Operator generates and uploads commercial invoice.
- App:
  - loads sender, receiver, and item data
  - creates PDF invoice
  - validates quality
  - uploads invoice to Chronopost `/IN`
- Chronopost uses it for customs clearance.

### ✈️ Transit
- Shipment travels physically:
  - Dhaka
  - Dubai
  - Brussels
- Chronopost scans parcel movement and creates tracking events.

### ⑥ Track
- Scheduler runs every **30 minutes**.
- App checks Chronopost FTP `/OUT`.
- App downloads EDIPOD tracking files.
- Events are parsed and saved to DB.
- Shipment status is updated automatically.

---

## Important Note

> ⚠️ **Critical before go-live:**  
> `SDEPOT`, `SYSDEPOT`, and `HARDWARE` values must be confirmed with Chronopost technical team.  
> Real SFTP credentials are required.  
> UAT should be run in **TEST mode first** using the SFTP `/TEST` folder.

---

## Invoice Direction

### You send (Export)
- You create the invoice
- You place it in SFTP `/IN`
- Chronopost receives it
- Chronopost uses it for customs clearance

**Example:**  
`INV_EXP_14576904_HY712000001JF.PDF`

### Chronopost sends (Import)
- Belgium customs may create an import invoice
- Chronopost places it in SFTP `/OUT`
- Your app pulls it every 30 minutes

**Example:**  
`INV_IMP_14576904_HY712000001JF.PDF`

### Why does Chronopost send an invoice?
When the parcel reaches Belgium, customs may calculate duty/tax and generate a customs invoice for your records.

### Practical Note
At the early stage of integration, **Import invoice (Chronopost → You)** may not always come.  
This depends on the agreement/contract with Chronopost.

For now, the main focus is:

- **You → Chronopost: GEODATA**
- **You → Chronopost: Export Invoice**
- **Chronopost → You: Tracking via EDIPOD**

---

## Legend

- **SFTP Push** = App → Chronopost `/IN`
- **FTP Pull** = App ← Chronopost `/OUT`
- **DB Read/Write** = Oracle 11g
- **Manual Action** = Performed by operator
- **Auto Scheduler** = Runs every 30 minutes

---

## Go-Live Enhancement Checklist

### 🔴 Critical (Before Go-Live)
1. **Login / Authentication**  
   No login exists now, so anyone can access the system.

2. **Role-based access**  
   Different permission levels needed for Operator, Manager, and Admin.

3. **Secure storage of SFTP credentials**  
   Credentials should not remain in plain text inside `application.yml`.

4. **Error retry queue**  
   If SFTP upload fails, system should retry automatically.

5. **MPSID Oracle sequence**  
   `AtomicInteger` resets after restart, so database sequence is required.

### 🟡 Business Logic
6. **Multi-parcel shipment support on label page**  
   Shipment supports multiple parcels, but batch label generation is missing.

7. **Shipment edit / cancel**  
   No proper edit or cancel workflow after creation.

8. **Return shipment workflow**  
   `R` event is tracked, but return process is not handled.

9. **Bulk shipment create**  
   Excel import for creating 100 shipments together is needed.

10. **Duplicate detection**  
   Same receiver + same day should trigger warning.

11. **Weight mismatch alert**  
   Declared vs actual weight mismatch should be flagged.

12. **Routing file auto-update**  
   Chronopost monthly `ROUCHR` file should reload automatically.

### 🟢 Frontend / UX
13. **Dashboard home**
14. **Shipment search improvement**
15. **Better tracking timeline**
16. **File upload status visibility**
17. **Error message usability improvement**

---

## Summary

This integration works in three main directions:

1. **Shipment data flow** → App sends shipment and pre-alert data to Chronopost  
2. **Document flow** → App sends export invoice to Chronopost  
3. **Tracking flow** → App pulls tracking events from Chronopost FTP `/OUT`

Chronopost does **not push tracking automatically**.  
Your app must **poll and pull tracking files every 30 minutes**.
