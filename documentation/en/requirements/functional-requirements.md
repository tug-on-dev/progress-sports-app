# Functional Requirements (MoSCoW Method)

> **📖 Complete Version**: For detailed requirements analysis, see [French Functional Requirements](../../fr/requirements/functional-requirements.md)

## MoSCoW Categories

- **Must Have**: Critical features required for launch
- **Should Have**: Important but not critical features
- **Could Have**: Desirable features if time permits
- **Won't Have**: Features explicitly excluded from current scope

## Must Have (Critical)

### MH-001: Customer Data Display
- Display customer list with basic information
- **Implementation**: `customer-data.p`, `customer-view.html`
- **Fields**: CustNum, Country, Name, Address, Address2

### MH-002: State Data Display
- Display US states with regional information
- **Implementation**: `state-data.p`, `state-view.html`
- **Fields**: State, StateName, Region

### MH-003: Responsive Web Interface
- Browser-accessible web application
- Navigation menu
- Interactive data grids
- **Implementation**: `index.html`, `menu.html`, Kendo UI components

### MH-004: REST/JSON APIs
- JSON endpoints for data access
- **Endpoints**:
  - `/web/customer-data.p`
  - `/web/state-data.p`
- **Headers**: application/json

### MH-005: Database Connection
- Auto-connect to sports2020 database
- Failover support
- **Implementation**: `dbconnect.p`, `autoreconnect.pf`

## Should Have (Important)

### SH-001: Data Filtering and Sorting
- Column-based filtering
- Multi-column sorting
- Text search
- **Implementation**: Kendo UI grid configuration

### SH-002: Data Pagination
- Configurable page sizes
- Page navigation
- **Implementation**: Kendo UI DataSource

### SH-003: Analytics Dashboard
- Geographic distribution chart
- **Implementation**: `main.html` with Kendo Chart

### SH-004: Database Replication
- 3-node configuration (1 source + 2 targets)
- Asynchronous replication
- **Implementation**: OpenEdge replication, `.repl.properties` files

### SH-005: Multi-Environment Deployment
- Local deployment (version 122)
- AWS Cloud deployment
- **Implementation**: `build.sh`, `deploy.sh`

## Could Have (Optional)

### CH-001: Admin Interface
- Session monitoring
- Performance statistics
- **Implementation**: Spring Security, `anonymousLoginModel.xml`

### CH-002: Data Caching
- In-memory result caching
- Configurable TTL
- **Implementation**: PASOE session configuration

### CH-003: Data Export
- CSV export
- PDF reports
- Excel export
- **Implementation**: Kendo UI extensions

### CH-004: Real-Time Notifications
- WebSocket communication
- Data change notifications

### CH-005: Internationalization
- Multi-language support (FR/EN)
- Localized date/number formats

## Won't Have (Excluded)

### WH-001: User Authentication
Not in current version - anonymous access configured
**Future**: v2.0

### WH-002: Data Editing
Read-only demonstration application
**Future**: CRUD in v2.0

### WH-003: GraphQL API
REST/JSON sufficient for current needs
**Future**: Consider in v3.0

### WH-004: Native Mobile App
Responsive web interface sufficient
**Future**: Possible in v3.0

### WH-005: AI/ML Features
Out of scope for demonstration
**Future**: Not planned

## Traceability Matrix

| ID | Component | Files | Tests |
|----|-----------|-------|-------|
| MH-001 | Customer API | customer-data.p, customer-view.html | test.sh |
| MH-002 | State API | state-data.p, state-view.html | test.sh |
| MH-003 | Web UI | index.html, menu.html, main.html | test.sh |
| MH-004 | REST APIs | *.p programs | JSON validation |
| MH-005 | DB Connection | dbconnect.p, autoreconnect.pf | Connection tests |
| SH-001 | Grid Features | grid.js, Kendo UI | UI tests |
| SH-004 | Replication | replication scripts | Replication tests |
| SH-005 | Multi-deploy | build.sh, deploy.sh | CI/CD pipeline |

## Version Roadmap

### Version 1.0 (Current)
- All **Must Have** (MH-001 to MH-005)
- Critical **Should Have** (SH-001, SH-002, SH-004, SH-005)

### Version 1.1 (Next)
- SH-003: Enhanced analytics dashboard
- CH-001: Basic admin interface

### Version 2.0 (Future)
- Authentication and authorization
- Data editing (CRUD operations)
- Advanced caching

---

**📚 For complete requirements including:**
- Detailed acceptance criteria
- Use case scenarios
- Test strategies
- Priority justifications

**See the [complete French documentation](../../fr/requirements/functional-requirements.md)**
