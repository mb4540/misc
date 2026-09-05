---
solutionId: "{{ORG_PREFIX}}-KG-0001"
title: "Repository Knowledge Graph — Generic Multi-Root Workspace Solution Design"
status: "TEMPLATE"
version: "1.0"
date: "{{YYYY-MM-DD}}"
author:
  name: "{{AUTHOR_NAME}}"
  attuid: "{{AUTHOR_ID}}"
reviewers: []
relatedPlanFile: "(none yet — generate via /scaffold-planfile once this design is approved)"
relatedSolutionDocs: []
tags: ["knowledge-graph", "multi-root-workspace", "developer-tooling", "template", "generic"]
---

# Repository Knowledge Graph — Multi-Root Workspace Solution Design

> **Goal**: Build a structured, queryable knowledge graph of an entire multi-root workspace so that AI agents and human developers can reason about cross-repo relationships, trace dependencies, and answer architectural questions without re-surveying the codebase every session. This document is a **reusable template** — proven on an 8-repo, ~60-node / ~120-edge production workspace — parameterized so it can be re-instantiated for any git-based multi-root workspace regardless of language or framework.

---

## 0. How to Use This Template

This is not a fill-in-the-blank form — it is a **working solution design with the workspace-specific facts replaced by placeholders**. To instantiate it for a new workspace:

1. **Copy this file** to `AIFCDocuments/SolutionDesigns/active/{NNN}-{workspace-slug}-knowledge-graph.md` (next available sequence number in that directory).
2. **Run the Workspace Discovery Survey** (§3) against the target workspace — either manually or via a subagent — and replace every `{{PLACEHOLDER}}` with the discovered facts.
3. **Fill in the frontmatter**: `solutionId`, `author`, `date`.
4. **Get the design reviewed/approved** — set `status: "PROPOSED"` → `"APPROVED"`.
5. **Generate the execution plan**: run `/scaffold-planfile` referencing this document. The workflow's Step 5 (Find Solution-Compare Document) will extract this doc's `## Recommendation`, `## Architecture Decisions`, `## Component Archetypes`, and `## Comparison Rationale` sections — keep those four headers intact and populated so the automated plan-file generation works without manual copy-paste.
6. Set `relatedPlanFile` once the plan file exists.

**Do not remove the four sections named in step 5** — they are the machine-readable contract this document exposes to plan-file tooling.

---

## 1. Applicability & Assumptions

This design applies to any workspace where:

- Multiple git repositories are checked out as **sibling directories under one workspace root** (a "multi-root" or "poly-repo" layout — not a single monorepo, though the same approach works inside monorepo subdirectories too).
- Repositories may be **polyglot** (Python, TypeScript/JavaScript, Java, Swift, Go, etc.) — the discovery approach uses language-agnostic signals (manifest files, config files, source-text patterns), not a single-language AST.
- There is **read access** to all repos from wherever the graph-building tooling runs (local clone, CI runner, or AI agent sandbox).
- The goal is a **living, version-controlled artifact** that both humans and AI agents can query — not a one-time diagram.

**Explicit non-assumption**: this design does **not** assume a graph database, a running service, or network access at query time. The graph is git-native, file-based, and offline-queryable by design (see §6 Comparison Rationale).

---

## 2. Problem Statement

In any workspace with more than 3-4 interdependent repositories, the following pain points recur:

- **"What calls what?"** requires opening N repos and grepping for hostnames, `axios`/`fetch`/`httpx` calls, or hardcoded URLs — every single time the question is asked.
- **"What breaks if I change this?"** (blast-radius analysis) requires manually tracing callers across repo boundaries, which is error-prone and rarely done thoroughly under deadline pressure.
- **"What's the tech stack here?"** for onboarding or architecture review requires reading `package.json`/`pyproject.toml`/`pom.xml` in every repo by hand.
- **AI coding agents re-survey the same codebase from scratch every session**, burning context and time on discovery that produced an identical answer last week.
- **Institutional knowledge about cross-repo contracts lives in Slack threads and tribal memory**, not in a queryable, version-controlled place.

---

## 3. Workspace Discovery & Survey Methodology

This section is the **generic recipe** for producing the workspace-specific facts that seed the graph. Run this before filling in the rest of the document.

### 3.1 Step 1 — Enumerate Repositories

List every git repository directory at the workspace root (or the configured set of roots):

```bash
find {{WORKSPACE_ROOT}} -maxdepth 1 -type d -exec test -d '{}/.git' \; -print
```

