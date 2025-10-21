# Component Architecture

## Module Overview

The Sports application is organized into distinct modules, each with specific responsibilities and well-defined interfaces.

```mermaid
graph TB
    subgraph "WebUI Module"
        A[Static Files Module]
        A1[index.html]
        A2[menu.html]
        A3[main.html]
        A4[grid.js]
    end
    
    subgraph "WebSpeed Module"
        B[Data API Module]
        B1[customer-data.p]
        B2[state-data.p]
        
        C[View Generation Module]
        C1[customer-view.html]
        C2[state-view.html]
        C3[about.html]
    end
    
    subgraph "Infrastructure Module"
        D[Include Files Module]
        D1[table-config.i]
        D2[header.i]
        D3[dbconnect.p]
        
        E[Compilation Module]
        E1[compile.p]
    end
    
    subgraph "Configuration Module"
        F[PASOE Config]
        F1[openedge.properties]
        F2[autoreconnect.pf]
        
        G[Web Config]
        G1[nginx.conf]
        G2[default site config]
    end
    
    subgraph "Deployment Module"
        H[Build Scripts]
        H1[build.sh]
        H2[deploy.sh]
        H3[test.sh]
        
        I[AWS Scripts]
        I1[create_stack.sh]
        I2[deploy_app.sh]
    end
    
    A --> B
    B --> D
    C --> D
    D --> F
    H --> G
    I --> H
```

## WebUI Module - User Interface

### Static Components

```mermaid
graph LR
    subgraph "HTML Structure"
        A[index.html<br/>Main Page]
        B[menu.html<br/>Navigation]
        C[main.html<br/>Dashboard]
    end
    
    subgraph "JavaScript Logic"
        D[grid.js<br/>Grid Component]
    end
    
    subgraph "External Libraries"
        E[Kendo UI<br/>UI Framework]
        F[jQuery<br/>DOM Manipulation]
        G[Progress JSDO<br/>Data Binding]
    end
    
    A --> B
    A --> C
    C --> D
    D --> E
    D --> F
    D --> G
```

#### Responsibilities
- **index.html**: Entry point with iframe layout
- **menu.html**: Navigation between views
- **main.html**: Dashboard with Kendo UI chart
- **grid.js**: Reusable component for data grids

## WebSpeed Module - 4GL Backend

### Data APIs

```mermaid
graph TB
    subgraph "Data APIs"
        A[customer-data.p]
        B[state-data.p]
    end
    
    subgraph "View Generators"
        C[customer-view.html]
        D[state-view.html]
        E[about.html]
    end
    
    subgraph "Common Pattern"
        F[Temp-Table Creation]
        G[Dataset Definition]
        H[JSON Serialization]
        I[HTTP Response]
    end
    
    A --> F
    B --> F
    F --> G
    G --> H
    H --> I
    
    C --> A
    D --> B
```

#### Common Data Pattern

```progress
// Standard pattern for data APIs
DEFINE TEMP-TABLE tt{Entity} LIKE {Entity}.
DEFINE DATASET ds{Entity} FOR tt{Entity}.

PROCEDURE process-web-request :
    // 1. Empty temp-table
    EMPTY TEMP-TABLE tt{Entity}.
    
    // 2. Load data from DB
    FOR EACH {Entity} NO-LOCK:
        CREATE tt{Entity}.
        BUFFER-COPY {Entity} TO tt{Entity}.
    END.
    
    // 3. Serialize to JSON
    oJsonObject = NEW JsonObject().
    oJsonObject:READ(DATASET ds{Entity}:HANDLE).
    
    // 4. Return response
    lChar = oJsonObject:GetJsonText().
    {&OUT-LONG} lChar.
END PROCEDURE.
```

## Infrastructure Module - Utilities

### Include Files

```mermaid
graph TB
    subgraph "Dynamic Configuration"
        A[table-config.i]
        A1[JSON config generation]
        A2[Schema metadata]
        A3[Grid configuration]
    end
    
    subgraph "Web Headers"
        B[header.i]
        B1[Kendo UI CSS links]
        B2[JavaScript scripts]
        B3[JSDO configuration]
    end
    
    subgraph "DB Connection"
        C[dbconnect.p]
        C1[sports2020 connection]
        C2[Connection parameters]
    end
    
    A --> A1
    A1 --> A2
    A2 --> A3
    
    B --> B1
    B1 --> B2
    B2 --> B3
    
    C --> C1
    C1 --> C2
```

#### table-config.i - Configuration Generator

This include file dynamically generates JSON configuration for Kendo UI grids by analyzing the database schema.

**Features**:
- Table metadata analysis
- Field configuration generation
- Automatic API URLs
- Grid column configuration

## PASOE Module - Application Server

