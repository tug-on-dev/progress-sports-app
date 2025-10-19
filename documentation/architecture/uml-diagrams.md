# Diagrammes UML

## Diagramme de Classes

```mermaid
classDiagram
    class WebSpeedProgram {
        <<abstract>>
        +JsonObject oJsonObject
        +LONGCHAR lChar
        +process-web-request()
        +outputHeader()
    }
    
    class CustomerDataProgram {
        +TEMP-TABLE ttCustomer
        +DATASET dsCustomer
        +process-web-request()
        +outputHeader()
    }
    
    class StateDataProgram {
        +TEMP-TABLE ttState
        +DATASET dsState
        +process-web-request()
        +outputHeader()
    }
    
    class CustomerViewProgram {
        +cConfig: CHARACTER
        +generateView()
    }
    
    class StateViewProgram {
        +cConfig: CHARACTER
        +generateView()
    }
    
    class TableConfigInclude {
        +EntityName: CHARACTER
        +oJsonObject: JsonObject
        +array: JsonArray
        +generateConfig()
    }
    
    class HeaderInclude {
        +generateHeaders()
    }
    
    class DBConnectProgram {
        +connectDatabase()
    }
    
    class CompileProgram {
        +cSrcDir: CHARACTER
        +cTargetDir: CHARACTER
        +compilePrograms()
        +compileSpeedScript()
    }
    
    WebSpeedProgram <|-- CustomerDataProgram
    WebSpeedProgram <|-- StateDataProgram
    WebSpeedProgram <|-- CustomerViewProgram
    WebSpeedProgram <|-- StateViewProgram
    
    CustomerViewProgram --> TableConfigInclude
    StateViewProgram --> TableConfigInclude
    CustomerViewProgram --> HeaderInclude
    StateViewProgram --> HeaderInclude
    
    CustomerDataProgram --> DBConnectProgram
    StateDataProgram --> DBConnectProgram
    
    CompileProgram --> CustomerDataProgram
    CompileProgram --> StateDataProgram
    CompileProgram --> CustomerViewProgram
    CompileProgram --> StateViewProgram
```

## Diagramme de Classes Frontend

```mermaid
classDiagram
    class GridConfiguration {
        +readURL: string
        +data: string
        +fields: FieldConfig[]
    }
    
    class FieldConfig {
        +field: string
    }
    
    class KendoGrid {
        +dataSource: DataSource
        +navigatable: boolean
        +filterable: boolean
        +groupable: boolean
        +createGrid()
    }
    
    class DataSource {
        +transport: Transport
        +schema: Schema
    }
    
    class Transport {
        +read: string
    }
    
    class Schema {
        +data: string
    }
    
    class MenuComponent {
        +generateMenu()
    }
    
    class MainComponent {
        +createChart()
    }
    
    GridConfiguration --> FieldConfig
    KendoGrid --> DataSource
    DataSource --> Transport
    DataSource --> Schema
```

## Diagramme de Séquence - Requête Customer Data

```mermaid
sequenceDiagram
    participant Browser
    participant nginx
    participant PASOE
    participant WebSpeed
    participant CustomerData
    participant Database
    participant JsonObject
    
    Browser->>nginx: GET /web/customer-data.p
    nginx->>PASOE: Proxy request to port 8810
    PASOE->>WebSpeed: Activate session
    WebSpeed->>CustomerData: Execute customer-data.p
    
    CustomerData->>Database: Connect via dbconnect.p
    activate Database
    CustomerData->>Database: CONNECT "-pf /psc/wrk/autoreconnect.pf"
    
    CustomerData->>CustomerData: EMPTY TEMP-TABLE ttCustomer
    CustomerData->>Database: FOR EACH Customer NO-LOCK
    Database-->>CustomerData: Customer records
    
    loop For each Customer
        CustomerData->>CustomerData: CREATE ttCustomer
        CustomerData->>CustomerData: BUFFER-COPY Customer TO ttCustomer
        CustomerData->>CustomerData: ASSIGN ttCustomer.Name = UPPER(Customer.Name)
    end
    
    CustomerData->>JsonObject: NEW JsonObject()
    CustomerData->>JsonObject: READ(DATASET dsCustomer:HANDLE)
    JsonObject-->>CustomerData: JSON representation
    
    CustomerData->>CustomerData: lChar = oJsonObject:GetJsonText()
    CustomerData->>CustomerData: outputHeader() - set content-type
    CustomerData-->>WebSpeed: {&OUT-LONG} lChar
    
    deactivate Database
    WebSpeed-->>PASOE: JSON response
    PASOE-->>nginx: JSON response
    nginx-->>Browser: application/json
    
    Browser->>Browser: Parse JSON
    Browser->>Browser: Render Kendo Grid
```