For each repo found, record: directory name, primary purpose (one line), and whether it is in scope for this graph (exclude tooling/archetype/wiki repos that contain no runtime services).

| # | Repo Directory | Purpose | Primary Stack | In Scope? |
|---|---|---|---|---|
| 1 | `{{REPO_1}}` | {{PURPOSE_1}} | {{STACK_1}} | Y/N |
| 2 | `{{REPO_2}}` | {{PURPOSE_2}} | {{STACK_2}} | Y/N |
| N | `{{REPO_N}}` | {{PURPOSE_N}} | {{STACK_N}} | Y/N |

### 3.2 Step 2 — Classify Each Repo by Tech-Stack Signal

Use manifest-file presence to classify each repo (language-agnostic, reusable across any workspace):

| Signal Found | Classification | Notes |
|---|---|---|
| `package.json` with React/Next.js/Vue/Angular deps | Frontend (SPA) | Look for a paired backend sibling to mark Full-Stack |
| `package.json` (Node) + no frontend framework | Backend (Node) | Express, Hono, Fastify, NestJS |
| `requirements.txt` / `pyproject.toml` with FastAPI/Flask/Django | Backend (Python) | |
| `requirements.txt` with mlflow/sklearn/torch/tensorflow | ML Pipeline | |
| `pom.xml` / `build.gradle` | Backend (Java/Kotlin) | Spring Boot, Micronaut, etc. |
| `*.xcodeproj` / `*.xcworkspace` / `Podfile` | Mobile (iOS/Swift) | No package.json/pyproject — classify by native tooling |
| `build.gradle` + `AndroidManifest.xml` | Mobile (Android) | |
| `*.tf` / `terraform/` | Infrastructure as Code | |
| `Dockerfile` + `helm/` or `k8s/` | Container/K8s deployment target | |
| `dags/` or Airflow config | Orchestration | |
| `*.sql` or `dbt_project.yml` | Data Engineering | |

Record the classification per repo — this becomes §8 Component Archetypes and feeds the archetype detection step of the downstream `/scaffold-planfile` workflow.

### 3.3 Step 3 — Identify Deployable Units (Service Nodes)

Within each in-scope repo, identify every independently deployable unit — not just the repo as a whole. A single repo commonly contains multiple services (a monorepo-of-services pattern). Look for:

- Separate top-level directories each with their own manifest file (`requirements.txt`, `package.json`) and `Dockerfile`
- Separate Helm chart entries or `docker-compose.yml` service blocks
- Distinct deployment namespaces in CI/CD config

For each deployable unit, capture: name, language, framework, entry point file, port (if found in Dockerfile/config), and a one-paragraph description.

### 3.4 Step 4 — Identify Data Stores

Search for datastore connection signals across all repos:

```bash
grep -rEl "mongodb://|mongodb\+srv://|MONGO_URI|redis://|REDIS_URL|postgres(ql)?://|jdbc:|milvus|COSMOS_|BLOB_|azure.storage|s3://" {{WORKSPACE_ROOT}} --include="*.py" --include="*.ts" --include="*.java" --include="*.yaml" --include="*.env*"
```

For each distinct datastore found, record: type (MongoDB / Redis / Postgres / Vector DB / Blob / etc.), host pattern, and which services connect to it (this becomes `READS_FROM` / `WRITES_TO` edges).

### 3.5 Step 5 — Identify Cross-Repo API Dependencies

This is the highest-value and most labor-intensive step. For each frontend or service, search for outbound HTTP/WebSocket calls that target another in-workspace service:

```bash
# JS/TS
grep -rn "axios\.\(get\|post\|put\|delete\)\|fetch(\|new WebSocket(" {{REPO}}/src --include="*.ts" --include="*.tsx"
# Python
grep -rn "httpx\.\|requests\.\(get\|post\)\|aiohttp\." {{REPO}}/src --include="*.py"
# Native mobile (Swift/Kotlin) — look for a central URL/environment service
grep -rn "baseURL\|BASE_URL\|Endpoints\b" {{REPO}} --include="*.swift" --include="*.kt"
```

For every discovered call, resolve: caller service, callee service, endpoint path(s), protocol (REST/WebSocket/gRPC), and any auth header pattern. This becomes the `DEPENDS_ON` / `CALLS` / `EXPOSES` edges.

**Tip**: hardcoded environment URLs (e.g. `https://{{ingress-host}}.{{env}}.example.com/{{path}}`) are the fastest way to disambiguate "is this really calling service X or does it just look similar" — resolve the ingress/Helm routing config (`values_*.yaml`) if the URL path doesn't map 1:1 to a repo name.

