---
name: do-ops
description: DigitalOcean operations cheatsheet for AI agents. Covers auth, Spaces file ops, Droplets, App Platform, Databases, DNS, and common gotchas. Use when working with any DigitalOcean resource.
---

# DigitalOcean Ops

Quick reference for agents working with DigitalOcean infrastructure. `doctl` is pre-authenticated on this machine.

> **Destructive actions (delete, destroy, reset, create) require operator approval. Read-only ops (list, get, logs) are always safe.**

---

## Auth Setup

Two separate auth systems — don't mix them up:

| Tool | Auth | Used For |
|------|------|----------|
| `doctl` | OAuth token, pre-configured | Infrastructure (droplets, apps, DNS, DBs) |
| AWS CLI | Spaces key/secret from `.env` | Spaces file operations |

```bash
# Verify doctl auth
doctl account get

# Load Spaces credentials before any aws s3 command
source ~/.env
# Provides: DO_TOKEN, DO_SPACES_KEY, DO_SPACES_SECRET,
#           DO_SPACES_ENDPOINT, DO_SPACES_BUCKET, DO_SPACES_CDN
```

Direct API calls when `doctl` doesn't cover something:
```bash
source ~/.env
curl -X GET "https://api.digitalocean.com/v2/<resource>" \
  -H "Authorization: Bearer $DO_TOKEN" \
  -H "Content-Type: application/json"
```

---

## Spaces (Object Storage)

All Spaces config comes from `.env` — bucket name, endpoint, and CDN URL are all environment-specific.

### Bucket Structure

```
$DO_SPACES_BUCKET/
├── project-name/
│   ├── images/        ← static images, brand assets
│   ├── videos/        ← video files
│   ├── docs/          ← PDFs, documents
│   └── uploads/
│       └── <user-id>/ ← user-generated content, scoped per user
└── another-project/
    └── ...
```

Rules:
- Top-level folder = project name (match repo/project name)
- Never write to bucket root
- Never write outside your project's folder
- User content goes under `uploads/<user-id>/`

### File Operations

Always `source ~/.env` first. Pass credentials inline — do not configure `~/.aws/credentials`.

```bash
source ~/.env

# Upload (public)
AWS_ACCESS_KEY_ID=$DO_SPACES_KEY \
AWS_SECRET_ACCESS_KEY=$DO_SPACES_SECRET \
AWS_DEFAULT_REGION=us-east-1 \
aws s3 cp ./file.jpg s3://$DO_SPACES_BUCKET/project-name/images/file.jpg \
  --endpoint-url $DO_SPACES_ENDPOINT \
  --acl public-read

# Upload (private)
AWS_ACCESS_KEY_ID=$DO_SPACES_KEY \
AWS_SECRET_ACCESS_KEY=$DO_SPACES_SECRET \
AWS_DEFAULT_REGION=us-east-1 \
aws s3 cp ./file.pdf s3://$DO_SPACES_BUCKET/project-name/docs/file.pdf \
  --endpoint-url $DO_SPACES_ENDPOINT

# List folder
AWS_ACCESS_KEY_ID=$DO_SPACES_KEY \
AWS_SECRET_ACCESS_KEY=$DO_SPACES_SECRET \
AWS_DEFAULT_REGION=us-east-1 \
aws s3 ls s3://$DO_SPACES_BUCKET/project-name/ \
  --endpoint-url $DO_SPACES_ENDPOINT

# Sync local dir to Spaces
AWS_ACCESS_KEY_ID=$DO_SPACES_KEY \
AWS_SECRET_ACCESS_KEY=$DO_SPACES_SECRET \
AWS_DEFAULT_REGION=us-east-1 \
aws s3 sync ./dist s3://$DO_SPACES_BUCKET/project-name/assets/ \
  --endpoint-url $DO_SPACES_ENDPOINT \
  --acl public-read

# Delete a file (⚠️ requires approval)
AWS_ACCESS_KEY_ID=$DO_SPACES_KEY \
AWS_SECRET_ACCESS_KEY=$DO_SPACES_SECRET \
AWS_DEFAULT_REGION=us-east-1 \
aws s3 rm s3://$DO_SPACES_BUCKET/project-name/images/file.jpg \
  --endpoint-url $DO_SPACES_ENDPOINT
```

