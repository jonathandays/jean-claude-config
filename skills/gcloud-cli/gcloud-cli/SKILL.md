---
name: gcloud-cli
description: Use when working with the Google Cloud CLI (gcloud) — authenticating, managing projects, configuring settings, working with Compute Engine, Cloud Run, GKE, Cloud SQL, Cloud Storage, IAM, secrets, deployments, or any other GCP resource management. Trigger on any mention of gcloud, Google Cloud CLI, GCP resources, Cloud Run, Cloud SQL, Cloud Storage, Secret Manager, GKE, Artifact Registry, Cloud Build, or "how do I do X in Google Cloud". Use this skill even for simple questions like "how do I list my projects" or "how do I set a default region" — always check here first before suggesting gcloud commands.
---

# gcloud CLI

The Google Cloud CLI (`gcloud`) manages authentication, configuration, and all interactions with GCP services. Commands follow the pattern: `gcloud <group> <subgroup> <command> [flags]`.

Use `gcloud <command> --help` at any point to see detailed docs. `gcloud cheat-sheet` prints a quick reference.

## First Steps

```bash
gcloud init                          # Interactive setup: login, project, defaults
gcloud version                       # Show installed version
gcloud components install COMPONENT  # Install a specific component
gcloud components update             # Update to latest
gcloud config set project PROJECT_ID # Set default project quickly
gcloud info                          # Show current environment (project, account, region)
gcloud cheat-sheet                   # Built-in quick reference
gcloud topic TOPIC                   # Supplementary help (e.g. gcloud topic filters)
```

## Auth

```bash
gcloud auth login                                      # Browser-based login
gcloud auth activate-service-account --key-file=KEY.json  # Use a service account key
gcloud auth application-default login                  # Set Application Default Credentials (for SDKs)
gcloud auth list                                       # See all credentialed accounts
gcloud auth revoke [ACCOUNT]                           # Remove credentials
gcloud auth print-access-token                         # Print current access token
gcloud auth configure-docker                           # Allow Docker to push to GCR/Artifact Registry
```

## Config & Projects

```bash
gcloud config list                                     # Show all active config values
gcloud config set project PROJECT_ID                   # Set default project
gcloud config set compute/region us-central1           # Set default region
gcloud config set compute/zone us-central1-a           # Set default zone
gcloud config get project                              # Get a single property

# Named configurations (useful for switching between projects/envs)
gcloud config configurations create dev
gcloud config configurations list
gcloud config configurations activate prod

# Projects
gcloud projects list
gcloud projects describe PROJECT_ID
gcloud projects create PROJECT_ID --name="My Project"
```

## IAM

```bash
gcloud iam service-accounts create NAME --display-name="Description"
gcloud iam service-accounts list
gcloud iam service-accounts keys create KEY.json \
  --iam-account=SA@PROJECT.iam.gserviceaccount.com
gcloud iam service-accounts keys list \
  --iam-account=SA@PROJECT.iam.gserviceaccount.com
gcloud iam service-accounts add-iam-policy-binding SA@PROJECT.iam.gserviceaccount.com \
  --member="user:USER@example.com" --role="roles/iam.serviceAccountUser"
gcloud iam service-accounts set-iam-policy-binding SA@PROJECT.iam.gserviceaccount.com \
  --policy-file=policy.json

# Grant roles on a project
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SA@PROJECT.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"

# Custom roles
gcloud iam roles create ROLE_ID --project=PROJECT_ID --file=role.yaml
gcloud iam roles list --project=PROJECT_ID
gcloud iam list-grantable-roles //cloudresourcemanager.googleapis.com/projects/PROJECT_ID
```

## Secrets (Secret Manager)

```bash
gcloud secrets create MY_SECRET --replication-policy="automatic"
gcloud secrets versions add MY_SECRET --data-file=secret.txt
echo -n "value" | gcloud secrets versions add MY_SECRET --data-file=-
gcloud secrets versions access latest --secret=MY_SECRET
gcloud secrets list
gcloud secrets describe MY_SECRET
gcloud secrets delete MY_SECRET
```

## Services / APIs

```bash
gcloud services list --enabled
gcloud services enable secretmanager.googleapis.com
gcloud services enable run.googleapis.com
gcloud services disable SERVICE_NAME
```

## Cloud Run

```bash
gcloud run deploy SERVICE_NAME \
  --image=IMAGE_URL \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated

gcloud run services list --region=us-central1
gcloud run services describe SERVICE_NAME --region=us-central1
gcloud run services delete SERVICE_NAME --region=us-central1

# Update env vars
gcloud run services update SERVICE_NAME \
  --update-env-vars KEY=VALUE \
  --region=us-central1

# Revisions
gcloud run revisions list --service=SERVICE_NAME --region=us-central1
gcloud run revisions describe REVISION_NAME --region=us-central1

# Logs
gcloud run services logs read SERVICE_NAME --region=us-central1
```

## Cloud Storage

```bash
# Newer gcloud storage commands
gcloud storage buckets create gs://BUCKET_NAME --location=us-central1
gcloud storage ls gs://BUCKET_NAME
gcloud storage cp FILE gs://BUCKET_NAME/
gcloud storage rm gs://BUCKET_NAME/FILE

# Classic gsutil (still widely used)
gsutil mb gs://BUCKET_NAME
gsutil ls gs://BUCKET_NAME
gsutil cp FILE gs://BUCKET_NAME/
gsutil cp gs://BUCKET_NAME/FILE .
gsutil rm gs://BUCKET_NAME/FILE
gsutil rsync -r LOCAL_DIR gs://BUCKET_NAME
```

