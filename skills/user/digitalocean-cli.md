---
name: digitalocean-cli
description: Use when working with the DigitalOcean CLI (doctl) — authenticating, managing projects, deploying apps, creating droplets, managing databases (PostgreSQL, Redis), Kubernetes clusters, domains, SSH keys, load balancers, volumes, or any other DigitalOcean resource. Trigger on any mention of doctl, DigitalOcean CLI, DO CLI, "deploy to DigitalOcean", App Platform, managed databases, or "how do I do X in DigitalOcean". Use this skill even for simple questions like "how do I list my droplets" or "how do I create a PostgreSQL database" — always check here first before suggesting doctl commands.
---

# DigitalOcean CLI (doctl)

The official DigitalOcean command-line interface (`doctl`) manages all DigitalOcean resources via terminal commands. Commands follow the pattern: `doctl <service> <command> [flags]`.

**Official Documentation**: https://docs.digitalocean.com/reference/doctl/
**GitHub**: https://github.com/digitalocean/doctl

## Installation

```bash
# macOS (Homebrew)
brew install doctl

# Linux (snap)
sudo snap install doctl

# Linux (manual download)
cd ~
wget https://github.com/digitalocean/doctl/releases/download/v1.151.0/doctl-1.151.0-linux-amd64.tar.gz
tar xf doctl-1.151.0-linux-amd64.tar.gz
sudo mv doctl /usr/local/bin

# Windows (Chocolatey)
choco install doctl

# Verify installation
doctl version
```

## First Steps

```bash
# Interactive authentication (opens browser)
doctl auth init

# Authenticate with token directly
doctl auth init --access-token YOUR_API_TOKEN

# List authenticated contexts
doctl auth list

# Switch between accounts
doctl auth switch --context personal
doctl auth switch --context work

# Remove authentication
doctl auth remove --context personal

# Check account info
doctl account get
doctl account ratelimit  # Check API usage and rate limits
```

## Global Flags

```bash
--access-token, -t    # Override default token
--context            # Use named authentication context
--output, -o         # Output format: text (default), json
--verbose, -v        # Enable verbose output
--help, -h          # Show help for any command
```

## App Platform (Django, Python, Node.js, etc.)

**Perfect for deploying Puny.bz Django application**

### Create App from GitHub

```bash
# Create app from app spec YAML/JSON
doctl apps create --spec app-spec.yaml

# Create with inline spec
doctl apps create --spec - <<EOF
name: puny-bz
region: nyc
services:
- name: web
  github:
    repo: your-username/puny-bz
    branch: main
    deploy_on_push: true
  source_dir: /
  environment_slug: python
  instance_count: 2
  instance_size_slug: professional-xs
  http_port: 8000
  run_command: gunicorn punybz.wsgi:application --bind 0.0.0.0:8000
  envs:
  - key: DJANGO_SETTINGS_MODULE
    value: punybz.settings
    type: GENERAL
  - key: SECRET_KEY
    value: your-secret-key
    type: SECRET
databases:
- name: db
  engine: PG
  version: "15"
  size: db-s-1vcpu-1gb
EOF

# Create app from container registry
doctl apps create --spec - <<EOF
name: puny-bz-docker
region: nyc
services:
- name: web
  image:
    registry_type: DOCR
    repository: puny-bz
    tag: latest
  http_port: 8000
  instance_count: 2
  instance_size_slug: professional-xs
EOF
```

### Manage Apps

```bash
# List all apps
doctl apps list
doctl apps list --format ID,Spec.Name,ActiveDeployment.ID,Created

# Get app details
doctl apps get APP_ID
doctl apps get APP_ID --format ID,Spec.Name,DefaultIngress

# Update app configuration
doctl apps update APP_ID --spec updated-spec.yaml

# Trigger new deployment
doctl apps create-deployment APP_ID
doctl apps create-deployment APP_ID --force-rebuild

# View deployment details
doctl apps get-deployment APP_ID DEPLOYMENT_ID

# List deployments
doctl apps list-deployments APP_ID

# Delete app
doctl apps delete APP_ID
```

### App Logs

