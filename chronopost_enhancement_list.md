# Enhancement List — Chronopost Integration App

## 🔴 Critical (Go-Live-এর আগে)

| # | Enhancement | কেন দরকার |
|---|---|---|
| 1 | Login / Authentication | এখন কোনো login নেই — যে কেউ ঢুকতে পারে |
| 2 | Role-based access | Operator, Manager, Admin আলাদা permission |
| 3 | SFTP credentials secure storage | এখন `application.yml`-এ plain text |
| 4 | Error retry queue | SFTP fail হলে auto-retry mechanism |
| 5 | MPSID Oracle sequence | এখন `AtomicInteger` — restart-এ reset হয় |

## 🟡 Business Logic

| # | Enhancement | কেন দরকার |
|---|---|---|
| 6 | Multi-parcel shipment | এখন ১টা shipment-এ N parcel support আছে কিন্তু label page-এ batch generate নেই |
| 7 | Shipment edit/cancel | Create-এর পরে cancel বা full edit নেই |
| 8 | Return shipment workflow | R event track হয় কিন্তু return process নেই |
| 9 | Bulk shipment create | Excel থেকে import করে একসাথে ১০০ shipment |
| 10 | Duplicate detection | Same receiver + same day → warning দেওয়া |
| 11 | Weight mismatch alert | Declared vs actual weight আলাদা হলে flag |
| 12 | Routing file auto-update | Chronopost মাসে ROUCHR দেয় — auto reload নেই |

## 🟢 Frontend / UX

| # | Enhancement | কেন দরকার |
|---|---|---|
| 13 | Dashboard home page | এখন কোনো home নেই — সরাসরি list-এ যায় |
| 14 | Real-time notification | Label ready / delivered হলে browser notification |
| 15 | Dark mode | — |
| 16 | Mobile responsive | এখন desktop-first |
| 17 | Batch label print | একসাথে ১০টা label print করা |
| 18 | Shipment search by date range | Date picker filter |
| 19 | Tracking map view | Route দেখানো (Brussels, CDG airport...) |
| 20 | Email notification | Delivered হলে customer-কে auto email |
| 21 | PDF label A4 / A5 toggle | দুই format support |
| 22 | Saved sender profiles | বারবার একই sender info — save করা |

## 🔵 Backend / Technical

| # | Enhancement | কেন দরকার |
|---|---|---|
| 23 | Swagger / OpenAPI | `/swagger-ui` — API test করা যাবে |
| 24 | Audit log table | কে কখন কী করেছে — track রাখা |
| 25 | Rate limiting | API flood protection |
| 26 | Health check endpoint | `/actuator/health` — monitoring |
| 27 | Metrics / Prometheus | Performance monitoring |
| 28 | Unit tests | এখন মাত্র ৪টা test class আছে |
| 29 | Integration tests | DB + API end-to-end test |
| 30 | Liquibase / Flyway | DB migration versioning |
| 31 | Redis cache | Routing lookup 417K rows — cache করলে faster |
| 32 | Async label generation | বড় batch-এ blocking না করে background-এ |
| 33 | File cleanup scheduler | পুরনো PDF file auto-delete |

## 🟣 Reporting

| # | Enhancement | কেন দরকার |
|---|---|---|
| 34 | Monthly shipment report PDF | Management-এর জন্য |
| 35 | Delivery success rate | কত % delivered, কত % failed |
| 36 | Average delivery time | Country-wise |
| 37 | Top destinations chart | কোন দেশে বেশি যায় |
| 38 | Failed delivery analysis | কোন reason সবচেয়ে বেশি |
| 39 | Invoice reconciliation | পাঠানো invoice vs received invoice match |

## ⚪ Future / Advanced

| # | Enhancement | কেন দরকার |
|---|---|---|
| 40 | Multi-tenant | অন্য company-ও ব্যবহার করতে পারবে |
| 41 | WooCommerce / Shopify integration | Order সরাসরি import |
| 42 | Customer portal | Customer নিজে tracking দেখতে পারবে |
| 43 | WhatsApp notification | Delivered হলে WhatsApp message |
| 44 | Barcode scanner | Parcel scan করে status update |
| 45 | HS Code lookup | Item লিখলে auto HS code suggest |

## Priority Matrix

### এখনই করো (1–5)
- Login
- Roles
- Secure credentials
- Retry queue
- MPSID sequence

### Soon (6–22)
- Dashboard
- Batch label
- Bulk import
- Notifications
- Routing auto-update

### Later (23–39)
- Swagger
- Tests
- Redis cache
- Reports
- Analytics

### Future (40–45)
- Multi-tenant
- E-commerce integration
- Customer portal

