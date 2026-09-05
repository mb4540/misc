---
title: "Jemba9 Multi-Root Workspace Knowledge Graph — Solution Design"
date: "2026-09-05"
solutionId: "JEMBA9-KG-0001"
status: "PROPOSED"
version: "2.0"
author:
  name: "Michael Berry"
reviewers: []
relatedPlanFile: null
relatedSolutionDocs:
  - "002-knowledge-graph-multiroot-template.md"
tags:
  - "knowledge-graph"
  - "multi-root-workspace"
  - "developer-tooling"
  - "jemba9"
---

# Jemba9 Multi-Root Workspace Knowledge Graph — Solution Design

> **Goal:** Build a version-controlled, offline-queryable knowledge graph for the current ten-root development workspace so human developers and AI agents can answer cross-repository architecture, provenance, migration, dependency, and ownership questions without surveying every root again. This document replaces the generic placeholders and single-root assumptions in `002-knowledge-graph-multiroot-template.md` with the workspace’s actual repositories, generated configuration mirrors, asset collection, service boundaries, and governance rules.

## 0. Design-Stage Boundary

This is a Stage 2 Solution Design artifact under `~/.config/devin/constitutions/dev-lifecycle-workflows.md`. It chooses an approach and identifies implementation files; it does not create graph data, scripts, skills, CI, or application changes.

The only file written during this stage is this requested solution document:

- `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/003-knowledge-graph-multiroot-template.md`

## 1. Problem Statement

The current IDE workspace spans seven git repositories and three non-git roots located under multiple parent directories, while important relationships—such as the `wcca-poc` to Board Lab port, MySkills-to-live-Devin configuration synchronization, QMS content vendored from `j9ccgit`, and planned application migrations—are recorded only in scattered documentation and source comments. The desired outcome is one trustworthy graph that represents every configured workspace root, distinguishes current dependencies from historical or planned relationships, and lets developers and AI agents trace impact and provenance without repeated multi-repo searches.

### 1.1 User outcome

A developer or AI agent can run a local query such as:

- `what depends on jemba9-ecosystem-platform`
- `what was ported from wcca-poc`
- `where do live Devin skills come from`
- `what applications use Lakebase Postgres`
- `show planned migrations for jemba9-qms`
- `which roots are generated mirrors rather than source repositories`

The answer must include relationship status and evidence paths so a historical port or planned migration is not mistaken for a current runtime call.

### 1.2 Current state

Concrete anchors found during the workspace survey include:

- `j9ccgit/README.md` defines the Jemba9 — Space Assurance Gateway monorepo, four React portals, an Express backend, Databricks Lakebase, LangGraph, Cognito, and AWS CDK.
- `Jemba9_Ecosystem/README.md` and `Jemba9_Ecosystem/Ecosystem/CLAUDE.md` define a platform plus independently built business applications under `Applications/`.
- `Jemba9_Ecosystem/Applications/boardlab/src/graph/board-graph.ts` and neighboring files explicitly identify code ported from `wcca-poc`.
- `Jemba9_QMS/app/README.md` defines QMS Lite, its Roadmap SPA, Cognito-authenticated notes, AWS infrastructure, and a documented migration path.
- `Jemba9_QMS/app/roadmap/src/` contains content explicitly vendored from `j9ccgit` deploy runbooks.
- `MySkills/CLAUDE.md` defines version-controlled skill and constitution masters, while `~/.config/devin/skills/` and `~/.config/devin/constitutions/` are live generated copies.
- `WorkingConcepts/TrainingPortal/` is a static prototype without a package manifest.
- `/Users/michaelberry/Desktop/J9Logos/` is a non-git binary brand-asset collection.
- `misc/knowledgeGraph/002-knowledge-graph-multiroot-template.md` provides a generic JSON graph design but assumes repositories are sibling directories under one root and does not model generated mirrors or non-git roots.

### 1.3 Gap

The generic design cannot accurately describe this workspace because:

1. There is no single common repository root.
2. Not every configured root is a git repository.
3. Some roots are generated mirrors whose canonical source is elsewhere.
4. Several relationships are historical, planned, vendored, or governance-related rather than runtime API dependencies.
5. Absolute machine-specific paths should not be committed as canonical identity.
6. A full-regeneration scanner would erase manually verified context and could turn ambiguous documentation into false architectural claims.

## 2. Scope, Constraints, and Non-Goals

### 2.1 In scope

- Represent all ten configured workspace roots.
- Deep-index the seven git repositories and inventory their deployable units.
- Represent the two live Devin configuration roots as generated mirrors of MySkills rather than duplicate source trees.
- Represent the Jemba9 logo directory as an asset root without parsing binary contents.
- Capture runtime, data-access, deployment, auth, governance, vendoring, porting, mirroring, and migration relationships.
- Attach evidence, lifecycle status, and discovery method to every non-trivial edge.
- Provide deterministic build, validation, and local query scripts using Python’s standard library.
- Provide a Devin skill from the canonical MySkills master so agents can query the graph.

