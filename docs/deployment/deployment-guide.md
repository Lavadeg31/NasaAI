# Deployment Guide

## Deployment Overview

### Deployment Strategy
The NASA AI application follows a continuous deployment strategy with multiple environments:
- **Development:** Feature development and testing
- **Staging:** Pre-production testing and validation
- **Production:** Live application serving users

### Deployment Environments

#### Development Environment
- **Purpose:** Development and initial testing
- **URL:** https://dev.nasaai.app
- **Deployment:** Automatic on merge to `develop` branch
- **Resources:** Minimal allocation for cost efficiency

#### Staging Environment
- **Purpose:** Pre-production validation and UAT
- **URL:** https://staging.nasaai.app
- **Deployment:** Manual trigger after development validation
- **Resources:** Production-like allocation

#### Production Environment
- **Purpose:** Live application for end users
- **URL:** https://nasaai.app
- **Deployment:** Manual approval required
- **Resources:** Full production allocation

## Prerequisites

### Infrastructure Requirements
- **Compute:** 2 vCPU, 4GB RAM minimum per instance
- **Storage:** 100GB SSD for application data
- **Network:** Load balancer with SSL termination
- **Database:** PostgreSQL 14+ or MongoDB 5+

### Required Accounts and Access
- Cloud provider account (AWS/Azure/GCP)
- Docker registry access
- NASA API credentials
- DNS management access
- SSL certificate management

### Required Tools
- Docker and Docker Compose
- Kubernetes CLI (kubectl) if using K8s
- Cloud CLI tools (aws-cli, az-cli, gcloud)
- Git and deployment scripts

## Deployment Process

### Automated Deployment (CI/CD)

#### Pipeline Overview
1. **Code Commit** → Triggers build pipeline
2. **Build & Test** → Runs automated tests
3. **Security Scan** → Scans for vulnerabilities
4. **Build Image** → Creates Docker image
5. **Deploy to Dev** → Automatic deployment
6. **Integration Tests** → Runs integration tests
7. **Deploy to Staging** → Manual approval
8. **UAT** → User acceptance testing
9. **Deploy to Production** → Manual approval

#### Environment Variables
```bash
# Application Configuration
APP_ENV=production
APP_DEBUG=false
APP_URL=https://nasaai.app

# Database Configuration
DB_HOST=prod-db.nasaai.app
DB_PORT=5432
DB_NAME=nasaai_prod
DB_USER=app_user
DB_PASSWORD=${DB_PASSWORD}

# NASA API Configuration
NASA_API_KEY=${NASA_API_KEY}
NASA_API_URL=https://api.nasa.gov

# Security Configuration
SECRET_KEY=${SECRET_KEY}
JWT_SECRET=${JWT_SECRET}

# External Services
REDIS_URL=${REDIS_URL}
STORAGE_BUCKET=${STORAGE_BUCKET}
```

### Manual Deployment

#### Step 1: Pre-deployment Checklist
- [ ] All tests passing in staging environment
- [ ] Database migrations reviewed and tested
- [ ] Environment variables configured
- [ ] SSL certificates valid and current
- [ ] Backup completed
- [ ] Rollback plan prepared

#### Step 2: Database Migration (if required)
```bash
# Backup current database
kubectl exec -it postgres-pod -- pg_dump nasaai_prod > backup_$(date +%Y%m%d).sql

# Apply migrations
kubectl apply -f k8s/migrations-job.yaml

# Verify migration success
kubectl logs migration-job
```

#### Step 3: Application Deployment
```bash
# Update image tag in deployment manifest
sed -i 's/image: nasaai:.*$/image: nasaai:v1.2.0/' k8s/deployment.yaml

# Apply deployment
kubectl apply -f k8s/

# Monitor rollout
kubectl rollout status deployment/nasaai-app

# Verify deployment
kubectl get pods -l app=nasaai
```