```bash
# Stream real-time logs
doctl apps logs APP_ID --type run --follow

# Get deployment logs
doctl apps logs APP_ID --type deploy

# Get build logs
doctl apps logs APP_ID --type build

# Filter by component
doctl apps logs APP_ID COMPONENT_NAME --follow

# Tail last 100 lines
doctl apps logs APP_ID --tail 100
```

### App Configuration

```bash
# Get app spec
doctl apps spec get APP_ID

# Validate app spec locally
doctl apps spec validate --spec app-spec.yaml

# Propose changes (dry-run)
doctl apps propose --spec updated-spec.yaml --app APP_ID

# List available instance sizes
doctl apps tier instance-size list

# List buildpacks
doctl apps list-buildpacks

# List regions
doctl apps list-regions
```

### Environment Variables

```bash
# App spec example with environment variables
envs:
- key: DATABASE_URL
  value: ${db.DATABASE_URL}  # Reference database component
  type: GENERAL
- key: REDIS_URL
  value: ${redis.DATABASE_URL}  # Reference Redis component
  type: GENERAL
- key: SECRET_KEY
  value: your-secret-key
  type: SECRET  # Encrypted at rest
- key: DEBUG
  value: "False"
  type: GENERAL
```

## Droplets (Virtual Machines)

### Create Droplets

```bash
# Basic droplet creation
doctl compute droplet create my-droplet \
  --size s-2vcpu-2gb \
  --image ubuntu-22-04-x64 \
  --region nyc1 \
  --ssh-keys YOUR_SSH_KEY_ID

# Multiple droplets at once
doctl compute droplet create web-{1..3} \
  --size s-2vcpu-4gb \
  --image ubuntu-22-04-x64 \
  --region nyc1 \
  --tag-names production,web

# With user data (cloud-init)
doctl compute droplet create django-server \
  --size s-2vcpu-4gb \
  --image ubuntu-22-04-x64 \
  --region nyc1 \
  --user-data-file cloud-init.yaml \
  --ssh-keys YOUR_SSH_KEY_ID

# With VPC
doctl compute droplet create app-server \
  --size s-2vcpu-4gb \
  --image ubuntu-22-04-x64 \
  --region nyc1 \
  --vpc-uuid VPC_ID

# Available sizes
doctl compute size list

# Available images
doctl compute image list
doctl compute image list --public  # Ubuntu, Debian, etc.

# Available regions
doctl compute region list
```

### Manage Droplets

```bash
# List droplets
doctl compute droplet list
doctl compute droplet list --format ID,Name,PublicIPv4,Status,Region

# Get droplet details
doctl compute droplet get DROPLET_ID

# Delete droplet
doctl compute droplet delete DROPLET_ID

# Power operations
doctl compute droplet-action power-on DROPLET_ID
doctl compute droplet-action power-off DROPLET_ID
doctl compute droplet-action reboot DROPLET_ID
doctl compute droplet-action shutdown DROPLET_ID

# Resize droplet
doctl compute droplet-action resize DROPLET_ID --size s-4vcpu-8gb

# Create snapshot
doctl compute droplet-action snapshot DROPLET_ID --snapshot-name backup-2026-03-11

# List droplet actions
doctl compute droplet actions DROPLET_ID

# Tag droplets
doctl compute droplet tag DROPLET_ID --tag-names production,database
doctl compute droplet untag DROPLET_ID --tag-names staging
```

## Databases (PostgreSQL, Redis/Valkey, MySQL, MongoDB, Kafka, OpenSearch)

**Critical for Puny.bz: PostgreSQL + Redis**

### Create Database Clusters

```bash
# PostgreSQL cluster
doctl databases create puny-postgres \
  --engine pg \
  --version 15 \
  --region nyc1 \
  --size db-s-2vcpu-4gb \
  --num-nodes 1

# PostgreSQL with high availability (2 standby nodes)
doctl databases create puny-postgres-ha \
  --engine pg \
  --version 15 \
  --region nyc1 \
  --size db-s-2vcpu-4gb \
  --num-nodes 3

# Redis (Valkey) cluster
doctl databases create puny-redis \
  --engine redis \
  --version 7 \
  --region nyc1 \
  --size db-s-1vcpu-1gb \
  --num-nodes 1

# Available engine versions
doctl databases options engines
doctl databases options versions --engine pg

# Available sizes
doctl databases options slugs --engine pg

# Available regions
doctl databases options regions --engine pg
```