### 2.2 Constraints

- Canonical graph files live in the git-tracked `misc` repository under `knowledgeGraph/graph/`, a neutral location outside any product repo.
- The graph must work offline and require no database or external service.
- The scanner must use an explicit allowlisted root manifest; it must not crawl `$HOME` or infer sibling roots.
- Stored paths use aliases plus `${HOME}` path expressions, not `/Users/michaelberry` as persistent identity.
- Secret values, `.env` contents, credentials, tokens, generated databases, user uploads, build outputs, dependency directories, and git internals are never ingested.
- Existing repository-specific rules remain authoritative. The graph records them; it does not replace them.
- MySkills is the canonical source for global Devin skills and runtime constitutions. Live `~/.config/devin/` roots are materialized mirrors.
- IDs are stable slugs and do not change when a directory or display name changes.
- Extraction is additive and evidence-backed. It may add detected facts and refresh derived views, but it may not delete or overwrite curated relationships.

### 2.3 Explicit non-goals

This design does not build an interactive visualizer, a graph database, a semantic-vector search service, runtime service discovery, a source-code AST index, or cross-repository CI automation. It also does not reconcile product architecture conflicts by guessing; it records documented relationships with `current`, `planned`, `historical`, or `superseded` status and evidence.

## 3. Workspace Discovery Survey

### 3.1 Root inventory

| Alias | Path expression | Kind | Git | Purpose | Scan policy |
|---|---|---|---:|---|---|
| `j9ccgit` | `${HOME}/Documents/Jemba9/CasecadeProjects/j9ccgit` | product monorepo | Y | Jemba9 — Space Assurance Gateway applications, API, data, agents, and AWS CDK | Deep |
| `jemba9-ecosystem` | `${HOME}/Documents/Jemba9/CasecadeProjects/Jemba9_Ecosystem` | product monorepo | Y | Jemba9 ecosystem platform and business applications | Deep |
| `jemba9-qms` | `${HOME}/Documents/Jemba9/CasecadeProjects/Jemba9_QMS` | product/documentation repo | Y | QMS Lite, roadmap, shell, notes services, QMS documents | Deep |
| `working-concepts` | `${HOME}/Documents/Jemba9/CasecadeProjects/WorkingConcepts` | prototype repo | Y | Static prototypes including Training Portal | Manifest + curated |
| `misc` | `${HOME}/Documents/Jemba9/CasecadeProjects/misc` | tooling/reference repo | Y | Cross-project guides, audits, experiments, and canonical knowledge graph | Curated |
| `wcca-poc` | `${HOME}/Documents/Jemba9/CasecadeProjects/wcca-poc` | proof-of-concept repo | Y | Standalone WCCA front-half reference implementation | Deep |
| `myskills` | `${HOME}/Documents/Consulting/MySkills` | developer-tooling repo | Y | Lifecycle skills, constitutions, artifacts, and companion webapp | Deep |
| `devin-constitutions-live` | `${HOME}/.config/devin/constitutions` | generated config mirror | N | Runtime copies of lifecycle constitutions | Metadata only |
| `devin-skills-live` | `${HOME}/.config/devin/skills` | generated config mirror | N | Runtime copies of globally loaded Devin skills | Metadata only |
| `jemba9-brand-assets` | `${HOME}/Desktop/J9Logos` | asset collection | N | Source logo files and packaged brand assets | Filename metadata only |

All ten roots are in scope at the workspace-root level. Deep source extraction is limited to the seven git repositories; the three non-git roots are represented without treating them as repositories.

### 3.2 Initial deployable-unit seed

The implementation survey must verify this seed rather than blindly copying it.

