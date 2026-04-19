# Changelog

All notable changes to the Business Operations System are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-04-15

### Added
- Initial release: Complete operational management system
- Power Apps canvas app with 5 main modules (Sales, Inventory, Purchases, Facilities, Programs)
- 7 Power Automate workflows for data orchestration and validation
- Excel data model with 6 normalized dimension/fact tables
- Power BI dashboard suite (3 dashboards: Revenue, Inventory, Procurement)
- Comprehensive documentation (Architecture, Database Design, Deployment Guide)
- Sample data for testing and training

### Features
- Real-time sales transaction capture from multiple revenue streams
- Automated inventory tracking with movement history
- Purchase order management with vendor tracking
- Daily facility operations summaries (parking, bathrooms)
- Environmental program attendance tracking
- Audit trail for all transactions
- Automated email alerts for critical events (low inventory, system errors)

### Technical
- Microsoft 365 stack: Power Apps, Power Automate, Excel, Power BI
- Normalized star schema (3NF)
- DirectQuery refresh strategy for BI
- Support for 100-500 monthly transactions at current scale

### Documentation
- `README.md` - Project overview & quick start
- `ARCHITECTURE.md` - System design, component interactions
- `DATABASE_DESIGN.md` - Data model, table schemas, normalization
- `DEPLOYMENT.md` - Step-by-step setup guide (30-45 minutes)
- `power-automate/CONNECTIONS_GUIDE.md` - Workflow configuration
- `powerbi/README.md` - Dashboard specifications & customization
- `powerapps/SETUP.md` - App import & configuration
- `data/DATA_DICTIONARY.md` - Field definitions & calculations

---

## [1.0.1] - 2025-04-20 (Planned)

### Planned Improvements
- Add multi-location support (region/branch filtering)
- Implement advanced forecasting in Power BI
- Add mobile app optimization to Power Apps
- Create user role-based access control

---

## Notes

### Version Strategy
- **MAJOR**: Breaking changes to data schema or architecture
- **MINOR**: New features or modules added
- **PATCH**: Bug fixes, performance improvements, documentation updates

### Maintenance Schedule
- **Weekly**: Monitor Power Automate run history for errors
- **Monthly**: Review and update sample data as needed
- **Quarterly**: Analyze usage patterns and collect user feedback
- **Annually**: Archive old data (> 2 years), optimize Excel model size
