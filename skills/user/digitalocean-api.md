---
name: digitalocean-api
description: Use when working with the DigitalOcean REST API — programmatic resource management, automation, CI/CD integration, webhooks, or building custom tools. Trigger on any mention of DigitalOcean API, DO API, REST API, API endpoints, API automation, "integrate DigitalOcean", or "programmatically manage DigitalOcean resources". Use this skill for Python, JavaScript, curl examples, or any programmatic interaction with DigitalOcean.
---

# DigitalOcean REST API

The DigitalOcean API is a RESTful HTTP API that allows programmatic management of all DigitalOcean resources. All API requests use HTTPS and return JSON responses.

**Base URL**: `https://api.digitalocean.com/v2`
**Official Documentation**: https://docs.digitalocean.com/reference/api/
**API Explorer**: https://docs.digitalocean.com/reference/api/api-reference/

## Authentication

All API requests require authentication using a Bearer token (Personal Access Token or OAuth token).

### Generate Personal Access Token

1. Log in to DigitalOcean control panel
2. Go to **API** → **Tokens/Keys**
3. Click **Generate New Token**
4. Name it and select scopes (read, write)
5. Copy the token (shown only once)

### Authentication Header

```bash
Authorization: Bearer YOUR_API_TOKEN
```

### Example: curl

```bash
curl -X GET \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.digitalocean.com/v2/account"
```

### Example: Python (requests)

```python
import requests

API_TOKEN = "your_api_token_here"
headers = {
    "Authorization": f"Bearer {API_TOKEN}",
    "Content-Type": "application/json"
}

response = requests.get("https://api.digitalocean.com/v2/account", headers=headers)
print(response.json())
```

### Example: Python (PyDo - Official Client)

```python
import os
from pydo import Client

# Install: pip install pydo
client = Client(token=os.environ.get("DIGITALOCEAN_TOKEN"))

# Get account info
account = client.account.get()
print(account)
```

### Example: JavaScript (Node.js)

```javascript
const axios = require('axios');

const API_TOKEN = process.env.DIGITALOCEAN_TOKEN;
const client = axios.create({
  baseURL: 'https://api.digitalocean.com/v2',
  headers: {
    'Authorization': `Bearer ${API_TOKEN}`,
    'Content-Type': 'application/json'
  }
});

// Get account info
client.get('/account')
  .then(response => console.log(response.data))
  .catch(error => console.error(error));
```

## Rate Limits

**Standard Rate Limit**: 5,000 requests per hour per token

### Rate Limit Headers (Returned in Every Response)

```http
RateLimit-Limit: 5000
RateLimit-Remaining: 4999
RateLimit-Reset: 1709251200
```

### Check Rate Limit

```bash
curl -X GET \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  "https://api.digitalocean.com/v2/account" \
  -I | grep RateLimit
```

### Handle Rate Limiting

```python
import time
import requests

def api_request_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)

        if response.status_code == 429:  # Too Many Requests
            retry_after = int(response.headers.get('Retry-After', 60))
            print(f"Rate limited. Retrying after {retry_after} seconds...")
            time.sleep(retry_after)
            continue

        response.raise_for_status()
        return response.json()

    raise Exception("Max retries exceeded")
```

## Pagination

List endpoints support pagination using `page` and `per_page` query parameters.

**Default**: 20 items per page
**Maximum**: 200 items per page

### Pagination Parameters

```bash
?page=2&per_page=50
```

### Pagination Response

```json
{
  "droplets": [...],
  "links": {
    "pages": {
      "first": "https://api.digitalocean.com/v2/droplets?page=1",
      "prev": "https://api.digitalocean.com/v2/droplets?page=1",
      "next": "https://api.digitalocean.com/v2/droplets?page=3",
      "last": "https://api.digitalocean.com/v2/droplets?page=10"
    }
  },
  "meta": {
    "total": 200
  }
}
```

### Example: Paginate All Results (Python)

```python
def get_all_droplets(client):
    all_droplets = []
    page = 1

    while True:
        response = client.get(f"/droplets?page={page}&per_page=200")
        data = response.json()

        all_droplets.extend(data['droplets'])

        # Check if there's a next page
        if 'next' not in data.get('links', {}).get('pages', {}):
            break

        page += 1

    return all_droplets
```

## Error Handling

### HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request succeeded |
| 201 | Created | Resource created successfully |
| 202 | Accepted | Request accepted (async operation) |
| 204 | No Content | Request succeeded, no response body |
| 400 | Bad Request | Invalid request parameters |
| 401 | Unauthorized | Missing or invalid authentication token |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable Entity | Validation error |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | DigitalOcean server error |
| 503 | Service Unavailable | Temporary maintenance |

### Error Response Format

```json
{
  "id": "not_found",
  "message": "The resource you requested could not be found."
}
```

### Example: Error Handling (Python)

```python
import requests

try:
    response = requests.get(
        "https://api.digitalocean.com/v2/droplets/12345",
        headers={"Authorization": f"Bearer {API_TOKEN}"}
    )
    response.raise_for_status()
    droplet = response.json()
except requests.exceptions.HTTPError as e:
    if e.response.status_code == 404:
        print("Droplet not found")
    elif e.response.status_code == 401:
        print("Invalid API token")
    elif e.response.status_code == 429:
        print("Rate limit exceeded")
    else:
        print(f"Error: {e.response.json()}")
```

## API Endpoints by Service

### Account

```bash
# Get account information
GET /v2/account
```

**Example**:
```bash
curl -X GET \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN" \
  "https://api.digitalocean.com/v2/account"
```

**Response**:
```json
{
  "account": {
    "droplet_limit": 25,
    "floating_ip_limit": 5,
    "email": "user@example.com",
    "uuid": "abc123",
    "email_verified": true,
    "status": "active",
    "status_message": ""
  }
}
```

### Droplets (Virtual Machines)

```bash
# List all droplets
GET /v2/droplets

# Get droplet by ID
GET /v2/droplets/{droplet_id}

# Create droplet
POST /v2/droplets

# Delete droplet
DELETE /v2/droplets/{droplet_id}

# List droplet actions
GET /v2/droplets/{droplet_id}/actions

# Perform droplet action
POST /v2/droplets/{droplet_id}/actions
```

#### Create Droplet

```bash
curl -X POST \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "example-droplet",
    "region": "nyc1",
    "size": "s-2vcpu-4gb",
    "image": "ubuntu-22-04-x64",
    "ssh_keys": [12345],
    "backups": true,
    "ipv6": true,
    "monitoring": true,
    "tags": ["production", "web"],
    "user_data": "#!/bin/bash\napt update && apt upgrade -y"
  }' \
  "https://api.digitalocean.com/v2/droplets"
```

**Python Example**:
```python
droplet_data = {
    "name": "puny-web-1",
    "region": "nyc1",
    "size": "s-2vcpu-4gb",
    "image": "ubuntu-22-04-x64",
    "ssh_keys": [12345],
    "backups": True,
    "monitoring": True,
    "tags": ["production", "web"],
    "vpc_uuid": "abc123"  # Optional VPC
}

response = requests.post(
    "https://api.digitalocean.com/v2/droplets",
    headers=headers,
    json=droplet_data
)

droplet = response.json()['droplet']
print(f"Created droplet ID: {droplet['id']}")
```

#### Droplet Actions

```bash
# Power on
POST /v2/droplets/{droplet_id}/actions
{"type": "power_on"}

# Power off
POST /v2/droplets/{droplet_id}/actions
{"type": "power_off"}

# Reboot
POST /v2/droplets/{droplet_id}/actions
{"type": "reboot"}

# Shutdown (graceful)
POST /v2/droplets/{droplet_id}/actions
{"type": "shutdown"}

# Resize
POST /v2/droplets/{droplet_id}/actions
{"type": "resize", "size": "s-4vcpu-8gb", "disk": true}

# Snapshot
POST /v2/droplets/{droplet_id}/actions
{"type": "snapshot", "name": "backup-2026-03-11"}

# Restore from snapshot
POST /v2/droplets/{droplet_id}/actions
{"type": "restore", "image": 12345}
```

### App Platform

```bash
# List all apps
GET /v2/apps

# Get app by ID
GET /v2/apps/{app_id}

# Create app
POST /v2/apps

# Update app
PUT /v2/apps/{app_id}

# Delete app
DELETE /v2/apps/{app_id}

# Create deployment
POST /v2/apps/{app_id}/deployments

# Get deployment
GET /v2/apps/{app_id}/deployments/{deployment_id}

# Get logs
GET /v2/apps/{app_id}/deployments/{deployment_id}/components/{component_name}/logs
```

#### Create App (Django/Python)

