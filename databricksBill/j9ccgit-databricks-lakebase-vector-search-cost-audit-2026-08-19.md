# J9CCGit Databricks Cost Audit — Lakebase and Vector Search

**Date:** 2026-08-19  
**Scope:** `j9ccgit` real application code and live-cost screenshot supplied by Mike Berry  
**Focus:** Databricks Lakebase and Databricks Vector Search  
**Safety boundary:** This report contains resource identifiers and hostnames only. It does **not** contain database URLs, passwords, OAuth client secrets, access tokens, or secret values.

---

## Executive Summary

The supplied Databricks Account Usage Dashboard shows two sustained cost drivers over the last full four weekly periods:

| Product | Approx. weekly cost shown | Approx. 4-week cost | Interpretation |
|---|---:|---:|---|
| Lakebase | $243–$294/week | roughly $1.0K–$1.2K | Largest identified driver; cost remains high and flat week-to-week |
| Vector Search | about $94/week | roughly $376 | Second driver; also flat week-to-week |

The dashboard shape matters more than the exact dollar labels: both products show **steady weekly spend**, not isolated spikes. That pattern is consistent with always-on/provisioned capacity or continuously active endpoints. It is **not sufficient by itself** to attribute cost to a particular table, query, index, or user action.

The `j9ccgit` codebase does have credible cost-producing paths:

1. The backend uses Lakebase Postgres as its primary system of record through a persistent Node `pg.Pool`.
2. Each API health check performs `SELECT 1` against Lakebase and a live Vector Search index metadata call.
3. RAG retrieval invokes a Vector Search query endpoint, requesting up to 10 results. The only cache is in-memory, per ECS process, and expires after five minutes.
4. Evidence building calls Vector Search once for each generated evidence record; a run can generate up to three evidence types per finding and does so with up to five concurrent tasks.
5. Evaluation runs submit an ephemeral Databricks job using an `i3.xlarge` driver, can run for up to 30 minutes per task / 60 minutes overall, call model endpoints, run MLflow evaluation with three judge metrics, and write results back to Lakebase.

The audit **cannot yet prove** which of these is responsible for the dashboard dollars. Databricks Genie correctly needs direct Lakebase statistics; the immediate next step is to query `pg_stat_statements` and the Lakebase counters from the correct projects, then correlate the results with Databricks usage by week and with ECS/application request volume.

---
## 1. Genie-Ready Lakebase Connection Context

Give Genie the following **non-secret** context. Do not provide it a connection string, password, access token, or client secret.

### SAG dev — primary candidate for day-to-day application cost

| Field Genie requested | Value / status | Evidence |
|---|---|---|
| Lakebase host | `ep-green-lake-d12qkqzq.database.us-west-2.cloud.databricks.com` | `documents/plans/active/25-staging-cdk-stack-and-deploy-pipeline-execution.md` §9.2, line 906 |
| Lakebase project | `sag-dev` | Plan 25 identifies staging as a separate project alongside existing `sag-dev` |
| Branch | **Unknown — do not send `production` to Genie as the API branch identifier** | Repository docs use `production` as the human-facing branch label, but Genie confirmed Lakebase requires the internal `br-...` identifier. The real identifier is not present in the inspected repo/configuration. |
| Database | `databricks_postgres` | Plan 25 §9.2 line 906 |
| Application schema | `public` | `infra/lib/stage-config.ts`, dev `dbSchema` line 39 |
| Workspace host (not the Lakebase host) | `dbc-10400a1c-b271.cloud.databricks.com` | `infra/lib/api-stack.ts` line 142 |

### How to obtain the exact SAG dev branch identifier

1. In the Databricks workspace `dbc-10400a1c-b271.cloud.databricks.com`, open **Compute → Lakebase → `sag-dev`**.
2. Select the branch that the application uses. The visible label may say `production`, but do **not** assume that label is the identifier Genie needs.
3. Copy the **Resource name** shown in Branch overview, expected in this form: `projects/sag-dev/branches/br-<adjective>-<noun>-<id>`; alternatively copy the `br-...` segment from the browser URL.
4. Give Genie either the full resource name or the `br-...` portion, plus the host/database/schema from the table above.

The OneJemba screenshot supplied during this investigation proves the convention: its display label is `production`, but the resource/URL carries a distinct internal `br-royal-block-...` branch identifier. SAG must be handled the same way.

