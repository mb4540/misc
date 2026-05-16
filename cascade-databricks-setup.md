# SAG Databricks Setup — Cascade Agent Instructions

> Written for the Windsurf Cascade AI agent to get a new MacBook connected to both Databricks workspaces used by the SAG project. Execute each step in order, verifying outputs before proceeding.

---

## Architecture Overview

SAG uses **two Databricks workspaces**:

| Workspace | Alias | Host | Purpose |
|---|---|---|---|
| sag-dev | Data | `dbc-30285e24-f917.cloud.databricks.com` | SQL warehouse, `sag_catalog.sag` tables, Unity Catalog |
| sag-backend | Deployment | `dbc-10400a1c-b271.cloud.databricks.com` | MLflow, Model Serving endpoints, Agents UI |

Auth is **OAuth M2M** (service principal). Databricks PATs are disabled at the org level.

Service principal: `sag-backend` (created in the Databricks Account Console by Chad — cfisher@jemba9.com).

---

## Prerequisites

Before starting, confirm these are in place (run `aws sso login --profile jemba9-dev` first):

```bash
# Python 3.11+ required
python3 --version

# AWS CLI (to pull secrets)
aws --version

# Homebrew (for python if needed)
brew --version
```

If Python 3.11 is missing:
```bash
brew install python@3.11
echo 'export PATH="/opt/homebrew/opt/python@3.11/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

## Step 1 — Get Databricks Credentials from AWS Secrets Manager

The SP credentials live in AWS Secrets Manager. Fetch them (SSO must be active):

```bash
# Client ID (non-sensitive, also in backend/.env.example)
aws secretsmanager get-secret-value \
  --secret-id sag/dev/databricks \
  --query SecretString --output text \
  --profile jemba9-dev
```

This returns a JSON blob. Extract `client_id` and `client_secret` from it.

Alternatively, the client secret is stored as an individual secret:
```bash
aws secretsmanager get-secret-value \
  --secret-id sag/dev/client-secret \
  --query SecretString --output text \
  --profile jemba9-dev
```

You will also need the **SQL warehouse HTTP path** from sag-dev. Ask Chad or check the Databricks sag-dev workspace UI at:
`https://dbc-30285e24-f917.cloud.databricks.com` → SQL Warehouses → the active warehouse → Connection Details → HTTP Path.

It looks like: `/sql/1.0/warehouses/<warehouse-id>`

---

## Step 2 — Create a Credentials Shell Script

Create `~/.sag-databricks-credentials` (never commit this file — it contains secrets):

```bash
cat > ~/.sag-databricks-credentials << 'EOF'
# SAG Databricks credentials — source this before running databricks scripts
# DO NOT commit this file

# sag-dev: SQL warehouse + Unity Catalog data source
export DATABRICKS_SERVER_HOSTNAME=dbc-30285e24-f917.cloud.databricks.com
export DATABRICKS_HTTP_PATH=/sql/1.0/warehouses/<WAREHOUSE_ID>   # fill in
export DATABRICKS_CATALOG=sag_catalog

# sag-backend: Model Serving + MLflow deployment workspace
export DATABRICKS_WORKSPACE_HOST=dbc-10400a1c-b271.cloud.databricks.com

# Service principal OAuth M2M credentials (same SP works on both workspaces)
export DATABRICKS_CLIENT_ID=<client_id_from_secrets_manager>     # fill in
export DATABRICKS_CLIENT_SECRET=<client_secret_from_secrets_manager>  # fill in
EOF

chmod 600 ~/.sag-databricks-credentials
```

Replace the `<...>` placeholders with values from Step 1.

Source it for your current shell session:
```bash
source ~/.sag-databricks-credentials
```

Add to `~/.zshrc` for automatic loading on every terminal:
```bash
echo 'source ~/.sag-databricks-credentials' >> ~/.zshrc
```

---

## Step 3 — Install Python Dependencies

From the repo root:

```bash
cd /Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit

# Create a virtual environment (recommended)
python3.11 -m venv .venv
source .venv/bin/activate

# Install SAG databricks requirements
pip install -r databricks/requirements.txt
```

Pinned packages:
```
databricks-sql-connector==3.1.2
pyarrow>=14.0.1,<15.0.0
databricks-sdk>=0.32.0
mlflow>=2.21.0
unitycatalog-ai>=0.1.0
openai>=1.0.0
pyyaml>=6.0
requests>=2.31.0
```

> **Note:** `databricks-sql-connector` is pinned to exactly `3.1.2` — do NOT upgrade; later versions break the UC function runtime.

---

## Step 4 — Run the Connectivity Verification Script

```bash
source ~/.sag-databricks-credentials
source .venv/bin/activate   # if using venv

python databricks/verify_connection.py
```

Expected output (all sections should show OK):