```bash
curl -X POST \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "spec": {
      "name": "puny-bz",
      "region": "nyc",
      "services": [{
        "name": "web",
        "github": {
          "repo": "your-username/puny-bz",
          "branch": "main",
          "deploy_on_push": true
        },
        "environment_slug": "python",
        "instance_count": 2,
        "instance_size_slug": "professional-xs",
        "http_port": 8000,
        "run_command": "gunicorn punybz.wsgi:application --bind 0.0.0.0:8000",
        "build_command": "pip install -r requirements.txt && python manage.py collectstatic --noinput",
        "health_check": {
          "http_path": "/health/"
        },
        "envs": [{
          "key": "DATABASE_URL",
          "value": "${db.DATABASE_URL}",
          "scope": "RUN_TIME"
        }, {
          "key": "SECRET_KEY",
          "value": "your-secret-key",
          "type": "SECRET"
        }]
      }],
      "databases": [{
        "engine": "PG",
        "name": "db",
        "production": true,
        "version": "15"
      }, {
        "engine": "REDIS",
        "name": "redis",
        "production": true,
        "version": "7"
      }]
    }
  }' \
  "https://api.digitalocean.com/v2/apps"
```

**Python Example**:
```python
app_spec = {
    "spec": {
        "name": "puny-bz",
        "region": "nyc",
        "services": [{
            "name": "web",
            "github": {
                "repo": "your-username/puny-bz",
                "branch": "main",
                "deploy_on_push": True
            },
            "environment_slug": "python",
            "instance_count": 2,
            "instance_size_slug": "professional-xs",
            "http_port": 8000,
            "run_command": "gunicorn punybz.wsgi:application --bind 0.0.0.0:8000 --workers 3",
            "build_command": "pip install -r requirements.txt && python manage.py collectstatic --noinput",
            "health_check": {
                "http_path": "/health/"
            },
            "envs": [
                {"key": "DATABASE_URL", "value": "${db.DATABASE_URL}", "scope": "RUN_TIME"},
                {"key": "REDIS_URL", "value": "${redis.DATABASE_URL}", "scope": "RUN_TIME"},
                {"key": "SECRET_KEY", "value": os.environ['DJANGO_SECRET_KEY'], "type": "SECRET"}
            ]
        }],
        "databases": [
            {"engine": "PG", "name": "db", "production": True, "version": "15"},
            {"engine": "REDIS", "name": "redis", "production": True, "version": "7"}
        ]
    }
}

response = requests.post(
    "https://api.digitalocean.com/v2/apps",
    headers=headers,
    json=app_spec
)

app = response.json()['app']
print(f"App created: {app['default_ingress']}")
```

#### Trigger Deployment

```python
def trigger_deployment(app_id, force_build=False):
    """Trigger a new deployment for an app"""
    deployment_data = {
        "force_build": force_build
    }

    response = requests.post(
        f"https://api.digitalocean.com/v2/apps/{app_id}/deployments",
        headers=headers,
        json=deployment_data
    )

    deployment = response.json()['deployment']
    return deployment['id']

# Usage
app_id = "abc123"
deployment_id = trigger_deployment(app_id, force_build=True)
print(f"Deployment triggered: {deployment_id}")
```

#### Get App Logs

```python
def get_app_logs(app_id, deployment_id, component_name, log_type="RUN"):
    """Get logs for app component

    log_type: BUILD, DEPLOY, or RUN
    """
    params = {
        "type": log_type,
        "follow": False
    }

    response = requests.get(
        f"https://api.digitalocean.com/v2/apps/{app_id}/deployments/{deployment_id}/components/{component_name}/logs",
        headers=headers,
        params=params
    )

    return response.json()['historic_urls']

# Usage
logs_url = get_app_logs("abc123", "deployment-123", "web", log_type="RUN")
print(f"Logs available at: {logs_url}")
```

### Databases

```bash
# List databases
GET /v2/databases

# Get database cluster
GET /v2/databases/{database_id}

# Create database cluster
POST /v2/databases

# Delete database cluster
DELETE /v2/databases/{database_id}

# Resize cluster
PUT /v2/databases/{database_id}/resize

# Migrate cluster
PUT /v2/databases/{database_id}/migrate

# Get connection details
GET /v2/databases/{database_id}

# List database users
GET /v2/databases/{database_id}/users

# Create database user
POST /v2/databases/{database_id}/users

# List databases in cluster
GET /v2/databases/{database_id}/dbs

# Create database
POST /v2/databases/{database_id}/dbs

# List connection pools
GET /v2/databases/{database_id}/pools

# Create connection pool
POST /v2/databases/{database_id}/pools

# List backups
GET /v2/databases/{database_id}/backups

# Restore from backup
POST /v2/databases/{database_id}/backups/{backup_id}/restore

# List replicas
GET /v2/databases/{database_id}/replicas

# Create replica
POST /v2/databases/{database_id}/replicas

# Manage firewall
GET /v2/databases/{database_id}/firewall
PUT /v2/databases/{database_id}/firewall
```