**Paste to Genie after filling the bracketed value:**

> Please inspect Lakebase project `sag-dev`, internal branch identifier `[br-... copied from Lakebase Branch overview]`, database `databricks_postgres`, schema `public`, host `ep-green-lake-d12qkqzq.database.us-west-2.cloud.databricks.com`. I need `pg_stat_statements` and database/table counters for the last 30 days, grouped by query fingerprint/user/application where available. Identify total calls, total execution time, mean latency, rows returned/changed, temp-file usage, cache hit indicators, connection count, and largest tables/indexes. Do not expose credentials.

### SAG staging — isolate separately from dev

| Field Genie requested | Value / status | Evidence |
|---|---|---|
| Lakebase host | `ep-wandering-tooth-d13zvqvq.database.us-west-2.cloud.databricks.com` | Plan 25 §10.4 and staging runbook |
| Lakebase project | `sag-staging` | Separate, dedicated project created during Plan 25 Phase 10 |
| Branch | **Unknown — retrieve internal `br-...` identifier from Lakebase Branch overview** | Not stated in the current real-app deployment record; a display label alone is insufficient for Genie/API queries. |
| Database | `databricks_postgres` | `documents/deploy-runbooks/active/04-staging-cdk-deploy-pipeline-runbook.md` |
| Application schema | `staging` | `infra/lib/stage-config.ts` line 63 and staging runbook |

**Paste to Genie after copying the staging `br-...` identifier:**

> Separately inspect Lakebase project `sag-staging`, internal branch identifier `[br-... copied from Lakebase Branch overview]`, database `databricks_postgres`, schema `staging`, host `ep-wandering-tooth-d13zvqvq.database.us-west-2.cloud.databricks.com`. Run the same top-query/table/connection analysis. Do not combine this with `sag-dev`; I need cost and activity by project.

### BoardLab — separate project, potentially relevant but not part of the deployed SAG backend

The real `j9ccgit` BoardLab redesign documents identify a separate Lakebase resource:

| Field | Value |
|---|---|
| Lakebase host | `ep-fancy-hill-d1ws6pzk.database.us-west-2.cloud.databricks.com` |
| Project | `board_lab` |
| Branch | `production` |
| Database | `boardlab` |
| Schemas | `boardlab`, `boardlab_graph` |

This is relevant if the dashboard aggregates all workspace usage. It should be inventoried separately, not assumed to be the SAG backend.

---
## 2. What the Screenshot Establishes — and What It Does Not

### Established by the supplied Account Usage Dashboard

- Lakebase and Vector Search are material cost categories in the selected Databricks workspace/dashboard view.
- Lakebase appears to be the larger recurring cost component in each full weekly bar.
- Both cost categories have a flat, repeated weekly pattern across the shown weeks, suggesting provisioned/always-on capacity or consistently recurring workload.
- The latest week is partial; it cannot be compared directly to full weeks without normalizing for elapsed days.

### Not established by the screenshot

- Which Lakebase project is billed (SAG dev, SAG staging, BoardLab, OneJemba, or another project).
- Which specific database/table/query generated the Lakebase use.
- Which Vector Search endpoint/index is billed.
- Whether the cost is endpoint baseline capacity, query volume, index sync, ingestion, embedding, evaluation jobs, or another workload.
- Whether the shown values are gross list price, net price, or allocated cost after tags/discounts.

**Do not treat the screenshot as root-cause evidence.** Treat it as the prioritization signal that justifies direct telemetry collection.

---
## 3. Confirmed J9CCGit Cost-Producing Paths

### 3.1 Lakebase Postgres is the backend system of record

`backend/src/db/connection.ts` creates a Node Postgres pool from `DATABASE_URL`:

```ts
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
```

The pool is used across the real Express backend for application reads/writes. There is no explicit `max`, `idleTimeoutMillis`, `connectionTimeoutMillis`, or `application_name` configuration in this file. Therefore, runtime defaults from the Node `pg` library apply unless overridden elsewhere.

**Cost implication:** this code confirms Lakebase is continuously in the request path, but it does not explain the flat baseline on its own. The more likely baseline driver is the Lakebase compute/branch configuration seen in the Databricks console; query pressure then adds workload on top of it.

### 3.2 Health checks touch both Lakebase and Vector Search