### Manage Database Clusters

```bash
# List clusters
doctl databases list
doctl databases list --format ID,Name,Engine,Version,Status,Region

# Get cluster details
doctl databases get DATABASE_ID
doctl databases get DATABASE_ID --format ID,Name,Connection

# Get connection details (IMPORTANT for Django settings.py)
doctl databases connection DATABASE_ID
doctl databases connection DATABASE_ID --format URI

# Example output:
# postgresql://doadmin:password@db-postgresql-nyc1-12345-do-user-123456-0.db.ondigitalocean.com:25060/defaultdb?sslmode=require

# Get CA certificate (for SSL connections)
doctl databases get-ca DATABASE_ID > ca-certificate.crt

# Delete cluster
doctl databases delete DATABASE_ID
```

### Database Configuration

```bash
# Get cluster configuration
doctl databases configuration get DATABASE_ID

# Resize cluster
doctl databases resize DATABASE_ID --size db-s-4vcpu-8gb --num-nodes 3

# Migrate cluster to new region
doctl databases migrate DATABASE_ID --region sfo3

# Configure maintenance window
doctl databases maintenance-window update DATABASE_ID \
  --day tuesday \
  --hour 02:00

# Enable/disable trusted sources (IP allowlist)
doctl databases firewalls append DATABASE_ID --rule ip_addr:203.0.113.0/24
doctl databases firewalls list DATABASE_ID
doctl databases firewalls remove DATABASE_ID --uuid RULE_UUID
```

### Database Users & Access

```bash
# List database users
doctl databases user list DATABASE_ID

# Create new user
doctl databases user create DATABASE_ID new_user

# Reset user password
doctl databases user reset DATABASE_ID doadmin --password NEW_PASSWORD

# Get user details
doctl databases user get DATABASE_ID USERNAME
```

### PostgreSQL-Specific Operations

```bash
# List databases in cluster
doctl databases db list DATABASE_ID

# Create new database
doctl databases db create DATABASE_ID puny_production

# Delete database
doctl databases db delete DATABASE_ID old_database

# List connection pools
doctl databases pool list DATABASE_ID

# Create connection pool (recommended for Django)
doctl databases pool create DATABASE_ID \
  --name django-pool \
  --db puny_production \
  --user doadmin \
  --size 25 \
  --mode transaction

# Get pool connection string
doctl databases pool get DATABASE_ID django-pool --format URI
```

### Backups & Recovery

```bash
# List backups
doctl databases backups list DATABASE_ID

# Restore from backup
doctl databases backups restore DATABASE_ID BACKUP_ID

# Configure backup retention (via API, not available in doctl)
# See API section for backup configuration
```

### Read-Only Replicas

```bash
# Create read replica
doctl databases replica create DATABASE_ID \
  --name puny-replica \
  --region sfo3 \
  --size db-s-2vcpu-4gb

# List replicas
doctl databases replica list DATABASE_ID

# Get replica connection string
doctl databases replica connection DATABASE_ID REPLICA_NAME

# Delete replica
doctl databases replica delete DATABASE_ID REPLICA_NAME
```

## Kubernetes (DOKS)

```bash
# Create cluster
doctl kubernetes cluster create puny-cluster \
  --region nyc1 \
  --version 1.28.2-do.0 \
  --node-pool "name=workers;size=s-2vcpu-4gb;count=3"

# List clusters
doctl kubernetes cluster list

# Get cluster details
doctl kubernetes cluster get CLUSTER_ID

# Update cluster
doctl kubernetes cluster update CLUSTER_ID --auto-upgrade=true

# Delete cluster
doctl kubernetes cluster delete CLUSTER_ID

# Get kubeconfig
doctl kubernetes cluster kubeconfig save CLUSTER_ID

# List available versions
doctl kubernetes options versions

# Node pools
doctl kubernetes cluster node-pool create CLUSTER_ID \
  --name workers \
  --size s-4vcpu-8gb \
  --count 3 \
  --auto-scale \
  --min-nodes 2 \
  --max-nodes 5

doctl kubernetes cluster node-pool list CLUSTER_ID
doctl kubernetes cluster node-pool delete CLUSTER_ID POOL_ID
```

