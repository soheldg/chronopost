# গল্পের ছলে — পুরো System Flow

## 🌅 সকালবেলা — Bangladesh থেকে শুরু

রাহেলা বেগম KBT Express-এর অফিসে বসে আছেন। তার হাতে একটা অর্ডার — বেলজিয়ামের ব্রাসেলসে থাকা এক customer **Women Cardigan** অর্ডার করেছে।

তিনি app খুললেন। **New Shipment** page-এ গিয়ে wizard শুরু করলেন।

### Step 1 — Sender
নিজের company-র info দিলেন:

- **Company:** KBT Express
- **City:** ঢাকা
- **Country:** বাংলাদেশ

### Step 2 — Receiver
Customer-এর info দিলেন:

- **Name:** Daniel Muller
- **City:** Brussels
- **Country:** Belgium
- **ZIP:** 1000

### Step 3 — Parcel
Parcel details দিলেন:

- **Weight:** 1440 গ্রাম
- **Dimensions:** 35 × 25 × 50 সেমি

### Step 4 — Customs
Customs info দিলেন:

- **Item:** Women Cardigan
- **Material:** 100% polyester
- **Value:** €83.58
- **HS Code:** 61101200
- **Origin:** BD

সব তথ্য দেওয়ার পর **Submit** করলেন।

App-এর পেছনে Oracle database-এ **৫টা table-এ data save** হলো।

- **Shipment ID:** 164
- **Status:** `CREATED`

---

## 🏷️ Label তৈরি

রাহেলা এখন **Labels** page-এ গেলেন।  
Shipment **164** দিলেন।

App সাথে সাথে backend-এ call করে parcel list নিয়ে এলো। Dropdown-এ দেখালো:

- `Parcel 1 · 1440g`

তিনি **Generate Label** click করলেন।

### Backend-এ যা হলো

1. প্রথমে weight দেখে বুঝলো:
   - **1440 গ্রাম**
   - অর্থাৎ **> 1 kg কিন্তু < 3 kg**
   - তাই **service code = 328 (Small)**
   - **suffix = "JF"**

2. তারপর parcel ID বানালো:

   - **Prefix:** `HY712` (KBT-এর prefix)
   - **Counter:** `000001` (DB-তে রাখা)
   - **Suffix:** `JF`

   তাই final Parcel ID হলো:

   - `HY712000001JF`

3. তারপর `CHR_ROUTING` table-এ খুঁজলো:

   - **country = BE**
   - **zip = 1000**
   - **origSort = BE1**
   - **distribSort = 011C**
   - **numCountry = 056**

4. সব মিলিয়ে routing barcode বানালো:

   - `%00010000HY712000001JF328056X`

5. এরপর iText দিয়ে PDF বানালো — **৫টা zone-এ সব information print** হলো।

6. DB-তে update হলো:

   - **Status:** `LABEL_READY`

রাহেলা PDF preview দেখলেন, download করলেন, printer-এ print করে parcel-এর গায়ে লাগালেন।

---

## 📡 Pre-Alert — Chronopost-কে আগেভাগে জানানো

Parcel এখনো ঢাকায় আছে।  
কিন্তু Chronopost France-কে আগেই জানাতে হবে:

> "আমি পাঠাচ্ছি, এই data।"

রাহেলা **Pre-Alert** page-এ গেলেন।  
Shipment **164** দিয়ে **Send to Chronopost** click করলেন।

### Backend validation করলো

- sender-এর VAT আছে কিনা
- parcel number real কিনা (`PENDING` না)
- weight match করছে কিনা

সব ঠিক থাকলে **GEODATA file** তৈরি হলো।

### GEODATA file details

- এটা একটা **semicolon-separated text file**
- **Encoding:** `ISO-8859-1`

**File name:**

```text
GEODATA_CONSO_14576904_CHR_D20260405T140000_000001
```

### SFTP journey শুরু

App backend France-এর Chronopost server-এ:

- **SSH connection** করলো
- **Port:** 22
- **Private key** দিয়ে authenticate করলো

এটা **plain FTP না**, বরং **encrypted SFTP**।

Server-এ গিয়ে `/IN` folder-এ file টা রেখে দিলো।

Chronopost-এর server file পড়লো, validate করলো, বলল:

> "ঠিক আছে।"

DB-তে update হলো:

- **Status:** `PREALERT_SENT`

---

## 📋 Invoice তৈরি এবং Upload

এবার customs-এর জন্য **commercial invoice** দরকার।

