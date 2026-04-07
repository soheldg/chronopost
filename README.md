# chronopost
# DB Tables — কোনটা কখন ব্যবহার হয়

## Table Usage Overview

| Table | কখন ব্যবহার হয় |
|---|---|
| `CHR_SHIPMENTS` | ধাপ ১ — create |
| `CHR_SENDERS` | ধাপ ১ — create, ধাপ ২ — edit |
| `CHR_RECEIVERS` | ধাপ ১ — create, ধাপ ২ — edit |
| `CHR_PARCELS` | ধাপ ১ — create, ধাপ ৩ — label update |
| `CHR_CUSTOMS` | ধাপ ১ — create |
| `CHR_INVOICE_LINES` | ধাপ ১ — create |
| `CHR_PRODUCTS` | ধাপ ৩ — weight দিয়ে code lookup |
| `CHR_ROUTING` | ধাপ ৩ — country+zip দিয়ে routing lookup |
| `CHR_PARCEL_COUNTER` | ধাপ ৩ — parcel ID counter |
| `CHR_PREALERT_FILES` | ধাপ ৪ — GEODATA file save |
| `CHR_INVOICE_FILES` | ধাপ ৫ — invoice PDF save |
| `CHR_TRACKING_EVENTS` | ধাপ ৬ — tracking events save |

---

## Short Summary

এই flow-তে:

- **Create stage**-এ মূল shipment data save হয়
- **Edit stage**-এ sender/receiver information update হয়
- **Label stage**-এ product, routing, parcel counter, এবং parcel label-related update হয়
- **File stage**-এ prealert ও invoice file store হয়
- **Tracking stage**-এ shipment event history save হয়
