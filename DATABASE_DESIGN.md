# Database Design & Schema

## Design Principles

- **Normalization**: 3NF for dimension tables, fact tables follow transaction-level grain
- **Surrogate Keys**: Numeric keys (INT) for all dimensions to support slow-changing dimensions
- **Immutability**: FACT tables are append-only (no updates to historical transactions)
- **Audit Trail**: Timestamps and user tracking on all tables
- **Dimensional Model**: Star schema optimized for analytical queries (Power BI)

## Entity-Relationship Diagram (ERD)

```
┌──────────────────┐          ┌──────────────────┐          ┌──────────────────┐
│   DIM_Dates      │          │   DIM_Products   │          │   DIM_Vendors    │
├──────────────────┤          ├──────────────────┤          ├──────────────────┤
│ DateKey (PK)     │          │ ProductKey (PK)  │          │ VendorKey (PK)   │
│ Date             │          │ ProductID        │          │ VendorID         │
│ Year             │          │ ProductName      │          │ VendorName       │
│ Month            │◄──────┐  │ Category         │◄──────┐  │ Address          │
│ Quarter          │       │  │ UnitOfMeasure    │       │  │ Phone            │
│ DayOfWeek        │       │  │ ReorderLevel     │       │  │ Email            │
│ IsWeekend        │       │  │ IsActive         │       │  │ IsActive         │
│ IsHoliday        │       │  │ DateInserted     │       │  │ DateInserted     │
│                  │       │  │ LastModified     │       │  │ LastModified     │
└──────────────────┘       │  └──────────────────┘       │  └──────────────────┘
                           │                             │
                           │      ┌─────────────┐        │
                           │      │ FACT_Sales  │        │
                           │      ├─────────────┤        │
                           │      │ SalesKey    │        │
                           └──────┤ DateKey (FK)│        │
                                  │ ProductKey  │        │
                                  │ Amount      │        │
                                  │ PaymentType │        │
                                  │ CreatedBy   │        │
                                  │ CreatedDate │        │
                                  │ Timestamp   │        │
                                  └─────────────┘        │
                           ┌──────────────────┐          │
                           │ FACT_Purchases   │          │
                           ├──────────────────┤          │
                           │ PurchaseKey      │          │
                           │ DateKey (FK)─────┼──────────┘
                           │ VendorKey (FK)───┼──────────┐
                           │ ProductKey (FK)──┼──────────┘
                           │ Quantity         │
                           │ UnitPrice        │
                           │ TotalAmount      │
                           │ PurchaseOrderNum │
                           │ CreatedBy        │
                           │ CreatedDate      │
                           │ Timestamp        │
                           └──────────────────┘
                                  ▲
                           ┌──────┴──────────┐
                           │ FACT_Inventory  │
                           ├─────────────────┤
                           │ InventoryKey    │
                           │ DateKey (FK)    │
                           │ ProductKey (FK) │
                           │ MovementType    │
                           │ Quantity        │
                           │ BeforeBalance   │
                           │ AfterBalance    │
                           │ Reason          │
                           │ CreatedBy       │
                           │ Timestamp       │
                           └─────────────────┘
```

## Detailed Table Schemas

### DIMENSION: DIM_Dates

**Purpose**: Time dimension for all fact tables. Pre-populated for 5 years.

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| **DateKey** | INTEGER | NO | YYYYMMDD format (e.g., 20250401) - Primary Key |
| **Date** | DATE | NO | Actual date (2025-04-01) |
| **Year** | INTEGER | NO | Calendar year (2025) |
| **YearString** | TEXT | NO | Year as string ('2025') |
| **Quarter** | INTEGER | NO | Quarter number (1-4) |
| **Month** | INTEGER | NO | Month number (1-12) |
| **MonthName** | TEXT | NO | Full month name ('April') |
| **MonthShort** | TEXT | NO | Abbreviated month ('Apr') |
| **Week** | INTEGER | NO | Week of year (1-52) |
| **DayOfWeek** | INTEGER | NO | Day number (1=Monday, 7=Sunday) |
| **DayName** | TEXT | NO | Day name ('Friday') |
| **IsWeekday** | INTEGER | NO | 1=weekday, 0=weekend |
| **IsWeekend** | INTEGER | NO | 1=Saturday/Sunday, 0=other |
| **IsHoliday** | INTEGER | NO | 1=public holiday, 0=regular (Chile holidays) |
| **IsYearStart** | INTEGER | NO | 1=January 1st, 0=other |
| **IsYearEnd** | INTEGER | NO | 1=December 31st, 0=other |

