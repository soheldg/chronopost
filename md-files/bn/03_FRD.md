# ফাংশনাল প্রয়োজনীয়তার দলিল (FRD)
**প্রকল্প:** Chronopost আন্তর্জাতিক ইন্টিগ্রেশন  
**সংস্করণ:** ১.০ | **তারিখ:** ২০২৬-০৪-০৫ | **অবস্থা:** ✅ বেসলাইন

> বিস্তারিত ইংরেজি সংস্করণের জন্য দেখুন: `../en/03_FRD.md`

---

## ১. সিস্টেম আর্কিটেকচার

```
React 18 (ফ্রন্টএন্ড)
    │  REST API (JSON)
    ▼
Spring Boot 3.2 (ব্যাকএন্ড) ──── Oracle 11g (ডেটাবেস)
    │                               │
    │  SFTP (পোর্ট ২২)              ├── ১৩টি টেবিল
    ▼                               └── CHR_ROUTING (৪,১৭,৬১২ সারি)
Chronopost SFTP
  /IN  → GEODATA + ইনভয়েস আপলোড
  /OUT ← ইনভয়েস ডাউনলোড
```

---

## ২. ফিচার তালিকা

### F01 — শিপমেন্ট ব্যবস্থাপনা
| ID | ফিচার | এন্ডপয়েন্ট |
|----|-------|-----------|
| F01.1 | শিপমেন্ট তৈরি (৫-ধাপের উইজার্ড) | `POST /api/shipment/create` |
| F01.2 | সব শিপমেন্ট তালিকা (sort, filter, search) | `GET /api/shipment/list` |
| F01.3 | শিপমেন্টের বিস্তারিত দেখা | `GET /api/shipment/{id}` |
| F01.4 | প্রেরকের ঠিকানা সম্পাদনা | `PUT /api/shipment/{id}/sender` |
| F01.5 | প্রাপকের ঠিকানা সম্পাদনা | `PUT /api/shipment/{id}/receiver` |
| F01.6 | CSV এক্সপোর্ট | ফ্রন্টএন্ড শুধু |
| F01.7 | ওয়ার্কফ্লো স্ট্যাটাস বার | `ShipmentDetailPage.jsx` |

### F02 — লেবেল জেনারেশন
| ID | ফিচার | বাস্তবায়ন |
|----|-------|---------|
| F02.1 | লেবেল PDF জেনারেট | `POST /api/label/generate` |
| F02.2 | ওজন অনুযায়ী প্রোডাক্ট অটো-সমাধান (৩২৭/৩২৮) | `ProductService.java` |
| F02.3 | দেশ+ZIP অনুযায়ী রাউটিং অটো-সমাধান | `RoutingService.java` |
| F02.4 | লেবেল PDF প্রিভিউ | `GET /api/label/preview/{parcelId}` |
| F02.5 | লেবেল PDF ডাউনলোড | `GET /api/label/download/{parcelId}` |
| F02.6 | DB-তে পার্সেল কাউন্টার | `CHR_PARCEL_COUNTER` |
| F02.7 | ১৫-অক্ষর ট্র্যাকিং ID + ISO 7064 চেকসাম | `TrackingIdGenerator.java` |

### F03 — প্রি-অ্যালার্ট (GEODATA)
| ID | ফিচার | বাস্তবায়ন |
|----|-------|---------|
| F03.1 | GEODATA CONSO ফাইল তৈরি (৮ সাবটাইপ) | `GeoDataFileBuilder.java` |
| F03.2 | পাঠানোর আগে যাচাই | `PreAlertValidator.java` |
| F03.3 | SFTP দিয়ে Chronopost-এ পাঠানো | `SftpService.java` |
| F03.4 | TEST মোড (/TEST ফোল্ডার) | `SftpService.setTestMode()` |

### F04 — ইনভয়েস বিনিময়
| ID | ফিচার | বাস্তবায়ন |
|----|-------|---------|
| F04.1 | ইনভয়েস PDF আপলোড → Chronopost SFTP /IN | `POST /api/invoice/upload` |
| F04.2 | PDF রেজোলিউশন যাচাই (≥৩০০ DPI) | `InvoiceValidator.java` |
| F04.3 | ৩০ মিনিটে স্বয়ংক্রিয় ডাউনলোড /OUT থেকে | `InvoiceScheduler.java` |
| F04.4 | ব্যর্থ আপলোড পুনরায় চেষ্টা | `POST /api/invoice/retry/{id}` |

### F05 — ট্র্যাকিং
| ID | ফিচার | বাস্তবায়ন |
|----|-------|---------|
| F05.1 | EDIPOD ফাইল সংগ্রহ | `POST /api/tracking/poll` |
| F05.2 | ১৩৩টি অফিশিয়াল ইভেন্ট কোড পার্স | `TrackingFileParser.java` |
| F05.3 | শিপমেন্ট স্ট্যাটাস আপডেট | `TrackingService.java` |
| F05.4 | পার্সেল ট্র্যাকিং টাইমলাইন | `GET /api/tracking/{parcelNumber}` |

### F06 — রেফারেন্স ডেটা
| ID | ফিচার | বাস্তবায়ন |
|----|-------|---------|
| F06.1 | CHR_PRODUCTS দেখা | `GET /api/admin/products` |
| F06.2 | দেশ+ZIP রাউটিং লুকআপ | `GET /api/admin/routing/lookup` |

---

## ৩. ইনপুট যাচাইকরণ নিয়ম

### প্রেরক
- `accountNo`: আবশ্যিক, সর্বোচ্চ ২০ অক্ষর
- `companyName` বা `name1`: কমপক্ষে একটি আবশ্যিক
- `street`, `zipcode`, `town`, `countryCode`: আবশ্যিক
- `phone` বা `email`: কমপক্ষে একটি আবশ্যিক
- `businessType`: B বা P

### পার্সেল
- `declaredWeight`: আবশ্যিক, > ০ গ্রাম
- `dimension`: ঐচ্ছিক, ফরম্যাট LLLWWWHHHcm (৯ অক্ষর)

### ইনভয়েস লাইন
- `content`: আবশ্যিক, সর্বোচ্চ ২০০ অক্ষর (ইংরেজিতে)
- `amount`: আবশ্যিক, > ০
- `hsCodeRecv`: আবশ্যিক, ৮ সংখ্যা
- `hsCodeSend`: আবশ্যিক, সর্বোচ্চ ১০ অক্ষর

---

*উৎস: সব Controller + Service Java ফাইল, ফ্রন্টএন্ড পেজ*  
*সর্বশেষ আপডেট: ২০২৬-০৪-০৫*
