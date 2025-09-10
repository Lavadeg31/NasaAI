# Configuration Management

## Configuration Overview

The NASA AI application uses a hierarchical configuration system that supports multiple environments and deployment scenarios.

### Configuration Hierarchy
1. **Default Configuration** - Base settings for all environments
2. **Environment Configuration** - Environment-specific overrides
3. **Local Configuration** - Local development overrides
4. **Environment Variables** - Runtime configuration
5. **Secrets Management** - Sensitive configuration data

## Environment Configurations

### Development Environment
```yaml
# config/development.yaml
app:
  name: "NASA AI - Development"
  debug: true
  log_level: DEBUG
  host: "0.0.0.0"
  port: 8000

database:
  host: "localhost"
  port: 5432
  name: "nasaai_dev"
  user: "dev_user"
  password: "dev_password"
  pool_size: 5
  ssl_mode: "disable"

nasa_api:
  base_url: "https://api.nasa.gov"
  timeout: 30
  retry_attempts: 3
  rate_limit: 100  # requests per hour

redis:
  host: "localhost"
  port: 6379
  database: 0
  password: ""

security:
  cors_enabled: true
  cors_origins: ["http://localhost:3000", "http://localhost:8080"]
  jwt_expiry: 24h
  session_timeout: 2h

features:
  image_analysis: true
  data_export: true
  admin_panel: true
  debug_mode: true
```

### Staging Environment
```yaml
# config/staging.yaml
app:
  name: "NASA AI - Staging"
  debug: false
  log_level: INFO
  host: "0.0.0.0"
  port: 8000

database:
  host: "${DB_HOST}"
  port: 5432
  name: "nasaai_staging"
  user: "${DB_USER}"
  password: "${DB_PASSWORD}"
  pool_size: 10
  ssl_mode: "require"

nasa_api:
  base_url: "https://api.nasa.gov"
  api_key: "${NASA_API_KEY}"
  timeout: 60
  retry_attempts: 5
  rate_limit: 500  # requests per hour

redis:
  host: "${REDIS_HOST}"
  port: 6379
  database: 0
  password: "${REDIS_PASSWORD}"

security:
  cors_enabled: true
  cors_origins: ["https://staging.nasaai.app"]
  jwt_expiry: 8h
  session_timeout: 1h

features:
  image_analysis: true
  data_export: true
  admin_panel: true
  debug_mode: false

monitoring:
  metrics_enabled: true
  health_check_interval: 30s
  log_retention_days: 30
```

### Production Environment
```yaml
# config/production.yaml
app:
  name: "NASA AI"
  debug: false
  log_level: WARN
  host: "0.0.0.0"
  port: 8000

database:
  host: "${DB_HOST}"
  port: 5432
  name: "nasaai_prod"
  user: "${DB_USER}"
  password: "${DB_PASSWORD}"
  pool_size: 20
  ssl_mode: "require"
  connection_timeout: 30s
  max_idle_connections: 5

nasa_api:
  base_url: "https://api.nasa.gov"
  api_key: "${NASA_API_KEY}"
  timeout: 120
  retry_attempts: 3
  rate_limit: 1000  # requests per hour
  circuit_breaker:
    enabled: true
    failure_threshold: 5
    timeout: 60s

redis:
  host: "${REDIS_HOST}"
  port: 6379
  database: 0
  password: "${REDIS_PASSWORD}"
  cluster_mode: true
  sentinel_hosts: "${REDIS_SENTINELS}"

security:
  cors_enabled: false
  allowed_hosts: ["nasaai.app", "www.nasaai.app"]
  jwt_expiry: 4h
  session_timeout: 30m
  csrf_protection: true
  rate_limiting:
    enabled: true
    requests_per_minute: 60

features:
  image_analysis: true
  data_export: true
  admin_panel: false
  debug_mode: false

monitoring:
  metrics_enabled: true
  health_check_interval: 10s
  log_retention_days: 90
  performance_monitoring: true

backup:
  enabled: true
  schedule: "0 2 * * *"  # Daily at 2 AM
  retention_days: 30
  storage_location: "${BACKUP_BUCKET}"
```

## Environment Variables

### Required Environment Variables
```bash
# Application Environment
APP_ENV=production                    # development, staging, production
APP_SECRET_KEY=your-secret-key-here   # Application secret key

# Database Configuration
DB_HOST=prod-db.nasaai.app           # Database hostname
DB_PORT=5432                         # Database port
DB_NAME=nasaai_prod                  # Database name
DB_USER=app_user                     # Database username
DB_PASSWORD=secure-db-password       # Database password

# NASA API Configuration
NASA_API_KEY=your-nasa-api-key       # NASA API key
NASA_API_BASE_URL=https://api.nasa.gov  # NASA API base URL

# Redis Configuration
REDIS_HOST=redis.nasaai.app          # Redis hostname
REDIS_PORT=6379                      # Redis port
REDIS_PASSWORD=redis-password        # Redis password
REDIS_DATABASE=0                     # Redis database number

# Security Configuration
JWT_SECRET=jwt-secret-key            # JWT signing secret
ENCRYPTION_KEY=encryption-key-here   # Data encryption key

# External Services
STORAGE_BUCKET=nasaai-storage        # Cloud storage bucket
CDN_URL=https://cdn.nasaai.app       # Content delivery network URL
MAIL_SERVICE_API_KEY=mail-api-key    # Email service API key

# Monitoring and Logging
SENTRY_DSN=https://sentry.io/dsn     # Error tracking service
LOG_LEVEL=INFO                       # Logging level
METRICS_ENDPOINT=http://prometheus:9090  # Metrics collection endpoint
```