**Sample Data**:
```
DateKey | Date       | Year | Quarter | Month | MonthName | DayOfWeek | DayName  | IsWeekend | IsHoliday
--------|------------|------|---------|-------|-----------|-----------|----------|-----------|----------
20250101| 2025-01-01 | 2025 | 1       | 1     | January   | 3         | Wednesday| 0         | 1 (New Year)
20250102| 2025-01-02 | 2025 | 1       | 1     | January   | 4         | Thursday | 0         | 0
20250104| 2025-01-04 | 2025 | 1       | 1     | January   | 6         | Saturday | 1         | 0
```

**Populate with SQL/Power Automate**:
```
Generate dates from 2020-01-01 to 2030-12-31
Calculate attributes (year, month, week, day of week)
Mark holidays (Chile national holidays)
```

---

### DIMENSION: DIM_Products

**Purpose**: Master data for all products (revenue streams, inventory items). Supports Slowly Changing Dimension Type 1.

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| **ProductKey** | INTEGER | NO | Surrogate key (auto-increment) - Primary Key |
| **ProductID** | TEXT | NO | Natural key (business identifier) - Unique |
| **ProductName** | TEXT | NO | Full product name |
| **Category** | TEXT | NO | Classification (e.g., 'Parking', 'Entry Fee', 'Merchandise', 'Beverage') |
| **SubCategory** | TEXT | YES | Further classification if needed |
| **UnitOfMeasure** | TEXT | NO | Unit (e.g., 'Each', 'Liter', 'KG', 'hour') |
| **ReorderLevel** | DECIMAL | YES | Minimum quantity threshold (for inventory alerts) |
| **ReorderQuantity** | DECIMAL | YES | Standard order quantity |
| **UnitPrice** | DECIMAL | YES | Standard selling price (informational, not normative) |
| **IsActive** | INTEGER | NO | 1=active, 0=inactive (soft delete) |
| **EffectiveDate** | DATE | NO | Date this record became effective |
| **DateInserted** | DATETIME | NO | Audit timestamp (never changes) |
| **LastModified** | DATETIME | NO | Last update timestamp |
| **ModifiedBy** | TEXT | YES | User who last modified |

**Sample Data**:
```
ProductKey | ProductID | ProductName        | Category | ReorderLevel | IsActive
-----------|-----------|-------------------|----------|--------------|----------
1          | PAR001    | Parking Daily      | Parking  | NULL         | 1
2          | ENT001    | Entry Single Adult | Entry    | NULL         | 1
3          | MER001    | T-Shirt Large      | Merch    | 10           | 1
4          | INV002    | Bottled Water      | Beverage | 50           | 1
5          | INV003    | Sunscreen SPF50    | Supplies | 5            | 1
```

**Considerations**:
- **For revenue streams** (parking, entry): ProductID = revenue code, ReorderLevel = NULL
- **For inventory items**: ProductID = SKU, ReorderLevel = min stock threshold
- **Soft deletes**: Set IsActive=0, don't remove rows (preserves referential integrity)
- **Slowly Changing**: When ProductName changes, overwrite LastModified + ModifiedBy (Type 1)

---

### DIMENSION: DIM_Vendors

**Purpose**: Master data for suppliers/vendors.

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| **VendorKey** | INTEGER | NO | Surrogate key (auto-increment) - Primary Key |
| **VendorID** | TEXT | NO | Natural key (business ID) - Unique |
| **VendorName** | TEXT | NO | Company/supplier name |
| **ContactName** | TEXT | YES | Primary contact person |
| **Address** | TEXT | YES | Mailing address |
| **City** | TEXT | YES | City |
| **Phone** | TEXT | YES | Phone number |
| **Email** | TEXT | YES | Email address |
| **PaymentTerms** | TEXT | YES | Terms (e.g., 'Net 30', 'COD') |
| **IsActive** | INTEGER | NO | 1=active, 0=inactive (soft delete) |
| **EffectiveDate** | DATE | NO | Date relationship started |
| **DateInserted** | DATETIME | NO | Audit timestamp |
| **LastModified** | DATETIME | NO | Last update timestamp |
| **ModifiedBy** | TEXT | YES | User who last modified |

**Sample Data**:
```
VendorKey | VendorID | VendorName | ContactName | Phone | Email | IsActive
-----------|----------|-----------|------------|-------|-------|----------
1          | VEN001   | Local Supplies | Juan | 227123456 | juan@local.cl | 1
2          | VEN002   | Metro Dist. | María | 227234567 | maria@metro.cl | 1
3          | VEN003   | Bio Clean (discontinued) | Pedro | 227345678 | pedro@bio.cl | 0
```

---