## Domains & DNS

```bash
# Create domain
doctl compute domain create puny.bz

# List domains
doctl compute domain list

# Get domain details
doctl compute domain get puny.bz

# Delete domain
doctl compute domain delete puny.bz

# DNS records
doctl compute domain records create puny.bz \
  --record-type A \
  --record-name @ \
  --record-data 203.0.113.1 \
  --record-ttl 3600

doctl compute domain records create puny.bz \
  --record-type CNAME \
  --record-name www \
  --record-data @ \
  --record-ttl 3600

doctl compute domain records create puny.bz \
  --record-type MX \
  --record-name @ \
  --record-data "mail.puny.bz" \
  --record-priority 10 \
  --record-ttl 3600

# List DNS records
doctl compute domain records list puny.bz

# Update DNS record
doctl compute domain records update puny.bz \
  --record-id RECORD_ID \
  --record-data 203.0.113.2

# Delete DNS record
doctl compute domain records delete puny.bz RECORD_ID
```

## Load Balancers

```bash
# Create load balancer
doctl compute load-balancer create \
  --name puny-lb \
  --region nyc1 \
  --forwarding-rules "entry_protocol:https,entry_port:443,target_protocol:http,target_port:8000,certificate_id:CERT_ID" \
  --health-check "protocol:http,port:8000,path:/health/,check_interval_seconds:10,response_timeout_seconds:5,healthy_threshold:3,unhealthy_threshold:3" \
  --droplet-ids DROPLET_ID_1,DROPLET_ID_2

# List load balancers
doctl compute load-balancer list

# Get load balancer details
doctl compute load-balancer get LB_ID

# Add droplets
doctl compute load-balancer add-droplets LB_ID --droplet-ids DROPLET_ID_3

# Remove droplets
doctl compute load-balancer remove-droplets LB_ID --droplet-ids DROPLET_ID_1

# Delete load balancer
doctl compute load-balancer delete LB_ID
```

## Block Storage (Volumes)

```bash
# Create volume
doctl compute volume create puny-storage \
  --region nyc1 \
  --size 100GiB \
  --desc "Puny.bz media storage"

# List volumes
doctl compute volume list

# Attach volume to droplet
doctl compute volume-action attach VOLUME_ID DROPLET_ID

# Detach volume
doctl compute volume-action detach VOLUME_ID

# Resize volume
doctl compute volume-action resize VOLUME_ID --size 200 --region nyc1

# Create snapshot
doctl compute volume-action snapshot VOLUME_ID --snapshot-name backup-2026-03-11

# Delete volume
doctl compute volume delete VOLUME_ID
```

## SSH Keys

```bash
# Upload SSH key
doctl compute ssh-key create laptop --public-key-file ~/.ssh/id_rsa.pub

# List SSH keys
doctl compute ssh-key list
doctl compute ssh-key list --format ID,Name,FingerPrint

# Get SSH key details
doctl compute ssh-key get KEY_ID

# Update SSH key
doctl compute ssh-key update KEY_ID --key-name new-name

# Delete SSH key
doctl compute ssh-key delete KEY_ID
```

## Spaces (Object Storage - S3 Compatible)

**Note**: `doctl` does NOT support Spaces API. Use `s3cmd` or AWS CLI instead.

```bash
# Install s3cmd
pip install s3cmd

# Configure s3cmd for DigitalOcean Spaces
s3cmd --configure \
  --access_key=YOUR_SPACES_KEY \
  --secret_key=YOUR_SPACES_SECRET \
  --host=nyc3.digitaloceanspaces.com \
  --host-bucket='%(bucket)s.nyc3.digitaloceanspaces.com'

# Create bucket
s3cmd mb s3://puny-media

# Upload files
s3cmd put media.jpg s3://puny-media/
s3cmd put --recursive media/ s3://puny-media/

# List buckets
s3cmd ls

# List files in bucket
s3cmd ls s3://puny-media/

# Download files
s3cmd get s3://puny-media/media.jpg

# Set public ACL
s3cmd setacl s3://puny-media --acl-public

# Delete files
s3cmd del s3://puny-media/media.jpg

# Generate Spaces access keys
doctl spaces keys create --name puny-spaces-key
doctl spaces keys list
```