### 3.6 Step 6 — Identify Auth Flows

Search for auth library imports (`MSAL`, `passport`, `oauth2`, `next-auth`, `Auth0`, `firebase-auth`) and classify each service as: delegated user auth (frontend), JWT validation (backend), or machine-to-machine (client credentials / managed identity).

### 3.7 Step 7 — Identify Shared Infrastructure

Enumerate cloud resources referenced across ≥2 repos: identity provider, secrets manager, container registry, orchestration cluster, CI/CD system, artifact repository. These become `Infrastructure` and `DataStore` nodes shared across many `DEPLOYED_TO` / `CONFIGURED_BY` edges.

---

## 4. Knowledge Graph Data Model

This schema is **already generic** — it was designed to be workspace-agnostic and has been validated against a real 60-node / 120-edge production graph. Reuse it as-is; extend only if a workspace has entity types not covered below.

### 4.1 Node Types

| Node Type | Description | `idFormat` | Required Fields | Optional Fields |
|---|---|---|---|---|
| `Repository` | A git repository in the workspace | `<repo-short-name>` | `@type, @id, name, path, primaryLanguage, description` | `frameworks, deploymentEnvironments, codeOwners, tags, services` |
| `Service` | A deployable unit — microservice, frontend app, or sidecar | `<service-slug>` | `@type, @id, name, repository, description` | `language, framework, port, entryPoint, environments, tags` |
| `APIEndpoint` | An HTTP/WebSocket route exposed or consumed | `<method>-<path-slug>` | `@type, @id, method, path, service` | `purpose, requestSchema, responseSchema, auth` |
| `DataStore` | A database, cache, blob container, message queue, or vector DB | `<store-slug>` | `@type, @id, name, storeType, description` | `host, databases, environments, tags` |
| `Collection` | A specific named collection, table, or container within a DataStore | `<store-id>.<collection-name>` | `@type, @id, name, dataStore` | `description, keyFields, sampleDocumentShape` |
| `DataModel` | A data contract type/schema (Pydantic, TS interface, Java class) | `<service-id>.<ModelName>` | `@type, @id, name, service, language` | `description, fields, sourceFile` |
| `Component` | A significant UI component, page, or view | `<repo-slug>.<ComponentName>` | `@type, @id, name, repository` | `description, sourceFile, route, framework` |
| `Library` | A major framework or library dependency | `<package-name>` | `@type, @id, name, ecosystem` | `version, purpose, homepage` |
| `Infrastructure` | A deployment target, CI/CD system, artifact registry, or cloud resource | `<infra-slug>` | `@type, @id, name, infraType` | `provider, region, description, url` |
| `Environment` | A named deployment environment (dev, test, stage, prod) | `<env-name>` | `@type, @id, name` | `baseUrl, description, isProd` |
| `Config` | A configuration file, environment variable, or secret **reference** (never a secret value) | `<config-key>` | `@type, @id, name, configType` | `description, defaultValue, requiredBy, secret` |
| `Team` | A code-owning team or group | `<team-slug>` | `@type, @id, name` | `githubTeam, email, repos` |

**Security guardrail**: `Config` nodes and `CONFIGURED_BY` edges must capture secret **names/keys only** (e.g. `MONGO_URI`, `LITELLM_API_KEY`) — never the actual secret value. This graph is intended to live in version control.

### 4.2 Edge Types

| Edge Type | From → To | Description |
|---|---|---|
| `CONTAINS` | Repository → Service | Repository contains one or more deployable services |
| `DEPENDS_ON` | Service → Service | Runtime API dependency between services |
| `CALLS` | Service → APIEndpoint | Service calls (consumes) an API endpoint |
| `EXPOSES` | Service → APIEndpoint | Service exposes (provides) an API endpoint |
| `READS_FROM` | Service → DataStore | Service reads data from a data store |
| `WRITES_TO` | Service → DataStore | Service writes data to a data store |
| `USES_COLLECTION` | Service → Collection | Service accesses a specific named collection or table |
| `DEFINES` | Service → DataModel | Service defines a data model or schema |
| `REFERENCES` | DataModel → DataModel | One data model references another |
| `RENDERS` | Component → Component | A UI component renders (composes) another component |
| `ROUTES_TO` | Component → APIEndpoint | A UI component calls a backend API endpoint |
| `BUILT_WITH` | Service → Library | Service is built with a library or framework |
| `DEPLOYED_TO` | Service → Infrastructure | Service is deployed to an infrastructure resource |
| `DEPLOYED_AT` | Service → Environment | Service is accessible at a specific environment URL |
| `CONFIGURED_BY` | Service → Config | Service requires a configuration variable or file |
| `OWNED_BY` | Repository → Team | Repository is owned or reviewed by a team |
| `AUTHENTICATES_VIA` | Service → Service | Service authenticates requests through another service or IdP |
| `FEDERATED_FROM` | Component → Service | UI component is loaded via Module Federation from a remote |
| `PROXIES_TO` | Service → Service | Service acts as a reverse proxy or gateway to another service |

