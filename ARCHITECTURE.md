# Technical Architecture

## System Overview

The Business Operations System is a **layered, event-driven architecture** using Microsoft 365 cloud services with no custom backend code. Data flows unidirectionally from operational capture through automated transformation into analytical outputs.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         PRESENTATION LAYER                          │
│   Power BI Dashboards (Analytics) + Power Apps (Operational Forms)  │
└────────────────┬──────────────────────────────────────────────────┘
                 │ Real-time refresh (5-15 min intervals)
                 │
┌────────────────▼──────────────────────────────────────────────────┐
│                      INTEGRATION LAYER                             │
│         Power Automate Workflows (Data Orchestration)              │
│  ├─ RegistroVentas (Sales capture → Excel)                        │
│  ├─ RegistroCompras (Purchase orders → Excel)                     │
│  ├─ GestionInventario (Stock movements → Excel)                   │
│  ├─ RegistroEstacionamiento (Parking transactions)                │
│  ├─ RegistroBaños (Facility usage tracking)                       │
│  ├─ AgregarProducto (Master data management)                      │
│  └─ EducacionAmbiental (Program participation)                    │
└────────────────┬──────────────────────────────────────────────────┘
                 │ REST API calls, Excel table operations
                 │
┌────────────────▼──────────────────────────────────────────────────┐
│                       PERSISTENCE LAYER                            │
│     Excel Workbook (OneDrive) + Normalized Data Model              │
│  ├─ DIM_Dates (time dimension)                                    │
│  ├─ DIM_Products (inventory catalog)                              │
│  ├─ DIM_Vendors (supplier directory)                              │
│  ├─ FACT_Sales (transaction-level revenue)                        │
│  ├─ FACT_Purchases (vendor transactions)                          │
│  └─ FACT_Inventory (movement history)                             │
└────────────────┬──────────────────────────────────────────────────┘
                 │ Built-in data validation + relationships
                 │
┌────────────────▼──────────────────────────────────────────────────┐
│                        SOURCE LAYER                                │
│                   Power Apps Canvas App                            │
│              (Mobile-optimized data entry forms)                   │
│  ├─ Sales Entry (multiple revenue streams)                        │
│  ├─ Inventory Transactions                                        │
│  ├─ Purchase Orders                                               │
│  ├─ Facility Operations                                           │
│  └─ Program Registration                                          │
└─────────────────────────────────────────────────────────────────────┘
```

## Component Details

### 1. Power Apps Canvas App

**Purpose**: Structured data capture optimized for field operations

**Interface Structure**:
```
Home Screen
├─ Sales Module
│  ├─ Revenue Stream Selector (parking, entry, merchandise, etc.)
│  ├─ Transaction Entry Form (date, amount, payment method)
│  ├─ Optional: Photo/receipt capture
│  └─ Real-time validation vs. master data
│
├─ Inventory Module  
│  ├─ Product Selector (linked to DIM_Products table)
│  ├─ Movement Type (in/out/adjustment)
│  ├─ Quantity & unit
│  └─ Location tracking
│
├─ Purchase Module
│  ├─ Vendor selection (auto-populated from DIM_Vendors)
│  ├─ Product selection with quantity
│  ├─ Invoice details (date, amount, tax)
│  └─ Attach document reference
│
└─ Reports Screen (read-only summary views)
```

**Technical Details**:
- **Data source**: Direct Excel connection via Power Automate trigger
- **Validation rules**: Applied in Power Apps controls + Power Automate flows
- **Offline capability**: Limited (requires online to write to Excel)
- **User experience**: Auto-population of dropdowns, input hints, error messages

### 2. Power Automate Workflows

**Orchestration Pattern**: **Event-Driven Transforms**

Each workflow follows this pattern:
```
Trigger (App submission) 
  → Validation logic
  → Data transformation
  → Excel table append
  → Error handling
  → Notifications
