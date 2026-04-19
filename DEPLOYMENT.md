# Deployment & Setup Guide

**Time to completion**: 30-45 minutes  
**Prerequisites**: Microsoft 365 account, Power Apps Studio, Power BI Desktop

## Overview

This guide walks through setting up the complete Business Operations System from scratch. Each step is sequenced to handle dependencies (e.g., Excel structure before Power Apps binding).

## Phase 1: Data Layer Setup (10 minutes)

### Step 1: Create OneDrive Folder Structure

1. Go to OneDrive → Create a new folder: `BusinessOperations_Data`
2. Inside, create these sub-folders:
   - `Tables` (for core data tables)
   - `Archives` (for backups)
   - `Reports` (for Power BI files)

### Step 2: Create Excel Workbook

1. Open Excel Online or Desktop
2. Create new workbook: `OperationalData.xlsx`
3. Create these worksheet tabs:
   ```
   DIM_Dates
   DIM_Products
   DIM_Vendors
   FACT_Sales
   FACT_Purchases
   FACT_Inventory
   InventorySummary
   AuditLog
   SalesControl_Daily
   ```

4. **Populate DIM_Dates** (required before anything else):
   - See `data/sample_data/sample_dates.xlsx` for template
   - Generate dates from 2023-01-01 to 2026-12-31
   - Copy sample data into DIM_Dates sheet
   - **Tip**: Use Excel `Fill Series` feature for date generation

5. **Populate DIM_Products**:
   - See `data/sample_data/sample_products.xlsx`
   - Add rows for your specific revenue streams
   - Add rows for inventory items
   - Copy sample data, update with your product names

6. **Populate DIM_Vendors**:
   - See `data/sample_data/sample_vendors.xlsx`
   - Add your supplier information
   - At minimum: VendorID, VendorName, Phone, Email

7. **Initialize FACT tables** (headers only, no data yet):
   - Copy headers from `data/sample_data/`
   - Leave rows empty (will be populated by Power Automate)

8. **Add Excel Table formatting**:
   - Select each sheet
   - Format as Table: `Insert` → `Table`
   - Name each table: `tblDIM_Dates`, `tblFACT_Sales`, etc.
   - **Why**: Power Automate needs structured table references

9. **Save & upload to OneDrive**:
   - Save locally first
   - Upload to `BusinessOperations_Data/Tables/OperationalData.xlsx`

✅ **Checkpoint**: You have Excel structure with dimension data ready

---

## Phase 2: Power Automate Setup (10 minutes)

### Step 3: Create Cloud Flow Connections

1. Go to Power Automate (make.powerautomate.com)
2. Create connections to:
   - **Excel Online (Business)**: Authorize with your M365 account
   - **Office 365 Outlook**: For email notifications
   - **OneDrive for Business**: For file access

### Step 4: Import Power Automate Flows

1. Download all 7 flow JSON files from `power-automate/flows/`
2. In Power Automate, go to `My cloud flows` → `Cloud flows`
3. For each flow:
   - Click `Import` → Select JSON file
   - Map connections: Select Excel, Outlook, OneDrive connections created in Step 3
   - Update Excel workbook reference: `OperationalData.xlsx`
   - Update Excel table names: `tblFACT_Sales`, `tblFACT_Purchases`, etc.
   - Click `Import`

**Flows to import** (in order):
1. `01-RegistroVentas.json` - Triggered by Power Apps
2. `02-RegistroCompras.json` - Triggered by Power Apps
3. `03-GestionInventario.json` - Triggered by Power Apps
4. `04-RegistroEstacionamiento.json` - Scheduled (daily aggregation)
5. `05-RegistroBaños.json` - Scheduled (daily aggregation)
6. `06-AgregarProducto.json` - Manual trigger (for admin use)
7. `07-EducacionAmbiental.json` - Triggered by Power Apps

### Step 5: Test Power Automate Flows

For each cloud flow:
1. Open the flow
2. Click `Test` → `Manually`
3. Provide sample input (e.g., SalesAmount = 5000, ProductKey = 1)
4. Click `Run flow`
5. Verify:
   - No red X errors
   - Check Excel workbook → verify row was added to appropriate FACT table
   - Check AuditLog → verify transaction was logged

✅ **Checkpoint**: Power Automate flows can read/write to Excel successfully

---

## Phase 3: Power Apps Setup (8 minutes)

### Step 6: Import Power Apps Application

1. Go to Power Apps Studio (make.powerapps.com)
2. Click `Create` → `Canvas app from blank`
   - **Alternative**: `Import` → Select `Prototipo_PPL_Operativo.msapp`

3. If importing the app:
   - Upload `powerapps/Prototipo_PPL_Operativo.msapp`
   - During import, map data sources:
     - Excel Online (Business) → OperationalData.xlsx
     - Power Automate flows → The 7 flows created in Step 4