#### Create PostgreSQL Database

```bash
curl -X POST \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "puny-postgres",
    "engine": "pg",
    "version": "15",
    "region": "nyc1",
    "size": "db-s-2vcpu-4gb",
    "num_nodes": 1,
    "tags": ["production"]
  }' \
  "https://api.digitalocean.com/v2/databases"
```

**Python Example**:
```python
database_spec = {
    "name": "puny-postgres",
    "engine": "pg",
    "version": "15",
    "region": "nyc1",
    "size": "db-s-2vcpu-4gb",
    "num_nodes": 1,
    "tags": ["production"],
    "private_network_uuid": "abc123"  # Optional VPC
}

response = requests.post(
    "https://api.digitalocean.com/v2/databases",
    headers=headers,
    json=database_spec
)

database = response.json()['database']
db_id = database['id']
print(f"Database created: {db_id}")

# Wait for database to be ready
import time
while True:
    db_info = requests.get(
        f"https://api.digitalocean.com/v2/databases/{db_id}",
        headers=headers
    ).json()['database']

    if db_info['status'] == 'online':
        print("Database is online!")
        break

    print(f"Status: {db_info['status']}, waiting...")
    time.sleep(30)
```

#### Create Connection Pool (Recommended for Django)

```python
def create_connection_pool(database_id, pool_name="django-pool", pool_size=25):
    """Create connection pool for Django app"""
    pool_data = {
        "name": pool_name,
        "mode": "transaction",
        "size": pool_size,
        "db": "defaultdb",
        "user": "doadmin"
    }

    response = requests.post(
        f"https://api.digitalocean.com/v2/databases/{database_id}/pools",
        headers=headers,
        json=pool_data
    )

    pool = response.json()['pool']
    return pool['connection']  # Connection details

# Usage
connection = create_connection_pool("db-abc123")
print(f"Pool connection: {connection['uri']}")
# Use this URI in Django DATABASE_URL
```

#### Configure Database Firewall

```python
def configure_database_firewall(database_id, allowed_ips):
    """Configure database firewall to allow specific IPs

    allowed_ips: List of IP addresses or CIDR blocks
    Example: ['203.0.113.1', '10.0.0.0/16']
    """
    firewall_rules = [{
        "type": "ip_addr",
        "value": ip
    } for ip in allowed_ips]

    firewall_data = {
        "rules": firewall_rules
    }

    response = requests.put(
        f"https://api.digitalocean.com/v2/databases/{database_id}/firewall",
        headers=headers,
        json=firewall_data
    )

    return response.json()

# Usage
configure_database_firewall("db-abc123", [
    "203.0.113.1",      # Office IP
    "10.10.0.0/16"      # VPC network
])
```

#### Create Read Replica

```python
def create_read_replica(database_id, replica_name, region="nyc1", size="db-s-2vcpu-4gb"):
    """Create read-only replica for scaling reads"""
    replica_data = {
        "name": replica_name,
        "region": region,
        "size": size
    }

    response = requests.post(
        f"https://api.digitalocean.com/v2/databases/{database_id}/replicas",
        headers=headers,
        json=replica_data
    )

    replica = response.json()['replica']
    return replica['connection']

# Usage
replica_connection = create_read_replica("db-abc123", "puny-replica-1", region="sfo3")
print(f"Replica connection: {replica_connection['uri']}")
```

### Kubernetes (DOKS)

```bash
# List clusters
GET /v2/kubernetes/clusters

# Get cluster
GET /v2/kubernetes/clusters/{cluster_id}

# Create cluster
POST /v2/kubernetes/clusters

# Update cluster
PUT /v2/kubernetes/clusters/{cluster_id}

# Delete cluster
DELETE /v2/kubernetes/clusters/{cluster_id}

# Get kubeconfig
GET /v2/kubernetes/clusters/{cluster_id}/kubeconfig

# List node pools
GET /v2/kubernetes/clusters/{cluster_id}/node_pools

# Add node pool
POST /v2/kubernetes/clusters/{cluster_id}/node_pools

# Delete node pool
DELETE /v2/kubernetes/clusters/{cluster_id}/node_pools/{pool_id}
```