## Container Registry (DOCR)

```bash
# Create registry
doctl registry create puny-registry

# Get registry details
doctl registry get

# Login to registry (configures Docker)
doctl registry login

# Tag and push image
docker tag puny-bz:latest registry.digitalocean.com/puny-registry/puny-bz:latest
docker push registry.digitalocean.com/puny-registry/puny-bz:latest

# List repositories
doctl registry repository list

# List repository tags
doctl registry repository list-tags puny-bz

# Delete repository tag
doctl registry repository delete-tag puny-bz latest

# Garbage collection
doctl registry garbage-collection start
doctl registry garbage-collection get-active
```

## Firewalls

```bash
# Create firewall
doctl compute firewall create \
  --name web-firewall \
  --inbound-rules "protocol:tcp,ports:80,sources:addresses:0.0.0.0/0,::/0 protocol:tcp,ports:443,sources:addresses:0.0.0.0/0,::/0 protocol:tcp,ports:22,sources:addresses:203.0.113.0/24" \
  --outbound-rules "protocol:tcp,ports:all,destinations:addresses:0.0.0.0/0,::/0 protocol:udp,ports:all,destinations:addresses:0.0.0.0/0,::/0" \
  --droplet-ids DROPLET_ID

# List firewalls
doctl compute firewall list

# Get firewall details
doctl compute firewall get FIREWALL_ID

# Add droplets to firewall
doctl compute firewall add-droplets FIREWALL_ID --droplet-ids DROPLET_ID_2

# Remove droplets
doctl compute firewall remove-droplets FIREWALL_ID --droplet-ids DROPLET_ID_1

# Add rules
doctl compute firewall add-rules FIREWALL_ID \
  --inbound-rules "protocol:tcp,ports:5432,sources:addresses:10.0.0.0/16"

# Delete firewall
doctl compute firewall delete FIREWALL_ID
```

## VPC (Virtual Private Cloud)

```bash
# Create VPC
doctl vpcs create \
  --name puny-vpc \
  --region nyc1 \
  --ip-range 10.10.0.0/16

# List VPCs
doctl vpcs list

# Get VPC details
doctl vpcs get VPC_ID

# Update VPC
doctl vpcs update VPC_ID --name new-name --description "Updated description"

# Delete VPC
doctl vpcs delete VPC_ID

# List VPC members
doctl vpcs list-members VPC_ID
```

## Monitoring & Alerts

```bash
# Create alert policy
doctl monitoring alert create \
  --type v1/insights/droplet/cpu \
  --description "High CPU usage" \
  --compare GreaterThan \
  --value 80 \
  --window 5m \
  --entities DROPLET_ID \
  --enabled=true

# List alert policies
doctl monitoring alert list

# Update alert policy
doctl monitoring alert update ALERT_ID --enabled=false

# Delete alert policy
doctl monitoring alert delete ALERT_ID

# Create uptime check
doctl monitoring uptime create \
  --name "Puny.bz Health Check" \
  --type https \
  --target https://puny.bz/health/ \
  --regions us_east,eu_west

# List uptime checks
doctl monitoring uptime list

# Get uptime check details
doctl monitoring uptime get CHECK_ID
```

## Projects

```bash
# Create project
doctl projects create \
  --name "Puny.bz Production" \
  --description "Production infrastructure for Puny.bz" \
  --purpose "Web Application" \
  --environment production

# List projects
doctl projects list

# Get project details
doctl projects get PROJECT_ID

# Update project
doctl projects update PROJECT_ID --name "New Name"

# Assign resources to project
doctl projects resources assign PROJECT_ID \
  --resource do:droplet:DROPLET_ID \
  --resource do:database:DATABASE_ID \
  --resource do:app:APP_ID

# List project resources
doctl projects resources list PROJECT_ID

# Delete project
doctl projects delete PROJECT_ID
```

## Serverless Functions

```bash
# Connect to namespace
doctl serverless connect

# Create namespace
doctl serverless namespaces create --label production --region nyc1

# List namespaces
doctl serverless namespaces list

# Deploy functions
doctl serverless deploy .

# Invoke function
doctl serverless functions invoke hello

# Get function logs
doctl serverless activations logs ACTIVATION_ID

# List activations
doctl serverless activations list

# Delete namespace
doctl serverless namespaces delete NAMESPACE_ID
```