### Step 7: Configure Power Apps Data Sources

1. In Power Apps Studio, go to `Data` (left sidebar)
2. Click `Add data`
3. Connect to:
   - Excel Online → OperationalData.xlsx → DIM_Products (for product dropdown)
   - Excel Online → OperationalData.xlsx → DIM_Vendors (for vendor dropdown)
   - Excel Online → OperationalData.xlsx → InventorySummary (for current stock)

4. For each form (Sales, Purchases, Inventory):
   - Bind data sources to Excel tables
   - Set form controls (dropdowns, text inputs) to reference Excel columns
   - Configure validation rules (see screenshots in `Manual de Usuario`)

### Step 8: Test Power Apps

1. Click `Play` (preview mode)
2. Test each form:
   - **Sales entry**: Fill in sale, submit → check Excel (FACT_Sales row added)
   - **Purchase entry**: Fill in PO, submit → check Excel (FACT_Purchases row added)
   - **Inventory move**: Record movement → check Excel (FACT_Inventory row added)

3. Verify each submission triggers Power Automate flow (check Power Automate run history)

✅ **Checkpoint**: Power Apps can submit data, triggering flows, writing to Excel

---

## Phase 4: Power BI Setup (8 minutes)

### Step 9: Create Power BI Model

1. Open Power BI Desktop
2. `Home` → `Get Data` → `Excel`
3. Navigate to OneDrive → OperationalData.xlsx
4. Select all tables (DIM_Dates, DIM_Products, DIM_Vendors, FACT_Sales, FACT_Purchases, FACT_Inventory)
5. Click `Load`

### Step 10: Create Relationships

In Power BI Desktop, go to `Model` view:

Create these relationships:
```
FACT_Sales.DateKey → DIM_Dates.DateKey (1:N)
FACT_Sales.ProductKey → DIM_Products.ProductKey (1:N)

FACT_Purchases.DateKey → DIM_Dates.DateKey (1:N)
FACT_Purchases.VendorKey → DIM_Vendors.VendorKey (1:N)
FACT_Purchases.ProductKey → DIM_Products.ProductKey (1:N)

FACT_Inventory.DateKey → DIM_Dates.DateKey (1:N)
FACT_Inventory.ProductKey → DIM_Products.ProductKey (1:N)
```

**Verify**:
- Each relationship shows cardinality (1:* is correct)
- No circular relationships
- All FKs are integers (matching PK types)

### Step 11: Create Calculated Measures

In Power BI, go to `Data` view and create these measures:

```DAX
// Revenue Metrics
Total Revenue = SUM(FACT_Sales[Amount])

Revenue YoY Growth % = 
VAR CurrentYear = YEAR(TODAY())
VAR PriorYear = CurrentYear - 1
VAR CurrentYearRevenue = 
  CALCULATE(
    [Total Revenue],
    YEAR(DIM_Dates[Date]) = CurrentYear
  )
VAR PriorYearRevenue = 
  CALCULATE(
    [Total Revenue],
    YEAR(DIM_Dates[Date]) = PriorYear
  )
RETURN
  DIVIDE(CurrentYearRevenue - PriorYearRevenue, PriorYearRevenue, 0)

Transaction Count = COUNTA(FACT_Sales[SalesKey])

Average Transaction = DIVIDE([Total Revenue], [Transaction Count], 0)

// Inventory Metrics
Total Units Sold = SUM(FACT_Inventory[Quantity])

Current Inventory Value = 
SUMPRODUCT(
  InventorySummary[CurrentQuantity],
  RELATED(DIM_Products[UnitPrice])
)

Inventory Turnover = 
DIVIDE(
  [Total Units Sold],
  [Current Inventory Value],
  0
)

// Procurement Metrics
Total Purchases = SUM(FACT_Purchases[GrandTotal])

Vendor Count = DISTINCTCOUNT(FACT_Purchases[VendorKey])

Average Order Value = DIVIDE([Total Purchases], COUNTA(FACT_Purchases[PurchaseKey]), 0)
```

### Step 12: Create Dashboards

See `powerbi/README.md` for detailed dashboard specifications. Quick overview:

**Dashboard 1: Revenue Overview**
- Card: Total Revenue (YTD)
- Card: Transaction count
- Card: Average transaction value
- Line chart: Revenue by day (last 30 days)
- Bar chart: Revenue by product/stream
- Slicer: Date range (for filtering)

**Dashboard 2: Inventory Status**
- Card: Total items in stock (by category)
- Card: Items below reorder level (alert count)
- Table: Current inventory (all products, quantities)
- Slicer: Product category
- Chart: Inventory turnover by product

