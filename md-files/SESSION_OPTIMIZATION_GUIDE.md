# Claude Session Optimization Guide
## কীভাবে Context Limit দ্রুত শেষ হলো এবং পরবর্তীতে কীভাবে ভালো করবেন

**তারিখ:** ২০২৬-০৪-০৫  
**প্রকল্প:** Chronopost Integration  
**ভাষা:** বাংলা

---

## ১. এই Session-এ কতটুকু Context ব্যবহার হয়েছে

### Claude-এর Context Window কী?
Claude Sonnet-এর context window = **~200,000 tokens**  
১ token ≈ ৩/৪টি ইংরেজি শব্দ বা ২/৩টি বাংলা শব্দ

### এই Session-এ যা পড়া/লেখা হয়েছে (আনুমানিক):

| কাজ | পরিমাণ | Token (আনুমানিক) |
|-----|--------|----------------|
| সব JSX page পড়া (একসাথে) | ~3,100 লাইন | ~25,000 |
| বড় Java file পড়া | ~1,900 লাইন | ~15,000 |
| PRODUIT.CSV সম্পূর্ণ পড়া | ~300 লাইন | ~8,000 |
| ROUCHR routing file analysis | ~500 লাইন output | ~5,000 |
| নতুন Frontend files তৈরি | ~1,500 লাইন | ~20,000 |
| নতুন Backend files তৈরি | ~600 লাইন | ~8,000 |
| সব Docs তৈরি (EN+BN) | ~2,900 লাইন | ~35,000 |
| Handoff file তৈরি | ~250 লাইন | ~4,000 |
| Chat conversation নিজেই | বহু back-and-forth | ~30,000 |
| Bash command outputs | অনেক বড় output | ~25,000 |
| **মোট আনুমানিক** | | **~175,000 tokens** |

> ⚠️ এই session প্রায় **87%** context limit ব্যবহার করেছে।  
> এজন্যই শেষের দিকে ধীর হয়ে গেছে এবং compacted হয়েছিল।

---

## ২. কোন কাজগুলো সবচেয়ে বেশি Token খেয়েছে

### 🔴 সবচেয়ে বেশি ক্ষতিকর কাজ

**১. একসাথে সব JSX file পড়া**
```bash
# এটা করেছিলেন — ভুল!
for f in /home/claude/project/frontend/src/pages/*.jsx; do
  cat "$f"   # সব 5টা file একসাথে = ~3,100 লাইন একবারে
done
```
→ শুধু NewShipmentPage.jsx-ই ছিল **967 লাইন**  
→ এই একটা command-এই ~25,000 token শেষ

**২. PRODUIT.CSV সম্পূর্ণ পড়া**
```bash
cat /mnt/user-data/uploads/PRODUIT.CSV  # শত শত লাইন
```
→ প্রয়োজন ছিল শুধু structure বোঝা (~10 লাইন দেখলেই হতো)

**৩. বারবার একই file re-read করা**
- `ShipmentService.java` এই session-এ **৩-৪ বার** পড়া হয়েছে
- প্রতিবার নতুন করে পড়া = প্রতিবার নতুন token খরচ

**৪. Audit script-এ সব file একসাথে scan করা**
```bash
# এটা করা হয়েছিল — প্রতিটা file পড়ে content match করা
grep -rh "..." /home/claude/project/backend/src --include="*.java"
```
→ ৬০+ Java file-এর content একসাথে context-এ লোড হয়

**৫. Documents দুই ভাষায় সম্পূর্ণ করা একই session-এ**
- ৬টা EN doc + ৬টা BN doc = ১২টা বড় file একই session-এ
- মোট ~2,900 লাইন = ~35,000 token

---

## ৩. আপনার ভুলগুলো (Honestly)

### ভুল ১: একটাই session-এ সব করার চেষ্টা
এই session-এ যা হয়েছে:
```
PRODUIT.CSV check → Routing file analysis → Frontend audit →
Backend fixes → New pages → Docs → Handoff
```
এত কাজ একটা session-এ করা ঠিক না।