```

**Workflow Details**:

#### 2.1 RegistroVentas (Sales Registration)
- **Trigger**: Power Apps → Submit button
- **Input**: Revenue stream, amount, date, payment method
- **Process**:
  1. Validate amount > 0
  2. Lookup revenue stream code from DIM_Products
  3. Create row in FACT_Sales with surrogate keys
  4. Update running totals in summary table
  5. Log in audit table
- **Output**: Row ID for user confirmation
- **Error handling**: Email notification if validation fails

#### 2.2 RegistroCompras (Purchase Orders)
- **Trigger**: App → Purchase form submit
- **Input**: Vendor, products, quantities, amounts
- **Process**:
  1. Validate vendor exists in DIM_Vendors
  2. Validate products exist in DIM_Products
  3. Create row in FACT_Purchases
  4. Update DIM_Products.last_vendor_price
  5. Generate PO number (lookup last number + 1)
- **Output**: PO confirmation with tracking number

#### 2.3 GestionInventario (Inventory Management)
- **Trigger**: App → Stock movement form
- **Input**: Product, quantity, movement type (in/out/adjustment), reason
- **Process**:
  1. Lookup current quantity from FACT_Inventory (latest state)
  2. Validate sufficient quantity for outbound moves
  3. Create transaction row
  4. Calculate new quantity (balance)
  5. Alert if quantity < reorder level
- **Output**: Updated inventory state + low-stock alerts

#### 2.4-2.5 Facility Operations (Parking, Bathrooms)
- **Trigger**: Direct entry + scheduled aggregation
- **Input**: Count/transaction data
- **Process**: Append to facility-specific fact table
- **Output**: Daily/hourly summaries for BI

#### 2.6 AgregarProducto (Master Data)
- **Trigger**: Manual or scheduled import
- **Purpose**: Maintain product catalog
- **Process**: Insert/update DIM_Products

#### 2.7 EducacionAmbiental (Programs)
- **Trigger**: Attendance/participation entry
- **Purpose**: Track environmental education program metrics
- **Output**: Program participation FACT table

### 3. Excel Data Layer (OneDrive)

**Design Principles**:
- **Normalized schema**: Avoid redundancy, ensure consistency
- **Single source of truth**: Each fact recorded once
- **Dimensional model**: Star schema for analytics
- **Auto-calculated fields**: DAX measures in Power BI, not Excel formulas (for performance)

**Core Tables** (see `DATABASE_DESIGN.md` for detailed schema):

```
DIMENSION TABLES
├─ DIM_Dates
│  ├─ DateKey (YYYYMMDD)
│  ├─ Date, Year, Month, Quarter, Day of Week
│  ├─ Week of Year, Is_WeekEnd, Is_Holiday
│  └─ (Pre-populated for 3+ years)
│
├─ DIM_Products  
│  ├─ ProductKey (surrogate key)
│  ├─ ProductID, ProductName, Category
│  ├─ UnitOfMeasure, ReorderLevel
│  └─ DateInserted, LastModified
│
└─ DIM_Vendors
   ├─ VendorKey (surrogate key)
   ├─ VendorID, VendorName, Contact
   ├─ Address, Phone, Email
   └─ DateInserted

FACT TABLES (Transaction-level)
├─ FACT_Sales
│  ├─ SalesKey (surrogate key)
│  ├─ DateKey (FK → DIM_Dates)
│  ├─ ProductKey (FK → DIM_Products) [Revenue Stream]
│  ├─ Amount, PaymentMethod
│  ├─ CreatedBy, CreatedDate, Timestamp
│  └─ [Supports: daily/hourly aggregation]
│
├─ FACT_Purchases
│  ├─ PurchaseKey (surrogate key)
│  ├─ DateKey (FK → DIM_Dates)
│  ├─ VendorKey (FK → DIM_Vendors)
│  ├─ ProductKey (FK → DIM_Products)
│  ├─ Quantity, UnitPrice, TotalAmount
│  ├─ PurchaseOrderNum, InvoiceNum
│  ├─ CreatedBy, CreatedDate, Timestamp
│  └─ [Supports: supplier analysis, cost tracking]
│
└─ FACT_Inventory
   ├─ InventoryKey (surrogate key)
   ├─ DateKey (FK → DIM_Dates)
   ├─ ProductKey (FK → DIM_Products)
   ├─ MovementType (in/out/adjustment)
   ├─ Quantity, BeforeBalance, AfterBalance
   ├─ Reason, CreatedBy, Timestamp
   └─ [Supports: turnover analysis, movement history]

OPERATIONAL TABLES (Summary/Reference)
├─ SalesControl_Daily (auto-updated by Power Automate)
├─ InventorySummary (current quantity + location)
└─ AuditLog (all transactions with user, timestamp)
```

**Key Design Decisions**:
- **Surrogate keys**: All dimensions use numeric keys instead of natural keys (future-proofs for SQL migration)
- **Slowly Changing Dimensions**: DIM_Products & DIM_Vendors track DateInserted/LastModified (Type 1: overwrite)
- **Timezone handling**: All timestamps in UTC; Power BI converts to local
- **Soft deletes**: Deleted products/vendors marked inactive, not removed (audit trail)

### 4. Power BI Analytics

**Data Model**:
```
Power BI → Excel (live connection, DirectQuery)
           ├─ Import DIM_Dates, DIM_Products, DIM_Vendors
           └─ Query FACT_* tables (100K+ rows OK with DirectQuery)

Calculated Measures (DAX):
├─ [Total Revenue] = SUM(FACT_Sales[Amount])
├─ [Revenue YoY %] = CALCULATE([Total Revenue], DATEADD(...))
├─ [Inventory Turnover] = [Cost of Goods Sold] / [Avg Inventory]
├─ [Avg Days Inventory] = 365 / [Inventory Turnover]
├─ [Purchase Efficiency] = [Total Purchases] / [Unique Vendors]
└─ (20+ measures for KPIs)