### 4.3 Node JSON Schema (example)

```jsonc
// {{KG_DIR}}/nodes/services/<id>.json
{
  "@type": "Service",
  "@id": "{{service-slug}}",
  "name": "{{Human Readable Name}}",
  "repository": "{{repo-@id}}",
  "language": "{{language + version}}",
  "framework": "{{framework}}",
  "port": {{port}},
  "entryPoint": "{{relative/path/to/main}}",
  "description": "{{one-to-three sentence description of responsibility}}",
  "tags": ["{{tag1}}", "{{tag2}}"],
  "environments": {
    "dev":   "{{dev url}}",
    "test":  "{{test url}}",
    "stage": "{{stage url}}",
    "prod":  "{{prod url}}"
  }
}
```

### 4.4 Edge JSON Schema (example)

```jsonc
// {{KG_DIR}}/edges/<relationship-type>.json
[
  {
    "from": "{{caller-service-@id}}",
    "to": "{{callee-service-@id}}",
    "type": "DEPENDS_ON",
    "via": "REST API",
    "protocol": "HTTPS + Bearer JWT",
    "endpoints": ["/path/one", "/path/two"],
    "notes": "{{why this dependency exists}}"
  }
]
```

---

## 5. Recommendation

**Recommended approach**: store the knowledge graph as **version-controlled JSON files** (nodes + edges), populated by a **repeatable, additive-only extraction script**, exposed via a **CLI query tool** and an **AI-agent skill**, and validated by a **referential-integrity checker**. This is the same pattern proven on the reference workspace (60 nodes, 120 edges, 0 validation violations, sub-second query times).

Concretely, for a new workspace:

1. Create `{{KG_DIR}}/` (suggested name: `KnowledgeGraph/`) at the workspace root, structured per §9.
2. Seed nodes and edges manually from the §3 survey results (fastest path to a useful graph — do not wait for perfect automation).
3. Build `build-graph.py` as an **additive-only** re-extraction script that enriches existing nodes with tech-stack signals but never deletes curated content (§10.1).
4. Build `validate-graph.py` to catch broken references and duplicate IDs before they are committed (§10.2).
5. Build `query-graph.py` as a natural-language-to-graph-traversal CLI so both humans and AI agents get instant answers (§10.3).
6. Optionally wire CI to keep the graph fresh on every push (§12) and expose it to an AI coding agent as a skill (§11).
7. Treat an **interactive visualizer** as a separate, follow-on solution design once the graph itself is stable and useful via CLI (§13) — do not build visualization before the data model is validated.

This recommendation is deliberately **incremental**: a useful graph exists after step 2 alone; steps 3-7 are compounding investments, not prerequisites.

---

## 6. Comparison Rationale

### 6.1 Storage Format: JSON Files vs. Alternatives

| Criterion | JSON Files (recommended) | Graph DB (Neo4j, etc.) | Wiki / Confluence page | Code comments only |
|---|---|---|---|---|
| Infrastructure required | None — files in the repo | Running DB server | Hosted wiki service | None |
| AI-agent readable | Direct `read` tool access | Requires query language + connector | Requires scraping/API | Requires re-scanning source every time |
| Version controlled | Git-native, diffable | Requires separate backup/export | Page history only, no diff | Git-native but unstructured |
| Human editable | Any text editor | Requires DB client or admin UI | WYSIWYG editor | Any text editor |
| Query capability | Python/`jq` scripts, or NetworkX | Rich query language (Cypher) | Full-text search only | grep only |
| Staleness risk | Low — CI can auto-refresh | Low — CI can auto-refresh | High — manual updates, easily forgotten | Highest — scattered, no single source of truth |
| Scales to | ~1,000 nodes comfortably | Effectively unlimited | N/A (not structured data) | N/A |