#### Step 4: Post-deployment Verification
```bash
# Health check
curl -f https://nasaai.app/health

# Smoke tests
./scripts/smoke-tests.sh production

# Monitor logs for errors
kubectl logs -f deployment/nasaai-app
```

## Docker Deployment

### Docker Compose (Development/Testing)
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - APP_ENV=development
      - DB_HOST=db
    depends_on:
      - db
      - redis
    
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: nasaai
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: dev_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    
  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Kubernetes Deployment

#### Deployment Manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nasaai-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nasaai
  template:
    metadata:
      labels:
        app: nasaai
    spec:
      containers:
      - name: nasaai
        image: nasaai:v1.2.0
        ports:
        - containerPort: 8000
        env:
        - name: APP_ENV
          value: "production"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: nasaai-secrets
              key: db-password
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

## Database Management

### Database Setup
```sql
-- Create production database
CREATE DATABASE nasaai_prod;
CREATE USER app_user WITH PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE nasaai_prod TO app_user;

-- Create indexes for performance
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_analysis_status ON analyses(status);
CREATE INDEX idx_nasa_data_date ON nasa_data(earth_date);
```

### Migration Strategy
```bash
# Development migrations
python manage.py makemigrations
python manage.py migrate

# Production migrations (with backup)
./scripts/backup-db.sh
python manage.py migrate --database=production
./scripts/verify-migration.sh
```

## Monitoring and Logging

### Application Monitoring
- **Health Checks:** `/health` and `/ready` endpoints
- **Metrics:** Prometheus metrics at `/metrics`
- **APM:** Application Performance Monitoring integration
- **Uptime:** External uptime monitoring service

### Log Management
```yaml
# Logging configuration
logging:
  level: INFO
  format: json
  outputs:
    - console
    - file: /var/log/nasaai/app.log
  loggers:
    nasa_api: DEBUG
    database: WARN
    security: INFO
```

### Alerting Rules
- Application error rate > 5%
- Response time > 5 seconds
- Database connection failures
- NASA API connection failures
- Disk usage > 80%
- Memory usage > 90%

## Rollback Procedures

### Automated Rollback
```bash
# Rollback to previous version
kubectl rollout undo deployment/nasaai-app

# Rollback to specific version
kubectl rollout undo deployment/nasaai-app --to-revision=2

# Check rollback status
kubectl rollout status deployment/nasaai-app
```

### Database Rollback
```bash
# Restore database from backup
kubectl exec -it postgres-pod -- psql -U postgres -d nasaai_prod < backup_20240101.sql

# Verify data integrity
./scripts/verify-db-integrity.sh
```

## Security Considerations

### SSL/TLS Configuration
- SSL certificates managed via Let's Encrypt or cloud provider
- TLS 1.2+ required
- HSTS headers enabled
- Secure cookie settings

### Access Control
- API keys rotated regularly
- Database credentials stored in secrets management
- Network security groups configured
- VPC/private networking where possible

### Data Protection
- NASA data encryption at rest and in transit
- Regular security updates
- Vulnerability scanning in CI/CD pipeline
- GDPR/compliance considerations

## Troubleshooting

### Common Issues

#### Database Connection Issues
```bash
# Check database connectivity
kubectl exec -it app-pod -- nc -zv db-host 5432

# Check database logs
kubectl logs deployment/postgres

# Verify credentials
kubectl get secret nasaai-secrets -o yaml
```

#### Application Won't Start
```bash
# Check pod status
kubectl describe pod nasaai-pod

# Check application logs
kubectl logs nasaai-pod

# Check resource limits
kubectl top pod nasaai-pod
```

#### NASA API Integration Issues
```bash
# Test API connectivity
curl -H "X-API-Key: $NASA_API_KEY" https://api.nasa.gov/planetary/apod

# Check API quota
curl -H "X-API-Key: $NASA_API_KEY" https://api.nasa.gov/planetary/apod?api_key=$NASA_API_KEY

# Monitor API response times
./scripts/api-health-check.sh
```

---
*Last updated: [Date]*