#### Create Kubernetes Cluster

```python
cluster_spec = {
    "name": "puny-cluster",
    "region": "nyc1",
    "version": "1.28.2-do.0",
    "auto_upgrade": True,
    "tags": ["production"],
    "node_pools": [{
        "name": "workers",
        "size": "s-2vcpu-4gb",
        "count": 3,
        "auto_scale": True,
        "min_nodes": 2,
        "max_nodes": 5,
        "tags": ["worker"]
    }],
    "vpc_uuid": "abc123"  # Optional VPC
}

response = requests.post(
    "https://api.digitalocean.com/v2/kubernetes/clusters",
    headers=headers,
    json=cluster_spec
)

cluster = response.json()['kubernetes_cluster']
print(f"Cluster created: {cluster['id']}")
```

### Domains & DNS

```bash
# List domains
GET /v2/domains

# Get domain
GET /v2/domains/{domain_name}

# Create domain
POST /v2/domains

# Delete domain
DELETE /v2/domains/{domain_name}

# List DNS records
GET /v2/domains/{domain_name}/records

# Create DNS record
POST /v2/domains/{domain_name}/records

# Update DNS record
PUT /v2/domains/{domain_name}/records/{record_id}

# Delete DNS record
DELETE /v2/domains/{domain_name}/records/{record_id}
```

#### Create Domain and DNS Records

```python
# Create domain
domain_data = {
    "name": "puny.bz",
    "ip_address": "203.0.113.1"  # Optional: creates A record for @
}

response = requests.post(
    "https://api.digitalocean.com/v2/domains",
    headers=headers,
    json=domain_data
)

# Create A record
a_record = {
    "type": "A",
    "name": "@",
    "data": "203.0.113.1",
    "ttl": 3600
}

requests.post(
    "https://api.digitalocean.com/v2/domains/puny.bz/records",
    headers=headers,
    json=a_record
)

# Create CNAME record
cname_record = {
    "type": "CNAME",
    "name": "www",
    "data": "@",
    "ttl": 3600
}

requests.post(
    "https://api.digitalocean.com/v2/domains/puny.bz/records",
    headers=headers,
    json=cname_record
)

# Create MX record
mx_record = {
    "type": "MX",
    "name": "@",
    "data": "mail.puny.bz",
    "priority": 10,
    "ttl": 3600
}

requests.post(
    "https://api.digitalocean.com/v2/domains/puny.bz/records",
    headers=headers,
    json=mx_record
)
```

### Load Balancers

```bash
# List load balancers
GET /v2/load_balancers

# Get load balancer
GET /v2/load_balancers/{lb_id}

# Create load balancer
POST /v2/load_balancers

# Update load balancer
PUT /v2/load_balancers/{lb_id}

# Delete load balancer
DELETE /v2/load_balancers/{lb_id}

# Add droplets
POST /v2/load_balancers/{lb_id}/droplets

# Remove droplets
DELETE /v2/load_balancers/{lb_id}/droplets
```

#### Create Load Balancer

```python
lb_spec = {
    "name": "puny-lb",
    "algorithm": "round_robin",
    "region": "nyc1",
    "forwarding_rules": [{
        "entry_protocol": "https",
        "entry_port": 443,
        "target_protocol": "http",
        "target_port": 8000,
        "certificate_id": "cert-abc123",
        "tls_passthrough": False
    }, {
        "entry_protocol": "http",
        "entry_port": 80,
        "target_protocol": "http",
        "target_port": 8000
    }],
    "health_check": {
        "protocol": "http",
        "port": 8000,
        "path": "/health/",
        "check_interval_seconds": 10,
        "response_timeout_seconds": 5,
        "healthy_threshold": 3,
        "unhealthy_threshold": 3
    },
    "sticky_sessions": {
        "type": "cookies",
        "cookie_name": "lb",
        "cookie_ttl_seconds": 3600
    },
    "droplet_ids": [12345, 67890],
    "tag": "web",  # Or use droplet_ids
    "vpc_uuid": "abc123"
}

response = requests.post(
    "https://api.digitalocean.com/v2/load_balancers",
    headers=headers,
    json=lb_spec
)

lb = response.json()['load_balancer']
print(f"Load balancer created: {lb['ip']}")
```