**Verdict**: for workspaces under ~1,000 nodes (the overwhelming majority of engineering workspaces), JSON files win on every axis except raw query power, which is not needed at this scale. Migrate to an embedded graph engine (e.g. SQLite + JSON1, or a lightweight graph DB) only if node count grows an order of magnitude beyond what a single `read` call can hold in an AI agent's context window.

### 6.2 Extraction Strategy: Additive-Only Script vs. Full Regeneration

| Approach | Additive-Only (recommended) | Full Regeneration |
|---|---|---|
| Preserves manual curation | Yes — merges into existing files, never overwrites scalar values that already exist | No — risks discarding hand-written descriptions, notes, and edges the scanner cannot infer |
| Safe to run repeatedly / in CI | Yes | Risky without a diff-review step |
| Initial seed effort | Higher (manual survey required first) | Lower (fully automated, but lower quality) |

**Verdict**: additive-only. Automated extraction is excellent at keeping tech-stack fields (frameworks, ports, languages) fresh, but it cannot infer *why* two services are coupled, what a data store is used for, or which auth flow applies — that context has to be captured once, by a human or an AI agent doing a deliberate survey, and preserved thereafter.

### 6.3 Query Interface: CLI Script vs. Chat-Embedded LLM vs. No Interface

| Approach | Pros | Cons |
|---|---|---|
| Python CLI (`query-graph.py`) (recommended, Phase 1) | Zero dependency beyond Python stdlib + optional `networkx`; instant; scriptable in CI | Requires learning the query vocabulary |
| Devin/AI-agent skill wrapping the CLI (recommended, Phase 2) | Natural language, zero learning curve, usable inside any agent session | Requires the agent runtime to support skills |
| Embedded LLM chat UI in a visualizer (optional, Phase 3+) | Best UX for non-technical stakeholders | Highest build cost; depends on the visualizer existing first |
| No interface — read JSON directly | Zero build cost | Poor UX; still useful as an AI agent fallback |

**Verdict**: build the CLI first (cheap, immediately useful), then wrap it as an agent skill (near-zero incremental cost, highest leverage), and treat a chat-embedded visualizer as an optional, separately-scoped enhancement (§13).

---

## 7. Architecture Decisions

- **Storage format**: JSON files under `{{KG_DIR}}/`, not a graph database — see §6.1.
- **Node identity**: `@id` is a short, permanent slug unique within its type directory (e.g. `"payments-api"`). Once assigned, an `@id` never changes even if the underlying service is renamed — cross-references in edge files use the slug, and renaming it would silently break every edge that points to it.
- **Edge identity**: edges are plain arrays in type-named JSON files (`edges/api-dependencies.json`, `edges/data-access.json`, …); no separate edge IDs are needed at this scale.
- **Extraction philosophy**: `build-graph.py` is **additive + update only** — it merges newly-discovered scalar/list/dict values into existing node files but never deletes manually-curated content (§6.2, §10.1).
- **Validation gate**: `validate-graph.py` is a **required pre-commit / CI check** — it fails the build if any edge references an unknown `@id`, if any `@id` is duplicated within a type directory, or if an edge uses an undeclared type (§10.2).
- **Query engine**: an in-memory graph built from the JSON files at query time (NetworkX `DiGraph` recommended for Python tooling; a simple adjacency-list dict is sufficient if avoiding the `networkx` dependency). Natural-language queries are pattern-matched to graph-traversal functions — no LLM call required for the CLI tier.
- **CI trigger** (if implemented): a path-filtered workflow that runs on push to any in-scope repo, re-runs `build-graph.py`, runs `validate-graph.py` as a gate, and commits the diff back to the graph directory using a bot identity.
- **Secrets discipline**: `Config` nodes and `CONFIGURED_BY` edges record configuration **key names** only, never resolved secret values — the graph is designed to live safely in a version-controlled repository.

---

## 8. Component Archetypes

Map each in-scope repo (from §3.2) to a development archetype. This table feeds the archetype-detection step of the downstream `/scaffold-planfile` workflow and determines which AI Development Prompts apply per phase.

| Repo | Tech Signal | Archetype | Notes |
|---|---|---|---|
| `{{REPO_1}}` | {{signal}} | `{{archetype-slug}}` | {{notes}} |
| `{{REPO_2}}` | {{signal}} | `{{archetype-slug}}` | {{notes}} |
| `{{REPO_N}}` | {{signal}} | `{{archetype-slug}}` | {{notes}} |
| *(the graph tooling itself)* | Python stdlib + optional `networkx`/`pyyaml` | `backend-only` or `documentation-evangelist` | No frontend; pure scripts + JSON + Markdown |