```
============================================================
SAG Phase 7D — Databricks Connection Verification
============================================================

--- SDK Versions ---
  databricks-sql-connector: 3.1.2
  databricks-sdk: <version>
  mlflow: <version>
  openai: <version>

--- Agent Framework Imports ---
  mlflow.pyfunc.ResponsesAgent: OK
  databricks.sdk.WorkspaceClient: OK
  unitycatalog.ai DatabricksFunctionClient: OK

--- SQL Connection (sag-dev) ---
  Host: dbc-30285e24-f917.cloud.databricks.com
  HTTP Path: /sql/1.0/warehouses/...
  OAuth token: OK
  Catalog: sag_catalog, Database: sag
  Tables in sag_catalog.sag: 25
    - tenant
    - user
    - membership
    - project
    - board
    ... and 20 more
  SQL connection: OK

--- Workspace API (sag-backend) ---
  Host: dbc-10400a1c-b271.cloud.databricks.com
  Authenticated as: sag-backend (<display name>)
  Serving endpoints: 8
    - sag-standards-librarian: READY
    - sag-derating-analyst: READY
    ...
  Workspace API connection: OK

============================================================
Verification complete.
============================================================
```

---

## Step 5 — Set Backend Environment Variables

Copy and populate the backend env file:

```bash
cp backend/.env.example backend/.env.local
```

Edit `backend/.env.local` and set these Databricks-specific values:

| Variable | Value |
|---|---|
| `DATABRICKS_HOST` | `dbc-10400a1c-b271.cloud.databricks.com` |
| `DATABRICKS_CLIENT_ID` | From Step 1 |
| `DATABRICKS_CLIENT_SECRET` | From Step 1 |
| `DATABRICKS_SERVER_HOSTNAME` | `dbc-30285e24-f917.cloud.databricks.com` |
| `DATABRICKS_WORKSPACE_HOST` | `dbc-10400a1c-b271.cloud.databricks.com` |
| `DATABRICKS_HTTP_PATH` | `/sql/1.0/warehouses/<warehouse-id>` |
| `DATABRICKS_AI_GATEWAY_URL` | `https://7474654462212433.ai-gateway.cloud.databricks.com` |
| `DATABASE_URL` | From `aws secretsmanager get-secret-value --secret-id sag/dev/database-url` |

The remaining variables (Cognito, CloudFront, S3) are documented in `cascade-sag-setup.md`.

---

## Step 6 — Verify Backend Connects to Databricks

Start the backend and hit the health endpoint:

```bash
cd /Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit
npm run dev --workspace=backend
```

In a second terminal:
```bash
curl http://localhost:3000/api/db-status
# Expected: {"connected":true}
```

If you see `{"connected":false}`, check:
1. `DATABASE_URL` is set and points to Lakebase Postgres (not Databricks SQL)
2. `DATABRICKS_HOST` and `DATABRICKS_CLIENT_*` are set in `backend/.env.local`
3. Lakebase may be cold-starting — wait 30 seconds and retry

---

## Troubleshooting

### "Invalid client credentials" / 401 on OAuth token request
- Confirm `DATABRICKS_CLIENT_ID` matches the SP created for `sag-backend` (not a personal account)
- PATs are disabled at the org level — **do not use a personal token**
- Ask Chad (cfisher@jemba9.com) to verify the SP `sag-backend` still has permissions on both workspaces

### "SQL connection: FAILED — unable to connect"
- Check `DATABRICKS_HTTP_PATH` is the correct warehouse path from sag-dev
- The SQL warehouse may be suspended — open sag-dev UI and start it manually

### "unitycatalog.ai DatabricksFunctionClient: IMPORT ERROR"
- Run: `pip install "unitycatalog-ai[databricks]>=0.1.0"`
- The `[databricks]` extra is required for UC function tool calls

### "Tables in sag_catalog.sag: 0" or "SHOW TABLES" fails
- The SP may have lost GRANT permissions on the catalog
- Ask Chad to re-run: `GRANT USE CATALOG ON CATALOG sag_catalog TO \`<sp-application-id>\``

### Lakebase Postgres connection timeout
- It scales to zero after 5 minutes idle — first connection takes ~30 seconds
- Retry the `curl` after 30 seconds

### `databricks-sql-connector` version conflict
- Must be exactly `3.1.2` — do not let pip upgrade it
- If wrong version installed: `pip install databricks-sql-connector==3.1.2 --force-reinstall`

---

## Quick Reference

| What | Command |
|---|---|
| Load credentials | `source ~/.sag-databricks-credentials` |
| Activate venv | `source .venv/bin/activate` |
| Verify Databricks | `python databricks/verify_connection.py` |
| Check DB status | `curl http://localhost:3000/api/db-status` |
| sag-dev UI | `https://dbc-30285e24-f917.cloud.databricks.com` |
| sag-backend UI | `https://dbc-10400a1c-b271.cloud.databricks.com` |

---

## Key Contacts

| Person | Role | Contact |
|---|---|---|
| Chad Fisher | Databricks admin, SP owner | cfisher@jemba9.com |
| Mike Berry | Dev, AWS + Databricks | mb4540@gmail.com |
