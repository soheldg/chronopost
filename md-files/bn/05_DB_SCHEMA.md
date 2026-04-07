# ডেটাবেস স্কিমা
**প্রকল্প:** Chronopost আন্তর্জাতিক ইন্টিগ্রেশন  
**ডেটাবেস:** Oracle 11g | **স্কিমা:** CHRONOPOST  
**সংস্করণ:** ১.০ | **তারিখ:** ২০২৬-০৪-০৫

> বিস্তারিত কলাম তালিকার জন্য দেখুন: `../en/05_DB_SCHEMA.md`

---

## সেটআপ ক্রম

```sql
১. schema.sql                        -- সব টেবিল
২. alter_add_routing_table.sql       -- CHR_ROUTING
৩. alter_add_counter_table.sql       -- CHR_PARCEL_COUNTER
৪. alter_tracking_events.sql         -- ৭টি নতুন কলাম
৫. seed_products.sql                 -- CHR_PRODUCTS ডেটা
৬. sqlldr ... load_routing.ctl       -- CHR_ROUTING ৪,১৭,৬১২ সারি
```

---

## টেবিল সংক্ষেপ (১৩টি টেবিল)

| টেবিল | উদ্দেশ্য | সারি সংখ্যা |
|-------|---------|-----------|
| CHR_SHIPMENTS | মূল শিপমেন্ট রেকর্ড | প্রতিটি শিপমেন্টে ১টি |
| CHR_SENDERS | প্রেরকের ঠিকানা | প্রতিটি শিপমেন্টে ১টি |
| CHR_RECEIVERS | প্রাপকের ঠিকানা | প্রতিটি শিপমেন্টে ১টি |
| CHR_PARCELS | পার্সেল বিবরণ | শিপমেন্টপ্রতি ১–N |
| CHR_CUSTOMS | কাস্টমস ঘোষণা | প্রতিটি শিপমেন্টে ১টি |
| CHR_INVOICE_LINES | ইনভয়েস লাইন আইটেম | শিপমেন্টপ্রতি ১–N |
| CHR_TRACKING_EVENTS | EDIPOD ট্র্যাকিং ইভেন্ট | পার্সেলপ্রতি N |
| CHR_PREALERT_FILES | GEODATA ফাইল রেকর্ড | শিপমেন্টপ্রতি ১টি |
| CHR_INVOICE_FILES | ইনভয়েস PDF রেকর্ড | শিপমেন্টপ্রতি N |
| CHR_PRODUCTS | সার্ভিস ক্যাটালগ (PRODUIT.CSV) | ~১৫০ সারি |
| CHR_ROUTING | রাউটিং টেবিল (ROUCHR) | **৪,১৭,৬১২ সারি** |
| CHR_PARCEL_COUNTER | পার্সেল ID কাউন্টার | প্রিফিক্সপ্রতি ১টি |

---

## শিপমেন্ট স্ট্যাটাস প্রবাহ

```
CREATED → LABEL_READY → PREALERT_SENT → INVOICE_SENT → IN_TRANSIT → DELIVERED
                                                                    → FAILED
                                                                    → RETURNED
```

---

## গুরুত্বপূর্ণ নোট

| বিষয় | বিবরণ |
|------|------|
| **MPSWEIGHT** | ডেকাগ্রামে সংরক্ষিত (গ্রাম ÷ ১০) |
| **ইভেন্ট কোড** | অফিশিয়াল: D=ডেলিভারি, N=না দেওয়া, P=অনুপস্থিত, R=ফেরত |
| **রাউটিং ফাইল** | মাসিক আপডেট — নতুন ROUCHR আসলে পুনরায় লোড করতে হবে |
| **পার্সেল ID** | `[৫-প্রিফিক্স][৬-কাউন্টার][২-সাফিক্স]` = ১৩ অক্ষর |

---

*উৎস: `schema.sql`, সব `@Entity` Java ক্লাস*  
*সর্বশেষ আপডেট: ২০২৬-০৪-০৫*