## Billing & Usage

```bash
# Get current balance
doctl balance get

# List invoices
doctl invoice list

# Get invoice details
doctl invoice get INVOICE_ID

# Download invoice CSV
doctl invoice csv INVOICE_ID

# Get invoice summary
doctl invoice summary INVOICE_ID

# List billing history
doctl billing-history list
```

## Common Workflows for Puny.bz

### 1. Deploy Django App to App Platform

```bash
# Create app spec (app-spec.yaml)
cat > app-spec.yaml <<'EOF'
name: puny-bz
region: nyc
services:
- name: web
  github:
    repo: your-username/puny-bz
    branch: main
    deploy_on_push: true
  source_dir: /
  environment_slug: python
  instance_count: 2
  instance_size_slug: professional-xs
  http_port: 8000
  run_command: gunicorn punybz.wsgi:application --bind 0.0.0.0:8000 --workers 3
  build_command: pip install -r requirements.txt && python manage.py collectstatic --noinput
  health_check:
    http_path: /health/
  envs:
  - key: DJANGO_SETTINGS_MODULE
    value: punybz.settings
  - key: DATABASE_URL
    value: ${db.DATABASE_URL}
    scope: RUN_TIME
  - key: REDIS_URL
    value: ${redis.DATABASE_URL}
    scope: RUN_TIME
  - key: SECRET_KEY
    value: your-secret-key
    type: SECRET
databases:
- engine: PG
  name: db
  production: true
  version: "15"
- engine: REDIS
  name: redis
  production: true
  version: "7"
EOF

# Deploy app
doctl apps create --spec app-spec.yaml

# Get app URL
doctl apps list --format DefaultIngress
```

### 2. Create PostgreSQL + Redis for Puny.bz

```bash
# Create PostgreSQL cluster
doctl databases create puny-postgres \
  --engine pg \
  --version 15 \
  --region nyc1 \
  --size db-s-2vcpu-4gb \
  --num-nodes 1

# Get connection string
DB_ID=$(doctl databases list --format ID --no-header | head -1)
doctl databases connection $DB_ID --format URI

# Create Redis cluster
doctl databases create puny-redis \
  --engine redis \
  --version 7 \
  --region nyc1 \
  --size db-s-1vcpu-1gb \
  --num-nodes 1

# Get Redis connection string
REDIS_ID=$(doctl databases list --format ID --no-header | tail -1)
doctl databases connection $REDIS_ID --format URI

# Create connection pool for Django
doctl databases pool create $DB_ID \
  --name django-pool \
  --db defaultdb \
  --user doadmin \
  --size 25 \
  --mode transaction

# Get pool connection string
doctl databases pool get $DB_ID django-pool --format URI
```

### 3. Set Up Domain for App Platform

```bash
# Create domain
doctl compute domain create puny.bz

# Add CNAME for app
doctl compute domain records create puny.bz \
  --record-type CNAME \
  --record-name @ \
  --record-data your-app-slug.ondigitalocean.app \
  --record-ttl 3600

# Add www subdomain
doctl compute domain records create puny.bz \
  --record-type CNAME \
  --record-name www \
  --record-data your-app-slug.ondigitalocean.app \
  --record-ttl 3600

# Configure custom domain in app spec
doctl apps update APP_ID --spec - <<EOF
domains:
- domain: puny.bz
  type: PRIMARY
- domain: www.puny.bz
  type: ALIAS
EOF
```

### 4. Monitor App Performance

```bash
# Create CPU alert
doctl monitoring alert create \
  --type v1/insights/droplet/cpu \
  --description "High CPU on Puny.bz" \
  --compare GreaterThan \
  --value 85 \
  --window 5m \
  --entities DROPLET_ID \
  --enabled=true

# Create uptime check
doctl monitoring uptime create \
  --name "Puny.bz Production" \
  --type https \
  --target https://puny.bz/health/ \
  --regions us_east,us_west,eu_west

# Stream app logs
doctl apps logs APP_ID --type run --follow
```

## Troubleshooting

### Authentication Issues