`backend/src/routes/health.ts` defines `GET /api/db-status` as a Lakebase + Vector Search connectivity check:

- Lakebase: `SELECT 1 AS ok`
- Vector Search: authenticated metadata request for the configured index

`infra/lib/platform-delivery-pipeline-stack.ts` and the staging plan establish that smoke checks call `/health` and `/api/db-status` after dev and staging deploys. This is legitimate operational traffic, but it means repeated smoke/monitoring activity is not free.

**Cost implication:** probably not the largest driver, but the endpoint should not be polled aggressively by dashboards, browser refresh loops, or external uptime tools because every request hits both products.

### 3.3 Vector Search is a live query dependency with a narrow cache

`backend/src/agents/tools/vector-search.ts`:

- Uses index `initialvectorindex` by default.
- Uses `VECTOR_SEARCH_ENDPOINT` when set; otherwise falls back to the Databricks workspace host.
- Posts to `/api/2.0/vector-search/indexes/{index}/query`.
- Requests up to `num_results: 10`.
- Uses an in-memory cache keyed on `runId::query` with a five-minute TTL.

The configured real backend environment sets:

| Variable | Current value in CDK source |
|---|---|
| `VECTOR_SEARCH_INDEX` | `initialvectorindex` |
| `DATABRICKS_HOST` | Databricks workspace host |
| `VECTOR_SEARCH_ENDPOINT` | Not set by the ECS task definition; fallback workspace-host route is used unless injected outside CDK |

**Cost implication:** the cache only helps when the **same ECS process** receives the same query/run combination within five minutes. It is not shared across ECS tasks, restarts, or users. Cache effectiveness is currently unmeasured.

### 3.4 Evidence generation can multiply Vector Search calls

`backend/src/agents/nodes/evidence-builder.ts` calls `searchDocuments()` once inside each evidence-generation operation. The node describes its target as “3 types each” and sets `MAX_CONCURRENT = 5`.

For each actionable finding:

1. It constructs a RAG query from component type, parameter, and standard.
2. It calls Vector Search.
3. It uses up to the first three returned chunks to enrich the LLM prompt.
4. It generates evidence records across standard-citation, datasheet, and calculation categories.

**Likely multiplier:** a run with many pass/fail findings can produce several Vector Search queries and model calls, potentially concurrently. The exact total must be measured from `agent_trace`, request logs, and Vector Search telemetry.

### 3.5 Evaluation runs can create significant Databricks compute and Lakebase load

`backend/src/routes/agent-evaluation.ts` submits evaluation jobs through `backend/src/lib/databricks-jobs.ts`.

The job submission code creates an ad hoc Databricks job with:

| Parameter | Current code behavior |
|---|---|
| Cluster | New cluster per submitted evaluation run |
| Driver | `i3.xlarge` |
| Workers | `0` (single-node job) |
| Task timeout | 1,800 seconds (30 min) |
| Overall timeout | 3,600 seconds (60 min) |
| Notebook | Workspace `sag/evaluation_runner` |
| Input/output store | Lakebase Postgres |
| Evaluation framework | MLflow + LLM judge metrics |

`databricks/notebooks/evaluation_runner.py` reads evaluation cases from Lakebase, invokes agents through the AI Gateway, writes evaluation results to Lakebase, logs MLflow data, and calls `mlflow.evaluate()` with three extra metrics:

- faithfulness
- relevance
- answer correctness

The evaluation UI/API also supports:

- standard runs;
- A/B tests (run each case twice);
- model comparisons (multiple models).

**Cost implication:** this is the strongest code-level candidate for bursty Databricks compute, model-serving, and Lakebase query/write cost. If evaluation runs are used often, A/B tests and model comparisons multiply work materially. The dashboard’s smooth weekly cost pattern could include scheduled/repeated evaluation usage, but the audit cannot prove frequency without Jobs history.

---
## 4. Vector Search Configuration to Give Genie

| Item | Known value | Confidence |
|---|---|---|
| Workspace host | `dbc-10400a1c-b271.cloud.databricks.com` | High — ECS CDK source |
| Vector Search endpoint name | `vectorsearchendpoint` | Medium/high — RAG integration plan and historical contract; verify in Vector Search UI |
| Index name | `initialvectorindex` | High — ECS CDK source and query helper default |
| Embedding model noted in RAG integration plan | `databricks-gte-large-en` | Medium — plan/history; verify current index configuration |
| Retrieval request size | 10 results | High — `vector-search.ts` |
| In-process cache TTL | 5 minutes | High — `vector-search.ts` |

