# OpsCommand — Enterprise Operations & Logistics Command Dashboard

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110%2B-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![SQLite WAL](https://img.shields.io/badge/SQLite-WAL_Mode-003B57?logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![PostgreSQL Ready](https://img.shields.io/badge/PostgreSQL-Ready-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A **white-label, enterprise-grade Operations & Logistics Command Dashboard** that you can fork, rebrand, and deploy for **any organization** — corporate event logistics, franchise rollouts, retail distribution, warehouse fulfillment, field supply chain, or institutional dispatch operations.

**27,000+ lines** of production code. **22 database tables**. **Zero hardcoded brand names**. Ready to scale.

---

## Table of Contents

1. [Why This Exists](#1-why-this-exists)
2. [The 4-Question Architecture](#2-the-4-question-architecture)
3. [Tech Stack](#3-tech-stack)
4. [Database Schema (22 Tables)](#4-database-schema-22-tables)
5. [Project Structure](#5-project-structure)
6. [Quickstart (3 Minutes)](#6-quickstart-3-minutes)
7. [White-Label Customization Guide](#7-white-label-customization-guide)
8. [Complete API Reference](#8-complete-api-reference)
9. [Frontend Architecture](#9-frontend-architecture)
10. [Scaling to PostgreSQL](#10-scaling-to-postgresql)
11. [Docker Deployment](#11-docker-deployment)
12. [Use Cases & Industry Examples](#12-use-cases--industry-examples)
13. [Contributing](#13-contributing)
14. [License](#14-license)

---

## 1. Why This Exists

Traditional logistics management relies on shared Excel spreadsheets, WhatsApp groups, and manual phone-call status updates. This creates:

- **No single source of truth** — 5 people editing 5 copies of the same tracker
- **No real-time visibility** — "Is the shipment delivered?" requires 3 phone calls
- **No audit trail** — changes are silently overwritten
- **No at-risk alerting** — deadlines pass unnoticed

**OpsCommand** replaces all of that with a single web dashboard that any operations team can deploy in minutes.

---

## 2. The 4-Question Architecture

Every screen in this dashboard is designed to answer **4 operational questions** in under 1 second:

| # | Question | What You See |
|---|----------|-------------|
| **1** | **WHEN** is the next deadline? | Countdown badge, exact date, color-coded urgency |
| **2** | **WHERE** is the destination? | Hub name, state/region, SPOC name & 1-tap phone |
| **3** | **IS IT DELIVERED?** | Courier carrier, live AWB tracking link, box count |
| **4** | **WHAT'S AT RISK?** | Unverified consignments approaching their deadline |

### The Decoupled Verification Model

This is the key architectural insight: **courier "Delivered" ≠ physically verified**.

```
PENDING → IN_TRANSIT → DELIVERED (carrier) → CONFIRMED (human SPOC on-site)
                                            ↗
                                    DISCREPANCY (items damaged/missing)
```

Standard couriers mark packages as "Delivered" when dropped at a facility gate or reception desk. OpsCommand adds a **second verification layer** — the local SPOC (Single Point of Contact) must physically open the carton, verify contents, and confirm receipt via phone/WhatsApp/email.

---

## 3. Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Backend** | FastAPI (Python 3.10+) | Async REST API with auto-generated OpenAPI docs |
| **ORM** | SQLAlchemy 2.0 | Database-agnostic — swap SQLite ↔ PostgreSQL with 1 line |
| **Database** | SQLite with WAL mode | Zero-config, 64MB cache, concurrent reads, portable `.db` file |
| **Frontend** | Vanilla ES6 SPA | Zero build tools, zero npm, zero React — just 3 files |
| **UI Pattern** | Linear / Stripe design | High-density tables, hairline borders, monospace figures |
| **Typography** | Inter + IBM Plex Mono | Professional body text + tabular-aligned numbers |
| **Excel Engine** | OpenPyXL | Multi-sheet audit exports with freeze panes & auto-widths |
| **Validation** | Pydantic v2 | Request/response schema validation with type safety |
| **Deployment** | Docker + Uvicorn | Production container with health checks |

---

## 4. Database Schema (22 Tables)

```mermaid
erDiagram
    Event ||--o{ EventDestination : "has destinations"
    Event ||--o{ Dispatch : "has dispatches"
    Event ||--o{ Participant : "has participants"
    Event ||--o{ PrintingJob : "has print jobs"
    
    Organiser ||--o{ EventDestination : "linked to"
    EventDestination ||--o{ Dispatch : "shipped to"
    
    Dispatch ||--o{ DispatchItem : "contains items"
    Dispatch ||--o| DeliveryConfirmation : "verified by"
    
    CollateralItem ||--o{ DispatchItem : "tracked as"
    CollateralItem ||--o{ StockTransaction : "inventory of"
    
    ImportJob ||--o{ ImportFile : "uploaded files"
    ImportFile ||--o{ ImportSheet : "detected sheets"
```

| # | Table | Purpose | Key Fields |
|---|-------|---------|------------|
| 1 | `users` | Operator accounts & roles | username, role (ADMIN/VIEWER), hashed_password |
| 2 | `events` | Campaigns / operations | event_code, start_date, delivery_deadline, status |
| 3 | `collateral_items` | Master SKU catalog | code, name, category (CORE/PREMIUM/ACCESSORY) |
| 4 | `participants` | Bulk attendee/recipient records | full_name, mobile, card_type, custom_fields_json |
| 5 | `organisers` | Reusable hub/destination directory | organiser_name, contact_person, full address |
| 6 | `event_destinations` | Event-specific dispatch targets | event_id, destination_name, required_quantities_json |
| 7 | `printing_jobs` | Print vendor job tracking | job_number, vendor, quantity, status |
| 8 | `physical_receipts` | Goods received from printers | quantity_received, challan_number |
| 9 | `stock_transactions` | Inventory ledger (+ve in, −ve out) | collateral_code, quantity, balance_after |
| 10 | `dispatch_batches` | Grouped shipment waves | batch_code, total_destinations |
| 11 | `dispatches` | Individual consignments | courier_partner, awb_number, courier_status, hist_* snapshot |
| 12 | `dispatch_items` | Collateral breakdown per shipment | collateral_code, quantity |
| 13 | `delivery_confirmations` | Physical SPOC verification | confirmed_by_name, confirmation_method, confirmation_date |
| 14 | `printing_invoices` | Vendor payment tracking | invoice_number, total_amount, payment_status |
| 15 | `courier_invoices` | Courier billing reconciliation | billing_period, billed_weight_kg, total_amount |
| 16 | `import_jobs` | Spreadsheet import sessions | status, total_files |
| 17 | `import_files` | Individual uploaded files | original_filename, detected_category, row counts |
| 18 | `import_sheets` | Sheet-level header detection | detected_headers_json, suggested_mapping_json |
| 19 | `import_mappings` | Learned column alias memory | source_alias → target_field, usage_count |
| 20 | `export_templates` | Reusable export configurations | column_definitions_json, header_start_row |
| 21 | `export_jobs` | Export audit trail | filter_criteria_json, row_count, generated_files |
| 22 | `audit_logs` | Full change history | entity_name, action, old_values_json, new_values_json |

---

## 5. Project Structure

```
Crm-dashboard-/
│
├── config.py                  # ← START HERE: Organization name, items, couriers, regions
├── .env.example               # Environment variable template (copy to .env)
├── database.py                # SQLAlchemy engine (SQLite WAL / PostgreSQL)
├── models.py                  # 22-table ORM schema
├── schemas.py                 # Pydantic v2 request/response validators
├── main.py                    # FastAPI app + router aggregation + middleware
├── seed_data.py               # Demo seeder (25 hubs, multi-carrier AWBs)
├── requirements.txt           # Python dependencies
│
├── routes/                    # Modular API endpoints
│   ├── config.py              # GET /api/config → dynamic branding + item catalog
│   ├── dashboard.py           # GET /api/dashboard → KPIs, telemetry, at-risk alerts
│   ├── dispatch.py            # CRUD consignment pipeline (book, track, update AWB)
│   ├── events.py              # CRUD campaign/operations management
│   ├── organisers.py          # Paginated destination directory + SPOC contacts
│   ├── participants.py        # Bulk attendee/recipient management
│   ├── tracking.py            # Multi-carrier AWB URL resolution
│   ├── stock.py               # Collateral inventory ledger
│   ├── printing.py            # WYSIWYG carton & thermal label engine
│   ├── billing.py             # Courier & vendor invoice ledgers
│   ├── export_engine.py       # Multi-sheet Excel workbook generation
│   ├── import_engine.py       # Smart spreadsheet ingestion with header detection
│   ├── datastore.py           # Database inspector, backup download, table counts
│   └── audit.py               # Change log & audit trail
│
├── static/                    # Frontend SPA (zero build tools)
│   ├── index.html             # Shell HTML with dynamic config loader
│   ├── app.js                 # 9,300-line Vanilla ES6 application engine
│   └── styles.css             # 11,400-line enterprise design system (Inter + Plex Mono)
│
├── Dockerfile                 # Production container (Python 3.11-slim)
├── docker-compose.yml         # 1-command deployment with volume persistence
├── start_server.bat           # Windows 1-click launcher
├── start_server.sh            # Linux/macOS 1-click launcher
└── LICENSE                    # MIT — free for commercial use
```

---

## 6. Quickstart (3 Minutes)

### Option A: Local Python (Development)

```bash
# 1. Clone
git clone https://github.com/Prashant20p95/Crm-dashboard-.git
cd Crm-dashboard-

# 2. Virtual environment
python -m venv venv
# Windows:
.\venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Seed demo data (25 regional hubs with live tracking AWBs)
python seed_data.py

# 5. Start the server
python main.py
```

Open `http://127.0.0.1:8000` — you'll see a fully populated dashboard with 25 regional hubs, multi-carrier dispatches, and live operational data.

### Option B: Docker (Production)

```bash
docker-compose up -d
# → http://localhost:8000
```

### Option C: Windows 1-Click

Double-click `start_server.bat`.

---

## 7. White-Label Customization Guide

### Step 1: Rebrand (30 seconds)

Copy `.env.example` to `.env` and change 3 lines:

```ini
ORGANIZATION_NAME=Acme Global Logistics
PORTAL_TITLE=Acme Operations & Dispatch Command
PORTAL_SUBTITLE=Pan-Region Distribution & Delivery Verification
```

The frontend reads `/api/config/` on startup and dynamically updates the header, document title, and all reports.

### Step 2: Define Your Tracked Items

Edit `config.py` → `TRACKED_ITEMS`:

```python
TRACKED_ITEMS = [
    {"code": "ITEM_PRIMARY",   "name": "Starter Kit",           "category": "CORE",      "unit": "kits",    "default_qty": 500},
    {"code": "ITEM_SECONDARY", "name": "Marketing Collateral",  "category": "CORE",      "unit": "packs",   "default_qty": 100},
    {"code": "ITEM_VIP",       "name": "Executive Hamper",      "category": "PREMIUM",   "unit": "hampers", "default_qty": 25},
    {"code": "ITEM_ACCESSORY", "name": "ID Badges & Lanyards",  "category": "ACCESSORY", "unit": "pcs",     "default_qty": 600},
]
```

### Step 3: Add Your Courier Partners

Edit `config.py` → `COURIER_PARTNERS`:

```python
COURIER_PARTNERS = {
    "BlueDart":    "https://www.bluedart.com/tracking?awb={awb}",
    "Delhivery":   "https://www.delhivery.com/track/package/{awb}",
    "FedEx":       "https://www.fedex.com/fedextrack/?trknbr={awb}",
    "DHL":         "https://www.dhl.com/en/express/tracking.html?AWB={awb}",
    "UPS":         "https://www.ups.com/track?tracknum={awb}",
    "Internal":    "",
}
```

### Step 4: Set Geographic Regions

```python
REGIONS = ["North", "South", "East", "West", "Central"]
# or for international:
REGIONS = ["APAC", "EMEA", "Americas", "ANZ"]
```

### Step 5: Configure Risk Threshold

```ini
# Flag destinations as "AT RISK" when deadline is within N days and not verified
AT_RISK_THRESHOLD_DAYS=3
```

---

## 8. Complete API Reference

All endpoints return JSON. Interactive Swagger UI available at `/docs`.

### System & Configuration

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/config/` | Organization branding, item catalog, courier list, regions |

### Campaigns / Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/events/` | List all campaigns with pagination |
| `POST` | `/api/events/` | Create new campaign |
| `PUT` | `/api/events/{id}` | Update campaign details |
| `DELETE` | `/api/events/{id}` | Delete campaign |

### Destinations & SPOC Directory

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/organisers/` | Paginated, filterable destination list with status |
| `PATCH` | `/api/destinations/{id}` | Quick inline edit (SPOC, dates, courier, AWB, stage) |

### Dispatch Pipeline

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/dispatch/` | Consignment ledger with filtering |
| `POST` | `/api/dispatch/` | Book new consignment (auto-generates dispatch code) |
| `PUT` | `/api/dispatch/{id}` | Update courier, AWB, status |

### Dashboard & Telemetry

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/dashboard/` | KPIs, status counters, at-risk alerts, telemetry strip |

### Excel Exports

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/export/quick/station-status-excel` | Station delivery tracker (multi-sheet) |
| `GET` | `/api/export/quick/dispatches-excel` | Dispatch manifest with item breakdowns |
| `GET/POST` | `/api/export/operations-summary-excel` | Full operations summary with parity audit |

### Data Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/datastore/health` | Database health, table counts, WAL status |
| `GET` | `/api/datastore/backup-db` | Download raw `.db` backup file |
| `GET` | `/api/datastore/template-excel` | Download blank station import template |
| `POST` | `/api/datastore/import-stations` | Smart Excel/CSV import with header detection |

### Audit Trail

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/audit/` | Paginated change history with entity filtering |

---

## 9. Frontend Architecture

The frontend is a **zero-dependency Vanilla ES6 Single-Page Application** — no React, no Vue, no npm, no webpack. Just 3 files:

| File | Lines | Role |
|------|-------|------|
| `index.html` | 92 | Shell: header, nav tabs, modal containers, script loader |
| `app.js` | 9,377 | Full application logic: routing, API calls, DOM rendering |
| `styles.css` | 11,406 | Enterprise design system: 4px/8px grid, Inter font, monospace figures |

### Tab Architecture

| Tab | Purpose |
|-----|---------|
| **Operations Dashboard** | KPI telemetry strip, at-risk alerts, filterable station table |
| **Dispatch Pipeline** | Consignment booking modal, courier ledger, AWB tracking links |
| **Stationery & Labels** | WYSIWYG thermal shipping label preview, multi-box carton labels |
| **Master Data Store** | Database inspector, table counts, backup download, Excel import |
| **Campaign Master** | Event/campaign CRUD, delivery deadlines, status management |

### Dynamic Configuration

On page load, `app.js` calls `GET /api/config/` and dynamically sets:
- Document title and header branding
- Item names in tables and modals
- Courier dropdown options
- Region filter options

This means **zero hardcoded brand names in the frontend**.

---

## 10. Scaling to PostgreSQL

The codebase is 100% database-agnostic via SQLAlchemy ORM. To switch:

```bash
# 1. Install driver
pip install psycopg2-binary

# 2. Update .env
DATABASE_URL=postgresql://user:pass@db.example.com:5432/opscommand

# 3. Initialize schema
python seed_data.py
```

The `database.py` engine auto-detects the URL prefix and configures connection pooling accordingly (SQLite pragmas are only applied for `sqlite:///` URLs).

---

## 11. Docker Deployment

### Single Container

```bash
docker build -t opscommand .
docker run -d -p 8000:8000 \
  -e ORGANIZATION_NAME="Your Company" \
  -e PORTAL_TITLE="Your Operations Portal" \
  -v opsdata:/app \
  opscommand
```

### Docker Compose (Recommended)

```yaml
# docker-compose.yml is included — just run:
docker-compose up -d
```

Includes health check: `curl -f http://localhost:8000/api/config/ || exit 1`

---

## 12. Use Cases & Industry Examples

| Industry | What You Track | Items Config Example |
|----------|---------------|---------------------|
| **Corporate Events** | Passes, lanyards, delegate kits → 200 venues | `ITEM_PRIMARY="Delegate Pass"` |
| **Franchise Rollout** | Signage, POS material, uniforms → 500 stores | `ITEM_PRIMARY="Store Kit"` |
| **Pharmaceutical** | Drug samples, brochures → 1,000 clinics | `ITEM_PRIMARY="Sample Pack"` |
| **Election Operations** | Ballot papers, booth material → 5,000 booths | `ITEM_PRIMARY="Booth Kit"` |
| **FMCG Distribution** | Promotional displays → 3,000 retailers | `ITEM_PRIMARY="POS Display"` |
| **Education** | Exam papers, certificates → 800 centers | `ITEM_PRIMARY="Exam Pack"` |
| **Construction** | Safety gear, PPE kits → 150 project sites | `ITEM_PRIMARY="Safety Kit"` |

In every case, the workflow is identical:
1. Define destinations (hubs/stores/clinics/booths)
2. Assign required item quantities per destination
3. Book courier dispatch with AWB
4. Track transit → delivery → physical verification

---

## 13. Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-module`)
3. Commit with conventional commits (`feat:`, `fix:`, `docs:`)
4. Push and open a Pull Request

---

## 14. License

MIT License — free for commercial and private use. See [LICENSE](LICENSE).

---

**Built for operations teams who need answers, not dashboards that need explaining.**