### Optional Environment Variables
```bash
# Performance Tuning
MAX_WORKERS=4                        # Number of worker processes
WORKER_TIMEOUT=300                   # Worker timeout in seconds
REQUEST_TIMEOUT=60                   # Request timeout in seconds

# Feature Flags
FEATURE_ADVANCED_ANALYSIS=true       # Enable advanced analysis features
FEATURE_EXPORT_LIMIT=1000           # Maximum records for export
FEATURE_CACHE_TTL=3600              # Cache time-to-live in seconds

# Development and Debugging
DEBUG_SQL=false                      # Enable SQL query debugging
PROFILING_ENABLED=false             # Enable performance profiling
TEST_MODE=false                     # Enable test mode features
```

## Secrets Management

### AWS Secrets Manager Configuration
```json
{
  "nasa-ai-secrets": {
    "db_password": "secure-database-password",
    "nasa_api_key": "nasa-api-key-value",
    "jwt_secret": "jwt-signing-secret",
    "encryption_key": "data-encryption-key",
    "redis_password": "redis-connection-password"
  }
}
```

### Kubernetes Secrets
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nasaai-secrets
type: Opaque
data:
  db-password: <base64-encoded-password>
  nasa-api-key: <base64-encoded-api-key>
  jwt-secret: <base64-encoded-jwt-secret>
  encryption-key: <base64-encoded-encryption-key>
  redis-password: <base64-encoded-redis-password>
```

### Docker Secrets
```bash
# Create secrets
echo "secure-db-password" | docker secret create db_password -
echo "nasa-api-key-value" | docker secret create nasa_api_key -

# Use in docker-compose
version: '3.8'
services:
  app:
    image: nasaai:latest
    secrets:
      - db_password
      - nasa_api_key
    environment:
      - DB_PASSWORD_FILE=/run/secrets/db_password
      - NASA_API_KEY_FILE=/run/secrets/nasa_api_key

secrets:
  db_password:
    external: true
  nasa_api_key:
    external: true
```

## Configuration Validation

### Configuration Schema
```python
from pydantic import BaseModel, validator
from typing import Optional, List

class DatabaseConfig(BaseModel):
    host: str
    port: int = 5432
    name: str
    user: str
    password: str
    pool_size: int = 10
    ssl_mode: str = "require"
    
    @validator('port')
    def validate_port(cls, v):
        if not 1 <= v <= 65535:
            raise ValueError('Port must be between 1 and 65535')
        return v

class NASAAPIConfig(BaseModel):
    base_url: str = "https://api.nasa.gov"
    api_key: str
    timeout: int = 60
    retry_attempts: int = 3
    rate_limit: int = 1000
    
    @validator('timeout')
    def validate_timeout(cls, v):
        if v <= 0:
            raise ValueError('Timeout must be positive')
        return v

class AppConfig(BaseModel):
    name: str
    debug: bool = False
    log_level: str = "INFO"
    host: str = "0.0.0.0"
    port: int = 8000
    
    database: DatabaseConfig
    nasa_api: NASAAPIConfig
    
    @validator('log_level')
    def validate_log_level(cls, v):
        if v not in ['DEBUG', 'INFO', 'WARN', 'ERROR']:
            raise ValueError('Invalid log level')
        return v
```

### Configuration Loading
```python
import os
import yaml
from pathlib import Path

def load_config(env: str = None) -> AppConfig:
    """Load configuration based on environment."""
    env = env or os.getenv('APP_ENV', 'development')
    
    # Load base configuration
    config_dir = Path(__file__).parent / 'config'
    base_config = yaml.safe_load((config_dir / 'base.yaml').read_text())
    
    # Load environment-specific configuration
    env_config_file = config_dir / f'{env}.yaml'
    if env_config_file.exists():
        env_config = yaml.safe_load(env_config_file.read_text())
        base_config.update(env_config)
    
    # Override with environment variables
    base_config = substitute_env_vars(base_config)
    
    # Validate configuration
    return AppConfig(**base_config)

def substitute_env_vars(config: dict) -> dict:
    """Substitute environment variables in configuration."""
    import re
    
    def replace_env_var(match):
        var_name = match.group(1)
        default_value = match.group(2) if match.group(2) else None
        return os.getenv(var_name, default_value)
    
    config_str = yaml.dump(config)
    config_str = re.sub(r'\$\{([^}:]+)(?::([^}]*))?\}', replace_env_var, config_str)
    return yaml.safe_load(config_str)
```

## Configuration Best Practices

### Security Guidelines
1. **Never commit secrets** to version control
2. **Use environment variables** for sensitive data
3. **Rotate secrets regularly** (quarterly for production)
4. **Use different secrets** for each environment
5. **Encrypt configuration files** containing sensitive data

### Environment Separation
1. **Isolate environments** completely (separate databases, APIs, etc.)
2. **Use different API keys** for each environment
3. **Implement proper access controls** for configuration
4. **Test configuration changes** in lower environments first

### Configuration Management
1. **Version control configurations** (except secrets)
2. **Document all configuration options**
3. **Validate configurations** before deployment
4. **Monitor configuration drift** between environments

### Deployment Considerations
1. **Use configuration as code** where possible
2. **Implement blue-green deployments** for configuration changes
3. **Have rollback procedures** for configuration changes
4. **Test configuration** in staging before production

---
*Last updated: [Date]*