### Configuration and Properties

```mermaid
graph TB
    subgraph "Session Management"
        A[Session Manager Config]
        A1[Agent Pool Size]
        A2[Timeouts]
        A3[Connection Limits]
    end
    
    subgraph "Database Connection"
        B[Auto-Reconnect Config]
        B1[Primary Database]
        B2[Alternate Databases]
        B3[Failover Logic]
    end
    
    subgraph "Web Transport"
        C[WebSpeed Compatibility]
        C1[Static Root Path]
        C2[Debug Settings]
        C3[Security Model]
    end
    
    A --> B
    B --> C
```

#### Key Configuration (openedge.properties)

```properties
# Automatic session activation
sessionActivateProc=dbconnect.p

# Agent configuration
minAgents=1
maxAgents=2
numInitialAgents=1

# Timeouts
idleSessionTimeout=1800000
connectionWaitTimeout=3000

# WebSpeed compatibility
wsRoot=/static/webspeed
```

## Compilation Module

### Build Process

```mermaid
flowchart TD
    A[compile.p] --> B{File Type}
    B -->|.p/.w| C[4GL Compilation]
    B -->|.html| D[SpeedScript Generation]
    
    C --> E[Save .r]
    D --> F[Generate temp .w]
    F --> G[Compile .w]
    G --> H[Delete .w]
    
    E --> I[Target Artifacts]
    H --> I
```

#### Compiler Features

- **Multi-format support**: .p, .w, .html
- **SpeedScript**: HTML to 4GL conversion
- **Organization**: Source/target directory structure
- **Parameters**: Configuration via session arguments

## Deployment Module

### Build Architecture

```mermaid
graph TB
    subgraph "Build Process"
        A[build.sh]
        A1[Source Compilation]
        A2[Asset Copying]
        A3[Package Creation]
    end
    
    subgraph "Deployment Targets"
        B[Local Deployment]
        B1[Version 117 - WebSpeed/Apache]
        B2[Version 122 - PASOE/nginx]
        
        C[Cloud Deployment]
        C1[AWS S3 Upload]
        C2[CloudFormation Stack]
    end
    
    subgraph "Package Types"
        D[web.tar.gz]
        E[pas.tar.gz]
        F[db.tar.gz]
    end
    
    A --> A1
    A1 --> A2
    A2 --> A3
    A3 --> D
    A3 --> E
    A3 --> F
    
    D --> B
    E --> B
    F --> B
    
    D --> C
    E --> C
    F --> C
```

## Database Module

### OpenEdge Replication

```mermaid
graph TB
    subgraph "Source Database (DB0)"
        A[sports2020 Primary]
        A1[Replication Server]
        A2[AI Archiver]
    end
    
    subgraph "Target Database 1 (DB1)"
        B[sports2020 Replica]
        B1[Replication Agent]
        B2[AI Recovery]
    end
    
    subgraph "Target Database 2 (DB2)"
        C[sports2020 Replica]
        C1[Replication Agent]
        C2[AI Recovery]
    end
    
    A1 -.->|Async Replication| B1
    A1 -.->|Async Replication| C1
    A2 --> A1
    B2 --> B1
    C2 --> C1
```

#### Replication Configuration

**Source Database**:
- Service: `replserv`
- Control agents: agent1, agent2
- Method: asynchronous

**Target Databases**:
- Service: `replagent`
- Connection to other agents
- Manual transition enabled

## Module Interactions

### Complete Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant WebUI
    participant PASOE
    participant WebSpeed
    participant Infrastructure
    participant Database
    
    Client->>WebUI: Load page
    WebUI->>PASOE: API request
    PASOE->>WebSpeed: Activate session
    WebSpeed->>Infrastructure: Include files
    Infrastructure->>Database: Connect
    Database-->>Infrastructure: Connection established
    Infrastructure-->>WebSpeed: Config loaded
    WebSpeed->>Database: Data query
    Database-->>WebSpeed: Result set
    WebSpeed-->>PASOE: JSON response
    PASOE-->>WebUI: HTTP response
    WebUI-->>Client: Rendered page
```

## Architectural Patterns

### 1. Template Method Pattern
WebSpeed programs use a template method pattern:
- Common structure (temp-table + dataset + JSON)
- Variations by entity (Customer, State)
- Include files for reuse

### 2. Configuration Pattern
Include files implement a configuration pattern:
- `table-config.i`: Dynamic configuration based on metadata
- `header.i`: Static header configuration
- Parameterization via global variables

### 3. Proxy Pattern
nginx acts as a proxy with multiple responsibilities:
- Static vs dynamic request routing
- Load balancing to PASOE
- SSL termination and security headers

This modular architecture ensures clear separation of responsibilities, high maintainability, and optimal scalability.