### Block Storage (Volumes)

```bash
# List volumes
GET /v2/volumes

# Get volume
GET /v2/volumes/{volume_id}

# Create volume
POST /v2/volumes

# Delete volume
DELETE /v2/volumes/{volume_id}

# Attach to droplet
POST /v2/volumes/actions
{"type": "attach", "volume_id": "...", "droplet_id": 12345}

# Detach from droplet
POST /v2/volumes/actions
{"type": "detach", "volume_id": "...", "droplet_id": 12345}

# Resize volume
POST /v2/volumes/{volume_id}/actions
{"type": "resize", "size_gigabytes": 200, "region": "nyc1"}
```

### Spaces (Object Storage)

**Note**: Spaces uses the S3-compatible API, not the standard DigitalOcean API.

**Spaces API Base URL**: `https://{region}.digitaloceanspaces.com`

#### Generate Spaces Access Keys

```bash
# Via API
POST /v2/spaces/keys
{
  "name": "puny-spaces-key"
}
```

#### Use boto3 (Python S3 Client)

```python
import boto3

# Configure boto3 for Spaces
session = boto3.session.Session()
client = session.client('s3',
    region_name='nyc3',
    endpoint_url='https://nyc3.digitaloceanspaces.com',
    aws_access_key_id='YOUR_SPACES_KEY',
    aws_secret_access_key='YOUR_SPACES_SECRET'
)

# Create bucket
client.create_bucket(Bucket='puny-media')

# Upload file
client.upload_file('local-file.jpg', 'puny-media', 'media/file.jpg')

# Set public ACL
client.put_object_acl(
    Bucket='puny-media',
    Key='media/file.jpg',
    ACL='public-read'
)

# Generate presigned URL
url = client.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'puny-media', 'Key': 'media/file.jpg'},
    ExpiresIn=3600
)
print(f"Download URL: {url}")

# List objects
response = client.list_objects_v2(Bucket='puny-media')
for obj in response.get('Contents', []):
    print(obj['Key'])

# Delete object
client.delete_object(Bucket='puny-media', Key='media/file.jpg')
```

### Container Registry

```bash
# Get registry info
GET /v2/registry

# Create registry
POST /v2/registry
{"name": "puny-registry", "subscription_tier_slug": "basic"}

# Delete registry
DELETE /v2/registry

# List repositories
GET /v2/registry/{registry_name}/repositories

# List repository tags
GET /v2/registry/{registry_name}/repositories/{repository_name}

# Delete repository tag
DELETE /v2/registry/{registry_name}/repositories/{repository_name}/tags/{tag}

# Validate registry name
POST /v2/registry/validate-name
{"name": "puny-registry"}

# Start garbage collection
POST /v2/registry/{registry_name}/garbage-collection

# Get active garbage collection
GET /v2/registry/{registry_name}/garbage-collection
```

### Firewalls

```bash
# List firewalls
GET /v2/firewalls

# Get firewall
GET /v2/firewalls/{firewall_id}

# Create firewall
POST /v2/firewalls

# Update firewall
PUT /v2/firewalls/{firewall_id}

# Delete firewall
DELETE /v2/firewalls/{firewall_id}

# Add droplets
POST /v2/firewalls/{firewall_id}/droplets

# Remove droplets
DELETE /v2/firewalls/{firewall_id}/droplets
```

#### Create Firewall

```python
firewall_spec = {
    "name": "web-firewall",
    "inbound_rules": [
        {
            "protocol": "tcp",
            "ports": "80",
            "sources": {
                "addresses": ["0.0.0.0/0", "::/0"]
            }
        },
        {
            "protocol": "tcp",
            "ports": "443",
            "sources": {
                "addresses": ["0.0.0.0/0", "::/0"]
            }
        },
        {
            "protocol": "tcp",
            "ports": "22",
            "sources": {
                "addresses": ["203.0.113.0/24"]  # Office IP
            }
        }
    ],
    "outbound_rules": [
        {
            "protocol": "tcp",
            "ports": "all",
            "destinations": {
                "addresses": ["0.0.0.0/0", "::/0"]
            }
        },
        {
            "protocol": "udp",
            "ports": "all",
            "destinations": {
                "addresses": ["0.0.0.0/0", "::/0"]
            }
        }
    ],
    "droplet_ids": [12345],
    "tags": ["web"]
}

response = requests.post(
    "https://api.digitalocean.com/v2/firewalls",
    headers=headers,
    json=firewall_spec
)
```