### ভুল ২: "Sob koro" বলা
যখন বললেন "Ok do it" বা "Sob e koro" — তখন আমি অনেক বড় কাজ একসাথে করেছি।  
প্রতিটা বড় কাজ = অনেক token।

### ভুল ৩: File দেখার আগে context না বলা
PRODUIT.CSV upload করে সরাসরি "Check this file" বললেন।  
আমি পুরো file পড়ে ফেললাম।  
বলা উচিত ছিল: **"Structure দেখো, শুধু প্রথম ১০ লাইন"**

### ভুল ৪: Audit বারবার করা
Session-এর শেষে "Akon ki front end backend sob thik ase" — তখন আবার সব file scan করা হলো।  
এই audit-টা handoff-এর আগে না করে করলে ভালো হতো।

### ভুল ৫: Documents একই session-এ
Docs তৈরি একটা আলাদা session-এ করা উচিত ছিল।  
Code session আলাদা, Documentation session আলাদা।

---

## ৪. Token বাঁচানোর সঠিক উপায়

### ✅ করণীয়

**১. File structure আগে, content পরে**
```
❌ "Check this file" (পুরো file পড়বে)
✅ "এই file-এর structure কী? শুধু column names দেখো"
✅ "প্রথম ৫ লাইন দেখো"
✅ "শুধু function names list করো"
```

**২. Specific question করুন**
```
❌ "Frontend-এ কী কী করা যেতে পারে?"
✅ "ShipmentListPage-এ কি sort feature আছে?"
✅ "TrackingPage-এ event code-এর legend কোথায়?"
```

**৩. একটা কাজ, একটা session**
```
Session 1: Backend fixes শুধু
Session 2: Frontend pages শুধু  
Session 3: Docs শুধু
Session 4: Audit + Handoff
```

**৪. Handoff ব্যবহার করুন (এখন যেটা তৈরি হলো)**
প্রতিটা session শেষে handoff → পরের session-এ দিয়ে শুরু করুন।  
এতে আমাকে সব context নতুন করে বুঝতে হবে না।

**৫. Code snippet চান, পুরো file না**
```
❌ "Show me ShipmentService.java"  (406 লাইন পুরো দেখাবে)
✅ "ShipmentService-এ getDetail method টা দেখাও"  (৫০ লাইন)
```

---

## ৫. এই Project-এ কীভাবে করলে ভালো হতো

### Recommended Session Breakdown:

```
📅 Session 1 — Project Setup + DB Schema
   Input:  "Spring Boot + Oracle 11g + React project বানাও"
   Output: schema.sql, pom.xml, application.yml, basic structure
   Token:  ~30,000 ✅ সহজ

📅 Session 2 — Spec Analysis (Doc 1, 2, 3, 4)
   Input:  শুধু Doc 1 upload → "এটা পড়ো, কী বানাতে হবে বলো"
   Output: Feature list, assumption list
   Token:  ~40,000 ✅ manageable

📅 Session 3 — Label Module
   Input:  "Label generate করার backend বানাও"
   Output: LabelController, LabelService, generators
   Token:  ~35,000 ✅

📅 Session 4 — PreAlert Module  
   Input:  "GEODATA file builder বানাও"
   Output: GeoDataFileBuilder, SftpService, validator
   Token:  ~35,000 ✅

📅 Session 5 — Tracking Module
   Input:  "EDIPOD tracking parser বানাও"
   Output: TrackingFileParser, TrackingService
   Token:  ~25,000 ✅

📅 Session 6 — Invoice Module
   Input:  "Invoice exchange module বানাও"
   Output: InvoiceController, InvoiceService, validator
   Token:  ~25,000 ✅

📅 Session 7 — Frontend (Core pages)
   Input:  "NewShipment + Label + PreAlert page বানাও"
   Output: 3 pages
   Token:  ~40,000 ✅

📅 Session 8 — Frontend (List + Detail pages)
   Input:  "ShipmentList + ShipmentDetail page বানাও"
   Output: 2 complex pages
   Token:  ~35,000 ✅

📅 Session 9 — Routing + Product DB
   Input:  "PRODUIT.CSV + ROUCHR routing DB integrate করো"
   Output: Product/Routing entities, services
   Token:  ~30,000 ✅

📅 Session 10 — Bug Fixes (Spec docs দিয়ে audit)
   Input:  "এই files দেখে কী ভুল আছে বলো"
   Output: Event codes, #DEF, version fixes
   Token:  ~35,000 ✅

📅 Session 11 — Documentation
   Input:  "Project docs বানাও (BRD, FRD, API, DB, UAT)"
   Output: 6 docs × 2 languages
   Token:  ~40,000 ✅
```