| Repository | Unit ID | Kind / stack | Evidence anchor |
|---|---|---|---|
| `j9ccgit` | `employee-console` | React/Vite SPA | `apps/employee-console/package.json` |
| `j9ccgit` | `customer-portal` | React/Vite SPA | `apps/customer-portal/package.json` |
| `j9ccgit` | `control-center` | React/Vite SPA | `apps/control-center/package.json` |
| `j9ccgit` | `stakeholder-portal` | React/Vite SPA | `apps/stakeholder-portal/package.json` |
| `j9ccgit` | `space-assurance-api` | Express/TypeScript/LangGraph API | `backend/package.json`, `README.md` |
| `jemba9-ecosystem` | `jemba9-ecosystem-platform` | Next.js platform and central authorization | `Ecosystem/package.json`, `Ecosystem/CLAUDE.md` |
| `jemba9-ecosystem` | `jemba9-ecosystem-worker` | BullMQ worker | `Ecosystem/src/worker/`, `Ecosystem/package.json` |
| `jemba9-ecosystem` | `okr-app` | Next.js business app | `Applications/OKR/package.json` |
| `jemba9-ecosystem` | `data-room-app` | Next.js business app | `Applications/dataroom/package.json` |
| `jemba9-ecosystem` | `board-lab-app` | Next.js/LangGraph business app | `Applications/boardlab/package.json` |
| `jemba9-qms` | `qms-lite` | React/Vite SPA | `app/package.json` |
| `jemba9-qms` | `qms-roadmap` | React/Vite SPA | `app/roadmap/package.json` |
| `jemba9-qms` | `j9works-shell` | React/Vite SPA | `app/shell/package.json` |
| `jemba9-qms` | `qms-notes-handler` | API Gateway/Lambda service | `infra/lambda/notes/package.json`, `app/README.md` |
| `jemba9-qms` | `qms-roadmap-handler` | Lambda service | `infra/lambda/roadmap/package.json` |
| `jemba9-qms` | `qms-wiki-handler` | Lambda service | `infra/lambda/wiki/package.json` |
| `working-concepts` | `training-portal-prototype` | Static HTML prototype | `TrainingPortal/index.html` |
| `wcca-poc` | `wcca-poc-api` | Express/TypeScript/LangGraph API | `package.json`, `src/server.ts` |
| `wcca-poc` | `wcca-poc-web` | React/Vite SPA | `web/`, `README.md` |
| `myskills` | `myskills-companion-webapp` | Next.js local-first application | `webapp/package.json` |

Infrastructure packages and CDK/Terraform stacks are modeled as `Infrastructure` nodes, not as runtime services unless they contain an independently invoked handler.

### 3.3 Initial datastore and state seed

| Store ID | Type | Principal users | Evidence anchor |
|---|---|---|---|
| `space-assurance-lakebase` | Databricks Lakebase Postgres | `space-assurance-api` | `j9ccgit/README.md` |
| `ecosystem-lakebase` | Standard Postgres / Databricks Lakebase | ecosystem platform and applications | `Jemba9_Ecosystem/Ecosystem/CLAUDE.md` |
| `ecosystem-redis` | Redis | ecosystem BullMQ worker/platform | `Jemba9_Ecosystem/Ecosystem/CLAUDE.md` |
| `board-lab-lakebase` | Databricks Lakebase Postgres | Board Lab application and LangGraph checkpointer via separate roles/schemas | `Applications/boardlab/README.md` |
| `qms-review-notes-codecommit` | Git branch-backed document store | QMS notes Lambda | `Jemba9_QMS/app/README.md` |
| `wcca-poc-sqlite` | SQLite | WCCA POC API | `wcca-poc/README.md` |
| `wcca-poc-local-data` | Local filesystem | WCCA POC uploads and datasheets | `wcca-poc/README.md` |
| `myskills-target-files` | Local filesystem | MySkills companion webapp | `MySkills/CLAUDE.md` |

Hostnames, account IDs, database URLs, role passwords, API keys, and token values are not graph fields. Environment and config nodes may store key names such as `GRAPH_DATABASE_URL`, but never resolved values.

### 3.4 Initial cross-root relationships

| From | Relationship | To | Status | Evidence |
|---|---|---|---|---|
| `board-lab-app` | `PORTS_FROM` | `wcca-poc-api` | historical/current provenance | `Applications/boardlab/src/graph/board-graph.ts`, `src/graph/state.ts`, `src/tools/` |
| `board-lab-app` | `VENDORS_FROM` | `jemba9-ecosystem-platform` | current | `Applications/boardlab/CLAUDE.md`, `Applications/README.md` |
| `okr-app` | `AUTHORIZES_VIA` | `jemba9-ecosystem-platform` | current design | `Applications/README.md` |
| `data-room-app` | `AUTHORIZES_VIA` | `jemba9-ecosystem-platform` | migrating | `Applications/README.md` |
| `board-lab-app` | `AUTHORIZES_VIA` | `jemba9-ecosystem-platform` | current design | `Applications/boardlab/README.md` |
| `qms-roadmap` | `VENDORS_FROM` | `j9ccgit` | current content provenance | `Jemba9_QMS/app/roadmap/src/pages/`, `src/runbooks/registry.ts` |
| `jemba9-qms` | `MIGRATES_TO` | `j9ccgit` | planned in source document | `Jemba9_QMS/app/README.md` |
| `jemba9-qms` | `PLANNED_TO_JOIN` | `jemba9-ecosystem` | planned in ecosystem registry | `Jemba9_Ecosystem/Applications/README.md` |
| `devin-skills-live` | `MIRRORS_FROM` | `myskills` | current | `MySkills/CLAUDE.md` |
| `devin-constitutions-live` | `MIRRORS_FROM` | `myskills` | current | `MySkills/CLAUDE.md` |
| workspace repos | `GOVERNED_BY` | `myskills` | current where adopted | per-repo `CLAUDE.md`, `.devin/rules/`, and plan references |