**Paste to Genie:**

> Please identify the Vector Search endpoint and index used by the SAG application: endpoint likely `vectorsearchendpoint`, index `initialvectorindex`, in workspace `dbc-10400a1c-b271.cloud.databricks.com`. For the last 30 days, report endpoint/index lifecycle state, baseline capacity type, index sync history, documents/chunks indexed, query count by day, latency, failures, and any compute/storage cost attribution. I need to distinguish endpoint baseline cost from query traffic and index sync/ingestion cost.

If Genie cannot access that telemetry, use the Databricks Vector Search UI/API with an authorized human session to capture:

1. endpoint type and current state;
2. index type (Delta Sync vs direct-access);
3. source Delta table or direct data source;
4. embedding endpoint/model;
5. index sync schedule / last sync / failures;
6. index size and document/chunk count;
7. endpoint query and capacity metrics.

---
## 5. Immediate Lakebase SQL Requests for Genie or a Lakebase SQL Client

Run these only against the intended Lakebase project/branch/database after confirming the environment mapping. The queries are read-only except for optional extension enablement, which should not be attempted without an owner’s approval.

### 5.1 Confirm context and available telemetry

```sql
SELECT
  current_database() AS database_name,
  current_schema() AS schema_name,
  current_user AS database_user,
  version() AS postgres_version;

SELECT extname, extversion
FROM pg_extension
ORDER BY extname;
```

If `pg_stat_statements` is available, query it. If it is not available, ask the Lakebase project owner whether it can be enabled and exported without restarting/altering production behavior.

### 5.2 Top query fingerprints — use the correct schema if Lakebase exposes it

```sql
SELECT
  calls,
  round(total_exec_time::numeric, 2) AS total_exec_ms,
  round(mean_exec_time::numeric, 2) AS mean_exec_ms,
  rows,
  shared_blks_hit,
  shared_blks_read,
  temp_blks_written,
  left(query, 500) AS query_sample
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 50;
```

Then examine the highest-call queries separately:

```sql
SELECT
  calls,
  round(mean_exec_time::numeric, 2) AS mean_exec_ms,
  round(total_exec_time::numeric, 2) AS total_exec_ms,
  rows,
  left(query, 500) AS query_sample
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 50;
```

### 5.3 Largest tables and indexes

```sql
SELECT
  schemaname,
  relname AS table_name,
  pg_size_pretty(pg_total_relation_size(relid)) AS total_size,
  pg_size_pretty(pg_relation_size(relid)) AS table_size,
  pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid)) AS indexes_size,
  n_live_tup,
  n_dead_tup,
  last_autovacuum,
  last_autoanalyze
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 50;
```

```sql
SELECT
  schemaname,
  relname AS table_name,
  indexrelname AS index_name,
  idx_scan,
  pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC
LIMIT 50;
```

### 5.4 Connection / idle-session inventory

```sql
SELECT
  application_name,
  usename,
  state,
  count(*) AS sessions,
  min(backend_start) AS oldest_connection,
  min(state_change) AS oldest_state_change
FROM pg_stat_activity
WHERE datname = current_database()
GROUP BY application_name, usename, state
ORDER BY sessions DESC, application_name;
```

This is specifically important because the Node pool does not set an `application_name` in `backend/src/db/connection.ts`. If all sessions show a generic pg/Node label, add an application name by environment/service to make future cost attribution possible.

---
## 6. A Safe, Ordered Investigation Plan

### Step 1 — Separate the bill by resource before tuning code

In Databricks usage/cost reporting, filter and export daily usage for:

- Lakebase project: `sag-dev`
- Lakebase project: `sag-staging`
- Lakebase project: `board_lab` (if it still exists)
- Lakebase project: `onejemba-dev` (new and likely too recent to explain the earlier four-week pattern)
- Vector Search endpoint/index: `vectorsearchendpoint` / `initialvectorindex`
- Jobs runs matching `sag-eval-*`
- Model Serving / AI Gateway separately, so evaluation model cost is not accidentally attributed to Vector Search or Lakebase

**Expected outcome:** identify whether the dashboard’s Lakebase spend is one always-on project, several projects, or a large volume of query/write traffic.

