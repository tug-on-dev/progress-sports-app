# Architecture Overview

## Introduction

The Sports application is a multi-tier application developed with Progress OpenEdge 4GL that demonstrates database replication capabilities and cloud deployment patterns. It comprises WebSpeed/PASOE backend services, database components, and a web frontend interface deployed via AWS infrastructure.

## 3-Tier Architecture

```mermaid
graph TB
    subgraph "Tier 1 - Presentation"
        A[Web Browser Client]
        B[nginx Web Server]
        C[Static HTML/CSS/JS Files]
        D[Kendo UI Grids]
    end
    
    subgraph "Tier 2 - Application"
        E[PASOE Application Server]
        F[WebSpeed Transaction Server]
        G[Progress 4GL Programs]
        H[JSON REST APIs]
    end
    
    subgraph "Tier 3 - Data"
        I[(sports2020 Database)]
        J[(DB1 Replica)]
        K[(DB2 Replica)]
    end
    
    A --> B
    B --> C
    C --> D
    B --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I -.->|OpenEdge Replication| J
    I -.->|OpenEdge Replication| K
```

## Main Components

### 1. Web Tier (Tier 1)
- **nginx**: Web server serving static files and reverse proxy to PASOE
- **User interface**: HTML5 with Kendo UI components
- **Dynamic grids**: Display of Customer and State data

### 2. Application Tier (Tier 2)
- **PASOE**: Progress Application Server for OpenEdge
- **WebSpeed**: Transaction server for 4GL programs
- **REST APIs**: JSON endpoints for data access

### 3. Data Tier (Tier 3)
- **Primary database**: sports2020 (replication source)
- **Replicated databases**: DB1 and DB2 (replication targets)
- **OpenEdge replication**: Automatic data synchronization

## AWS Deployment Architecture

```mermaid
graph TB
    subgraph "AWS VPC"
        subgraph "Public Subnet"
            ALB[Application Load Balancer]
            WEB[Web Instance<br/>nginx]
        end
        
        subgraph "Private Subnet 1"
            PAS[PASOE Instance<br/>Application Server]
        end
        
        subgraph "Private Subnet 2"
            DB0[(DB0 Instance<br/>Primary Database)]
        end
        
        subgraph "Private Subnet 3"
            DB1[(DB1 Instance<br/>Replica Database)]
        end
        
        subgraph "Private Subnet 4"
            DB2[(DB2 Instance<br/>Replica Database)]
        end
    end
    
    subgraph "AWS Services"
        S3[S3 Buckets<br/>Deployment Packages]
        CF[CloudFormation<br/>Infrastructure as Code]
        ASG[Auto Scaling Group]
    end
    
    Internet --> ALB
    ALB --> WEB
    WEB --> PAS
    PAS --> DB0
    DB0 -.->|Replication| DB1
    DB0 -.->|Replication| DB2
    CF --> S3
    ASG --> PAS
```

## Data Flow

```mermaid
sequenceDiagram
    participant Browser
    participant nginx
    participant PASOE
    participant WebSpeed
    participant Database
    
    Browser->>nginx: HTTP Request
    nginx->>Browser: Static files (HTML/JS)
    Browser->>nginx: API Request (/web/customer-data.p)
    nginx->>PASOE: Proxy to port 8810
    PASOE->>WebSpeed: Session activation
    WebSpeed->>Database: SQL Query
    Database-->>WebSpeed: Data
    WebSpeed-->>PASOE: JSON Response
    PASOE-->>nginx: JSON Response
    nginx-->>Browser: JSON Response
    Browser->>Browser: Kendo UI Grid rendering
```

## Technologies and Frameworks

### Backend
- **Progress OpenEdge 11.7+**: 4GL development platform
- **WebSpeed**: Web framework for Progress 4GL
- **PASOE**: Modern application server
- **Progress.Json.ObjectModel**: Native JSON serialization

### Frontend
- **Kendo UI 2017.3.1026**: UI component library
- **jQuery**: DOM manipulation and AJAX
- **Progress JSDO**: JavaScript data binding

### Infrastructure
- **nginx**: Web server and reverse proxy
- **AWS EC2**: Virtual instances
- **AWS CloudFormation**: Infrastructure as Code
- **AWS S3**: Deployment package storage

## Architectural Patterns

### 1. Repository Pattern
4GL programs follow the repository pattern with:
- Temp-tables for data manipulation
- Datasets for JSON serialization
- Centralized connection procedures

### 2. Proxy Pattern
nginx acts as a reverse proxy:
- Routing static vs dynamic requests
- Load balancing to PASOE instances
- SSL/TLS termination

### 3. MVC Pattern (Client Side)
- **Model**: Dynamically generated JSON configuration
- **View**: HTML templates with placeholders
- **Controller**: Kendo UI JavaScript scripts

## Security

### Authentication
- Anonymous configuration for public access
- Spring Security for endpoint protection
- Security headers (X-Frame-Options, XSS-Protection)

### Network
- AWS VPC with private/public subnets
- Security Groups to control access
- Load Balancer for traffic distribution

## Scalability

### Horizontal
- Auto Scaling Groups for PASOE instances
- Multi-region database replication
- Load Balancer for traffic distribution

### Vertical
- PASOE agent configuration (min/max)
- Connection pool parameters
- User session cache

## Monitoring and Observability

### Logs
- nginx logs: access and errors
- PASOE logs: agents and sessions
- Database logs: replication and transactions

### Metrics
- PASOE metrics collection enabled
- AWS CloudWatch monitoring
- Performance alerts

This architecture ensures high availability, scalability, and optimal maintainability for the Sports application.
