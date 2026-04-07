# API চুক্তি
**প্রকল্প:** Chronopost আন্তর্জাতিক ইন্টিগ্রেশন  
**বেস URL:** `http://localhost:8080/api`  
**সংস্করণ:** ১.০ | **তারিখ:** ২০২৬-০৪-০৫

> বিস্তারিত request/response body-র জন্য দেখুন: `../en/04_API_CONTRACT.md`

---

## সম্পূর্ণ এন্ডপয়েন্ট তালিকা (২৯টি)

### শিপমেন্ট
| মেথড | পথ | কাজ |
|------|-----|-----|
| POST | /shipment/create | নতুন শিপমেন্ট তৈরি |
| GET | /shipment/list | সব শিপমেন্ট তালিকা |
| GET | /shipment/{id} | শিপমেন্টের বিস্তারিত (sender+receiver+parcels+customs+invoiceLines) |
| PUT | /shipment/{id}/sender | প্রেরকের ঠিকানা আপডেট |
| PUT | /shipment/{id}/receiver | প্রাপকের ঠিকানা আপডেট |

### লেবেল
| মেথড | পথ | কাজ |
|------|-----|-----|
| POST | /label/generate | লেবেল PDF জেনারেট |
| GET | /label/preview/{parcelId} | ব্রাউজারে প্রিভিউ |
| GET | /label/download/{parcelId} | PDF ডাউনলোড |
| GET | /label/status/{shipmentId} | লেবেল স্ট্যাটাস |

### প্রি-অ্যালার্ট
| মেথড | পথ | কাজ |
|------|-----|-----|
| POST | /prealert/generate/{id} | GEODATA ফাইল তৈরি |
| POST | /prealert/send/{id} | তৈরি + যাচাই + পাঠানো |
| GET | /prealert/status/{id} | প্রি-অ্যালার্ট স্ট্যাটাস |

### ইনভয়েস
| মেথড | পথ | কাজ |
|------|-----|-----|
| POST | /invoice/upload | PDF আপলোড → SFTP /IN |
| POST | /invoice/fetch | SFTP /OUT থেকে ডাউনলোড |
| GET | /invoice/all | সব ইনভয়েস তালিকা |
| GET | /invoice/view/{id} | ব্রাউজারে দেখা |
| GET | /invoice/download/{id} | ডাউনলোড |
| POST | /invoice/retry/{id} | ব্যর্থ আপলোড পুনরায় চেষ্টা |
| POST | /invoice/test-mode | TEST মোড চালু/বন্ধ |

### ট্র্যাকিং
| মেথড | পথ | কাজ |
|------|-----|-----|
| POST | /tracking/poll | FTP থেকে EDIPOD ফাইল সংগ্রহ |
| GET | /tracking/dashboard | স্ট্যাটাস গণনা |
| GET | /tracking/{parcelNumber} | পার্সেল টাইমলাইন |
| GET | /tracking/shipment/{id} | শিপমেন্টের সব পার্সেল |
| GET | /tracking/search?q= | অনুসন্ধান |

### রেফারেন্স ডেটা
| মেথড | পথ | কাজ |
|------|-----|-----|
| GET | /admin/products | CHR_PRODUCTS সব সারি |
| GET | /admin/routing/lookup | দেশ+ZIP রাউটিং লুকআপ |

---

## সাধারণ Response ফরম্যাট
```json
{
  "success": true,
  "message": "বার্তা",
  "data": { ... }
}
```

---

*সর্বশেষ আপডেট: ২০২৬-০৪-০৫*