**মোট: ১১টা clean session vs আমাদের ১টা ভারী session**

---

## ৬. Claude-এর সাথে কাজের Best Practices

### কথা বলার ভালো style:

| পরিস্থিতি | ❌ ভুল | ✅ সঠিক |
|----------|-------|--------|
| File দেখতে চান | "Check this file" | "এই file-এর প্রথম ১০ লাইন দেখাও" |
| কাজ করতে চান | "Sob koro" | "শুধু [X] করো" |
| Bug fix চান | "Sob fix koro" | "শুধু event code-এর bug fix করো" |
| Feature চান | "Frontend improve koro" | "ShipmentList-এ sort feature যোগ করো" |
| Review চান | "Thik ase?" | "ShipmentController-এ PUT endpoint আছে কিনা check করো" |

### Session শুরুর সঠিক উপায়:
```
"আমি [project name]-এ কাজ করছি।
Handoff: [handoff content paste করুন]
এই session-এ শুধু [X] করতে চাই।"
```

### Session শেষের সঠিক উপায়:
```
"এই session শেষ।
Handoff file তৈরি করো পরের session-এর জন্য।"
```

---

## ৭. Context বাঁচানোর Quick Tips

```
📌 Rule 1: এক session = এক module বা এক feature
📌 Rule 2: File upload করার আগে বলো কোন part দরকার
📌 Rule 3: "সব করো" না বলে specific বলো
📌 Rule 4: প্রতিটা session-এ handoff নিয়ে রাখো
📌 Rule 5: Audit/review আলাদা session-এ করো
📌 Rule 6: বড় output-এর পর নতুন session শুরু করো
📌 Rule 7: Docs লেখা সবসময় আলাদা session-এ
📌 Rule 8: "দেখাও" না বলে "কী আছে বলো" বলো
```

---

## ৮. এই Project-এ পরের Session-এ কীভাবে শুরু করবেন

### Template:
```
আমি Chronopost Integration project-এ কাজ করছি।

নিচে Handoff document:
[CHRONOPOST_HANDOFF.md এর content paste করুন]

এই session-এ আমি শুধু [একটা কাজ] করতে চাই:
- [X] fix করতে চাই
অথবা
- [Y] feature যোগ করতে চাই
অথবা  
- [Z] test করতে চাই

সংশ্লিষ্ট file: [শুধু একটা বা দুটো file নাম]
```

---

## সারসংক্ষেপ

| বিষয় | এই Session | ভালো উপায় |
|------|-----------|-----------|
| Session সংখ্যা | ১টা বিশাল | ১১টা ছোট |
| Token ব্যবহার | ~175,000 (~87%) | প্রতিটায় ~35,000 |
| File পড়া | পুরো file | শুধু দরকারি অংশ |
| কাজের ধরন | "সব করো" | "এটা করো" |
| Handoff | শেষে একটা | প্রতি session শেষে |
| Docs | Code-এর সাথে | আলাদা session |

> **মূল কথা:** Claude-কে যত কম পড়তে হবে, তত বেশি কাজ করার জায়গা থাকবে।  
> ছোট ছোট session = বেশি কাজ = ভালো output।

---

*এই guide নিজেই `docs/` folder-এ রাখুন পরের reference-এর জন্য।*