Universal archetypes that always apply regardless of workspace, per standard plan-file convention:

- `code-reviewer` → `/compare-code-reviewer`
- `documentation-evangelist` → `/document-documentation-evangelist`
- `regression-test-coverage` → `/test-regression-test-coverage`
- `unit-test-code-coverage` → `/test-unit-test-code-coverage`

---

## 9. Proposed Directory Layout

```
{{KG_DIR}}/
├── README.md                    ← usage guide (query examples, update instructions)
├── schema/
│   ├── node-types.json          ← node type definitions (§4.1, reuse verbatim)
│   └── edge-types.json          ← edge type definitions (§4.2, reuse verbatim)
├── nodes/
│   ├── repositories/            ← one file per repo
│   ├── services/                ← one file per deployable unit
│   ├── datastores/               ← databases, caches, blob stores, vector DBs
│   ├── components/              ← significant UI components (optional — later phase)
│   ├── models/                  ← key data contracts/schemas (optional — later phase)
│   ├── infrastructure/          ← AKS/EKS/GKE, registries, Key Vault, etc. (optional — later phase)
│   └── config/                  ← env var / secret-key references (optional — later phase)
├── edges/
│   ├── api-dependencies.json    ← service-to-service API calls
│   ├── data-access.json         ← service-to-datastore reads/writes
│   ├── ui-composition.json      ← component composition / module federation
│   ├── deployment.json          ← service-to-infrastructure, CONTAINS, CONFIGURED_BY
│   └── auth.json                ← authentication flows
├── views/                       ← pre-computed, cached query results (regenerated by build-graph.py)
│   ├── dependency-matrix.json   ← N×N matrix of service dependency weights
│   ├── tech-stack-summary.json  ← aggregated technology inventory
│   ├── api-catalog.json         ← all known endpoints across all services
│   └── env-map.json             ← service → environment URL mapping
└── scripts/
    ├── build-graph.py           ← additive-only re-extraction from source (§10.1)
    ├── validate-graph.py        ← referential integrity checker (§10.2)
    ├── query-graph.py           ← CLI query interface (§10.3)
    └── requirements.txt         ← e.g. `networkx>=3.3`, `pyyaml>=6.0,<7.0`
```

**Note**: the four "optional — later phase" node directories (`components/`, `models/`, `infrastructure/`, `config/`) should exist as empty directories from day one (so the schema is complete and discoverable) but do not need to be populated in the first pass — seed `repositories/`, `services/`, and `datastores/` first, since those three answer the highest-value questions ("what exists, what talks to what, what stores data").

---

## 10. Extraction & Automation Scripts

### 10.1 `build-graph.py` — Additive Extraction

**Purpose**: re-scan the workspace and merge freshly-discovered tech-stack signals into existing node files, without ever discarding manually-curated content.

**Key design constraints** (validated on the reference implementation):

- Maintain an explicit **repo-prefix → `@id` map** (a Python dict) rather than trying to auto-infer IDs from directory names — directory naming conventions vary too much across workspaces to infer reliably, and a wrong auto-inferred ID silently breaks edges.
- **Deep-merge, never overwrite**: for each field, if it doesn't exist on the target node, add it; if it's a list, append new unique items; if it's a dict, recurse; if it's a scalar that already has a value, **keep the existing value** (assume prior manual curation was deliberate).
- Extract tech-stack signals from manifest files only (`package.json` dependencies, `pyproject.toml`/`requirements.txt`, `pom.xml`/`build.gradle`) — do not attempt full AST parsing; simple regex/text matching is sufficient and far more portable across languages.
- **Route/endpoint detection is report-only, not auto-committed** in the first version — print discovered route counts for human/agent review, but do not auto-generate `EXPOSES` edges, since noisy auto-detected endpoint lists tend to duplicate or conflict with curated `DEPENDS_ON` edges that already capture the meaningful subset.
- Regenerate all `views/*.json` files unconditionally on every run — these are fully derived data and safe to overwrite.
- Support `--dry-run` to preview changes before writing, and `--repos-root` / `--graph-root` overrides so the script is portable to a different workspace layout without editing the script body.

**CLI usage**:
```bash
python3 {{KG_DIR}}/scripts/build-graph.py --dry-run
python3 {{KG_DIR}}/scripts/build-graph.py
python3 {{KG_DIR}}/scripts/build-graph.py --repos-root /path/to/other/workspace
```

