# SAG New MacBook Setup — Cascade Agent Instructions

> This file is written for the Windsurf Cascade AI agent to complete the SAG development environment setup on a new macbook. Execute each step in order.

---

## Current State (Updated May 16, 2026)

- ✅ Homebrew installed (`/opt/homebrew`)
- ✅ AWS CLI installed (aws-cli/2.34.48, arm64)
- ✅ AWS SSO profile `jemba9-dev` configured in `~/.aws/config`
  - SSO start URL: `https://onej9.awsapps.com/start`
  - SSO region: `us-west-2`
  - Account ID: `824999955649`
  - Role: `AdministratorAccess`
  - Default region: `us-west-2`
- ✅ AWS SSO login verified — account `824999955649`, role `AdministratorAccess`
- ✅ `~/.zshrc` updated with `brew shellenv` and `AWS_PROFILE=jemba9-dev`
- ✅ CodeCommit credential helper configured (global git config)
- ✅ Repo cloned to `/Users/mikeberry/Documents/Jemba9/CasecadeProjects/j9ccgit`
  - **Note:** Repo is in `us-east-2` (Ohio), not `us-west-2`
- ✅ Node.js v20.20.2 / npm 10.8.2 installed
- ✅ `npm install` complete (1026 packages)
- ✅ `backend/.env.local` populated (all values resolved from AWS)
- ✅ `apps/employee-console/.env.local` populated
- ✅ `apps/customer-portal/.env.local` populated
- ⏳ `DLT_PIPELINE_ID` in `backend/.env.local` — awaiting Databricks setup instructions
- ⏳ `VECTOR_SEARCH_ENDPOINT` in `backend/.env.local` — optional, awaiting Databricks setup
- ⏸️ Steps 9–10 (start dev servers, verify health) — paused pending Databricks config

---

## Step 1 — AWS SSO Login

Run the following and approve access in the browser that opens:

```bash
aws sso login --profile jemba9-dev
```

Verify it worked:

```bash
AWS_PROFILE=jemba9-dev aws sts get-caller-identity
```

Expected output: account `824999955649`, role `AdministratorAccess`.

---

## Step 2 — Shell Environment Persistance