```bash
# Verify authentication
doctl account get

# Check rate limits
doctl account ratelimit

# Re-authenticate
doctl auth init --access-token YOUR_TOKEN

# Switch contexts
doctl auth switch --context production
```

### App Platform Deployment Failures

```bash
# Check deployment logs
doctl apps logs APP_ID --type deploy --tail 100

# Check build logs
doctl apps logs APP_ID --type build

# Validate app spec
doctl apps spec validate --spec app-spec.yaml

# Force rebuild
doctl apps create-deployment APP_ID --force-rebuild
```

### Database Connection Issues

```bash
# Verify database is running
doctl databases get DATABASE_ID

# Check firewall rules
doctl databases firewalls list DATABASE_ID

# Add your IP to firewall
doctl databases firewalls append DATABASE_ID --rule ip_addr:$(curl -s ifconfig.me)/32

# Test connection
doctl databases connection DATABASE_ID
psql "$(doctl databases connection DATABASE_ID --format URI | grep postgresql)"
```

### Common Errors

**Error**: "Unable to authenticate you"
**Solution**: Run `doctl auth init` or check API token permissions

**Error**: "This resource is not available in your region"
**Solution**: Use `doctl compute region list` to find available regions

**Error**: "You have reached your droplet limit"
**Solution**: Request limit increase via support ticket or delete unused droplets

**Error**: "Database cluster not ready"
**Solution**: Wait 5-10 minutes for cluster provisioning to complete

## Output Formats

```bash
# Default text output (human-readable tables)
doctl compute droplet list

# JSON output (for scripting)
doctl compute droplet list --output json

# Custom columns
doctl compute droplet list --format ID,Name,PublicIPv4,Status,Region

# No header
doctl compute droplet list --no-header

# Example: Get all droplet IPs
doctl compute droplet list --format PublicIPv4 --no-header
```

## Scripting Examples

### Batch Create Droplets

```bash
#!/bin/bash
for i in {1..5}; do
  doctl compute droplet create "web-$i" \
    --size s-2vcpu-4gb \
    --image ubuntu-22-04-x64 \
    --region nyc1 \
    --ssh-keys YOUR_SSH_KEY_ID \
    --tag-names production,web &
done
wait
```

### Backup All Databases

```bash
#!/bin/bash
doctl databases list --format ID --no-header | while read db_id; do
  echo "Backing up database $db_id..."
  # Backups are automatic in managed databases
  doctl databases backups list $db_id
done
```

### Scale App Platform Based on Time

```bash
#!/bin/bash
HOUR=$(date +%H)
APP_ID="your-app-id"

if [ $HOUR -ge 8 ] && [ $HOUR -lt 18 ]; then
  # Business hours: scale up
  doctl apps update $APP_ID --spec - <<EOF
services:
- name: web
  instance_count: 5
EOF
else
  # Off hours: scale down
  doctl apps update $APP_ID --spec - <<EOF
services:
- name: web
  instance_count: 2
EOF
fi
```

## Best Practices

1. **Use Named Contexts**: Manage multiple accounts/projects with `doctl auth init --context`
2. **Enable JSON Output**: Use `--output json` for scripting and automation
3. **Tag Resources**: Organize resources with tags for easier management and cost tracking
4. **Use App Platform for Django**: Simpler than managing droplets + databases manually
5. **Enable Auto-Scaling**: Configure node pools/instance counts to scale automatically
6. **Use Connection Pools**: For Django apps, always use database connection pools
7. **Monitor API Rate Limits**: Check `doctl account ratelimit` regularly
8. **Use Projects**: Organize resources by environment (production, staging, dev)
9. **Automate Backups**: Use `doctl` in cron jobs for automated backup verification
10. **Version Control App Specs**: Keep `app-spec.yaml` in git for reproducible deployments

## Reference Links

- **Official Docs**: https://docs.digitalocean.com/reference/doctl/
- **GitHub**: https://github.com/digitalocean/doctl
- **App Platform**: https://docs.digitalocean.com/products/app-platform/
- **Managed Databases**: https://docs.digitalocean.com/products/databases/
- **Kubernetes**: https://docs.digitalocean.com/products/kubernetes/
- **API Reference**: https://docs.digitalocean.com/reference/api/

## Version

This guide is based on **doctl v1.151.0**. Check your version with `doctl version`.