Dashboards:
├─ Revenue Dashboard (daily/monthly trends, by stream)
├─ Inventory Dashboard (levels, turnover, low-stock alerts)
├─ Procurement Dashboard (vendor performance, cost analysis)
├─ Operations Summary (holistic KPIs, comparative views)
└─ Detailed Reports (drill-down capabilities)
```

**Refresh Strategy**:
- **Scheduled**: Daily 6 AM (configurable)
- **On-demand**: Manual refresh available
- **Incremental**: Can implement if FACT tables exceed 500K rows

## Data Flow Diagram

```
User (Field Operations)
  │
  ├─ Opens Power Apps on tablet/phone
  │  │
  │  ├─ Selects transaction type (sale/inventory/purchase)
  │  │  │
  │  │  ├─ App loads dimension data (read) from Excel
  │  │  │  │
  │  │  │  └─ Populates dropdowns (products, vendors, etc.)
  │  │  │
  │  │  └─ User enters transaction details + submits
  │  │     │
  │  │     └─ Power Apps triggers Power Automate flow
  │  │        │
  │  │        ├─ Validates data
  │  │        │  └─ If invalid: return error to app
  │  │        │
  │  │        └─ Transforms & appends to Excel FACT table
  │  │           │
  │  │           ├─ Logs to AuditLog
  │  │           ├─ Updates InventorySummary (if applicable)
  │  │           └─ Sends confirmation + tracking ID
  │  │
  │  └─ User sees confirmation screen
  │
  └─ (Every 5-15 min) Power BI refreshes
     └─ Pulls updated FACT tables from Excel
        └─ Recalculates KPI measures
           └─ Dashboards update automatically
              └─ Stakeholders view latest analytics

Evening Admin Review:
  └─ Opens Power BI dashboard
     └─ Reviews daily/weekly KPIs
     └─ Takes action based on insights
```

## Performance Considerations

### Current Scale (Operating Range)
- **Transactions/month**: ~100-500 (small operations)
- **Excel file size**: 2-10 MB (acceptable in OneDrive)
- **Power BI refresh**: < 2 minutes
- **Power Apps load time**: < 3 seconds

### Scaling Limits & Migration Path

| Metric | Safe Limit | Action Needed |
|--------|-----------|---------------|
| Transactions/month | 500-1000 | Monitor refresh time |
| Transactions/month | 1000-5000 | Consider SQL Server (on-prem) or Cosmos DB |
| Excel file size | > 50 MB | Implement incremental refresh or archive old data |
| Power BI refresh | > 10 min | Add aggregation tables, implement incremental refresh |
| Concurrent users | > 20 | Publish Power BI to Premium capacity |

### Migration to SQL Server
If volumes exceed 1000 transactions/month:
1. Export normalized schema to SQL Server / PostgreSQL
2. Maintain Excel for backward compatibility (view only)
3. Point Power Automate to SQL instead of Excel
4. Point Power BI DirectQuery to SQL (identical model)
5. Estimated effort: 1-2 weeks

## Security Architecture

### Access Control

**Power Apps**:
- Azure AD integration (optional)
- Row-level security via context variables
- Audit trail of all submissions logged

**Excel (OneDrive)**:
- Built-in OneDrive sharing (per-user permissions)
- Managed access via Microsoft 365 admin
- Version history (30-day retention)

**Power Automate**:
- Connection-level auth (stored in Flow Service)
- No hardcoded credentials
- Service account for Excel access (optional)

**Power BI**:
- Row-level security (filter data by org unit if applicable)
- Granular sharing (specific users, groups)
- Audit logging in admin portal

### Data Protection
- **Encryption in transit**: HTTPS/TLS for all connections
- **Encryption at rest**: OneDrive automatic encryption
- **Backup**: OneDrive version history (30 days), Microsoft backup (90 days)

## Deployment Architecture

```
Development         → Testing          → Production
─────────────────────────────────────────────────────
Power Apps Dev       → Power Apps Dev   → Power Apps Prod
(local save)            (test org)         (live org)
     │                      │                  │
Power Automate Dev      Power Automate    Power Automate
(saved as draft)        (published)       (published)
     │                      │                  │
Excel (OneDrive Dev)    Excel (OneDrive)  Excel (OneDrive Prod)
```

**CI/CD Strategy**:
- Manual promotion (no automated deployment pipeline required)
- Test flows before publishing
- Excel backups before schema changes
- Change log in CHANGELOG.md

## Integration Points

### Available Connectors (for future extensions)
- **Microsoft Teams**: Notification of anomalies
- **Slack**: Daily digest of KPIs
- **SharePoint**: Document storage for invoices/receipts
- **Dynamics 365**: CRM integration for customer-linked transactions
- **Custom APIs**: Webhook triggers for external systems

### Current Integrations
- Power Apps ↔ Excel (read/write)
- Power Automate ↔ Excel (table operations, HTTP)
- Power BI ↔ Excel (DirectQuery live)

## Monitoring & Alerts

**Implemented**:
- Audit log in Excel (all transactions)
- Email alerts on Power Automate failures
- Power BI visual alerts (low inventory, revenue anomalies)

**Recommended**:
- Daily scheduled email digest of summary KPIs
- Slack notification on critical errors
- Excel file health checks (Power Automate scheduled flow)

---

**For setup instructions**, see [`DEPLOYMENT.md`](./DEPLOYMENT.md)  
**For database details**, see [`DATABASE_DESIGN.md`](./DATABASE_DESIGN.md)