রাহেলা **Invoice** page-এ গেলেন।  
**Generate** tab-এ গিয়ে Shipment **164** দিলেন।

Backend DB থেকে সব data নিয়ে—

- sender
- receiver
- items
- HS codes
- amounts

সব combine করে iText দিয়ে professional PDF বানালো।

### Preview-এ যা ছিল

- seller address
- buyer address
- item table
- grand total
- signature field

সব দেখে তিনি **Generate + Upload** click করলেন।

### Backend আবার SFTP connection করলো

এবার file rename হলো:

```text
INV_EXP_14576904_HY712000001JF.PDF
```

তারপর `/IN` folder-এ upload করে দিলো।

DB-তে update হলো:

- **Status:** `INVOICE_SENT`

---

## ✈️ Physical Journey শুরু

রাহেলা parcel টা courier-এ দিলেন।  
Parcel বিমানে উঠলো:

- Dhaka → Dubai → Brussels

এই সময় Chronopost তাদের system থেকে tracking events পাঠাতে শুরু করলো।

কিন্তু এখানে একটু আলাদা process হয়।

---

## 📥 Tracking — FTP থেকে Pull

Chronopost **push করে না**।  
তারা একটা **FTP server-এ tracking file রেখে দেয়**।

App-এর **Scheduler** প্রতি **৩০ মিনিটে একবার** ঘুম থেকে উঠে কাজ করে।

সে FTP server-এ যায়, folder check করে, নতুন কোনো **EDIPOD file** আছে কিনা দেখে।

### EDIPOD file কী?

- এটা আরেকটা **text file**
- **৫১টা semicolon-separated field** থাকে
- প্রতিটা line মানে **একটা tracking event**

নতুন file থাকলে app সেটা download করে।

### তারপর parse করে

উদাহরণ:

```text
HY712000001JF...SM...20260405...1430...CDG...
```

এখান থেকে পাওয়া যায়:

- **Parcel:** `HY712000001JF`
- **Event:** `SM` (In transit / scanned)
- **Date:** `05 April 2026`
- **Time:** `14:30`
- **Location:** `CDG (Charles de Gaulle Airport)`

তারপর DB-তে `CHR_TRACKING_EVENTS` table-এ save করে।

Shipment status update হয়:

- **Status:** `IN_TRANSIT`

### ৩০ মিনিট পরে আরও event এলো

- `TI` → Arrived at hub (Brussels sorting center)
- `TO` → Sorted out from hub
- `TA` → Out for delivery

এরপর সেই দিন বিকেলে:

- `D` → **DELIVERED ✅**

DB-তে Shipment **164**-এর status চূড়ান্তভাবে update হলো।

---

## 📍 Tracking Page-এ দেখা

রাহেলা **Tracking** page খুললেন।

Dashboard-এ দেখলেন:

- **Delivered:** 1

তিনি `HY712000001JF` দিয়ে search করলেন।

### Timeline দেখালো

- ✓ **Delivered** — 05 Apr · 16:45
- 🚚 **Out for delivery** — 05 Apr · 09:00
- ⬤ **Sorted at hub** — 04 Apr · 22:30
- ⬤ **Arrived at hub** — 04 Apr · 18:00
- → **In transit / scanned** — 04 Apr · 14:30
- 📦 **Picked up by driver** — 04 Apr · 10:00

রাহেলা হাসলেন। Customer-কে notify করলেন।

---

## 🔄 পুরো journey এক নজরে

```text
KBT Express (BD)          App                    Chronopost (FR)
─────────────────────────────────────────────────────────────────

Create Shipment    →    DB save
Print Label        →    PDF generate + DB update
                        SFTP → /IN  ────────────→  GEODATA file receive
Invoice generate   →    PDF create
                        SFTP → /IN  ────────────→  Invoice file receive

                                           ↓ Physical parcel travels

                   ←── FTP pull (30 min) ←──  EDIPOD tracking file
                        Parse events
                        DB update
                        Status: DELIVERED
```

---

## সারকথা

- **App → Chronopost:** সবসময় **SFTP push** (`/IN` folder-এ file দেওয়া)
- **Chronopost → App:** সবসময় **FTP pull** (App নিজে গিয়ে file নিয়ে আসে)
- **Invoice ও GEODATA:** App push করে
- **Tracking events:** App pull করে
- **Chronopost push করে না**
- **সব scheduling:** ৩০ মিনিট পরপর automatic

---

## এক লাইনে পুরো flow

**Shipment Create → Label Generate → Pre-Alert Send → Invoice Upload → Parcel Travel → Tracking Pull → Delivered**