### CDN URL Format

```
$DO_SPACES_CDN/project-name/images/file.jpg
```

CDN URL = `$DO_SPACES_CDN` + `/` + path within bucket. Never use the raw Spaces endpoint URL in app code — always use CDN.

---

## Droplets

```bash
# List all
doctl compute droplet list

# Get details
doctl compute droplet get <id>

# Create (⚠️ requires approval)
doctl compute droplet create <name> \
  --region sgp1 \
  --size s-1vcpu-1gb \
  --image ubuntu-22-04-x64 \
  --tag-names project-name

# SSH into droplet
doctl compute ssh <id>

# Get droplet IP
doctl compute droplet get <id> --format PublicIPv4 --no-header

# Delete (⚠️ requires approval)
doctl compute droplet delete <id>
```

---

## App Platform

```bash
# List all apps
doctl apps list

# Get app details (includes ID, URL, status)
doctl apps get <id>

# Logs
doctl apps logs <id> --type run        # runtime logs
doctl apps logs <id> --type build      # build logs
doctl apps logs <id> --type deploy     # deploy logs
doctl apps logs <id> --no-follow | tail -50  # recent only

# Deploy from spec (⚠️ requires approval)
doctl apps create --spec app.yaml

# Trigger re-deploy (⚠️ requires approval)
doctl apps create-deployment <id>

# Delete (⚠️ requires approval)
doctl apps delete <id>
```

---

## Databases

```bash
# List clusters
doctl databases list

# Get cluster details
doctl databases get <id>

# Connection string
doctl databases connection <id>

# List DBs within a cluster
doctl databases db list <id>

# List users
doctl databases user list <id>

# Delete (⚠️ requires approval)
doctl databases delete <id>
```

---

## DNS / Domains

```bash
# List domains
doctl compute domain list

# List DNS records for a domain
doctl compute domain records list example.com

# Create A record (⚠️ requires approval)
doctl compute domain records create example.com \
  --record-type A \
  --record-name @ \
  --record-data <ip>

# Create CNAME (⚠️ requires approval)
doctl compute domain records create example.com \
  --record-type CNAME \
  --record-name www \
  --record-data @

# Delete record (⚠️ requires approval)
doctl compute domain records delete example.com <record-id>
```

---

## Container Registry

```bash
doctl registry get
doctl registry repository list
doctl registry repository list-tags <repo-name>
```

---

## Spaces Metadata (via doctl)

```bash
doctl spaces list
doctl spaces get <bucket-name>
```

---

## Gotchas

**AWS CLI region must be `us-east-1`** even if the bucket is in another region (e.g. SGP1). This is a Spaces quirk — always use `us-east-1`.

**Never configure `~/.aws/credentials`** — pass credentials inline. Multiple agents share this machine.

**`doctl` vs AWS CLI are separate auth systems.** `doctl` auth doesn't give Spaces access. Spaces needs `DO_SPACES_KEY`/`DO_SPACES_SECRET`, not `DO_TOKEN`.

**Always source `.env` before Spaces commands.** `$DO_SPACES_BUCKET`, `$DO_SPACES_ENDPOINT`, and `$DO_SPACES_CDN` won't exist otherwise — commands will silently fail or write to wrong paths.

**CDN URLs, not raw Spaces URLs.** The raw endpoint bypasses CDN. Always construct URLs from `$DO_SPACES_CDN`.

**`--acl public-read` is required for public files.** Files uploaded without it are private by default.

**Tag resources with project name** for cost tracking:
```bash
doctl compute droplet create <name> --tag-names my-project
```

**App IDs are UUIDs, not names.** Use `doctl apps list` to find the ID before running any app command.
