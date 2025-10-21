# Architecture des Composants

## Vue d'Ensemble des Modules

L'application Sports est organisée en modules distincts, chacun ayant des responsabilités spécifiques et des interfaces bien définies.

```mermaid
graph TB
    subgraph "Module WebUI"
        A[Static Files Module]
        A1[index.html]
        A2[menu.html]
        A3[main.html]
        A4[grid.js]
    end
    
    subgraph "Module WebSpeed"
        B[Data API Module]
        B1[customer-data.p]
        B2[state-data.p]
        
        C[View Generation Module]
        C1[customer-view.html]
        C2[state-view.html]
        C3[about.html]
    end
    
    subgraph "Module Infrastructure"
        D[Include Files Module]
        D1[table-config.i]
        D2[header.i]
        D3[dbconnect.p]
        
        E[Compilation Module]
        E1[compile.p]
    end
    
    subgraph "Module Configuration"
        F[PASOE Config]
        F1[openedge.properties]
        F2[autoreconnect.pf]
        
        G[Web Config]
        G1[nginx.conf]
        G2[default site config]
    end
    
    subgraph "Module Deployment"
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

## Module WebUI - Interface Utilisateur

### Composants Statiques

```mermaid
graph LR
    subgraph "Structure HTML"
        A[index.html<br/>Page principale]
        B[menu.html<br/>Navigation]
        C[main.html<br/>Dashboard]
    end
    
    subgraph "Logique JavaScript"
        D[grid.js<br/>Composant grille]
    end
    
    subgraph "Bibliothèques Externes"
        E[Kendo UI<br/>Framework UI]
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

#### Responsabilités
- **index.html**: Point d'entrée avec layout en iframes
- **menu.html**: Navigation entre les vues
- **main.html**: Tableau de bord avec graphique Kendo UI
- **grid.js**: Composant réutilisable pour les grilles de données

## Module WebSpeed - Backend 4GL

### APIs de Données

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

#### Pattern de Données Commun

```progress
// Pattern standard pour les APIs de données
DEFINE TEMP-TABLE tt{Entity} LIKE {Entity}.
DEFINE DATASET ds{Entity} FOR tt{Entity}.

PROCEDURE process-web-request :
    // 1. Vider la temp-table
    EMPTY TEMP-TABLE tt{Entity}.
    
    // 2. Charger les données depuis la DB
    FOR EACH {Entity} NO-LOCK:
        CREATE tt{Entity}.
        BUFFER-COPY {Entity} TO tt{Entity}.
    END.
    
    // 3. Sérialiser en JSON
    oJsonObject = NEW JsonObject().
    oJsonObject:READ(DATASET ds{Entity}:HANDLE).
    
    // 4. Retourner la réponse
    lChar = oJsonObject:GetJsonText().
    {&OUT-LONG} lChar.
END PROCEDURE.
```

## Module Infrastructure - Utilitaires

### Include Files

```mermaid
graph TB
    subgraph "Configuration Dynamique"
        A[table-config.i]
        A1[Génération config JSON]
        A2[Métadonnées de schéma]
        A3[Configuration de grille]
    end
    
    subgraph "Headers Web"
        B[header.i]
        B1[Liens CSS Kendo UI]
        B2[Scripts JavaScript]
        B3[Configuration JSDO]
    end
    
    subgraph "Connexion DB"
        C[dbconnect.p]
        C1[Connexion sports2020]
        C2[Paramètres de connexion]
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

#### table-config.i - Générateur de Configuration

Ce fichier include génère dynamiquement la configuration JSON pour les grilles Kendo UI en analysant le schéma de la base de données.

**Fonctionnalités**:
- Analyse des métadonnées de table
- Génération des configurations de champs
- URLs d'API automatiques
- Configuration des colonnes de grille

## Module PASOE - Serveur d'Applications

### Configuration et Propriétés

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

#### Configuration Clé (openedge.properties)

```properties
# Activation automatique des sessions
sessionActivateProc=dbconnect.p

# Configuration des agents
minAgents=1
maxAgents=2
numInitialAgents=1

# Timeouts
idleSessionTimeout=1800000
connectionWaitTimeout=3000

# WebSpeed compatibility
wsRoot=/static/webspeed
```

## Module de Compilation

### Processus de Build

```mermaid
flowchart TD
    A[compile.p] --> B{Type de fichier}
    B -->|.p/.w| C[Compilation 4GL]
    B -->|.html| D[SpeedScript Generation]
    
    C --> E[Sauvegarde .r]
    D --> F[Génération .w temporaire]
    F --> G[Compilation .w]
    G --> H[Suppression .w]
    
    E --> I[Artefacts Target]
    H --> I
```

#### Fonctionnalités du Compilateur

- **Support multi-format**: .p, .w, .html
- **SpeedScript**: Conversion HTML vers programmes 4GL
- **Organisation**: Structure de répertoires source/target
- **Paramètres**: Configuration via arguments de session

## Module de Déploiement

### Architecture de Build

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

## Module de Base de Données

### Réplication OpenEdge

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

#### Configuration de Réplication

**Source Database**:
- Service: `replserv`
- Agents de contrôle: agent1, agent2
- Méthode: asynchrone

**Target Databases**:
- Service: `replagent`
- Connexion aux autres agents
- Transition manuelle activée

## Interactions Entre Modules

### Flux de Requête Complète

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

## Patterns d'Architecture

### 1. Pattern Template Method
Les programmes WebSpeed utilisent un template method pattern:
- Structure commune (temp-table + dataset + JSON)
- Variations par entité (Customer, State)
- Include files pour la réutilisation

### 2. Pattern Configuration
Les include files implémentent un pattern de configuration:
- `table-config.i`: Configuration dynamique basée sur métadonnées
- `header.i`: Configuration statique des headers
- Paramétrage via variables globales

### 3. Pattern Proxy
nginx agit comme proxy avec plusieurs responsabilités:
- Routage des requêtes statiques vs dynamiques
- Load balancing vers PASOE
- Terminaison SSL et headers de sécurité

Cette architecture modulaire assure une séparation claire des responsabilités, une maintenabilité élevée et une scalabilité optimale.