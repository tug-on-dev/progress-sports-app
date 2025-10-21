# Deployment Guide

> **📖 Complete Version**: For detailed deployment instructions, see [French Deployment Guide](../../fr/deployment/deployment-guide.md)

## Quick Start

### Prerequisites

- Progress OpenEdge 11.7+
- Java 8+
- nginx or Apache
- Linux/Unix OS

### Environment Variables
```bash
export DLC=/psc/dlc
export PATH=$DLC/bin:$PATH
export WRKDIR=/psc/wrk
```

## Local Deployment (Version 122)

### 1. Build
```bash
./build.sh 122
```

### 2. Database Setup
```bash
cd /psc/wrk
prodb sports2020 sports2020
proserve sports2020 -S 20000
```

### 3. PASOE Configuration
```bash
cd /psc/wrk
pasman create -v oepas1
```

Copy configuration:
```bash
cp /artifacts/sports-app/122/conf/openedge.properties oepas1/conf/
cp /artifacts/sports-app/122/webspeed/*.r oepas1/openedge/
```

### 4. Start PASOE
```bash
cd oepas1
./bin/tcman.sh start
```

### 5. Configure nginx
```bash
sudo cp /artifacts/sports-app/122/conf/default /etc/nginx/sites-available/
sudo cp /artifacts/sports-app/122/webui/* /var/www/html/
sudo systemctl start nginx
```

### 6. Test
```bash
./test.sh localhost:8080
```

Access: http://localhost:8080

## AWS Cloud Deployment

### 1. AWS Setup
```bash
# Configure AWS CLI
aws configure

# Create S3 buckets
aws s3 mb s3://sports-app-deploy
```

### 2. Build for Cloud
```bash
./build.sh 123
```

### 3. Upload Packages
```bash
aws s3 cp /artifacts/sports-app/123/web.tar.gz s3://sports-app-deploy/
aws s3 cp /artifacts/sports-app/123/pas.tar.gz s3://sports-app-deploy/
aws s3 cp /artifacts/sports-app/123/db.tar.gz s3://sports-app-deploy/
```

### 4. Deploy Stack
```bash
./deploy.sh aws
```

### 5. Monitor
```bash
aws cloudformation describe-stacks --stack-name sports-app-prod
```

## Database Replication

### Source (DB0)
```bash
proutil sports2020 -C enableSiteReplication source
proserve sports2020 -DBService replserv -S 20000 -aiarcdir aiArchives
```

### Targets (DB1, DB2)
```bash
proutil sports2020 -C enableSiteReplication target
proserve sports2020 -DBService replagent -S 20000 -aiarcdir aiArchives
```

## Troubleshooting

### PASOE Not Starting
```bash
cd /psc/wrk/oepas1
./bin/tcman.sh status
tail -f logs/catalina.out
```

### Database Connection Failed
```bash
proutil sports2020 -C idxcheck
echo "Test" | pro -db sports2020 -S 20000 -b
```

### nginx 502 Error
```bash
# Test PASOE directly
curl -I http://localhost:8810/web/

# Check logs
sudo tail -f /var/log/nginx/error.log
```

## Common Commands

### PASOE Management
```bash
# Start
./bin/tcman.sh start

# Stop
./bin/tcman.sh stop

# Status
./bin/tcman.sh status

# Restart
./bin/tcman.sh restart
```

### Database Management
```bash
# Start database
proserve sports2020 -S 20000

# Stop database
proshut sports2020

# Check status
promon sports2020

# Backup
probkup sports2020 /backups/sports2020.backup
```

### nginx Management
```bash
# Test configuration
sudo nginx -t

# Reload
sudo systemctl reload nginx

# Restart
sudo systemctl restart nginx

# Logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

## Deployment Scenarios

| Scenario | Use Case | Command |
|----------|----------|---------|
| Development | Local testing | `./build.sh && ./deploy.sh 122` |
| Version 117 | Legacy WebSpeed | `./build.sh 117 && ./deploy.sh 117` |
| Version 122 | Modern PASOE | `./build.sh 122 && ./deploy.sh 122` |
| AWS Cloud | Production | `./build.sh 123 && ./deploy.sh aws` |

## Configuration Files

### Key Locations
- PASOE config: `/psc/wrk/oepas1/conf/openedge.properties`
- nginx config: `/etc/nginx/sites-available/default`
- DB connection: `/psc/wrk/autoreconnect.pf`
- Application: `/psc/wrk/oepas1/openedge/*.r`
- Static files: `/var/www/html/`

### Important Ports
- nginx: 8080
- PASOE: 8810
- Database: 20000

---

**📚 For complete deployment guide including:**
- Detailed step-by-step instructions
- Configuration examples
- Troubleshooting procedures
- Performance tuning
- Security best practices
- CI/CD pipeline setup

**See the [complete French documentation](../../fr/deployment/deployment-guide.md)**