### 10.2 `validate-graph.py` — Referential Integrity

**Checks to implement**:

1. Every edge's `from` and `to` value must resolve to a known node `@id`.
2. No duplicate `@id` within a single node-type directory.
3. Every edge's `type` must be declared in `schema/edge-types.json`.
4. *(Optional, stricter)* every edge's `from`/`to` node `@type` matches the `from`/`to` constraint declared for that edge type in the schema.

**CLI usage**:
```bash
python3 {{KG_DIR}}/scripts/validate-graph.py
# ✅ Graph valid — {{N}} nodes, {{M}} edges, 0 violations
```

Wire this as a CI gate (§12) and/or a pre-commit hook so a broken graph never reaches `main`.

### 10.3 `query-graph.py` — Natural-Language CLI

Load the full graph into memory (NetworkX `DiGraph` or a plain adjacency dict) and pattern-match common question shapes to graph-traversal calls. Minimum recommended query patterns:

| Pattern | Example | Traversal |
|---|---|---|
| `what depends on <id>` | `what depends on payments-api` | in-edges of type `DEPENDS_ON` |
| `what does <id> depend on` | `what does checkout-ui depend on` | out-edges of type `DEPENDS_ON` |
| `what reads from <id>` / `what writes to <id>` | `what writes to orders-db` | in-edges of type `READS_FROM`/`WRITES_TO` |
| `tech stack for <id>` | `tech stack for checkout-ui` | node fields: `language`, `framework`, `frameworks` |
| `auth for <id>` | `auth for payments-api` | out-edges of type `AUTHENTICATES_VIA` |
| `list all <type>` | `list all services` | all nodes of a given `@type` |
| `list services in <repo>` | `list services in checkout-monorepo` | `CONTAINS` edges from that repo |
| `path between <id> and <id>` | `path between checkout-ui and orders-db` | shortest path (Dijkstra/BFS) |
| bare `<id>` lookup | `payments-api` | full node JSON |

Output Markdown, not raw JSON — the CLI's primary consumer is an AI agent that will paraphrase the answer for a human, and Markdown is both human- and agent-readable without extra parsing.

---

## 11. AI Agent Integration (Recommended, Phase 2)

Once `query-graph.py` exists, wrap it as an agent-invocable skill (e.g. a Devin skill at `.devin/skills/{{skill-name}}/SKILL.md`, or the equivalent construct for whichever agent runtime the team uses). The skill should:

- Accept natural-language questions and forward them to `query-graph.py`.
- Optionally offer a "refresh the graph" action that runs `build-graph.py` before answering, for workspaces where staleness is a concern.
- Document 3-5 example questions in the skill file itself so agents discover its query vocabulary without trial and error.

---

## 12. CI Integration (Optional, Phase 3+)

If the workspace has CI (GitHub Actions, Azure Pipelines, etc.), add a path-filtered workflow:

```yaml
name: Update Knowledge Graph
on:
  push:
    branches: [main]
    paths:
      - '{{REPO_1}}/**'
      - '{{REPO_2}}/**'
      # ... one path filter per in-scope repo

jobs:
  update-graph:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: actions/setup-python@v5
        with: { python-version: '3.12' }
      - run: pip install -r {{KG_DIR}}/scripts/requirements.txt
      - run: python3 {{KG_DIR}}/scripts/build-graph.py
      - run: python3 {{KG_DIR}}/scripts/validate-graph.py
      - name: Commit graph diff
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add {{KG_DIR}}/
          git diff --staged --quiet || git commit -m "chore(knowledge-graph): auto-update from CI"
          git push
```

**Failure mode by design**: if `validate-graph.py` exits non-zero, the workflow fails and nothing is committed — a broken graph never lands on `main`.

---

## 13. Optional Follow-On: Interactive Visualizer

Once the graph itself is stable, validated, and actively used via CLI/agent skill, consider a **separate solution design** for an interactive, browser-based visualization (force-directed graph, filter/search, node detail panel, shortest-path finder). This was built successfully on the reference workspace using:

- React 18 + TypeScript + Vite
- D3.js v7 (force simulation) for rendering
- `@dagrejs/graphlib` for shortest-path queries
- Zustand for selection/filter state
- Fuse.js for fuzzy search
- Static JSON imports via `import.meta.glob` — no backend required
- Deployed as a static site (GitHub Pages / Azure Static Web Apps)

