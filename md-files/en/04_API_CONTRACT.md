# API Contract
**Project:** Chronopost International Integration  
**Base URL:** `http://localhost:8080/api`  
**Version:** 1.0 | **Date:** 2026-04-05

> All responses follow: `{ "success": bool, "message": "...", "data": {...} }`

---

## Shipment Endpoints

### POST /shipment/create
Create a new shipment.

**Request Body:**
```json
{
  "linehaul": "AI",
  "sender": {
    "accountNo": "14576904", "companyName": "KBT Express",
    "street": "1 Rue des Tests", "zipcode": "1230",
    "town": "Dhaka", "countryCode": "BD",
    "phone": "+880123456789", "businessType": "B"
  },
  "receiver": {
    "name1": "Daniel Muller", "street": "11 Rue des Glaieuls",
    "zipcode": "75001", "town": "Paris", "countryCode": "FR",
    "phone1": "+33634567890", "businessType": "P"
  },
  "parcels": [{ "declaredWeight": 1440, "dimension": "035025050", "parcelRank": 1 }],
  "customs": { "parcelType": "P", "customsAmount": 109.09, "currency": "EUR",
               "reasonForExport": "01", "incoterms": "07" },
  "invoiceLines": [{
    "quantity": 2, "content": "Women Cardigans(100% Chemical fiber)",
    "amount": 83.58, "countryOrigin": "BD",
    "hsCodeRecv": "61101200", "hsCodeSend": "6110120000",
    "netWeight": 480, "grossWeight": 480
  }]
}
```
**Response:** `201` with shipmentId, mpsId, parcelCount

---

### GET /shipment/list
List all shipments. Supports `?status=CREATED|IN_TRANSIT|DELIVERED|FAILED`.

**Response fields:** id, mpsId, status, senderName, receiverName, receiverCountry, parcelCount, totalWeightGrams, linehaul, createdAt, labelReady, prealertSent, invoiceSent

---

### GET /shipment/{id}
Full shipment detail with nested: sender, receiver, parcels, customs, invoiceLines.

---

### PUT /shipment/{id}/sender
Update sender address fields.

**Request Body:** `{ "name1": "...", "street": "...", "zipcode": "...", "town": "...", "phone": "...", "email": "..." }`

---

### PUT /shipment/{id}/receiver
Update receiver address fields.

**Request Body:** `{ "name1": "...", "street": "...", "zipcode": "...", "town": "...", "phone1": "...", "email": "...", "doorCode": "..." }`

---

## Label Endpoints

### POST /label/generate
Generate shipping label PDF.

**Request Body:**
```json
{
  "shipmentId": 1,
  "parcelId": 1,
  "customerPrefix": "HY712",
  "geopostSenderId": "0407112",
  "customsClearanceCode": "HY-DP"
}
```
> `productSuffix`, `serviceCode`, `origSort`, `distribSort`, `numCountryCode` are **auto-resolved** from CHR_PRODUCTS and CHR_ROUTING.

**Response:** chronopostParcelId, geopostTrackingId, routingBarcodeDisplay, labelStatus, generatedAt

---

### GET /label/preview/{parcelId}
Returns PDF bytes (Content-Disposition: inline).

### GET /label/download/{parcelId}
Returns PDF bytes (Content-Disposition: attachment).

---

## Pre-Alert Endpoints

### POST /prealert/generate/{shipmentId}
Generate GEODATA file (does not send).

### POST /prealert/send/{shipmentId}
Generate + validate + send to Chronopost SFTP.

**Response:** fileName, ftpStatus, validationErrors[], fileContent (preview)

### GET /prealert/status/{shipmentId}
Returns: shipmentId, mpsId, shipmentStatus, prealertSent, fileName, sentAt

---

## Invoice Endpoints

### POST /invoice/upload
Upload PDF to Chronopost SFTP /IN.

**Request:** `multipart/form-data` — file (PDF), shipmentId (number), type (EXP|IMP)

### POST /invoice/fetch
Manually trigger download from Chronopost SFTP /OUT.

**Response:** totalFound, downloaded, failed

### GET /invoice/all
List all invoice files: id, fileName, fileSize, direction, ftpStatus, createdAt

### GET /invoice/view/{id}
Returns PDF (inline).

### GET /invoice/download/{id}
Returns PDF (attachment).

### POST /invoice/retry/{id}
Retry failed upload.

### POST /invoice/test-mode?enabled=true|false
Toggle TEST mode (sends to /TEST instead of /IN).

---

## Tracking Endpoints

### POST /tracking/poll
Manually poll Chronopost FTP for EDIPOD files.

**Response:** filesFound, filesProcessed, totalEventsProcessed, filesFailed

### GET /tracking/dashboard
Returns: totalShipments, delivered, inTransit, failed, returned

### GET /tracking/{parcelNumber}
Single parcel timeline: parcelNumber, geopostTrackingId, currentStatus, isDelivered, totalEvents, events[]

**Event fields:** eventCode, eventDescription, eventDate, eventTime, location, reasonCode, deliveryInstruction, pickupPudoCode, newDeliveryDate

### GET /tracking/shipment/{id}
All parcels for a shipment (array of tracking cards).

### GET /tracking/search?q={query}
Search by parcel number or tracking ID.

---

## Admin / Reference Data Endpoints

### GET /admin/products
List CHR_PRODUCTS. Optional: `?natInter=I&isPrimary=P`

**Response:** codeGeopost, serviceMark, serviceText, description, natInter, jour, suffixe, libel25, libelleFa, isPrimary

### GET /admin/routing/lookup?country={cc}&zip={zip}&service={svc}
Lookup routing info for a country+zip combination.

**Response:** country, zip, serviceType, origSort, agencyCode, distribSort, numCountryCode, normalizedZip

---

*All endpoints return CORS headers for `http://localhost:3000`*  
*Last updated: 2026-04-05*
