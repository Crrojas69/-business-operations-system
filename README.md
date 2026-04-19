# Business Operations Management System

**Integrated operational management platform** for multi-revenue tourism/hospitality centers. Automates income registration, inventory control, vendor management, and real-time analytics through a low-cost Microsoft 365 stack.

## 🎯 Problem Statement

Tourism and hospitality centers typically manage multiple revenue streams (accommodations, parking, food services, entry fees) and operational areas (inventory, purchasing, facility management) with disconnected Excel spreadsheets and manual processes, leading to:

- **Data silos**: Information scattered across files and email
- **Delayed insights**: Manual consolidation creates analysis bottlenecks  
- **Inventory blind spots**: No real-time tracking of stock movements
- **Compliance gaps**: Difficulty generating audit trails for financial reports

## ✅ Solution Overview

A **unified operational system** that captures data once and flows automatically through workflows into structured analytics—delivering real-time dashboards and actionable insights without requiring custom development or external cloud infrastructure.

### Key Capabilities

| Module | Capability | Outcome |
|--------|-----------|---------|
| **Sales & Revenue** | Mobile app for POS transactions (parking, entry, merchandise) | Real-time revenue tracking by stream & period |
| **Inventory** | Automated stock movements on purchase/sale | Visibility into inventory levels, turnover rates |
| **Purchasing** | Purchase order capture & vendor tracking | Cost analysis, supplier performance metrics |
| **Multi-module** | Unified data warehouse | Comparative analytics across all operations |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Power Apps Frontend                      │
│  (Mobile-optimized forms for field operations)               │
└────────────────────┬────────────────────────────────────────┘
                     │ Captures structured data
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                Power Automate Orchestration                  │
│  (7 automated workflows: sales, inventory, purchasing, etc.) │
└────────────────────┬────────────────────────────────────────┘
                     │ Transforms & validates
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Excel Data Lake (OneDrive)                      │
│  (Normalized tables: transactions, inventory, dimensions)    │
└────────────────────┬────────────────────────────────────────┘
                     │ Live connection
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                Power BI Dashboards                           │
│  (Real-time reporting: KPIs, trends, comparatives)           │
└─────────────────────────────────────────────────────────────┘
```

**Technology Stack:**
- **Frontend**: Power Apps (Canvas app, optimized for mobile/tablet)
- **Integration**: Power Automate (7 cloud workflows, scheduled + trigger-based)
- **Data layer**: Excel workbook in OneDrive (normalized schema, 6 core tables)
- **Analytics**: Power BI (5+ dashboards, live query on Excel data model)
- **Infrastructure**: Microsoft 365 (no additional cloud costs)

## 📋 Project Structure

```
business-operations-system/
├── README.md (this file)
├── ARCHITECTURE.md (detailed technical architecture)
├── DATABASE_DESIGN.md (schema, normalization, relationships)
├── DEPLOYMENT.md (step-by-step replication guide)
├── CHANGELOG.md (version history & updates)
│
├── powerapps/
│   ├── Prototipo_PPL_Operativo.msapp (exported app file)
│   └── SETUP.md (how to import into Power Apps Studio)
│
├── power-automate/
│   ├── flows/ (7 workflow exports as .json with documentation)
│   │   ├── 01-RegistroVentas.md
│   │   ├── 02-RegistroCompras.md
│   │   ├── 03-GestionInventario.md
│   │   ├── 04-RegistroEstacionamiento.md
│   │   ├── 05-RegistroBaños.md
│   │   ├── 06-AgregarProducto.md
│   │   └── 07-EducacionAmbiental.md
│   └── CONNECTIONS_GUIDE.md (connection setup between PA and Excel)
│
├── powerbi/
│   ├── dashboards/ (PBIX files & export PDFs)
│   │   ├── 01-Dashboard_Ingresos.pbix
│   │   ├── 02-Dashboard_Inventario.pbix
│   │   ├── 03-Dashboard_Compras.pbix
│   │   └── README.md (dashboard guide)
│   └── DATA_MODEL.md (DAX formulas, relationships, calculated columns)
│
├── data/
│   ├── sample_data/ (anonymized example data)
│   │   ├── sample_ventas.xlsx
│   │   ├── sample_compras.xlsx
│   │   └── sample_inventario.xlsx
│   └── DATA_DICTIONARY.md (field definitions, units, sources)
│
├── .github/
│   └── workflows/ (CI/CD if extending to code-based deployment)
│
├── LICENSE (MIT)
└── .gitignore
```

## 🚀 Quick Start

### Prerequisites
- Microsoft 365 account (Power Apps, Power Automate, OneDrive)
- Power BI Desktop (for development) or Power BI Pro (for sharing)
- 30 minutes for setup

### Setup in 4 Steps

1. **Create Excel data structure** (from `data/sample_data/`)  
   - Copy normalized tables to OneDrive
   - See: [`DATABASE_DESIGN.md`](./DATABASE_DESIGN.md)

2. **Import Power Apps**  
   - Open `powerapps/Prototipo_PPL_Operativo.msapp` in Power Apps Studio
   - Bind to Excel data source  
   - See: [`powerapps/SETUP.md`](./powerapps/SETUP.md)

3. **Deploy Power Automate flows**  
   - Import 7 JSON files from `power-automate/flows/`
   - Configure Excel and email connections  
   - See: [`power-automate/CONNECTIONS_GUIDE.md`](./power-automate/CONNECTIONS_GUIDE.md)

4. **Create Power BI dashboards**  
   - Import PBIX files or rebuild from scratch using Excel live connection
   - See: [`powerbi/README.md`](./powerbi/README.md)

**Complete guide**: [`DEPLOYMENT.md`](./DEPLOYMENT.md)

## 📊 Key Metrics & Impact

**Operational improvements observed:**
- **Data capture**: ~150+ transactions/month automated (vs. manual entry)
- **Analysis time**: 70% reduction (from 2-3 hours to 30 minutes monthly)
- **Visibility**: Real-time KPIs instead of weekly/monthly reviews
- **Accuracy**: Automated validation reduces entry errors by ~85%
- **Cost**: $0 infrastructure (leverages existing M365)

## 🔐 Security & Data Privacy

- **No PII**: System uses anonymized example data  
- **Excel in OneDrive**: Built-in encryption, access control, version history
- **Power Apps**: Row-level security available via Azure AD
- **Audit trail**: Power Automate logs all workflow executions
- **Compliance-ready**: Can extend with DLP policies, data loss prevention

## 🛠️ Technical Highlights

### Database Design
- **Normalized schema**: 6 core tables (Sales, Inventory, Purchases, Vendors, Products, Dates)
- **Relationships**: Proper dimensional modeling (fact/dimension structure)
- **Data integrity**: Validations in Power Apps + constraints in Excel

### Power Automate Workflows
- **Trigger-based**: Data entry in app → immediate flow execution
- **Error handling**: Retry logic, email notifications on failure
- **Data transformation**: Validation, formatting, cross-table lookups

### Power BI Analytics
- **Live connection**: DirectQuery to Excel for real-time updates
- **Calculated measures**: DAX formulas for KPIs (revenue growth, inventory turnover, etc.)
- **Interactivity**: Slicers for date ranges, departments, revenue streams

## 📚 Documentation Map

| Document | Purpose |
|----------|---------|
| [`ARCHITECTURE.md`](./ARCHITECTURE.md) | System design, component interactions, data flows |
| [`DATABASE_DESIGN.md`](./DATABASE_DESIGN.md) | Schema, table definitions, relationships, normalization approach |
| [`DEPLOYMENT.md`](./DEPLOYMENT.md) | Step-by-step setup guide (30 min to production-ready) |
| [`powerapps/SETUP.md`](./powerapps/SETUP.md) | How to import & configure the app |
| [`power-automate/CONNECTIONS_GUIDE.md`](./power-automate/CONNECTIONS_GUIDE.md) | Workflow setup, connection strings, testing |
| [`powerbi/README.md`](./powerbi/README.md) | Dashboard overview, refresh settings, custom reports |
| [`data/DATA_DICTIONARY.md`](./data/DATA_DICTIONARY.md) | Field definitions, units, calculation methods |

## 🔄 Extensibility

This system is designed for modification:

- **Add new revenue streams**: Create form in Power Apps → add Power Automate flow → add Power BI measures
- **New dashboards**: Use Excel data directly in Power BI without code
- **Integrate external APIs**: Power Automate connects to 500+ services (Slack, Teams, SharePoint, etc.)
- **Scale to database**: Export normalized Excel schema to SQL / Snowflake when volume exceeds 100K rows/month

## 👨‍💼 Author Notes

This system demonstrates:
- **End-to-end data pipeline** design (capture → transform → store → analyze)
- **Low-code/no-code** architecture decision trade-offs
- **Operational BI** for decision-making vs. enterprise analytics
- **Practical problem-solving** with existing technology stack

Built as part of a professional internship focusing on operations management and data-driven decision support.

## 📄 License

MIT License - See [`LICENSE`](./LICENSE) file for details.

---

**Questions or Issues?**  
Refer to [`DEPLOYMENT.md`](./DEPLOYMENT.md) for troubleshooting, or review individual component docs in their directories.