## Cloud SQL

```bash
gcloud sql instances list
gcloud sql instances describe INSTANCE_NAME
gcloud sql instances create INSTANCE_NAME \
  --database-version=POSTGRES_15 \
  --tier=db-f1-micro \
  --region=us-central1

gcloud sql databases list --instance=INSTANCE_NAME
gcloud sql databases create DB_NAME --instance=INSTANCE_NAME

gcloud sql users list --instance=INSTANCE_NAME
gcloud sql users create USER --instance=INSTANCE_NAME --password=PASS

# Connect (opens psql/mysql shell via Cloud SQL Auth Proxy)
gcloud sql connect INSTANCE_NAME --user=postgres

# Import/Export
gcloud sql export sql INSTANCE_NAME gs://BUCKET/dump.sql --database=DB_NAME
gcloud sql import sql INSTANCE_NAME gs://BUCKET/dump.sql --database=DB_NAME
```

## Compute Engine (VMs)

```bash
gcloud compute instances list
gcloud compute instances create VM_NAME \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-family=debian-12 \
  --image-project=debian-cloud

gcloud compute instances describe VM_NAME --zone=us-central1-a
gcloud compute instances start VM_NAME --zone=us-central1-a
gcloud compute instances stop VM_NAME --zone=us-central1-a
gcloud compute instances delete VM_NAME --zone=us-central1-a

gcloud compute ssh VM_NAME --zone=us-central1-a
gcloud compute zones list
gcloud compute regions list

# Disk snapshots
gcloud compute disks snapshot DISK_NAME --zone=us-central1-a --snapshot-names=SNAP_NAME
gcloud compute snapshots describe SNAP_NAME
gcloud compute snapshots delete SNAP_NAME
```

## GKE (Kubernetes)

```bash
gcloud container clusters list
gcloud container clusters create CLUSTER_NAME \
  --zone=us-central1-a \
  --num-nodes=3

# Connect kubectl to the cluster
gcloud container clusters get-credentials CLUSTER_NAME --zone=us-central1-a
gcloud container clusters delete CLUSTER_NAME --zone=us-central1-a

gcloud container images list --repository=gcr.io/PROJECT_ID
```

## App Engine

```bash
gcloud app create --region=us-central  # Create the App Engine app (once per project)
gcloud app deploy                      # Deploy from app.yaml
gcloud app versions list
gcloud app browse                      # Open in browser
gcloud app logs read
gcloud app services list
```

## Artifact Registry

```bash
gcloud artifacts repositories list
gcloud artifacts repositories create REPO_NAME \
  --repository-format=docker \
  --location=us-central1

gcloud artifacts docker images list us-central1-docker.pkg.dev/PROJECT/REPO

# Auth Docker to push to Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev
```

## Cloud Build

```bash
gcloud builds submit --tag=IMAGE_URL .          # Build and push image from local dir
gcloud builds submit --config=cloudbuild.yaml . # Use custom build config
gcloud builds list
gcloud builds describe BUILD_ID
gcloud builds log BUILD_ID
```

## Logging

```bash
gcloud logging logs list
gcloud logging read "resource.type=cloud_run_revision AND severity>=ERROR" --limit=50
gcloud logging read "timestamp>=\"2026-01-01T00:00:00Z\"" --format=json
gcloud logging read 'protoPayload.status.code!=0' --limit=20
```

## KMS (Key Management)

```bash
gcloud kms keyrings create KEYRING --location=global
gcloud kms keys create KEY --keyring=KEYRING --location=global --purpose=encryption
gcloud kms encrypt --key=KEY --keyring=KEYRING --location=global \
  --plaintext-file=plain.txt --ciphertext-file=cipher.enc
gcloud kms decrypt --key=KEY --keyring=KEYRING --location=global \
  --ciphertext-file=cipher.enc --plaintext-file=decrypted.txt
```

## Cloud SQL (Additional)

```bash
gcloud sql backups describe BACKUP_ID --instance=INSTANCE_NAME
gcloud sql backups list --instance=INSTANCE_NAME
```

## Output Formatting & Filtering

These flags work with almost every `gcloud` command — very useful for scripting:

```bash
# Output formats
gcloud ... --format=json
gcloud ... --format=yaml
gcloud ... --format="table(name,status,region)"
gcloud ... --format="value(name)"           # Just the value, no headers

# Filter results
gcloud ... --filter="status=RUNNING"
gcloud ... --filter="name~prod"             # regex match on name
gcloud ... --filter="labels.env=production"

# Limit and sort
gcloud ... --limit=10                 # Max results to return
gcloud ... --sort-by=name             # Sort by field (prefix ~ for descending)
gcloud ... --sort-by=~createTime      # Newest first

# Quiet mode (no interactive prompts — good for scripts/CI)
gcloud ... --quiet

# Override project for one command
gcloud ... --project=OTHER_PROJECT_ID

# Verbosity
gcloud ... --verbosity=debug          # debug | info | warning | error | none
```

## Getting Help

```bash
gcloud help                          # Top-level help
gcloud <group> --help                # Help for a command group
gcloud <group> <command> --help      # Help for a specific command
gcloud cheat-sheet                   # Built-in quick reference
```

Full reference: https://cloud.google.com/sdk/gcloud/reference