The two QMS migration edges are intentionally both present. They are documented statements from different repositories, not proof that both migrations will occur. Evidence and status preserve the ambiguity without converting it into an unresolved design question.

## 4. Approaches Considered

### 4.1 Approach A — Central manifest-driven JSON graph

Create one graph in `misc/knowledgeGraph/graph/`. An explicit root manifest defines all ten roots, their kinds, path expressions, canonical/source status, and scan policy. Curated JSON nodes and edges are enriched by additive Python standard-library extraction; derived views are regenerated; a validator enforces identity, referential integrity, provenance, status, and path safety; a MySkills-owned Devin skill wraps the query CLI.

**Affected systems:** `misc/knowledgeGraph/graph/`, the seven source repositories as read-only scan inputs, `MySkills/skills/workspace-knowledge-graph/SKILL.md`, and the two live Devin config roots as generated destinations after normal skill synchronization.

**Key tradeoff:** One canonical graph makes cross-root queries simple and consistent, but freshness depends on an explicit local refresh because the roots do not share one CI system.

### 4.2 Approach B — Per-repository graph fragments with federation

Store a graph fragment inside each git repository and build a federated index that loads all available fragments. Each repository owns its own facts and can validate them in its own CI; cross-repository edges live either with the caller or in a separate federation manifest.

**Affected systems:** new graph directories in `j9ccgit`, `Jemba9_Ecosystem`, `Jemba9_QMS`, `WorkingConcepts`, `misc`, `wcca-poc`, and `MySkills`, plus a federation tool and conventions shared across all repos.

**Key tradeoff:** Ownership and repository-local freshness improve, but schema rollout, ID coordination, generated mirrors, non-git roots, and atomic cross-repo relationship changes become substantially harder.

### 4.3 Approach C — Fully regenerated workspace inventory

Generate a single machine-produced inventory from manifests, source-text patterns, and git metadata on every run. Treat generated output as authoritative and avoid manual curation except for scanner configuration.

**Affected systems:** a root manifest, scanner, generated nodes/edges/views, and query/validation tooling under `misc/knowledgeGraph/graph/`.

**Key tradeoff:** Refresh is simple, but the scanner cannot reliably infer why code was ported, whether a migration is current or planned, which document supersedes another, or whether a detected URL is an actual runtime dependency. Regeneration would produce a less trustworthy graph for the workspace’s highest-value questions.

## 5. Recommendation

**Choose Approach A: a central, manifest-driven, curated JSON graph with additive extraction and provenance-aware edges.** It reuses the generic template’s strongest pattern—git-native JSON plus validation and a local CLI—while making the minimal upstream correction required by this workspace: replace the single sibling-directory assumption with an explicit typed root manifest.

This approach also preserves manually verified historical and planned context that automation cannot infer. The accepted tradeoff is manual/local refresh rather than cross-repository CI in the first implementation.

### 5.1 Chosen approach in three sentences

Store the canonical graph in the `misc` git repository and identify every configured root through a path-portable, allowlisted root manifest. Seed curated root, repository, service, datastore, infrastructure, and cross-root provenance records, then enrich them with additive standard-library extraction while regenerating only derived views. Validate every reference, evidence path, lifecycle status, and scan boundary before queries or agent use.

### 5.2 What this gives up

- No automatic refresh after every commit in every repository.
- No rich graph-database query language.
- No repository-local ownership of graph fragments.
- No interactive visualization in this scope.

### 5.3 Open questions

None.

Resolved decisions:

1. **Canonical location:** `misc/knowledgeGraph/graph/`, because `misc` is git-tracked and neutral across product repositories.
2. **Workspace membership:** all ten IDE roots are represented; seven git roots are deep-scanned, two live config roots are metadata-only generated mirrors, and the logo root is filename-metadata-only.
3. **Path portability:** aliases plus `${HOME}` expressions are stored; resolved absolute paths are runtime-only.
4. **Conflicting migration statements:** retain both as evidence-backed `planned` edges rather than selecting an undocumented winner.
5. **Dependencies:** Python standard library only for build, validation, and query tooling; use an adjacency map and BFS rather than adding NetworkX.
6. **Agent integration:** author the skill in the MySkills master and install it through the existing `/sync-skills` process; do not hand-edit the live skill root as the source of truth.
7. **Freshness:** explicit local refresh is required in the first implementation; multi-repo CI is deferred because it is not needed to meet the stated outcome and has no single existing pipeline boundary.

## 6. Comparison Rationale