**Dashboard 3: Procurement Analysis**
- Card: Total spend (YTD)
- Card: Vendor count
- Bar chart: Spend by vendor
- Line chart: Average unit cost trends
- Table: Recent purchases detail

### Step 13: Publish to Power BI Service

1. In Power BI Desktop, `File` → `Publish`
2. Select workspace (or create new: "BusinessOperations")
3. Click `Select`
4. Go to Power BI Service (app.powerbi.com)
5. Open workspace → Your published report
6. Pin key visuals to dashboard

### Step 14: Schedule Refresh

In Power BI Service:
1. Navigate to dataset (`OperationalData`)
2. `Settings` → `Scheduled refresh`
3. Enable refresh: Daily at 6:00 AM
4. Frequency: Daily
5. Notification: "Send refresh failure notification to..."

✅ **Checkpoint**: Power BI dashboards refresh automatically from Excel data

---

## Phase 5: End-to-End Testing (5 minutes)

### Step 15: Complete Workflow Test

1. **Day 1: Test data entry**
   - Open Power Apps on tablet/mobile
   - Submit a sales transaction (e.g., parking fee)
   - Verify: 
     - Submission confirmed in app
     - Power Automate flow triggered (check flow history)
     - Row appeared in FACT_Sales
     - AuditLog updated

2. **Day 2: Test analytics**
   - Open Power BI dashboard
   - Verify new transaction is reflected in charts
   - Check "Total Revenue" card updated
   - Filter by product → verify count correct

3. **Day 3: Test inventory**
   - Submit inventory movement in Power Apps
   - Check FACT_Inventory row added
   - Check InventorySummary updated
   - Verify Power BI "Current Inventory" reflects change

4. **Day 4: Test purchasing**
   - Create purchase order in Power Apps
   - Verify FACT_Purchases row added
   - Check PO number auto-generated
   - Verify "Total Purchases" KPI in Power BI updated

✅ **Checkpoint**: Complete data pipeline works: App → Automate → Excel → BI

---

## Production Checklist

Before going live:

- [ ] Excel workbook backed up to OneDrive Archive folder
- [ ] All Power Automate flows tested and published
- [ ] Power Apps users trained (show them "Manual de Usuario")
- [ ] Power BI dashboards shared with decision-makers
- [ ] Scheduled refresh enabled (Power BI)
- [ ] Email notifications configured (Power Automate errors)
- [ ] Audit trail enabled (Excel version history enabled)
- [ ] Data validation rules tested
- [ ] Sample data removed from production (if not needed)
- [ ] User access permissions configured (OneDrive sharing, Power Apps sharing)

---

## Troubleshooting

### Issue: Power Automate flow fails with "Excel table not found"
**Solution**: 
- Verify table names match exactly: `tblFACT_Sales`, `tblDIM_Products`, etc.
- Check workbook is in OneDrive (not local disk)
- Refresh flow connection: `Edit flow` → `Save`

### Issue: Power Apps shows "No data" when selecting products
**Solution**:
- Ensure DIM_Products has data (not empty)
- Check Excel Online connection is authorized
- Try "Refresh": `Insert` → `Refresh` button in app

### Issue: Power BI shows "No data" or blank charts
**Solution**:
- Verify relationships created correctly (`Model` view)
- Check date range in slicer matches data dates
- Try `Refresh` in Power BI Service
- Ensure FACT tables have rows (not empty)

### Issue: Scheduled refresh fails
**Solution**:
- Check Power BI email for error message
- Verify Excel file is still at expected path
- Try manual refresh first: `Settings` → `Refresh now`
- Re-authenticate Excel Online connection in Power BI

### Issue: Power Apps form won't submit
**Solution**:
- Check validation rules (red error messages on form?)
- Verify required fields are filled
- Check Power Automate flow is published (not draft)
- Try submitting from different browser/device

---

## Next Steps

Once live:
1. **Monitor**: Check Power Automate run history weekly (any failures?)
2. **Gather feedback**: Ask field users about app usability
3. **Optimize**: Review Power BI dashboards - are users asking for different views?
4. **Extend**: Add more revenue streams or inventory categories as business grows
5. **Archive**: Move old data (> 2 years) to archive for performance

---

For more details on specific components:
- [`ARCHITECTURE.md`](./ARCHITECTURE.md) - System design
- [`DATABASE_DESIGN.md`](./DATABASE_DESIGN.md) - Data model details
- [`power-automate/CONNECTIONS_GUIDE.md`](./power-automate/CONNECTIONS_GUIDE.md) - Flow configuration
- [`powerbi/README.md`](./powerbi/README.md) - Dashboard details
- [`powerapps/SETUP.md`](./powerapps/SETUP.md) - App import & customization