### VPCs

```bash
# List VPCs
GET /v2/vpcs

# Get VPC
GET /v2/vpcs/{vpc_id}

# Create VPC
POST /v2/vpcs

# Update VPC
PATCH /v2/vpcs/{vpc_id}

# Delete VPC
DELETE /v2/vpcs/{vpc_id}

# List VPC members
GET /v2/vpcs/{vpc_id}/members
```

### Monitoring & Alerts

```bash
# Create alert policy
POST /v2/monitoring/alerts

# List alert policies
GET /v2/monitoring/alerts

# Update alert policy
PUT /v2/monitoring/alerts/{alert_uuid}

# Delete alert policy
DELETE /v2/monitoring/alerts/{alert_uuid}

# Get droplet metrics
GET /v2/monitoring/metrics/droplet/{metric_name}?host_id={droplet_id}&start={timestamp}&end={timestamp}
```

#### Create CPU Alert

```python
alert_spec = {
    "type": "v1/insights/droplet/cpu",
    "description": "High CPU usage on Puny.bz",
    "compare": "GreaterThan",
    "value": 85,
    "window": "5m",
    "entities": ["12345"],  # Droplet IDs
    "enabled": True,
    "tags": []
}

response = requests.post(
    "https://api.digitalocean.com/v2/monitoring/alerts",
    headers=headers,
    json=alert_spec
)
```

### Projects

```bash
# List projects
GET /v2/projects

# Get project
GET /v2/projects/{project_id}

# Create project
POST /v2/projects

# Update project
PATCH /v2/projects/{project_id}

# Delete project
DELETE /v2/projects/{project_id}

# List project resources
GET /v2/projects/{project_id}/resources

# Assign resources to project
POST /v2/projects/{project_id}/resources
```

## Common Workflows for Puny.bz

### 1. Complete Django App Deployment

```python
import os
import requests
import time

API_TOKEN = os.environ['DIGITALOCEAN_TOKEN']
headers = {
    "Authorization": f"Bearer {API_TOKEN}",
    "Content-Type": "application/json"
}

# Step 1: Create VPC
vpc_spec = {
    "name": "puny-vpc",
    "region": "nyc1",
    "ip_range": "10.10.0.0/16"
}
vpc = requests.post(
    "https://api.digitalocean.com/v2/vpcs",
    headers=headers,
    json=vpc_spec
).json()['vpc']
vpc_id = vpc['id']
print(f"VPC created: {vpc_id}")

# Step 2: Create PostgreSQL database
db_spec = {
    "name": "puny-postgres",
    "engine": "pg",
    "version": "15",
    "region": "nyc1",
    "size": "db-s-2vcpu-4gb",
    "num_nodes": 1,
    "private_network_uuid": vpc_id
}
db = requests.post(
    "https://api.digitalocean.com/v2/databases",
    headers=headers,
    json=db_spec
).json()['database']
db_id = db['id']
print(f"Database created: {db_id}")

# Wait for database to be online
while True:
    db_info = requests.get(
        f"https://api.digitalocean.com/v2/databases/{db_id}",
        headers=headers
    ).json()['database']
    if db_info['status'] == 'online':
        break
    time.sleep(30)

# Step 3: Create Redis
redis_spec = {
    "name": "puny-redis",
    "engine": "redis",
    "version": "7",
    "region": "nyc1",
    "size": "db-s-1vcpu-1gb",
    "num_nodes": 1,
    "private_network_uuid": vpc_id
}
redis = requests.post(
    "https://api.digitalocean.com/v2/databases",
    headers=headers,
    json=redis_spec
).json()['database']
redis_id = redis['id']
print(f"Redis created: {redis_id}")

# Step 4: Create connection pool
pool_data = {
    "name": "django-pool",
    "mode": "transaction",
    "size": 25,
    "db": "defaultdb",
    "user": "doadmin"
}
pool = requests.post(
    f"https://api.digitalocean.com/v2/databases/{db_id}/pools",
    headers=headers,
    json=pool_data
).json()['pool']
db_uri = pool['connection']['uri']
print(f"Connection pool created: {db_uri}")

# Step 5: Deploy app
app_spec = {
    "spec": {
        "name": "puny-bz",
        "region": "nyc",
        "services": [{
            "name": "web",
            "github": {
                "repo": "your-username/puny-bz",
                "branch": "main",
                "deploy_on_push": True
            },
            "environment_slug": "python",
            "instance_count": 2,
            "instance_size_slug": "professional-xs",
            "http_port": 8000,
            "run_command": "gunicorn punybz.wsgi:application --bind 0.0.0.0:8000",
            "envs": [
                {"key": "DATABASE_URL", "value": db_uri},
                {"key": "SECRET_KEY", "value": os.environ['DJANGO_SECRET_KEY'], "type": "SECRET"}
            ]
        }]
    }
}
app = requests.post(
    "https://api.digitalocean.com/v2/apps",
    headers=headers,
    json=app_spec
).json()['app']
print(f"App deployed: {app['default_ingress']}")
```

