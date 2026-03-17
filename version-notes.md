# IBM ODM Version Notes

## ODM 8.9.x

- **Rule Designer**: Eclipse-based IDE (Eclipse 4.x)
- **Decision Center**: Classic web UI (Enterprise Console + Business Console)
- **Deployment**: Traditional WAS (WebSphere Application Server) or Liberty
- **XOM management**: Manual JAR upload or Maven integration
- **Git support**: Basic — via Eclipse EGit, no native DC integration
- **REST API**: Limited — primarily SOAP-based for RES management
- **Decision tables**: Standard partition-based model
- **Rule governance**: Built-in workflow (draft → review → approve → deploy)

### 8.9 Specifics
- BOM editor uses the classic tree view
- Simulation requires explicit data provider configuration (CSV, database, or code)
- RuleApp deployment via Decision Center console or `deployer` CLI
- Testing via DVS (Decision Validation Services) or JUnit with `IlrSessionFactory`

---

## ODM 8.10.x

- **Decision Center**: Modernized Business Console UI (responsive design)
- **Git integration**: Native Git support in Decision Center — rule projects can be stored in Git repos
- **REST API**: Enhanced REST API for RES management (`/res/api/v1/`)
  - Deploy, undeploy, query rulesets
  - Retrieve execution traces
  - Manage XOM resources
- **Rule Designer**: Still Eclipse-based, minor UX improvements
- **Decision tables**: Improved editor with better multi-dimensional support
- **Simulation**: Streamlined simulation setup, better reporting

### 8.10 Migration from 8.9
- Decision Center database schema updated — requires migration tool
- RuleApp format unchanged — 8.9 RuleApps deploy on 8.10 RES
- BOM/XOM format unchanged
- Rule file formats (`.brl`, `.dta`, `.rfl`, `.trl`) are backward compatible

---

## ODM 8.11.x and Later

- **Container-native**: First-class support for deployment on OpenShift and Kubernetes
  - Helm charts for Decision Server (Runtime + Console)
  - Operator-based deployment option
- **Decision Runtime**: Runs on Open Liberty (lightweight)
- **Decision Center**: Further UI modernization, improved collaboration features
- **Cloud Pak for Automation**: ODM becomes part of IBM Cloud Pak for Business Automation
- **Enhanced REST APIs**: Broader coverage, better documentation
- **Operator pattern**: Kubernetes operator manages lifecycle (install, upgrade, scale)

### Container Deployment Model
```
┌─────────────────────────────────────┐
│  Kubernetes / OpenShift Cluster     │
│                                     │
│  ┌──────────────┐  ┌─────────────┐  │
│  │ Decision     │  │ Decision    │  │
│  │ Center       │  │ Server      │  │
│  │ (Business    │  │ Runtime     │  │
│  │  Console)    │  │ (RES)       │  │
│  └──────┬───────┘  └──────┬──────┘  │
│         │                 │         │
│  ┌──────┴─────────────────┴──────┐  │
│  │     Shared Database (DB2/    │  │
│  │     PostgreSQL)              │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
```

### 8.11 Migration Notes
- Rule project format remains compatible with 8.9/8.10
- XOM packaging may need adjustment for container classloaders
- Decision Center database requires schema migration
- REST API endpoints may have new paths under container deployment

---

## Format Compatibility Matrix

| Artifact      | 8.9 | 8.10 | 8.11+ | Notes                        |
|---------------|-----|------|-------|------------------------------|
| `.brl` (BAL)  | ✓   | ✓    | ✓     | Format unchanged across all  |
| `.dta` (DT)   | ✓   | ✓    | ✓     | Format unchanged across all  |
| `.trl` (IRL)  | ✓   | ✓    | ✓     | Format unchanged across all  |
| `.rfl` (Flow) | ✓   | ✓    | ✓     | Format unchanged across all  |
| `.bom`         | ✓   | ✓    | ✓     | Format unchanged across all  |
| `.dep`         | ✓   | ✓    | ✓     | Minor additions in 8.11      |
| RuleApp        | ✓   | ✓    | ✓     | Binary compatible            |
| DC Database    | ✗   | ✗    | ✗     | Requires migration per major |

The rule artifact XML formats are remarkably stable across ODM versions. The main differences are in deployment infrastructure, APIs, and tooling — not in the rule files themselves.