Add these lines to `~/.zshrc` if not already present (Cascade: check first, only append what's missing):

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
export AWS_PROFILE=jemba9-dev
```

Check current `~/.zshrc`:
```bash
grep -E 'brew shellenv|AWS_PROFILE' ~/.zshrc
```

If either line is missing, append it:
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
echo 'export AWS_PROFILE=jemba9-dev' >> ~/.zshrc
```

---

## Step 3 — Clone the Main Monorepo

The SAG codebase lives at CodeCommit. Clone it if not already present at `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit`:

```bash
# First authenticate CodeCommit via the SSO credential helper
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true

# Clone
git clone https://git-codecommit.us-west-2.amazonaws.com/v1/repos/j9ccgit \
  /Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit
```

If the directory already exists, just verify the remote:
```bash
git -C /Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit remote -v
```

---

## Step 4 — Node.js & npm

Verify Node is installed (need v20+):
```bash
node --version
npm --version
```

If missing, install via Homebrew:
```bash
brew install node@20
echo 'export PATH="/opt/homebrew/opt/node@20/bin:$PATH"' >> ~/.zshrc
```

---

## Step 5 — Install Monorepo Dependencies

```bash
npm install
```

Run from the repo root: `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit`

---

## Step 6 — Backend Environment File

Copy the example env file and populate secrets from AWS Secrets Manager:

```bash
cp backend/.env.example backend/.env.local
```

Fetch the two secrets from AWS Secrets Manager (SSO must be active):

```bash
# Databricks SP client secret
aws secretsmanager get-secret-value \
  --secret-id sag/dev/client-secret \
  --query SecretString --output text \
  --profile jemba9-dev

# Lakebase Postgres connection string
aws secretsmanager get-secret-value \
  --secret-id sag/dev/database-url \
  --query SecretString --output text \
  --profile jemba9-dev
```

Then set these values in `backend/.env.local`:

| Variable | Source |
|---|---|
| `DATABRICKS_CLIENT_SECRET` | `sag/dev/client-secret` value |
| `DATABASE_URL` | `sag/dev/database-url` value |
| `DATABRICKS_HOST` | `dbc-10400a1c-b271.cloud.databricks.com` |
| `DATABRICKS_CLIENT_ID` | See `backend/.env.example` comment |
| `DATABRICKS_SERVER_HOSTNAME` | `dbc-30285e24-f917.cloud.databricks.com` |
| `COGNITO_USER_POOL_ID` | `us-west-2_G3cWUrDk6` |
| `COGNITO_CLIENT_ID` | `46k0lr6vneoqfkd8opij4bdcsk` |
| `COGNITO_REGION` | `us-west-2` |

---

## Step 7 — Employee Console Environment File

```bash
cp apps/employee-console/.env.example apps/employee-console/.env.local
```

Set in `apps/employee-console/.env.local`:
```
VITE_COGNITO_USER_POOL_ID=us-west-2_G3cWUrDk6
VITE_COGNITO_CLIENT_ID=46k0lr6vneoqfkd8opij4bdcsk
VITE_COGNITO_REGION=us-west-2
```

**Do NOT set `VITE_API_BASE_URL`** — leave it unset so the Vite proxy routes `/api/*` to `localhost:3000` in dev.

---

## Step 8 — Customer Portal Environment File

```bash
cp apps/customer-portal/.env.example apps/customer-portal/.env.local
```

Same Cognito values as Step 7.

---

## Step 9 — Start Dev Servers

In separate terminals (or use the smoke script):

```bash
# Terminal 1 — Backend
cd /Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit
npm run dev --workspace=backend

# Terminal 2 — Employee Console
npm run dev --workspace=apps/employee-console

# Terminal 3 — Customer Portal (optional)
npm run dev --workspace=apps/customer-portal
```

Or use the convenience script:
```bash
bash scripts/smoke/dev.sh
```

---

## Step 10 — Verify Health

```bash
curl http://localhost:3000/health
# Expected: {"status":"ok"}

curl http://localhost:3000/api/db-status
# Expected: {"connected":true}
```

---

## Key URLs & Account Info

| Resource | Value |
|---|---|
| AWS Account ID | `824999955649` |
| AWS Region | `us-west-2` |
| AWS SSO Profile | `jemba9-dev` |
| Employee Console (prod) | `https://d1gxvxrdrv7g44.cloudfront.net` |
| Customer Portal (prod) | `https://d3hzxtenpd9cq2.cloudfront.net` |
| Control Center (prod) | `https://dkkr6lmn8bwz9.cloudfront.net` |
| Stakeholder Portal (prod) | `https://dxzx31lf1jjiv.cloudfront.net` |
| Backend ALB (prod) | `http://sag-alb-dev-1203440618.us-west-2.elb.amazonaws.com` |
| Databricks sag-dev | `dbc-30285e24-f917.cloud.databricks.com` |
| Databricks sag-backend | `dbc-10400a1c-b271.cloud.databricks.com` |
| Cognito User Pool | `us-west-2_G3cWUrDk6` |
| Cognito Client | `46k0lr6vneoqfkd8opij4bdcsk` |
| ECR | `824999955649.dkr.ecr.us-west-2.amazonaws.com/sag-backend-dev` |

---

## Recurring Tasks

### Re-login after token expiry (~8 hours)
```bash
aws sso login --profile jemba9-dev
```

### CDK deploy (infra changes)
```bash
export AWS_PROFILE=jemba9-dev
npx cdk deploy --all --profile jemba9-dev
```
Run from `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit/infra`

### Docker push (backend changes)
```bash
aws ecr get-login-password --region us-west-2 --profile jemba9-dev | \
  docker login --username AWS --password-stdin \
  824999955649.dkr.ecr.us-west-2.amazonaws.com

docker build --platform linux/amd64 -t sag-backend-dev .
docker tag sag-backend-dev:latest \
  824999955649.dkr.ecr.us-west-2.amazonaws.com/sag-backend-dev:latest
docker push 824999955649.dkr.ecr.us-west-2.amazonaws.com/sag-backend-dev:latest
```
Run from `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit/backend`

---

## Notes for Cascade

- The `aws-sso-setup-guide.md` in this directory has an **outdated SSO start URL** (`https://jemba9.awsapps.com/start`). The correct URL confirmed in `~/.aws/config` is **`https://onej9.awsapps.com/start`**.
- SSO region is `us-west-2` (not `us-east-1` as the old guide states).
- **CodeCommit repo `j9ccgit` is in `us-east-2`** (Ohio), not `us-west-2`. Clone URL: `https://git-codecommit.us-east-2.amazonaws.com/v1/repos/j9ccgit`
- `DATABRICKS_CLIENT_ID` resolved from ECS task definition: `e637cca1-47d2-4ce2-9936-7c7175cda5a3`
- `S3_DOCUMENTS_BUCKET` resolved from ECS task definition: `sag-documents-dev`
- `DLT_PIPELINE_ID` and `VECTOR_SEARCH_ENDPOINT` must come from Databricks workspace — not stored in AWS.
- `--platform linux/amd64` is **required** on Apple Silicon (M-series) when building Docker images for Fargate.
- Lakebase Postgres scales to zero after 5 min idle — the first connection after idle takes ~30s.