### 2. Automated Database Backups Monitor

```python
def check_database_backups():
    """Monitor database backups and alert if outdated"""
    databases = requests.get(
        "https://api.digitalocean.com/v2/databases",
        headers=headers
    ).json()['databases']

    for db in databases:
        db_id = db['id']
        backups = requests.get(
            f"https://api.digitalocean.com/v2/databases/{db_id}/backups",
            headers=headers
        ).json()['backups']

        if not backups:
            print(f"⚠️ No backups for {db['name']}")
            continue

        latest = backups[0]
        created_at = datetime.fromisoformat(latest['created_at'])
        age = datetime.now(timezone.utc) - created_at

        if age.days > 1:
            print(f"⚠️ Backup for {db['name']} is {age.days} days old")
        else:
            print(f"✓ {db['name']} backup is recent ({age.hours} hours old)")
```

### 3. Auto-Scale Based on Metrics

```python
def auto_scale_app(app_id, cpu_threshold=80):
    """Scale app based on CPU usage"""
    # Get current deployment
    app = requests.get(
        f"https://api.digitalocean.com/v2/apps/{app_id}",
        headers=headers
    ).json()['app']

    current_count = app['spec']['services'][0]['instance_count']

    # Get metrics (requires droplet IDs from app components)
    # This is simplified - actual implementation would query metrics API
    avg_cpu = 85  # Placeholder

    if avg_cpu > cpu_threshold and current_count < 5:
        # Scale up
        new_count = current_count + 1
        app['spec']['services'][0]['instance_count'] = new_count

        requests.put(
            f"https://api.digitalocean.com/v2/apps/{app_id}",
            headers=headers,
            json={"spec": app['spec']}
        )
        print(f"Scaled up to {new_count} instances")

    elif avg_cpu < 30 and current_count > 2:
        # Scale down
        new_count = current_count - 1
        app['spec']['services'][0]['instance_count'] = new_count

        requests.put(
            f"https://api.digitalocean.com/v2/apps/{app_id}",
            headers=headers,
            json={"spec": app['spec']}
        )
        print(f"Scaled down to {new_count} instances")
```

## Best Practices

1. **Use Environment Variables for Tokens**: Never hardcode API tokens
2. **Handle Rate Limits**: Implement exponential backoff for 429 errors
3. **Use Pagination**: Always paginate list endpoints for large result sets
4. **Error Handling**: Catch and handle all HTTP errors appropriately
5. **VPCs for Security**: Deploy databases and apps in private VPCs
6. **Connection Pools**: Use database connection pools for Django apps
7. **Monitoring**: Set up alerts for CPU, memory, disk usage
8. **Backups**: Enable automated backups for databases and droplets
9. **Tags**: Tag all resources for cost tracking and organization
10. **Projects**: Organize resources by environment (production, staging)

## Reference Links

- **API Documentation**: https://docs.digitalocean.com/reference/api/
- **PyDo (Official Python Client)**: https://github.com/digitalocean/pydo
- **API Explorer**: https://docs.digitalocean.com/reference/api/api-reference/
- **Status Page**: https://status.digitalocean.com/
- **Community**: https://www.digitalocean.com/community

## Rate Limit Summary

- **Limit**: 5,000 requests per hour
- **Headers**: `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`
- **Error Code**: 429 (Too Many Requests)
- **Best Practice**: Implement exponential backoff

## Authentication Summary

- **Method**: Bearer token in `Authorization` header
- **Generate Token**: Control Panel → API → Tokens/Keys
- **Scopes**: Read, Write
- **Best Practice**: Use environment variables, rotate regularly