### Step 2 — Establish baseline capacity / idle behavior

For each Lakebase project, record:

- project and branch;
- compute SKU / size;
- auto-stop or scale-to-zero behavior, if any;
- active hours by day;
- storage size and growth rate;
- connection/session count while apparently idle;
- backup/restore retention and whether it has a separate billable component.

The screenshot’s repeating weekly Lakebase pattern strongly suggests this step will produce an actionable finding even before query tuning.

### Step 3 — Attribute Lakebase workload

Run the Genie/Lakebase queries above. Group the output into:

1. high call-count, cheap queries (often health checks/polling);
2. high total execution-time queries (often genuine workload);
3. slow queries with missing indexes or large scans;
4. large tables/indexes with churn/vacuum pressure;
5. idle-in-transaction or excess pooled connections.

Map fingerprints back to `backend/src/routes/`, LangGraph node helpers, and evaluation endpoints before changing indexes or caching.

### Step 4 — Attribute Vector Search workload

Measure index sync and query traffic separately. In particular:

- Count RAG calls per completed agent run.
- Count evidence-builder findings × evidence types.
- Measure cache hit/miss ratio; today there is no metric for it.
- Confirm whether index sync is continuous and whether the source document/chunk table is generating unnecessary churn.
- Determine the endpoint’s baseline capacity and idle policy.

### Step 5 — Attribute evaluation/job workload

In Databricks Jobs history, filter runs created today/back through the billing period by prefix `sag-eval-` or notebook `sag/evaluation_runner`.

For each run collect:

- duration;
- cluster creation/startup time;
- number of evaluation test cases;
- run type (`standard`, `ab_test`, `model_compare`);
- model count;
- MLflow evaluation use;
- result status; and
- any retry/cancel pattern.

This identifies whether an evaluation workflow is creating expensive transient clusters or repeating judge calls at high frequency.

---
## 7. Cost-Reduction Candidates — Do Not Apply Blindly

| Candidate | Expected target | Evidence | Validate first |
|---|---|---|---|
| Right-size or schedule Lakebase compute | Lakebase baseline | Four nearly flat weekly cost bars; project size/idle policy not yet known | Project-specific active hours, compute SKU, scale-to-zero policy, production SLA |
| Stop duplicate/unused Lakebase projects | Lakebase baseline | At least SAG dev, SAG staging, BoardLab, and OneJemba resources are documented | Confirm which projects are live, which contain data, and backup/retention needs |
| Set `application_name` and pool limits/timeouts | Lakebase attribution / connection pressure | `new Pool({ connectionString })` has no visible pool controls | Current connection inventory and backend service task count |
| Reduce `/api/db-status` polling | Lakebase + Vector Search small recurring load | Each call does `SELECT 1` and index metadata API call | Actual call frequency from ALB/API logs and monitoring probes |
| Add shared RAG cache / dedupe | Vector Search query volume | Cache is per ECS process, five minutes, keyed `runId::query` | Cache hit/miss and semantic acceptability of reusing results |
| Reduce RAG result count or call sites | Vector Search + downstream model cost | Every query asks for 10 results; evidence builder uses max 3 | Retrieval quality metrics / human validation; do not degrade assurance evidence blindly |
| Batch or schedule evaluations | Jobs + model serving + Lakebase writes | Evaluation runs create `i3.xlarge` job clusters with three LLM judge metrics | Jobs history, evaluation volume, required turnaround time |
| Restrict A/B and model-comparison runs | Jobs + model serving | A/B doubles cases; model comparisons multiply model calls | Who triggers them, how often, and whether full-suite execution is required |
| Add metrics before optimizations | All | No code-level RAG cache-hit/query-count metric is visible | Instrument first, then change one lever at a time |

---
## 8. Recommended Instrumentation Backlog

Add the following before broad tuning. They make cost attribution repeatable rather than a one-time forensic exercise.

### Backend / Lakebase

- Set `application_name` per service and environment in the Postgres connection configuration, e.g. `sag-backend-dev` / `sag-backend-staging`.
- Emit pool metrics: total/open/idle/waiting connections, checkout latency, query count, error count.
- Tag recurring health probes separately from user/business queries.
- Record query-duration histograms at a route/service level without logging values or raw sensitive query parameters.

### Vector Search