| Criterion | Central manifest JSON | Federated fragments | Full regeneration |
|---|---:|---:|---:|
| Cross-root querying | Best | Requires federation | Good |
| Represents non-git roots | Directly | Awkward | Directly |
| Preserves curated provenance | Yes | Yes, but distributed | No |
| Schema/ID consistency | One validation boundary | Multi-repo coordination | One boundary |
| Infrastructure required | None | None, but more orchestration | None |
| Handles planned/historical edges | Explicitly | Explicitly but distributed | Poorly |
| Initial complexity | Moderate | High | Low |
| Ongoing freshness | Manual refresh | Potentially repo-local CI | Simple regeneration |
| Workspace fit | Best | Over-distributed | Insufficiently trustworthy |

The graph is expected to remain far below 1,000 nodes. JSON and in-memory adjacency maps therefore meet performance needs without a runtime dependency or service.

## 7. Architecture Decisions

1. **Canonical host:** `misc/knowledgeGraph/graph/` in the `misc` git repository.
2. **Root registry:** `manifest/workspace-roots.json` is the only source of scan targets; no sibling discovery and no recursive home-directory discovery.
3. **Root typing:** `git-repository`, `generated-config`, and `asset-collection` are distinct root kinds.
4. **Identity:** permanent, globally unique slug IDs. Display names and paths may change without changing IDs.
5. **Path storage:** `${HOME}` expressions and root-relative evidence paths; no username-specific absolute paths in graph records.
6. **Curation model:** curated JSON is authoritative; extraction is additive/update-only and never deletes curated fields or edges.
7. **Derived data:** `views/*.json` is fully generated and may be replaced on every build.
8. **Evidence:** every cross-root, auth, porting, vendoring, governance, and migration edge requires at least one source-root alias plus relative evidence path.
9. **Lifecycle status:** edges support `current`, `planned`, `historical`, and `superseded`; queries show status by default.
10. **Discovery method:** records identify `curated`, `manifest`, or `source-pattern`; inferred facts cannot silently overwrite curated facts.
11. **Query engine:** Python standard-library adjacency maps and BFS; no LLM and no NetworkX dependency.
12. **Agent ownership:** the query skill’s master lives under MySkills and is synchronized to the live Devin configuration.
13. **Secrets discipline:** scan exclusions are deny-by-default for secret-bearing and generated paths; config records store names only.
14. **Validation gate:** invalid references, duplicate IDs, missing evidence, invalid statuses, unsafe paths, escaped symlinks, or secret-like values fail validation.

## 8. Knowledge Graph Data Model

### 8.1 Node types

| Node type | Purpose | Required fields |
|---|---|---|
| `WorkspaceRoot` | Any configured IDE root, including repos, generated config, and assets | `@type`, `@id`, `name`, `pathExpression`, `rootKind`, `scanPolicy` |
| `Repository` | A git repository | `@type`, `@id`, `name`, `workspaceRoot`, `description`, `primaryLanguage` |
| `Service` | An independently run/deployed app, API, worker, static site, or Lambda | `@type`, `@id`, `name`, `repository`, `serviceKind`, `description` |
| `APIEndpoint` | A significant exposed or consumed route | `@type`, `@id`, `method`, `path`, `service` |
| `DataStore` | Database, cache, branch-backed store, object store, or local persisted state | `@type`, `@id`, `name`, `storeType`, `description` |
| `Collection` | A table, schema, collection, bucket, or branch where useful | `@type`, `@id`, `name`, `dataStore` |
| `DataModel` | A significant schema or contract | `@type`, `@id`, `name`, `service`, `sourceFile` |
| `Component` | A significant UI page or component when needed for a traced relationship | `@type`, `@id`, `name`, `repository`, `sourceFile` |
| `Library` | A major framework or shared package | `@type`, `@id`, `name`, `ecosystem` |
| `Infrastructure` | Cloud resource, deployment target, CI/CD system, or local runtime | `@type`, `@id`, `name`, `infraType` |
| `Environment` | A named deployment environment | `@type`, `@id`, `name` |
| `Config` | A config file or key reference, never a secret value | `@type`, `@id`, `name`, `configType`, `secret` |
| `AssetCollection` | A non-code asset set such as the Jemba9 logo root | `@type`, `@id`, `name`, `workspaceRoot`, `assetType` |
| `Team` | A code-owning team or group when documented | `@type`, `@id`, `name` |

### 8.2 Edge types

The generic runtime edge types remain: `CONTAINS`, `DEPENDS_ON`, `CALLS`, `EXPOSES`, `READS_FROM`, `WRITES_TO`, `USES_COLLECTION`, `DEFINES`, `REFERENCES`, `RENDERS`, `ROUTES_TO`, `BUILT_WITH`, `DEPLOYED_TO`, `DEPLOYED_AT`, `CONFIGURED_BY`, `OWNED_BY`, `AUTHENTICATES_VIA`, and `PROXIES_TO`.