## Diagramme de Séquence - Rendu Customer View

```mermaid
sequenceDiagram
    participant Browser
    participant nginx
    participant PASOE
    participant WebSpeed
    participant CustomerView
    participant TableConfig
    participant HeaderInclude
    
    Browser->>nginx: GET /web/customer-view.html
    nginx->>PASOE: Proxy request
    PASOE->>WebSpeed: Activate session
    WebSpeed->>CustomerView: Execute customer-view.html
    
    CustomerView->>TableConfig: { table-config.i &TableName = "Customer" }
    activate TableConfig
    
    TableConfig->>TableConfig: FIND _File WHERE _File-Name = "Customer"
    TableConfig->>TableConfig: Generate JSON configuration
    TableConfig->>TableConfig: Create field array from database schema
    TableConfig->>TableConfig: oJsonObject:ADD("readURL", "/web/customer-data.p")
    TableConfig->>TableConfig: oJsonObject:ADD("data", "dsCustomer.ttCustomer")
    TableConfig-->>CustomerView: cConfig = JSON string
    
    deactivate TableConfig
    
    CustomerView->>HeaderInclude: {header.i}
    activate HeaderInclude
    HeaderInclude->>HeaderInclude: Generate Kendo UI CSS/JS links
    HeaderInclude-->>CustomerView: HTML headers
    deactivate HeaderInclude
    
    CustomerView->>CustomerView: Generate HTML template
    CustomerView->>CustomerView: Inject cConfig into JavaScript
    CustomerView-->>WebSpeed: Complete HTML page
    
    WebSpeed-->>PASOE: HTML response
    PASOE-->>nginx: HTML response
    nginx-->>Browser: text/html
    
    Browser->>Browser: Parse HTML
    Browser->>Browser: Load grid.js
    Browser->>Browser: Execute createGrid() with config
```

## Diagramme de Composants

```mermaid
graph TB
    subgraph "Frontend Components"
        A[index.html]
        B[menu.html]
        C[main.html]
        D[grid.js]
    end
    
    subgraph "WebSpeed Programs"
        E[customer-data.p]
        F[state-data.p]
        G[customer-view.html]
        H[state-view.html]
        I[about.html]
    end
    
    subgraph "Include Files"
        J[table-config.i]
        K[header.i]
        L[dbconnect.p]
    end
    
    subgraph "Configuration"
        M[openedge.properties]
        N[autoreconnect.pf]
        O[nginx.conf]
    end
    
    subgraph "Database"
        P[(sports2020)]
    end
    
    A --> B
    A --> C
    C --> D
    
    G --> J
    H --> J
    G --> K
    H --> K
    
    E --> L
    F --> L
    L --> P
    
    G --> E
    H --> F
    
    M --> E
    M --> F
    N --> P
    O --> E
    O --> F
```

## Diagramme d'États - Session PASOE

```mermaid
stateDiagram-v2
    [*] --> Inactive
    
    Inactive --> Starting : tcman.sh start
    Starting --> Active : Session created
    
    Active --> Processing : Request received
    Processing --> Active : Request completed
    
    Active --> Idle : Timeout
    Idle --> Active : New request
    Idle --> Terminated : Idle timeout exceeded
    
    Active --> Stopping : tcman.sh stop
    Processing --> Stopping : tcman.sh stop
    Stopping --> Terminated : All sessions closed
    
    Terminated --> [*]
    
    note right of Processing
        Execute 4GL program
        Connect to database
        Generate JSON response
    end note
    
    note right of Idle
        Agent waiting for requests
        Connection pool maintained
    end note
```

## Diagramme d'États - Réplication Base de Données

```mermaid
stateDiagram-v2
    [*] --> Initializing
    
    Initializing --> SourceReady : Source DB configured
    SourceReady --> Replicating : Targets connected
    
    Replicating --> Synchronizing : Transaction log
    Synchronizing --> Replicating : Sync complete
    
    Replicating --> FailoverPending : Source failure
    FailoverPending --> TargetActive : Manual transition
    TargetActive --> Replicating : Source recovered
    
    Replicating --> Stopped : Stop replication
    Stopped --> [*]
    
    note right of Replicating
        DB0 (source) → DB1, DB2 (targets)
        Async replication enabled
        AI archiver active
    end note
    
    note right of FailoverPending
        Manual transition required
        Backup method: mark
        Restart after transition
    end note
```

Ces diagrammes UML fournissent une vue détaillée de la structure, du comportement et des interactions des composants de l'application Sports.