### FACT: FACT_Sales

**Purpose**: Transaction-level sales detail. Append-only (no updates).

**Grain**: One row per transaction (by revenue stream)

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| **SalesKey** | INTEGER | NO | Surrogate key (auto-increment) - Primary Key |
| **DateKey** | INTEGER | NO | FK → DIM_Dates (transaction date) |
| **ProductKey** | INTEGER | NO | FK → DIM_Products (revenue stream) |
| **Amount** | DECIMAL(10,2) | NO | Transaction amount (in local currency) |
| **Currency** | TEXT | NO | Currency code ('CLP', 'USD') |
| **PaymentMethod** | TEXT | YES | Payment type ('Cash', 'Card', 'Check', 'Online') |
| **TransactionID** | TEXT | YES | External identifier (if from POS system) |
| **Quantity** | DECIMAL | YES | Number of items sold (if applicable) |
| **Discount** | DECIMAL(10,2) | YES | Discount applied (if any) |
| **Notes** | TEXT | YES | Free-form notes |
| **CreatedBy** | TEXT | NO | User who entered transaction (audit) |
| **CreatedDate** | DATE | NO | Date transaction was recorded |
| **Timestamp** | DATETIME | NO | Exact timestamp of entry |
| **IsActive** | INTEGER | NO | 1=valid, 0=voided (for corrections) |

**Sample Data**:
```
SalesKey | DateKey  | ProductKey | Amount | PaymentMethod | CreatedBy | CreatedDate | Timestamp
---------|----------|-----------|--------|---------------|-----------|------------|------------------------
1        | 20250401 | 1         | 5000   | Cash          | jsmith    | 2025-04-01 | 2025-04-01 08:30:15
2        | 20250401 | 2         | 12000  | Card          | jsmith    | 2025-04-01 | 2025-04-01 09:15:22
3        | 20250401 | 1         | 5000   | Cash          | mgarcia   | 2025-04-01 | 2025-04-01 10:45:33
```

**Analytical Queries**:
```
Total revenue by day: SELECT DateKey, SUM(Amount) FROM FACT_Sales GROUP BY DateKey
Revenue by stream: SELECT ProductKey, SUM(Amount) FROM FACT_Sales GROUP BY ProductKey
Revenue by payment: SELECT PaymentMethod, COUNT(*), SUM(Amount) FROM FACT_Sales GROUP BY PaymentMethod
```

---

### FACT: FACT_Purchases

**Purpose**: Purchase order and receiving transaction detail.

**Grain**: One row per product per purchase order

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| **PurchaseKey** | INTEGER | NO | Surrogate key (auto-increment) - Primary Key |
| **DateKey** | INTEGER | NO | FK → DIM_Dates (purchase order date) |
| **VendorKey** | INTEGER | NO | FK → DIM_Vendors (supplier) |
| **ProductKey** | INTEGER | NO | FK → DIM_Products (item purchased) |
| **Quantity** | DECIMAL | NO | Number of units ordered |
| **UnitPrice** | DECIMAL(10,2) | NO | Price per unit at time of purchase |
| **TotalAmount** | DECIMAL(10,2) | NO | Quantity × UnitPrice (before tax) |
| **TaxAmount** | DECIMAL(10,2) | YES | Tax applied (e.g., VAT 19%) |
| **GrandTotal** | DECIMAL(10,2) | NO | TotalAmount + TaxAmount |
| **Currency** | TEXT | NO | Currency code ('CLP') |
| **PurchaseOrderNum** | TEXT | NO | PO number (natural identifier) - Unique |
| **InvoiceNum** | TEXT | YES | Vendor invoice number |
| **DeliveryDate** | DATE | YES | When goods were received |
| **Status** | TEXT | NO | 'Ordered', 'Received', 'Cancelled' |
| **CreatedBy** | TEXT | NO | User who created PO |
| **CreatedDate** | DATE | NO | Date PO was created |
| **Timestamp** | DATETIME | NO | Exact timestamp |

**Sample Data**:
```
PurchaseKey | DateKey  | VendorKey | ProductKey | Quantity | UnitPrice | TotalAmount | PurchaseOrderNum | Status
------------|----------|-----------|-----------|----------|-----------|------------|------------------|----------
1           | 20250401 | 1         | 4         | 100      | 500       | 50000      | PO-2025-0001     | Received
2           | 20250401 | 1         | 5         | 20       | 1200      | 24000      | PO-2025-0001     | Received
3           | 20250402 | 2         | 3         | 50       | 8000      | 400000     | PO-2025-0002     | Ordered
```