Workspace-specific relationship types add the semantics the generic template lacked:

| Edge type | From → To | Purpose |
|---|---|---|
| `MIRRORS_FROM` | WorkspaceRoot → Repository | Generated live root is materialized from canonical source |
| `PORTS_FROM` | Service/Repository → Service/Repository | Implementation originated as a deliberate port |
| `VENDORS_FROM` | Service/Repository → Service/Repository | Source, tokens, docs, or SDK copied and maintained locally |
| `MIGRATES_TO` | Repository/Service → Repository/Service | Documented migration destination |
| `PLANNED_TO_JOIN` | Repository/Service → Repository/Service | Declared future ecosystem membership |
| `GOVERNED_BY` | Repository/WorkspaceRoot → Repository | Lifecycle rules or constitutions come from another repository |
| `USES_ASSETS_FROM` | Repository/Service → AssetCollection | Documented brand-asset consumption |

Each non-trivial edge also contains:

```json
{
  "from": "board-lab-app",
  "to": "wcca-poc-api",
  "type": "PORTS_FROM",
  "status": "historical",
  "discoveryMethod": "curated",
  "evidence": [
    {
      "root": "jemba9-ecosystem",
      "path": "Applications/boardlab/src/graph/board-graph.ts",
      "lines": "1-5"
    }
  ],
  "notes": "The production application ported and expanded the POC graph topology."
}
```

## 9. Component Archetypes

| Root/repository | Tech signal | Archetype | Notes |
|---|---|---|---|
| `j9ccgit` | npm workspaces, React/Vite, Express, Drizzle, CDK | full-stack monorepo | Four portals, API, agents, shared packages, infrastructure |
| `jemba9-ecosystem` | Multiple Next.js package roots, Prisma, Terraform | platform + multi-application monorepo | Platform and business apps have independent build boundaries |
| `jemba9-qms` | React/Vite apps, Lambda packages, CDK, Markdown corpus | full-stack + documentation | QMS presentation, roadmap/shell, authenticated notes, infrastructure |
| `working-concepts` | Static HTML | static prototype | No package manifest; curate service metadata |
| `misc` | Markdown/HTML/reference files | documentation/tooling | Canonical host for the graph |
| `wcca-poc` | Express, React/Vite, SQLite, LangGraph | full-stack POC | Historical source for Board Lab portions |
| `myskills` | pnpm, Next.js, filesystem APIs, Markdown skills | developer tooling | Canonical workflow/skill source and companion app |
| live Devin config roots | SKILL.md and constitution mirrors | generated configuration | Metadata and mirror relationships only |
| Jemba9 logo root | PNG/AI/PDF/ZIP | asset library | Filename metadata only; no binary parsing |
| graph tooling | Python standard library, JSON, Markdown | developer tooling | Build, validation, query, and agent wrapper |

Universal review/documentation/test archetypes apply during planning, but this solution does not prescribe implementation phases beyond the file-scoped handoff below.

## 10. Proposed Directory Layout

```text
misc/knowledgeGraph/
├── 002-knowledge-graph-multiroot-template.md
├── 003-knowledge-graph-multiroot-template.md
└── graph/
    ├── README.md
    ├── manifest/
    │   └── workspace-roots.json
    ├── schema/
    │   ├── node-types.json
    │   └── edge-types.json
    ├── nodes/
    │   ├── workspace-roots/
    │   ├── repositories/
    │   ├── services/
    │   ├── datastores/
    │   ├── collections/
    │   ├── infrastructure/
    │   ├── config/
    │   └── assets/
    ├── edges/
    │   ├── containment.json
    │   ├── runtime-dependencies.json
    │   ├── data-access.json
    │   ├── auth.json
    │   ├── deployment.json
    │   ├── provenance.json
    │   ├── migration.json
    │   └── governance.json
    ├── views/
    │   ├── workspace-inventory.json
    │   ├── dependency-matrix.json
    │   ├── tech-stack-summary.json
    │   ├── datastore-map.json
    │   ├── provenance-map.json
    │   └── migration-map.json
    ├── scripts/
    │   ├── build_graph.py
    │   ├── validate_graph.py
    │   └── query_graph.py
    └── tests/
        ├── test_build_graph.py
        ├── test_validate_graph.py
        └── test_query_graph.py

MySkills/skills/workspace-knowledge-graph/
└── SKILL.md
```

The live copy at `${HOME}/.config/devin/skills/workspace-knowledge-graph/SKILL.md` is generated by `/sync-skills install`; it is not the authored source.

## 11. Extraction, Validation, and Query Behavior

### 11.1 Root resolution and scan boundaries

`build_graph.py` reads only `manifest/workspace-roots.json`. For each enabled root it expands `${HOME}`, resolves the real path, verifies the expected root kind, and applies the declared scan policy.

The scanner must:

- Refuse unregistered roots unless the user explicitly supplies a manifest override.
- Never follow a symlink whose resolved target leaves the registered root.
- Ignore `.git/`, dependency directories, build outputs, caches, generated clients, `.env*`, local databases, uploads, fixtures containing customer data, archives, and binary files.
- Read only approved manifest, configuration, documentation, and source extensions.
- Report missing roots without deleting their curated nodes.
- Support `--dry-run`, `--root <alias>`, and `--manifest <path>`.

### 11.2 Additive extraction

Automated extraction may add or update machine-verifiable facts such as:

- git HEAD and current branch as derived metadata;
- package names, languages, frameworks, scripts, and workspace membership;
- declared ports and entry points;
- route candidates as review output;
- datastore/config key names from approved source files;
- root availability and mirror drift metadata.

It must not auto-create high-impact cross-root relationships such as `PORTS_FROM`, `MIGRATES_TO`, `AUTHENTICATES_VIA`, or `GOVERNED_BY` without curated evidence. It must never overwrite curated scalar values and must never delete curated data.

### 11.3 Validation

`validate_graph.py` fails when:

1. A node ID is duplicated globally.
2. An edge endpoint does not resolve.
3. A node or edge type is undeclared.
4. A typed edge violates its allowed endpoint types.
5. A non-trivial edge lacks `status`, `discoveryMethod`, or evidence.
6. An evidence root is unknown, a relative path escapes its root, or a cited file is missing when the root is available.
7. A root path is absolute without an approved variable expression.
8. A record contains a secret-like field/value, connection string, private key marker, bearer token, or prohibited `.env` evidence path.
9. A generated mirror is modeled as canonical source.
10. A derived view differs from a fresh in-memory regeneration.

### 11.4 Query interface

`query_graph.py` supports deterministic patterns and structured filters, including:

| Query | Result |
|---|---|
| `list roots` | All roots with kind, availability, scan policy, and canonical/mirror status |
| `list services in <repo>` | Contained services |
| `what depends on <id>` | Incoming current dependency edges by default |
| `what does <id> depend on` | Outgoing current dependency edges |
| `show provenance for <id>` | Port, vendoring, mirroring, and evidence paths |
| `show planned migrations` | Planned migration/join edges only |
| `what uses <store-id>` | Data readers/writers |
| `auth for <id>` | Auth/authz relationships and status |
| `governance for <id>` | Governing repository/rules evidence |
| `path between <id> and <id>` | Shortest directed path, with optional status filter |
| `find <text>` | Case-insensitive node field lookup |

Output is Markdown by default and JSON with `--json`. Current edges are the default; `--status planned,historical,superseded` opts into non-current relationships so blast-radius queries do not accidentally include old ports or future architecture.

## 12. Devin Integration

The canonical skill is authored at:

- `${HOME}/Documents/Consulting/MySkills/skills/workspace-knowledge-graph/SKILL.md`

It invokes `query_graph.py`, provides example questions, defaults to read-only queries, and never refreshes or modifies the graph unless the user explicitly requests a refresh. The existing `/sync-skills` process installs it to `${HOME}/.config/devin/skills/`.

The skill must use the actual graph path from this design, quote all paths, and pass user query text as an argument list rather than constructing a shell command string.

## 13. Files and Systems That Will Change

The implementation plan is limited to these authored areas:

1. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/README.md`
2. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/manifest/workspace-roots.json`
3. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/schema/node-types.json`
4. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/schema/edge-types.json`
5. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/nodes/**`
6. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/edges/**`
7. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/views/**`
8. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/scripts/build_graph.py`
9. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/scripts/validate_graph.py`
10. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/scripts/query_graph.py`
11. `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc/knowledgeGraph/graph/tests/**`
12. `/Users/michaelberry/Documents/Consulting/MySkills/skills/workspace-knowledge-graph/SKILL.md`

Read-only inputs are the ten roots in §3.1. No application source file is changed to produce the first graph.

Generated but not directly authored:

- `${HOME}/.config/devin/skills/workspace-knowledge-graph/SKILL.md`, installed from MySkills by `/sync-skills`.

## 14. Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---:|---:|---|
| Secret or credential material is accidentally indexed | Medium | High | Deny-by-default scan extensions/paths, never read `.env*`, validate for secret patterns, store config names only |
| A symlink escapes an allowlisted root | Low | High | Resolve every candidate path and reject targets outside the root realpath |
| Historical/planned relationships pollute blast-radius answers | Medium | High | Mandatory edge status; current-only query default; always display status |
| Generated Devin mirrors are mistaken for canonical source | Medium | Medium | Root kinds plus required `MIRRORS_FROM`; validator rejects canonical designation for generated roots |
| Absolute local paths make the graph non-portable | High without controls | Medium | `${HOME}` expressions, aliases, relative evidence paths, runtime-only resolution |
| Additive extraction leaves stale curated facts | Medium | Medium | Include `verifiedAt`, source evidence, and stale-report output; corrections remain explicit reviewed edits |
| Missing root causes destructive graph shrinkage | Medium | Medium | Mark unavailable and report; never delete curated nodes or edges automatically |
| Conflicting architecture documents produce false certainty | Medium | High | Preserve both evidence-backed edges with status and notes; do not infer resolution |
| Binary assets or large generated trees make scans slow | Medium | Low | Filename-only asset policy, size limits, extension allowlist, ignored directories |
| Central graph freshness depends on manual action | Medium | Medium | Document one refresh command and expose an explicit agent refresh action; CI remains out of scope |

## 15. Security Consideration

1. **Auth — N.** The graph records authentication and authorization architecture but does not introduce, modify, or bypass any product authentication or authorization boundary.
2. **Secrets — N.** The design forbids reading `.env*` and secret stores and records configuration key names only; no credential, token, API key, private key, or connection-string value may be written or logged.
3. **User input — Y.** The build/query tools accept a manifest path, root alias, and query text that can influence file lookup and query parsing; all paths are resolved against allowlisted roots and all query text is parsed as data rather than executed.
4. **Data exposure — N.** The graph creates no user/role record access path and inherits the `misc` repository’s existing filesystem/git access; it stores architecture metadata and evidence paths only, not application records.
5. **Dependencies — N.** Build, validation, and query tooling use the Python standard library, and the Devin skill wraps those local scripts; no runtime package or external service is added.

### User-input mitigation

The implementation must avoid `shell=True`, `eval`, dynamic imports, and command-string concatenation. Manifest overrides must be regular files, root aliases must match declared IDs, evidence and scan paths must remain under the resolved allowlisted root, symlink escapes must be rejected, and query parsing must use fixed patterns with bounded input length. The Devin skill must pass arguments as an array and quote paths so query text cannot become a shell command.

## 16. Success Metrics

| Metric | Target |
|---|---|
| Configured workspace roots represented | 10/10 |
| Git repositories represented | 7/7 |
| Non-git roots correctly typed | 3/3 |
| Initial deployable units represented | 100% of verified §3.2 inventory |
| Seed cross-root relationships carry status and evidence | 100% |
| Validation result | 0 violations |
| Secret values in graph | 0 |
| Query response on local machine | Under 1 second for the expected graph size |
| Sample query coverage | At least one verified query for dependency, provenance, migration, datastore, auth, governance, and root inventory |
| AI-agent resurvey avoidance | Agent answers the seven sample question classes without scanning product repos |

## 17. Acceptance Criteria

- [ ] `manifest/workspace-roots.json` represents all ten roots with aliases, path expressions, kinds, and scan policies.
- [ ] The seven git roots are `Repository` nodes; the two live config roots are generated mirrors; J9Logos is an asset collection.
- [ ] Every verified deployable unit in §3.2 has a `Service` node and a containment edge.
- [ ] Every seed datastore in §3.3 has evidence-backed data-access relationships where verified.
- [ ] Every cross-root edge in §3.4 includes status, discovery method, and evidence.
- [ ] Both documented QMS destinations remain distinct planned relationships until a source repository records a superseding decision.
- [ ] Build extraction is additive and cannot delete or overwrite curated facts.
- [ ] Missing roots do not delete nodes or edges.
- [ ] Validation rejects duplicate IDs, dangling references, missing evidence, invalid statuses, unsafe paths, symlink escapes, secret-like values, and stale derived views.
- [ ] Query defaults exclude planned, historical, and superseded edges from current blast-radius answers.
- [ ] No graph file contains an absolute `/Users/michaelberry` path, secret value, token, API key, private key, or database connection string.
- [ ] The MySkills master skill queries the graph and is installable through `/sync-skills`.
- [ ] No product repository source code is modified for the initial graph implementation.

## 18. Validation of This Solution Design

- [x] No application code or implementation artifact was created.
- [x] The problem is stated in the user’s terms and anchored to concrete workspace paths.
- [x] Three approaches are offered with affected systems and key tradeoffs.
- [x] One approach is explicitly recommended.
- [x] The implementation file list is concrete.
- [x] Open questions are resolved; conflicting source statements are represented rather than guessed away.
- [x] The chosen approach is expressible in three sentences.
- [x] The Security Consideration section answers all five required prompts with Y/N and justification.
- [x] The explicit non-goals prevent visualization, graph-database, AST-index, runtime-discovery, and CI scope creep.

## 19. Handoff

**APPROVED APPROACH: Central manifest-driven JSON graph with additive extraction, evidence-backed lifecycle edges, deterministic validation/query tooling, and a MySkills-owned Devin query skill.**

Next: `/plan` to convert this into an implementation plan.