- Emit `rag_query_total`, `rag_cache_hit_total`, `rag_cache_miss_total`, `rag_query_latency_ms`, `rag_result_count`, and endpoint/index dimensions.
- Record the calling agent/node (`standards_librarian`, `evidence_builder`, etc.) and run ID only where the ID is non-sensitive/internal.
- Add an alert for index sync failure or sudden query-volume change.

### Evaluation jobs

- Record a cost-attribution row before/after each submitted evaluation: run type, test case count, model count, Databricks job run ID, start/end time, and aggregate result.
- Make full-suite, A/B, and model comparison actions visibly distinct in the UI and require an explicit confirmation for the more expensive variants.
- Consider a concurrency limit for evaluation submissions until workload and budget are understood.

---
## 9. Direct Answers to the Original Questions

### Information to send Databricks Genie now

For the immediate SAG dev analysis, provide:

```text
Lakebase host: ep-green-lake-d12qkqzq.database.us-west-2.cloud.databricks.com
Lakebase project: sag-dev
Lakebase branch: [internal br-... identifier copied from Branch overview; do not use display label `production`]
Database: databricks_postgres
Application schema: public
Workspace host: dbc-10400a1c-b271.cloud.databricks.com
```

For staging, run an independent analysis with:

```text
Lakebase host: ep-wandering-tooth-d13zvqvq.database.us-west-2.cloud.databricks.com
Lakebase project: sag-staging
Lakebase branch: [internal br-... identifier copied from Branch overview]
Database: databricks_postgres
Application schema: staging
Workspace host: dbc-10400a1c-b271.cloud.databricks.com
```

### What is most likely driving the cost?

**Highest confidence:** provisioned Lakebase and Vector Search baseline capacity, because the dashboard shows regular, flat weekly spend.

**Highest-confidence code paths to investigate after baseline capacity:**

1. Agent RAG calls, especially evidence building across many findings.
2. Evaluation runs, especially A/B and model-comparison workflows using transient `i3.xlarge` job clusters and MLflow LLM judges.
3. Lakebase query/write activity from the normal application and evaluation result persistence.
4. Excess polling of `/api/db-status`, if external monitoring or UI code calls it frequently.

### What is not yet proven?

No telemetry in the inspected codebase proves the actual count of Vector Search calls, evaluation jobs, Lakebase query fingerprints, database connections, endpoint capacity, or index sync activity. The direct Lakebase and Vector Search telemetry collection steps above are required before changing capacity, indexes, caching, or scheduling.

---
## Evidence Reviewed

### Real J9CCGit code

- `backend/src/db/connection.ts` — Node Postgres pool and schema search path.
- `backend/src/routes/health.ts` — Lakebase and Vector Search connectivity endpoint.
- `backend/src/agents/tools/vector-search.ts` — 5-minute in-process cache, index query path, 10-result request.
- `backend/src/agents/nodes/evidence-builder.ts` — RAG call inside evidence generation, three evidence types, concurrent tasks.
- `backend/src/routes/agent-evaluation.ts` — evaluation run submission and status polling.
- `backend/src/lib/databricks-jobs.ts` — ephemeral job cluster, `i3.xlarge`, timeouts, notebook submission.
- `databricks/notebooks/evaluation_runner.py` — Lakebase reads/writes, MLflow, LLM judge metrics.
- `infra/lib/api-stack.ts` — deployed backend Databricks host and Vector Search index configuration.
- `infra/lib/stage-config.ts` — dev/staging schemas and secret references.
- `documents/plans/active/25-staging-cdk-stack-and-deploy-pipeline-execution.md` — documented dev/staging Lakebase projects, hosts, schemas, and deployment context.
- `documents/deploy-runbooks/active/04-staging-cdk-deploy-pipeline-runbook.md` — staging database details.

### User-provided evidence

- Databricks Account Usage Dashboard screenshot, week view ending 2026-08-19, showing Lakebase and Vector Search as prominent cost categories.

### Limitations

- This audit did not read credentials, secret values, database contents, Databricks Jobs history, Vector Search endpoint telemetry, Lakebase telemetry tables, or system billing tables.
- Databricks Genie reported that Lakebase observability tables are not exported to Delta/Unity Catalog in the workspace; this report therefore recommends direct Lakebase `pg_stat_statements` / counter queries.
- Resource hostnames are infrastructure identifiers, not credentials. Verify branch names in the Lakebase UI before running project-specific analysis.
