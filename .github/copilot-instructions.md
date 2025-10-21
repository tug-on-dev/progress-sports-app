# Copilot Instructions: Sports App (Progress OpenEdge)

## Architecture Overview

This is a modernized **Progress OpenEdge Sports Application** with a 3-tier architecture:
- **Database Layer**: Progress OpenEdge Database (`sports2020`) with Customer/State tables
- **Application Server**: PAS (Pacific Application Server) for OpenEdge running compiled `.r` files
- **Web Layer**: Nginx reverse proxy serving static HTML/JS + proxying API calls to PAS

The application demonstrates database modernization from traditional Progress to cloud-deployable architecture.

## Key Components & Data Flow

1. **Source Code** (`src/`) contains Progress ABL (Advanced Business Language) files:
   - `.p` files compile to `.r` executables for PAS runtime
   - `.html` files with embedded SpeedScript compile to `.w` then `.r`
   - Database connection handled via `dbconnect.p` and `autoreconnect.pf`

2. **Build Process** (`build.sh`): Compiles all `.p`/`.html` files, creates database backup, packages into deployable `.tar.gz` files (web/pas/db)

3. **Deployment Targets**:
   - Version `117`: Legacy local deployment
   - Version `122`: Local PAS + web server
   - `aws`: Cloud deployment via CloudFormation stack

## Critical Developer Workflows

### Build & Deploy Locally
```bash
./build.sh 122        # Compiles to /artifacts/sports-app/122/
./deploy.sh 122       # Deploys to local PAS + nginx
./test.sh localhost:8080  # Validates all endpoints
```

### Cloud Deployment
```bash
./deploy.sh aws       # Uploads to S3, creates CloudFormation stack
```

### Database Operations
- Database created via `prodb sports2020 sports2020`
- Served on port 20000: `proserve sports2020 -S 20000`
- Connection string in `autoreconnect.pf` supports multi-host failover

## Progress-Specific Patterns

### File Structure Conventions
- `.p` files: Procedures/business logic (compile to `.r`)
- `.html` with SpeedScript: Web templates (compile via `e4gl-gen.r`)
- `.i` files: Include files for shared code/styling
- `.pf` files: Parameter files for database connections

### Data Access Patterns
```progress
FOR EACH Customer NO-LOCK:
    CREATE ttCustomer.
    BUFFER-COPY Customer TO ttCustomer.
END.
```
- Temp-tables (`ttCustomer`) for data transfer
- Datasets (`dsCustomer`) for JSON serialization
- `BUFFER-COPY` for efficient record copying

### Web API Pattern
Files like `customer.p` expose JSON APIs:
- Include `{src/web2/wrap-cgi.i}` for CGI handling
- Use `Progress.Json.ObjectModel.JsonObject` for JSON output
- Set content-type via `output-content-type ("application/json":U)`

## Configuration Management

### Environment-Specific Deployment (`app/deploy.sh`)
- `OE_ENV` variable determines deployment type: `db0`/`db1`/`db2`/`pasoe`/`webserver`
- Template substitution for hostnames: `sed -i "s/DBHostName/${DBHostName}/"`
- PAS configuration in `openedge.properties` with PROPATH settings

### Nginx Proxy Configuration
- Static files served from `/var/www/html`
- API calls proxied: `location /web { proxy_pass PASOEURL/web; }`
- Template variable: `PASOEURL` replaced during deployment

## Testing Strategy

The `test.sh` script validates:
1. Static HTML files against known-good outputs in `test/output/`
2. JSON API endpoints via `curl` + `json_reformat` validation
3. Both direct file serving and proxied API calls

## Development Environment Requirements

- Progress OpenEdge DLC installation (`/psc/dlc` or `/psc/122/dlc`)
- Environment variables: `DLC`, `PATH`, `PROPATH`
- Database connection requires `sports2020` database on port 20000
- Local testing assumes PAS on port 8810, nginx on 8080

## AWS Infrastructure Notes

Uses CloudFormation quickstart template with parameters for:
- Instance types, scaling groups, availability zones
- S3 buckets for deployment artifacts (`PrivateBucket`/`PublicBucket`)
- Package separation: `web.tar.gz`, `pas.tar.gz`, `db.tar.gz`

## Common Pitfalls

- Always compile before deployment - `.p` files must become `.r` files
- Database connections require `autoreconnect.pf` with correct hostnames
- PAS must be stopped before deploying new `.r` files
- Nginx configuration templates need variable substitution before service restart
- Progress sessions require specific PROPATH for include resolution

## My Rules

- Prioritize clarity and conciseness in explanations
- Use technical terminology accurately