**Do not build the visualizer before the data model is validated** — a visual layer amplifies data-quality problems (missing nodes, dangling edges) rather than hiding them. Author the visualizer as its own solution design once this graph's `validate-graph.py` has been passing cleanly for at least one full extraction cycle, and set that document's `relatedSolutionDocs` to reference this one.

---

## 14. Implementation Phases (Summary)

Detailed phase breakdown belongs in the execution plan file generated via `/scaffold-planfile` — this section is the input to that step, not a replacement for it.

| Phase | Deliverable | Depends On |
|---|---|---|
| 1 — Discover & Seed | `schema/`, `nodes/repositories`, `nodes/services`, `nodes/datastores`, initial `edges/*.json` populated from §3 survey | §3 survey complete |
| 2 — Extraction Scripts | `build-graph.py`, `validate-graph.py` | Phase 1 |
| 3 — Query Interface | `query-graph.py`, agent skill (§11) | Phase 2 |
| 4 — CI Integration | `.github/workflows/update-knowledge-graph.yml` (or equivalent) | Phase 2 |
| 5 — Quality & Documentation | `{{KG_DIR}}/README.md`, tests for extraction/validation scripts, final review | Phases 1-4 |
| *(Optional, separate solution doc)* — Visualizer | Interactive browser UI (§13) | Phase 5 complete and stable |

---

## 15. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Repos not colocated under one root (fetched separately, different machines) | Medium | Medium | Parameterize `--repos-root`; document that CI must check out all in-scope repos before running `build-graph.py` |
| Private/inaccessible repos block discovery | Medium | Medium | Document required access up front; degrade gracefully — missing repos produce TBD placeholders, not a hard failure |
| Schema drift (new node/edge type needed mid-project) | Medium | Low | Schema files are just JSON — add a new type definition and re-validate; no migration needed since there's no DB schema to alter |
| Secrets accidentally captured in `Config` nodes | Low (if guardrail in §4.1/§7 followed) | High | Code review checklist item: `CONFIGURED_BY` edges must never contain resolved secret values, only key names |
| Graph grows beyond ~1,000 nodes, JSON approach strains | Low for most workspaces | Medium | Documented escape hatch in §6.1 — migrate to embedded graph engine only if this threshold is actually reached |
| Additive-merge script silently keeps stale data that should have changed | Medium | Low | `build-graph.py` only merges *new* discoveries; deliberate corrections to existing fields must be edited by hand — document this behavior clearly in the script's own docstring |

---

## 16. Success Metrics

| Metric | Target |
|---|---|
| All in-scope repos represented as `Repository` nodes | 100% |
| All identified deployable units represented as `Service` nodes | 100% |
| `validate-graph.py` exit code | 0 (zero referential integrity violations) |
| `query-graph.py` response time for any single query | < 1 s on a laptop |
| An AI agent can answer "what depends on X" without re-scanning the repo | Yes, verified with a sample question per repo |
| Graph freshness (if CI wired) | Auto-updates within one CI run of a relevant push |

---

## 17. Acceptance Criteria

- [ ] `{{KG_DIR}}/schema/node-types.json` and `edge-types.json` exist and match §4
- [ ] Every in-scope repo from §3.1 has a corresponding `Repository` node
- [ ] Every deployable unit identified in §3.3 has a corresponding `Service` node
- [ ] Every datastore identified in §3.4 has a corresponding `DataStore` node with correct `READS_FROM`/`WRITES_TO` edges
- [ ] Every cross-repo dependency identified in §3.5 has a corresponding `DEPENDS_ON` edge with endpoint list
- [ ] `python3 {{KG_DIR}}/scripts/validate-graph.py` exits 0
- [ ] `python3 {{KG_DIR}}/scripts/query-graph.py "<sample question>"` returns a correct, human-readable answer for at least 5 distinct question types (§10.3)
- [ ] `{{KG_DIR}}/README.md` documents directory layout, query usage, and update process
- [ ] Config/secret guardrail (§4.1) verified — no resolved secret values present anywhere in `{{KG_DIR}}/`

---

## 18. Related Documents & Dependencies

| Document | Status | Notes |
|---|---|---|
| This document | TEMPLATE | Instantiate per §0 for a specific workspace |
| `/scaffold-planfile` workflow | Existing | Consumes §5/§6/§7/§8 of this document to auto-generate the execution plan |
| `{{KG_DIR}}/README.md` | To be authored in Phase 5 | End-user query and update guide |
| *(Future)* Visualizer solution design | Not yet authored | See §13 — separate document, references this one via `relatedSolutionDocs` |