**Analytical Queries**:
```
Cost by vendor: SELECT VendorKey, SUM(GrandTotal) FROM FACT_Purchases GROUP BY VendorKey
Average unit cost: SELECT ProductKey, AVG(UnitPrice) FROM FACT_Purchases GROUP BY ProductKey
Cost trend: SELECT DateKey, SUM(GrandTotal) FROM FACT_Purchases GROUP BY DateKey
```

---

### FACT: FACT_Inventory

**Purpose**: Inventory movement history (inbound, outbound, adjustments). Append-only.

**Grain**: One row per inventory transaction

| Column | Type | Nullable | Description |
|--------|------|----------|-------------|
| **InventoryKey** | INTEGER | NO | Surrogate key (auto-increment) - Primary Key |
| **DateKey** | INTEGER | NO | FK → DIM_Dates (transaction date) |
| **ProductKey** | INTEGER | NO | FK → DIM_Products (item) |
| **MovementType** | TEXT | NO | 'In' (purchase received), 'Out' (sale), 'Adjustment' (correction), 'Return' |
| **Quantity** | DECIMAL | NO | Quantity moved (always positive) |
| **BeforeBalance** | DECIMAL | NO | Inventory level before this transaction |
| **AfterBalance** | DECIMAL | NO | Inventory level after this transaction |
| **Reason** | TEXT | YES | Explanation (e.g., 'Regular sale', 'Damaged goods', 'Inventory recount') |
| **ReferenceID** | TEXT | YES | Link to related transaction (e.g., SalesKey, PurchaseKey) |
| **Location** | TEXT | YES | Where item is stored (if applicable) |
| **CreatedBy** | TEXT | NO | User who recorded movement |
| **Timestamp** | DATETIME | NO | Exact timestamp |

**Sample Data**:
```
InventoryKey | DateKey  | ProductKey | MovementType | Quantity | BeforeBalance | AfterBalance | Reason | ReferenceID
-------------|----------|-----------|--------------|----------|---------------|--------------|--------|---------------
1            | 20250401 | 4         | In           | 100      | 50            | 150          | PO received | PO-2025-0001
2            | 20250401 | 4         | Out          | 5        | 150           | 145          | Sale | SALE-001
3            | 20250402 | 4         | Out          | 10       | 145           | 135          | Sale | SALE-002
4            | 20250403 | 4         | Adjustment   | -2       | 135           | 133          | Damage writeoff | ADJ-001
```

**Analytical Queries**:
```
Current inventory: SELECT ProductKey, AfterBalance FROM FACT_Inventory 
                   WHERE (ProductKey, Timestamp) IN (SELECT ProductKey, MAX(Timestamp) FROM FACT_Inventory GROUP BY ProductKey)

Inventory turnover: SELECT ProductKey, COUNT(*) as movements, SUM(CASE WHEN MovementType='Out' THEN Quantity ELSE 0 END) as units_sold
                    FROM FACT_Inventory GROUP BY ProductKey

Days in stock: SELECT ProductKey, AVG(AfterBalance) FROM FACT_Inventory GROUP BY ProductKey
```

---

### OPERATIONAL: InventorySummary

**Purpose**: Current inventory snapshot (auto-updated by Power Automate).

| Column | Type | Description |
|--------|------|-------------|
| ProductKey | INTEGER | FK → DIM_Products |
| CurrentQuantity | DECIMAL | Latest balance from FACT_Inventory |
| LastUpdated | DATETIME | When last transaction occurred |
| ReorderRequired | INTEGER | 1 if below ReorderLevel, else 0 |
| Location | TEXT | Current storage location |

**Updated by**: GestionInventario Power Automate flow (on every inventory transaction)

---

### OPERATIONAL: AuditLog

**Purpose**: Complete transaction log (compliance & troubleshooting).

| Column | Type | Description |
|--------|------|-------------|
| AuditKey | INTEGER | Primary key |
| TransactionType | TEXT | 'Sales', 'Purchase', 'Inventory', 'ProductMaster', 'VendorMaster' |
| TransactionID | TEXT | Foreign key to related table (e.g., SalesKey) |
| Action | TEXT | 'Insert', 'Update', 'Delete', 'Void' |
| ChangedFields | TEXT | JSON or pipe-delimited list of what changed |
| OldValue | TEXT | Previous value (for updates) |
| NewValue | TEXT | New value (for updates) |
| ChangedBy | TEXT | User who made change |
| ChangedDateTime | DATETIME | Exact timestamp |

**Updated by**: All Power Automate flows (automatic logging)

---

## Data Integrity Rules

### Constraints

```
DIM_Dates
  PRIMARY KEY (DateKey)
  UNIQUE (Date)
  CHECK (DateKey > 0)

DIM_Products
  PRIMARY KEY (ProductKey)
  UNIQUE (ProductID)
  FOREIGN KEY (Category) references allowed list
  CHECK (ReorderLevel >= 0)

DIM_Vendors
  PRIMARY KEY (VendorKey)
  UNIQUE (VendorID)

FACT_Sales
  PRIMARY KEY (SalesKey)
  FOREIGN KEY (DateKey) references DIM_Dates(DateKey)
  FOREIGN KEY (ProductKey) references DIM_Products(ProductKey)
  CHECK (Amount > 0)
  CHECK (Quantity >= 0 OR Quantity IS NULL)
  --> Enforced in Power Automate validation

FACT_Purchases
  PRIMARY KEY (PurchaseKey)
  UNIQUE (PurchaseOrderNum)
  FOREIGN KEY (DateKey) references DIM_Dates(DateKey)
  FOREIGN KEY (VendorKey) references DIM_Vendors(VendorKey)
  FOREIGN KEY (ProductKey) references DIM_Products(ProductKey)
  CHECK (Quantity > 0)
  CHECK (UnitPrice > 0)
  CHECK (GrandTotal = TotalAmount + COALESCE(TaxAmount, 0))

FACT_Inventory
  PRIMARY KEY (InventoryKey)
  FOREIGN KEY (DateKey) references DIM_Dates(DateKey)
  FOREIGN KEY (ProductKey) references DIM_Products(ProductKey)
  CHECK (Quantity >= 0)
  CHECK (AfterBalance >= 0)
  CHECK (BeforeBalance + Quantity = AfterBalance) [for In/Out]
```

### Referential Integrity
- All FK references enforced in Power Automate (no orphaned records)
- Soft deletes used (never hard-delete from DIM tables)
- If product is discontinued: set DIM_Products.IsActive = 0, keep historical FACT rows

---

## Index Strategy (for SQL migration)

When migrating to SQL Server/PostgreSQL:
```sql
-- Fact table indexes (for BI queries)
CREATE INDEX idx_FACT_Sales_DateKey ON FACT_Sales(DateKey)
CREATE INDEX idx_FACT_Sales_ProductKey ON FACT_Sales(ProductKey)
CREATE INDEX idx_FACT_Sales_CreatedDate ON FACT_Sales(CreatedDate)

CREATE INDEX idx_FACT_Purchases_DateKey ON FACT_Purchases(DateKey)
CREATE INDEX idx_FACT_Purchases_VendorKey ON FACT_Purchases(VendorKey)
CREATE INDEX idx_FACT_Purchases_ProductKey ON FACT_Purchases(ProductKey)

CREATE INDEX idx_FACT_Inventory_DateKey ON FACT_Inventory(DateKey)
CREATE INDEX idx_FACT_Inventory_ProductKey ON FACT_Inventory(ProductKey)
CREATE INDEX idx_FACT_Inventory_Timestamp ON FACT_Inventory(Timestamp)

-- Dimension table indexes
CREATE INDEX idx_DIM_Products_ProductID ON DIM_Products(ProductID)
CREATE INDEX idx_DIM_Products_Category ON DIM_Products(Category)
CREATE INDEX idx_DIM_Vendors_VendorID ON DIM_Vendors(VendorID)
```

---

## Data Flow Through Schema

```
User enters sale in Power Apps
  ↓
Power Automate: RegistroVentas
  ├─ Validates: Amount > 0, ProductKey exists
  ├─ Appends row to FACT_Sales (with DateKey, ProductKey, FKs)
  ├─ Logs to AuditLog (TransactionType='Sales', Action='Insert')
  └─ Updates InventorySummary if applicable
  ↓
Power BI queries FACT_Sales
  ├─ Joins with DIM_Dates (for year, month filtering)
  ├─ Joins with DIM_Products (for product name)
  ├─ Calculates measures: SUM(Amount), COUNT(*), etc.
  └─ Displays in dashboards
```

---

## Sample Data Population Script

See `data/sample_data/` for example Excel files with:
- DIM_Dates (1000 rows, 2023-2025)
- DIM_Products (15 rows, mixed revenue streams & inventory items)
- DIM_Vendors (5 rows)
- FACT_Sales (500 rows, realistic transactions)
- FACT_Purchases (100 rows)
- FACT_Inventory (300 rows, movements)

**To use**:
1. Download sample Excel files
2. Copy data into your production workbook
3. Update connections in Power Automate
4. Test end-to-end workflow

---

For setup instructions, see [`DEPLOYMENT.md`](./DEPLOYMENT.md)
