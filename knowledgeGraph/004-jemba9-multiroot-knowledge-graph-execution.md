# Jemba9 Multi-Root Workspace Knowledge Graph — Execution Plan

**Date:** 2026-09-05
**Branch:** `feat/workspace-knowledge-graph` in both `misc` and `MySkills`
**Status:** READY FOR EXECUTION
**Mockup Source:** N/A — local CLI and JSON tooling; no UI is in scope
**Solution Source:** `knowledgeGraph/003-knowledge-graph-multiroot-template.md` — approved Approach A
**Template:** Follows `/Users/michaelberry/.config/devin/constitutions/NEW-plan-file-constitution.md` with the two operator-approved adaptations recorded below

**Constitution adaptations approved before authoring:**

1. This plan lives at the operator-selected `knowledgeGraph/` location rather than `documents/plans/active/`.
2. The mandatory execution-rules block’s Board Lab-specific verification line is generalized to this project’s Python checks. All other execution-rule text is retained.
3. Phase prerequisites use prior-phase `COMPLETE`, not prior-phase `APPROVED`, because the constitution’s continuous-execution protocol explicitly removes inter-phase human approval gates. Human approval remains retrospective in the Phase Checklist.

## Goal

Create a trustworthy, version-controlled, offline-queryable knowledge graph for all ten configured workspace roots, using the central manifest-driven JSON architecture selected as Approach A. The implementation will preserve curated context, safely enrich machine-verifiable facts, validate every relationship and evidence path, answer deterministic cross-root questions, and expose those queries to Devin through the canonical MySkills skill workflow.

Key outcomes:

1. **Complete workspace inventory** — Represent ten workspace roots, seven git repositories, three non-git roots, verified deployable units, datastores, infrastructure, configuration references, and asset collections.
2. **Evidence-backed relationships** — Capture runtime, data-access, deployment, authentication, authorization, provenance, migration, mirroring, and governance relationships with explicit lifecycle status and source evidence.
3. **Safe additive maintenance** — Refresh discoverable facts without deleting curated records, overwriting curated scalar values, escaping allowlisted roots, reading secret-bearing files, or parsing binary content.
4. **Validated deterministic queries** — Provide standard-library build, validation, and query CLIs with unit tests and representative dependency, provenance, migration, datastore, auth, governance, and inventory queries.
5. **Agent-accessible knowledge** — Add a canonical MySkills Devin skill that wraps the read-only query CLI and installs through the existing `/sync-skills` process.
6. **No product-code impact** — Build the first graph without modifying application source in any scanned product repository.

## EXECUTION RULES — READ BEFORE ANY WORK
CONTINUOUS-EXECUTION PROTOCOL (replaced the stop-and-review protocol on 2026-08-01 at the operator's direction)

The agent works through phases autonomously and continuously — no approval gate between phases. What replaces the gate is a mandatory written handoff at the end of every phase, so that if the session is lost, hits its context limit, or is deliberately restarted, a brand-new session can read this file and resume with no loss of fidelity. The handoff is the deliverable, not a courtesy. A phase is not finished until it is written down.

The loop
Execute ONE phase at a time, in order. Do not skip steps, do not combine phases, do not work ahead. Sequential execution is what makes the written record trustworthy.
Before starting phase N, read: this rules block, the Phase Checklist, the top-level Implementation Notes (deviations from plan), and the N.3 Implementation Notes of every prior phase. Those notes — not the original phase text — are the ground truth for as-built state.
Implement the phase's steps.
Verify — run every command in the phase's own Verification checklist. Once `validate_graph.py` and the full test suite exist, also run `python3 -m unittest discover -s knowledgeGraph/graph/tests -p 'test_*.py'` and `python3 knowledgeGraph/graph/scripts/validate_graph.py` from the `misc` repository. Every item in the phase's own Verification checklist must pass. Tick the checkboxes — - [x], not - [ ].
Write the handoff (MANDATORY — see below).
Commit with the phase's exact commit message. Do NOT push.
Continue straight to phase N+1 without waiting for approval.
The handoff — required before moving on
Both artifacts, every phase, no exceptions:

Phase Checklist row → status COMPLETE, short commit hash, timestamp. Leave Approved By blank; the operator fills it in retrospectively.
N.3 Implementation Notes → files created (count + one line each), files modified (count + what changed), dependencies added (with version and publish date), deviations from plan with rationale, design decisions, and a verification line quoting real command output.
Top-level Implementation Notes (deviations from plan) → append a **Phase N:** entry, or **Phase N:** No deviations.
Write these as if the next session has never seen this conversation — because it may not have. Anything that lives only in chat history is lost.

Honesty rules (these exist because they were broken)
Never record a verification result you did not observe. Phase 6 was marked COMPLETE and APPROVED on a written claim of "tsc — zero errors" while nine existed (review v2, finding V-1). Paste real output. npm run build passing is not evidence that npm run typecheck passes — Next does not typecheck tests/**.
If a verification step fails, the phase stays IN PROGRESS. Fix it or write down precisely why it cannot be fixed. Do not mark COMPLETE and carry the failure forward silently.
Do not describe behaviour the code does not have. Two notes claimed a BOM Quantity mismatch was recorded and inspectable when the check was an empty if block (finding V-3). If something was considered and not built, say "not implemented".
When correcting an earlier note, strike it through and date the correction rather than quietly rewriting it. The audit trail is the point.
When to stop and ask anyway
Autonomy is for execution, not for judgement calls the operator owns. Stop and ask when:

A phase depends on an engineering input that has not arrived (see §"Open Decisions" and the Follow-Up Items). Do not guess past these gates — build what is buildable, mark the rest blocked, and move on to the next phase if it is independent.
The work would require an architectural or product decision not already settled in the plan or solution design (e.g. B-4, S-5 — both now closed).
The work is destructive or irreversible: dropping/truncating tables, rewriting migration history that has been deployed, force-pushing, rewriting git history, or anything with real-world side effects.
The same defect resurfaces after a fix — that means the plan has a gap, not that the fix needs another attempt.
Otherwise: keep going.

Plan maintenance
If what you build diverges from what a later phase assumes, edit that later phase now — do not leave a contradiction for a future session to trip over. Record the edit in the top-level deviations.
Keep the header Status line current as phases complete.

## Phase Checklist

Update this checklist after each phase completes. Mark `APPROVED` only after human review.

| Phase | Description | Status | Commit Hash | Approved By | Date (Time & Date) |
|---|---|---|---|---|---|
| Phase 0 | Pre-flight — clean and branch both `misc` and `MySkills` | `NOT STARTED` | | | |
| Phase 1 | Define the root manifest and graph schemas | `NOT STARTED` | | | |
| Phase 2 | Seed workspace-root, repository, and asset nodes | `NOT STARTED` | | | |
| Phase 3 | Seed service, datastore, collection, infrastructure, and config nodes | `NOT STARTED` | | | |
| Phase 4 | Seed containment, runtime, data-access, and deployment edges | `NOT STARTED` | | | |
| Phase 5 | Seed auth, provenance, migration, and governance edges | `NOT STARTED` | | | |
| Phase 6 | Implement shared graph I/O and additive build tooling | `NOT STARTED` | | | |
| Phase 7 | Generate the six derived workspace views | `NOT STARTED` | | | |
| Phase 8 | Implement graph validation and security checks | `NOT STARTED` | | | |
| Phase 9 | Implement deterministic graph queries | `NOT STARTED` | | | |
| Phase 10 | Document graph operation and maintenance | `NOT STARTED` | | | |
| Phase 11 | Add and install the Devin query skill; run final verification | `NOT STARTED` | | | |

**Status values:** `NOT STARTED` → `IN PROGRESS` → `COMPLETE` → `APPROVED`

## Implementation Notes (deviations from plan)

Record deviations here after each phase so subsequent phases can account for them.

## Repository Context (As-Built Baseline)

### 5.1 Multi-root structure

The graph implementation is hosted by `misc`; the Devin skill is hosted by `MySkills`. All other roots are read-only inputs.

```text
${HOME}/Documents/Jemba9/CasecadeProjects/
├── j9ccgit/                         git · product monorepo · deep scan
├── Jemba9_Ecosystem/                git · platform/apps monorepo · deep scan
├── Jemba9_QMS/                      git · QMS apps/docs/infra · deep scan
├── WorkingConcepts/                 git · prototypes · manifest + curated scan
├── misc/                            git · tooling/reference · graph host
│   └── knowledgeGraph/
│       ├── 002-knowledge-graph-multiroot-template.md
│       ├── 003-knowledge-graph-multiroot-template.md
│       └── 004-jemba9-multiroot-knowledge-graph-execution.md
└── wcca-poc/                        git · WCCA proof of concept · deep scan

${HOME}/Documents/Consulting/
└── MySkills/                        git · lifecycle tooling · deep scan
    └── skills/                      canonical Devin skill masters

${HOME}/.config/devin/
├── constitutions/                   non-git · generated configuration mirror
└── skills/                          non-git · generated skill mirror

${HOME}/Desktop/
└── J9Logos/                         non-git · brand asset collection
```

Target implementation layout:

```text
misc/knowledgeGraph/graph/
├── README.md
├── manifest/workspace-roots.json
├── schema/{node-types.json,edge-types.json}
├── nodes/
│   ├── workspace-roots/index.json
│   ├── repositories/index.json
│   ├── services/index.json
│   ├── datastores/index.json
│   ├── collections/index.json
│   ├── infrastructure/index.json
│   ├── config/index.json
│   └── assets/index.json
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
├── scripts/{graph_io.py,build_graph.py,validate_graph.py,query_graph.py}
└── tests/{test_build_graph.py,test_validate_graph.py,test_query_graph.py}

MySkills/skills/workspace-knowledge-graph/SKILL.md
${HOME}/.config/devin/skills/workspace-knowledge-graph/SKILL.md  generated live copy
```

### 5.2 Shared packages and reuse inventory

| Reusable resource | Source | Use in this plan |
|---|---|---|
| Python `argparse` | Standard library | Fixed CLI arguments and bounded query input |
| Python `json` | Standard library | Manifest, schema, node, edge, and view serialization |
| Python `pathlib` / `os` | Standard library | `${HOME}` expansion, realpath containment, deterministic traversal |
| Python `subprocess` | Standard library | Optional git metadata using argument arrays with `shell=False` |
| Python `collections` | Standard library | Adjacency maps, grouped views, and BFS traversal |
| Python `re` | Standard library | Fixed query grammar and secret-pattern validation |
| Python `tempfile` / `unittest` | Standard library | Isolated tests without touching real roots |
| Existing `/sync-skills` workflow | `MySkills/skills/sync-skills/SKILL.md` | Install the authored skill from master to live config |
| Existing solution inventory | `knowledgeGraph/003-knowledge-graph-multiroot-template.md` | Seed roots, services, datastores, relationships, and acceptance criteria |

No runtime package or external service is added. Do not add NetworkX, PyYAML, a graph database, a vector database, or a JavaScript application.

### 5.3 Current state of the target

- `misc/knowledgeGraph/` currently contains only `002-knowledge-graph-multiroot-template.md`, `003-knowledge-graph-multiroot-template.md`, and this plan after authoring.
- `knowledgeGraph/graph/` does not exist.
- `MySkills/skills/workspace-knowledge-graph/` does not exist.
- The `misc` repository is on `main` and currently contains multiple unrelated untracked paths, including `knowledgeGraph/` itself.
- The `MySkills` repository is on `main` and currently has unrelated work: deleted `pnpm-lock.yaml` and untracked `skills/plan/PlanSkill.md`.
- Phase 0 is therefore blocked until the operator commits, removes, or otherwise resolves those changes. Execution must not stash, delete, restore, stage, or commit unrelated work.
- The solution design already resolves the architecture: central JSON files in `misc`, explicit root manifest, additive extraction, standard-library query engine, and MySkills-owned skill.

### 5.4 Design inventory

There is no UI mock. The binding behavior is defined by these solution sections:

| Solution section | Plan use |
|---|---|
| §3.1 | Ten-root manifest and workspace-root seed |
| §3.2 | Initial deployable-unit seed |
| §3.3 | Initial datastore seed |
| §3.4 | Initial cross-root relationship seed |
| §7 | Architecture decisions and validation rules |
| §8 | Node and edge types |
| §10 | Directory layout |
| §11 | Build, validation, and query behavior |
| §12 | Devin integration |
| §15 | Security answers and user-input mitigation |
| §17 | Acceptance criteria |

### 5.5 Route table

Not applicable. This plan creates local file-based CLI tooling and no HTTP routes, pages, server actions, API handlers, or ports.

### 5.6 New tooling components

| Component | Location | Responsibility |
|---|---|---|
| Graph I/O module | `knowledgeGraph/graph/scripts/graph_io.py` | Load graph records, resolve roots safely, normalize paths, and expose shared traversal primitives |
| Additive builder | `knowledgeGraph/graph/scripts/build_graph.py` | Enrich machine-verifiable facts and regenerate derived views without deleting curated data |
| Validator | `knowledgeGraph/graph/scripts/validate_graph.py` | Enforce schema, identity, evidence, path, mirror, derived-view, and secret-safety rules |
| Query CLI | `knowledgeGraph/graph/scripts/query_graph.py` | Answer fixed natural-language patterns and structured filters in Markdown or JSON |
| Devin skill | `MySkills/skills/workspace-knowledge-graph/SKILL.md` | Route agent questions to the read-only CLI and expose explicit refresh behavior |

## Security Consideration

1. **Auth — N.** The graph records authentication and authorization relationships but does not introduce, modify, or bypass any product authentication or authorization boundary.
2. **Secrets — N.** The implementation must not read `.env*`, credential stores, private keys, tokens, API keys, or connection-string values; `Config` nodes store approved key names only.
3. **User input — Y.** Manifest paths, root aliases, status filters, output mode, and query text affect file lookup or graph traversal; inputs must use fixed parsers, bounded lengths, allowlisted aliases, resolved path containment, and argument arrays rather than shell strings.
4. **Data exposure — N.** The graph inherits the existing filesystem and `misc` git-repository access boundary and stores architecture metadata rather than application records; it adds no user- or role-based read path.
5. **Dependencies — N.** All graph scripts and tests use the Python standard library, and skill installation uses the existing MySkills synchronization workflow.

**User-input mitigation:** Never use `shell=True`, `eval`, `exec`, dynamic imports, or command-string concatenation. Accept only declared root aliases and the `${HOME}` path expression, require manifest overrides to resolve to regular files, reject paths and symlinks that resolve outside their registered root, cap query length, parse only fixed query patterns, and pass every subprocess argument as an array. Tests must prove `.env` exclusion, symlink-escape rejection, query bounding, and the distinction between allowed secret key names and prohibited secret-like values.

## Backend Endpoints — Complete Map

### 6.1 Endpoints That Exist and Are Ready

None. Product endpoints discovered in source may be represented as graph metadata in later curated work, but no product endpoint is called by the graph tooling.

### 6.2 New Endpoints Needed

None. The implementation is a local CLI and file-based data model.

### 6.3 Endpoints Where Existing Data May Be Insufficient

None. Missing architectural facts are reported for curation rather than fetched through runtime APIs.

## 0. Phase 0: Pre-Flight — Create Feature Branches

**Commit message:** N/A — branch setup only; no code changes.
**Prerequisite:** Operator has resolved all unrelated dirty-tree changes in both repositories.

### 0.0 Context

Create and check out `feat/workspace-knowledge-graph` from an up-to-date `main` in both repositories that will receive authored files: `misc` and `MySkills`. The same branch name provides traceability, but each repository retains its own history and phase-specific commits. Do not alter any scanned product repository.

### 0.1 Implementation Steps

1. In `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc`, run `git status --short` and `git status`.
   - Expected before branch creation: a clean tree on `main` synchronized with `origin/main`.
   - Current known blocker: multiple unrelated untracked paths, including `knowledgeGraph/`.
   - If any changes remain, stop and ask the operator; do not stash, delete, restore, stage, or commit them.
2. In `/Users/michaelberry/Documents/Consulting/MySkills`, run `git status --short` and `git status`.
   - Expected before branch creation: a clean tree on `main` synchronized with `origin/main`.
   - Current known blockers: deleted `pnpm-lock.yaml` and untracked `skills/plan/PlanSkill.md`.
   - If either remains, stop and ask the operator; do not alter them.
3. For each repository, switch to `main` and run `git pull origin main` only after the tree is clean.
4. In each repository, create `feat/workspace-knowledge-graph` from the synchronized `origin/main` tip.
5. Confirm no branch is created in the seven read-only git inputs.

### 0.2 Verification

- [ ] Both repositories were clean before branch creation.
- [ ] `git -C /Users/michaelberry/Documents/Jemba9/CasecadeProjects/misc branch --show-current` returns `feat/workspace-knowledge-graph`.
- [ ] `git -C /Users/michaelberry/Documents/Consulting/MySkills branch --show-current` returns `feat/workspace-knowledge-graph`.
- [ ] Each branch tip matches its own `origin/main` tip before Phase 1 edits.
- [ ] No branch, commit, stage, stash, restore, or file change occurred in any scanned input repository.

### 0.3 Implementation Notes

_(to be filled during execution)_

## 1. Phase 1: Define Root Manifest and Graph Schemas

**Commit message:** `feat(knowledge-graph): phase 1 — define graph contracts`
**Commit repository:** `misc`
**Prerequisite:** Phase 0 `COMPLETE`

### 1.0 Context

Establish the stable contract before creating data. The root manifest eliminates the generic template’s single-parent assumption; schemas define all supported node/edge types, required fields, endpoint constraints, lifecycle statuses, and discovery methods. Later scripts must consume these files rather than duplicating constants.

### 1.1 Files

1. `knowledgeGraph/graph/manifest/workspace-roots.json`
2. `knowledgeGraph/graph/schema/node-types.json`
3. `knowledgeGraph/graph/schema/edge-types.json`

### 1.2 Implementation Steps

1. Create `workspace-roots.json` with exactly ten records from solution §3.1.
   - Required fields: `id`, `name`, `pathExpression`, `rootKind`, `git`, `scanPolicy`, `canonical`, `enabled`.
   - Allowed root kinds: `git-repository`, `generated-config`, `asset-collection`.
   - Allowed scan policies: `deep`, `manifest-curated`, `curated`, `metadata-only`, `filename-metadata-only`.
   - Paths use `${HOME}` and never `/Users/michaelberry`.
   - Mark MySkills as canonical and both live Devin roots as non-canonical mirrors.
2. Create `node-types.json` for `WorkspaceRoot`, `Repository`, `Service`, `APIEndpoint`, `DataStore`, `Collection`, `DataModel`, `Component`, `Library`, `Infrastructure`, `Environment`, `Config`, `AssetCollection`, and `Team`.
   - Define required fields and allowed enums without adding an external JSON Schema dependency.
3. Create `edge-types.json` for all solution edge types.
   - Include both `AUTHENTICATES_VIA` and `AUTHORIZES_VIA`: identity/session establishment and permission-decision delegation are different relationships.
   - Require `status`, `discoveryMethod`, and `evidence` on non-trivial cross-root, auth, provenance, migration, and governance edges.
   - Allowed status values: `current`, `planned`, `historical`, `superseded`.
   - Allowed discovery methods: `curated`, `manifest`, `source-pattern`.
4. Keep schemas data-driven so validator and query code can load them directly.

### 1.3 Verification

- [ ] `python3 -m json.tool knowledgeGraph/graph/manifest/workspace-roots.json >/dev/null` succeeds.
- [ ] `python3 -m json.tool knowledgeGraph/graph/schema/node-types.json >/dev/null` succeeds.
- [ ] `python3 -m json.tool knowledgeGraph/graph/schema/edge-types.json >/dev/null` succeeds.
- [ ] Manifest contains exactly 10 unique IDs: seven git repositories and three non-git roots.
- [ ] No manifest path contains `/Users/michaelberry`.
- [ ] Edge schema distinguishes `AUTHENTICATES_VIA` from `AUTHORIZES_VIA`.
- [ ] No dependency file or package manifest was added.

### 1.4 Implementation Notes

_(to be filled during execution)_

## 2. Phase 2: Seed Workspace, Repository, and Asset Nodes

**Commit message:** `feat(knowledge-graph): phase 2 — seed workspace roots`
**Commit repository:** `misc`
**Prerequisite:** Phase 1 `COMPLETE`

### 2.0 Context

Represent every configured root before describing services or relationships. Aggregate each node category in one deterministic `index.json` array to keep review and phase sizing manageable while retaining stable per-node IDs.

### 2.1 Files

1. `knowledgeGraph/graph/nodes/workspace-roots/index.json`
2. `knowledgeGraph/graph/nodes/repositories/index.json`
3. `knowledgeGraph/graph/nodes/assets/index.json`

### 2.2 Implementation Steps

1. Create ten `WorkspaceRoot` nodes matching the manifest one-to-one.
   - Include purpose and scan policy.
   - Generated roots must identify their expected canonical source ID.
2. Create seven `Repository` nodes: `j9ccgit`, `jemba9-ecosystem`, `jemba9-qms`, `working-concepts`, `misc`, `wcca-poc`, and `myskills`.
   - Store root-relative purpose, primary language/category, and evidence.
   - Do not store remote credentials or machine-specific absolute paths.
3. Create one `AssetCollection` node for `jemba9-brand-assets`.
   - Record asset type and filename-metadata-only policy.
   - Do not hash, parse, copy, or embed PNG, AI, PDF, or ZIP content.
4. Sort arrays by `@id` for stable diffs.
5. Add `verifiedAt` only to curated node records that were actually checked during execution; use the execution date rather than copying the solution date blindly.

### 2.3 Verification

- [ ] All three files parse with `python3 -m json.tool`.
- [ ] Workspace-root node count is 10 and repository node count is 7.
- [ ] Every repository references one known git workspace-root ID.
- [ ] The two Devin live roots are typed as generated configuration, not repositories.
- [ ] The Jemba9 logos root is typed as an asset collection, not a repository or service.
- [ ] IDs are globally unique across the three files.
- [ ] No binary content, absolute username path, secret value, or connection string is present.

### 2.4 Implementation Notes

_(to be filled during execution)_

## 3. Phase 3: Seed Service, Data, Infrastructure, and Config Nodes

**Commit message:** `feat(knowledge-graph): phase 3 — seed system inventory`
**Commit repository:** `misc`
**Prerequisite:** Phase 2 `COMPLETE`

### 3.0 Context

Populate the high-value entities needed for dependency and architecture questions. Facts must be verified against current manifests and repository guidance; the solution’s inventory is a seed, not permission to preserve a stale claim.

### 3.1 Files

1. `knowledgeGraph/graph/nodes/services/index.json`
2. `knowledgeGraph/graph/nodes/datastores/index.json`
3. `knowledgeGraph/graph/nodes/collections/index.json`
4. `knowledgeGraph/graph/nodes/infrastructure/index.json`
5. `knowledgeGraph/graph/nodes/config/index.json`

### 3.2 Implementation Steps

1. Verify and create service nodes for the 20 initial units in solution §3.2:
   - Five `j9ccgit` units.
   - Five Jemba9 Ecosystem units.
   - Six Jemba9 QMS units.
   - Training Portal, WCCA POC API/web, and MySkills companion webapp.
   - If current manifests prove the seed count changed, record the evidence and update this phase’s Implementation Notes before proceeding.
2. Create datastore nodes from solution §3.3 only after confirming evidence:
   - Space Assurance Lakebase, ecosystem Lakebase/Postgres, ecosystem Redis, Board Lab Lakebase, QMS review-notes CodeCommit branch, WCCA SQLite/local data, and MySkills target files.
3. Add collections only where a named schema, branch, bucket, or table is necessary to explain an edge.
   - Do not expand into a table-by-table database catalog in this plan.
4. Add infrastructure nodes necessary for seeded deployment relationships, such as Cognito, CloudFront/S3, API Gateway/Lambda, ECS/Fargate, Terraform/CDK-managed targets, and local runtimes, only when evidence is present.
5. Add config nodes for approved key names needed to explain behavior.
   - `secret: true` may classify a key name, but no value/default may be captured for a secret.
6. Sort every file by `@id` and include root-relative evidence.

### 3.3 Verification

- [ ] All five files parse with `python3 -m json.tool`.
- [ ] Every service references one of the seven repository IDs.
- [ ] Every collection references a known datastore ID.
- [ ] Infrastructure records contain no account ID, credential, token, or private hostname unless the source explicitly classifies it as public and the design permits it; default is omission.
- [ ] Config records contain names only; no `.env` evidence path or secret value is present.
- [ ] IDs remain globally unique across all node files.
- [ ] The service inventory’s final count and any evidence-backed differences from the 20-unit seed are recorded.

### 3.4 Implementation Notes

_(to be filled during execution)_

## 4. Phase 4: Seed Current Topology Edges

**Commit message:** `feat(knowledge-graph): phase 4 — map current topology`
**Commit repository:** `misc`
**Prerequisite:** Phase 3 `COMPLETE`

### 4.0 Context

Connect roots, repositories, services, datastores, and infrastructure using current structural/runtime facts. Keep high-judgment provenance and migration relationships out of this phase so automated-looking topology cannot blur historical or planned architecture.

### 4.1 Files

1. `knowledgeGraph/graph/edges/containment.json`
2. `knowledgeGraph/graph/edges/runtime-dependencies.json`
3. `knowledgeGraph/graph/edges/data-access.json`
4. `knowledgeGraph/graph/edges/deployment.json`

### 4.2 Implementation Steps

1. Add containment edges for root→repository and repository→service relationships.
2. Add current `DEPENDS_ON`, `CALLS`, `EXPOSES`, `ROUTES_TO`, or `PROXIES_TO` edges only when current source or authoritative docs identify both endpoints.
   - Do not infer runtime calls solely from similar names.
   - Endpoint-level expansion is optional only when the exact route is already documented.
3. Add `READS_FROM`, `WRITES_TO`, and `USES_COLLECTION` edges for the verified datastore inventory.
4. Add `DEPLOYED_TO`, `DEPLOYED_AT`, `BUILT_WITH`, and `CONFIGURED_BY` edges needed for the current architecture.
   - Do not store deployment URLs containing private hosts or account identifiers.
5. Give non-obvious edges evidence and `status: current` even where the schema does not require the full provenance envelope.
6. Sort edges by `type`, `from`, and `to` for stable diffs.

### 4.3 Verification

- [ ] All four files parse with `python3 -m json.tool`.
- [ ] Every `from` and `to` ID resolves against seeded nodes by a temporary standard-library check.
- [ ] Every edge type appears in `schema/edge-types.json`.
- [ ] No historical port or planned migration is represented as a current runtime dependency.
- [ ] Datastore edges match current evidence and do not expose credentials or connection strings.
- [ ] Every repository and service has the expected containment path.

### 4.4 Implementation Notes

_(to be filled during execution)_

## 5. Phase 5: Seed Auth, Provenance, Migration, and Governance Edges

**Commit message:** `feat(knowledge-graph): phase 5 — capture lifecycle relationships`
**Commit repository:** `misc`
**Prerequisite:** Phase 4 `COMPLETE`

### 5.0 Context

Capture the relationships that make this graph more useful than a generated inventory: identity/authz boundaries, porting, vendoring, planned migrations, generated mirrors, and lifecycle governance. Every edge in this phase requires explicit evidence and normalized status.

### 5.1 Files

1. `knowledgeGraph/graph/edges/auth.json`
2. `knowledgeGraph/graph/edges/provenance.json`
3. `knowledgeGraph/graph/edges/migration.json`
4. `knowledgeGraph/graph/edges/governance.json`

### 5.2 Implementation Steps

1. Add auth edges using distinct meanings:
   - `AUTHENTICATES_VIA` for identity/session establishment.
   - `AUTHORIZES_VIA` for central permission decisions.
   - Normalize incomplete/migrating applications to `planned` unless current implementation evidence proves the relationship is active.
2. Add provenance edges from solution §3.4:
   - Board Lab `PORTS_FROM` WCCA POC as `historical`.
   - Current SDK/token/document copies as `VENDORS_FROM` only when evidence proves local vendoring.
   - Both live Devin roots `MIRRORS_FROM` MySkills as `current`.
3. Add both QMS destination statements as separate `planned` edges:
   - `MIGRATES_TO` `j9ccgit` from the QMS source document.
   - `PLANNED_TO_JOIN` `jemba9-ecosystem` from the ecosystem registry.
   - Do not choose a winner or convert either to current.
4. Add `GOVERNED_BY` edges only for repositories with evidence that MySkills or a named constitution governs their lifecycle artifacts.
5. Every edge must include `status`, `discoveryMethod: curated`, and at least one evidence item with root alias and relative path; line ranges are included when stable and known.
6. Sort edges deterministically.

### 5.3 Verification

- [ ] All four files parse with `python3 -m json.tool`.
- [ ] Every edge endpoint resolves against a known node.
- [ ] Every edge has allowed status, discovery method, and evidence.
- [ ] Evidence paths are relative and contain no `..`, `${HOME}`, or absolute prefix.
- [ ] Both QMS planned relationships exist and neither is `current`.
- [ ] Board Lab’s port relationship is `historical`, not a runtime dependency.
- [ ] Live Devin roots point to MySkills through `MIRRORS_FROM` and are not marked canonical.

### 5.4 Implementation Notes

_(to be filled during execution)_

## 6. Phase 6: Implement Shared Graph I/O and Additive Build Tooling

**Commit message:** `feat(knowledge-graph): phase 6 — implement additive extraction`
**Commit repository:** `misc`
**Prerequisite:** Phase 5 `COMPLETE`

### 6.0 Context

Build the common loading/path-safety layer and the additive extractor before validation and query tooling. Tests are written first against temporary roots so failures cannot alter curated workspace data.

### 6.1 Files

1. `knowledgeGraph/graph/scripts/graph_io.py`
2. `knowledgeGraph/graph/scripts/build_graph.py`
3. `knowledgeGraph/graph/tests/test_build_graph.py`

### 6.2 Implementation Steps

1. In `test_build_graph.py`, define isolated temporary-manifest tests for:
   - `${HOME}`-style root expansion through an injected home directory.
   - Unknown alias rejection.
   - Missing root reporting without deletion.
   - Symlink escape rejection.
   - `.env*`, `.git`, dependency, build, cache, generated, database, upload, archive, binary, and oversized-file exclusions.
   - Deep merge preserving curated scalar values, adding unique list values, and recursively enriching dictionaries.
   - `--dry-run` producing no file changes.
   - Derived output order and idempotence.
2. Implement `graph_io.py` with shared constants loaded from manifest/schema, graph-root discovery relative to the script, deterministic JSON load/write helpers, strict `${HOME}` expansion, realpath containment, node/edge loading, and adjacency helpers.
3. Implement `build_graph.py` with `argparse` options:
   - `--dry-run`
   - `--root <alias>` repeatable filter
   - `--manifest <path>` regular-file override
   - `--json` machine-readable report
4. Scan only declared enabled roots and only according to their scan policy.
   - Git roots: approved manifests/docs/source extensions and optional git metadata via `subprocess.run([...], shell=False)`.
   - Generated config roots: filename and drift metadata only.
   - Asset root: filename metadata only; no file-content reads.
5. Implement additive merge rules:
   - Missing scalar: add.
   - Existing curated scalar: retain and report conflict.
   - Lists: append unique values deterministically.
   - Dicts: recurse.
   - Missing roots: mark availability in derived output/report only; never delete curated nodes/edges.
6. Generate all six view payloads in memory, but write them only when not in `--dry-run` mode.
7. Route detection is report-only; do not automatically create high-judgment auth, provenance, migration, or governance edges.

### 6.3 Verification

- [ ] Run `python3 -m unittest knowledgeGraph.graph.tests.test_build_graph` from `misc`; all tests pass.
- [ ] `python3 knowledgeGraph/graph/scripts/build_graph.py --help` succeeds.
- [ ] `python3 knowledgeGraph/graph/scripts/build_graph.py --dry-run` completes without changing tracked graph files.
- [ ] Running dry-run twice produces equivalent reports apart from explicitly documented volatile metadata.
- [ ] Tests prove curated scalar preservation and missing-root non-deletion.
- [ ] Tests prove symlink escape and prohibited-path rejection.
- [ ] No third-party dependency was added.

### 6.4 Implementation Notes

_(to be filled during execution)_

## 7. Phase 7: Generate Derived Workspace Views

**Commit message:** `feat(knowledge-graph): phase 7 — generate workspace views`
**Commit repository:** `misc`
**Prerequisite:** Phase 6 `COMPLETE`

### 7.0 Context

Materialize deterministic query accelerators and human-review summaries from the curated graph. These six files are fully derived and are the only graph records the builder may replace unconditionally.

### 7.1 Files

1. `knowledgeGraph/graph/views/workspace-inventory.json`
2. `knowledgeGraph/graph/views/dependency-matrix.json`
3. `knowledgeGraph/graph/views/tech-stack-summary.json`
4. `knowledgeGraph/graph/views/datastore-map.json`
5. `knowledgeGraph/graph/views/provenance-map.json`
6. `knowledgeGraph/graph/views/migration-map.json`

### 7.2 Implementation Steps

1. Run `build_graph.py` without `--dry-run` to generate the six views.
2. Ensure each view declares its derivation metadata without embedding machine-specific absolute paths.
3. Ensure current dependency views exclude `planned`, `historical`, and `superseded` edges by default.
4. Ensure provenance and migration views retain lifecycle status and evidence references.
5. Run the builder a second time and confirm byte-stable output when source facts have not changed.
6. Review the diff to ensure only the six declared derived files changed in this phase.

### 7.3 Verification

- [ ] All six files parse with `python3 -m json.tool`.
- [ ] `workspace-inventory.json` reports 10 roots, 7 repositories, and 3 non-git roots.
- [ ] `dependency-matrix.json` includes current dependency relationships only.
- [ ] `provenance-map.json` includes Board Lab/WCCA and Devin/MySkills relationships with statuses.
- [ ] `migration-map.json` includes both planned QMS destination statements.
- [ ] A second build produces no diff in the six views.
- [ ] No view contains `/Users/michaelberry`, secret values, connection strings, or prohibited source content.

### 7.4 Implementation Notes

_(to be filled during execution)_

## 8. Phase 8: Implement Graph Validation and Security Checks

**Commit message:** `feat(knowledge-graph): phase 8 — enforce graph integrity`
**Commit repository:** `misc`
**Prerequisite:** Phase 7 `COMPLETE`

### 8.0 Context

Turn the solution’s trust requirements into a blocking validator. Test invalid fixtures first, then implement checks at the shared graph boundary rather than scattering defensive checks through query handlers.

### 8.1 Files

1. `knowledgeGraph/graph/scripts/validate_graph.py`
2. `knowledgeGraph/graph/tests/test_validate_graph.py`

### 8.2 Implementation Steps

1. Add fixture-based tests for every required failure:
   - Duplicate global node ID.
   - Dangling edge endpoint.
   - Unknown node/edge type.
   - Invalid edge endpoint types.
   - Missing status, discovery method, or evidence.
   - Unknown evidence root, missing evidence file on an available root, absolute evidence path, `..` traversal, and symlink escape.
   - Absolute username path in graph data.
   - Generated mirror marked canonical or missing `MIRRORS_FROM`.
   - Stale/tampered derived view.
   - Prohibited secret-like values, connection strings, bearer tokens, and private-key markers.
   - Allowed config key names such as `GRAPH_DATABASE_URL` when no value is present.
2. Implement `validate_graph.py` by loading schemas rather than hard-coding type lists.
3. Resolve evidence paths only under the manifest root and check existence only when that root is available.
4. Recompute derived views in memory through shared builder functions and compare canonical serialized payloads.
5. Produce concise Markdown/default output and `--json` output; use non-zero exit status on any violation.
6. Never print the full suspicious value in a validation error; identify file and JSON path while redacting content.

### 8.3 Verification

- [ ] `python3 -m unittest knowledgeGraph.graph.tests.test_validate_graph` passes.
- [ ] `python3 -m unittest discover -s knowledgeGraph/graph/tests -p 'test_*.py'` passes.
- [ ] `python3 knowledgeGraph/graph/scripts/validate_graph.py` exits 0 on the real graph.
- [ ] Each invalid fixture produces non-zero validation and its expected error category.
- [ ] Secret-value errors are redacted.
- [ ] `GRAPH_DATABASE_URL` as a key name is accepted while a connection-string value is rejected.
- [ ] Tampering with a temporary derived view is detected.

### 8.4 Implementation Notes

_(to be filled during execution)_

## 9. Phase 9: Implement Deterministic Graph Queries

**Commit message:** `feat(knowledge-graph): phase 9 — add deterministic queries`
**Commit repository:** `misc`
**Prerequisite:** Phase 8 `COMPLETE`

### 9.0 Context

Expose the validated graph through a predictable CLI without an LLM or graph database. Tests define supported grammar, status defaults, output shape, ambiguity handling, and bounded input before implementation.

### 9.1 Files

1. `knowledgeGraph/graph/scripts/query_graph.py`
2. `knowledgeGraph/graph/tests/test_query_graph.py`

### 9.2 Implementation Steps

1. Add tests for all solution §11.4 patterns:
   - `list roots`
   - `list services in <repo>`
   - `what depends on <id>`
   - `what does <id> depend on`
   - `show provenance for <id>`
   - `show planned migrations`
   - `what uses <store-id>`
   - `auth for <id>`
   - `governance for <id>`
   - `path between <id> and <id>`
   - `find <text>`
2. Add tests for Markdown default output, `--json`, unknown IDs, ambiguous text matches, unsupported grammar, empty results, and deterministic ordering.
3. Make `current` the default status for dependency/blast-radius questions.
   - `--status planned,historical,superseded` explicitly opts into other lifecycle states.
   - Provenance and migration commands select their semantically relevant statuses but always display status.
4. Bound query length and status-list size; parse fixed patterns as data.
5. Use adjacency maps and BFS from `graph_io.py` for path queries; do not add NetworkX.
6. Include evidence paths in provenance, migration, auth, and governance output without reading evidence-file contents during query execution.

### 9.3 Verification

- [ ] `python3 -m unittest knowledgeGraph.graph.tests.test_query_graph` passes.
- [ ] `python3 -m unittest discover -s knowledgeGraph/graph/tests -p 'test_*.py'` passes.
- [ ] `python3 knowledgeGraph/graph/scripts/validate_graph.py` exits 0.
- [ ] Each eleven supported query patterns returns correct Markdown against the real graph.
- [ ] Representative queries return valid JSON with `--json`.
- [ ] Current blast-radius queries exclude historical and planned edges by default.
- [ ] Overlong or unsupported input fails safely without shell execution or traceback leakage.
- [ ] Representative query execution remains under one second on the local graph.

### 9.4 Implementation Notes

_(to be filled during execution)_

## 10. Phase 10: Document Graph Operation and Maintenance

**Commit message:** `docs(knowledge-graph): phase 10 — document graph operations`
**Commit repository:** `misc`
**Prerequisite:** Phase 9 `COMPLETE`

### 10.0 Context

Document the supported workflow after behavior is implemented and verified. The README must distinguish curated source data from derived views, explain safe refresh/query commands, and make lifecycle statuses visible so users do not treat planned or historical edges as current.

### 10.1 Files

1. `knowledgeGraph/graph/README.md`

### 10.2 Implementation Steps

1. Document graph purpose, directory layout, root kinds, canonical-vs-mirror semantics, and stable ID rules.
2. Document build commands:
   - Safe preview with `--dry-run`.
   - Root-scoped refresh.
   - Full refresh.
   - Missing-root behavior and curated-conflict reports.
3. Document validation and all supported query patterns with examples.
4. Explain lifecycle status defaults and how to opt into planned/historical/superseded edges.
5. Document curation rules:
   - Evidence required for judgment-heavy edges.
   - Config names only; no values.
   - Derived views are never hand-edited.
   - Corrections to curated facts are reviewed manual edits.
6. Document scan exclusions, symlink safety, portability through `${HOME}`, and troubleshooting for unavailable roots.
7. State that visualization, CI, graph databases, vector search, AST indexing, and runtime discovery remain out of scope.

### 10.3 Verification

- [ ] Every documented command succeeds exactly as written.
- [ ] README examples cover dependency, provenance, migration, datastore, auth, governance, and root inventory queries.
- [ ] README clearly states current-only defaults for blast-radius queries.
- [ ] README contains no secret, private host, account ID, or machine-specific username path.
- [ ] `python3 -m unittest discover -s knowledgeGraph/graph/tests -p 'test_*.py'` passes.
- [ ] `python3 knowledgeGraph/graph/scripts/validate_graph.py` exits 0.
- [ ] `python3 knowledgeGraph/graph/scripts/build_graph.py --dry-run` reports no unexpected mutation.

### 10.4 Implementation Notes

_(to be filled during execution)_

## 11. Phase 11: Add Devin Query Skill and Run Final Verification

**Commit message:** `feat(workspace-knowledge-graph): phase 11 — add Devin query skill`
**Commit repository:** `MySkills`
**Prerequisite:** Phase 10 `COMPLETE`

### 11.0 Context

Expose the completed graph to Devin from the canonical MySkills master, install it through the existing synchronization workflow, and run the end-to-end acceptance suite. The live skill is generated configuration, not authored source, and requires a Devin CLI restart before discovery.

### 11.1 Files

1. `skills/workspace-knowledge-graph/SKILL.md` in `/Users/michaelberry/Documents/Consulting/MySkills`
2. `${HOME}/.config/devin/skills/workspace-knowledge-graph/SKILL.md` — generated by `/sync-skills install workspace-knowledge-graph`; not committed

### 11.2 Implementation Steps

1. Create the master skill with:
   - Directory and frontmatter `name: workspace-knowledge-graph` match.
   - Single-line description and natural-language argument hint.
   - Context pointing to the actual `misc/knowledgeGraph/graph/` path.
   - Read-only query as the default action.
   - Explicit refresh action that requires the user to request refresh.
   - Three to five representative query examples.
   - No `$ARGUMENTS` substitution assumption and no Windsurf-only anchors.
2. Invoke `/sync-skills check workspace-knowledge-graph` and confirm it is `MASTER ONLY` before installation.
3. Invoke `/sync-skills install workspace-knowledge-graph`.
   - If a live file unexpectedly exists and is newer, stop and ask before overwrite.
   - Do not sync any unrelated skill or constitution.
4. Confirm master and live files are identical.
5. Stage and commit only `skills/workspace-knowledge-graph/SKILL.md` in MySkills.
   - Leave the existing unrelated `pnpm-lock.yaml` and `skills/plan/PlanSkill.md` state untouched if it reappears after Phase 0.
6. From `misc`, run the complete graph unit, build dry-run, validator, and representative query suite.
7. Verify git status in both repositories contains no unplanned task changes and no scanned product repository changed.
8. Tell the operator to restart Devin CLI to load the installed skill; do not claim the skill was discovered in the current session unless a restart actually occurred.

### 11.3 Verification

- [ ] Master skill path and frontmatter name match `workspace-knowledge-graph`.
- [ ] Skill defaults to read-only query behavior and requires explicit refresh intent.
- [ ] Skill passes query text as data/arguments and does not construct a shell command string.
- [ ] `/sync-skills check workspace-knowledge-graph` reports master and live copies identical after installation.
- [ ] Only `skills/workspace-knowledge-graph/SKILL.md` is staged and committed in MySkills.
- [ ] `python3 -m unittest discover -s knowledgeGraph/graph/tests -p 'test_*.py'` passes from `misc`.
- [ ] `python3 knowledgeGraph/graph/scripts/build_graph.py --dry-run` succeeds without mutation.
- [ ] `python3 knowledgeGraph/graph/scripts/validate_graph.py` exits 0 with 10 roots, 7 repositories, and zero violations.
- [ ] The seven acceptance query classes return correct, status-labeled answers in Markdown and representative JSON output.
- [ ] Query response time remains under one second locally.
- [ ] No graph file contains `/Users/michaelberry`, a secret value, token, API key, private key, or database connection string.
- [ ] No product repository has task-related working-tree changes.
- [ ] The operator receives the required Devin CLI restart notice.

### 11.4 Implementation Notes

_(to be filled during execution)_

## Shared Component Reuse Summary

| Shared resource | Package/source | Used by phases |
|---|---|---|
| `argparse`, `json`, `pathlib`, `os` | Python standard library | 6, 8, 9 |
| `subprocess` with argument arrays | Python standard library | 6 |
| `collections` adjacency/BFS structures | Python standard library | 6, 7, 9 |
| `re` fixed grammar and redacted secret checks | Python standard library | 8, 9 |
| `tempfile`, `unittest` | Python standard library | 6, 8, 9 |
| `graph_io.py` shared loading/path layer | Knowledge graph tooling | 6–11 |
| Root and edge schemas | Knowledge graph data contract | 1–11 |
| Existing `/sync-skills` workflow | MySkills | 11 |

**No component, parser, traversal utility, schema constant, or synchronization workflow is duplicated when an earlier shared resource provides it.**

## Follow-Up Items (out of scope for this plan)

1. **Interactive visualizer** — Create a separate solution design only after the graph passes validation, supports the acceptance-query set, and completes one successful refresh cycle. A future visualizer must consume validated JSON and must not redefine graph semantics.
2. **Cross-repository CI automation** — Design a trusted trigger and write boundary before automatically refreshing or committing a graph hosted in a separate repository.
3. **AST-level source indexing** — Consider only if manifest/text extraction demonstrably cannot answer required questions; do not add language-specific parsers preemptively.
4. **Graph database or semantic-vector search** — Reconsider only if graph size or query requirements exceed the measured capability of JSON plus adjacency maps.
5. **Runtime service discovery** — Treat observed deployment topology as a separate operational-data design rather than mixing it with version-controlled architectural intent.

## Timeline Estimate

This constitution-required estimate is a planning aid, not a delivery commitment.

| Phase | Description | Effort | Cumulative |
|---|---|---:|---:|
| 0 | Clean-tree verification and two-repository branch setup | 0.25 days | 0.25 days |
| 1 | Root manifest and schemas | 0.5 days | 0.75 days |
| 2 | Root, repository, and asset seed | 0.5 days | 1.25 days |
| 3 | Service, data, infrastructure, and config seed | 0.75 days | 2.0 days |
| 4 | Current topology edges | 0.75 days | 2.75 days |
| 5 | Auth, provenance, migration, and governance edges | 0.75 days | 3.5 days |
| 6 | Shared I/O and additive builder with tests | 1.25 days | 4.75 days |
| 7 | Derived views | 0.5 days | 5.25 days |
| 8 | Validator and security tests | 1.25 days | 6.5 days |
| 9 | Query CLI and tests | 1.25 days | 7.75 days |
| 10 | Operator documentation | 0.5 days | 8.25 days |
| 11 | Devin skill, installation, and final verification | 0.75 days | 9.0 days |

## Success Criteria

- [ ] All ten configured roots are represented: seven git repositories and three correctly typed non-git roots.
- [ ] Every verified deployable unit and seed datastore is represented with evidence.
- [ ] Cross-root auth, provenance, migration, mirroring, and governance edges carry valid status, discovery method, and evidence.
- [ ] Both QMS destination statements remain distinct planned relationships until superseding evidence exists.
- [ ] Additive extraction preserves curated scalars, never deletes unavailable-root data, and regenerates only derived views.
- [ ] Root resolution rejects unknown aliases, traversal, and symlink escape.
- [ ] `.env*`, credentials, tokens, private keys, connection-string values, generated databases, uploads, archives, and binary contents are not ingested.
- [ ] `python3 -m unittest discover -s knowledgeGraph/graph/tests -p 'test_*.py'` passes with zero failures.
- [ ] `python3 knowledgeGraph/graph/scripts/validate_graph.py` exits 0 with zero violations.
- [ ] Dependency, provenance, migration, datastore, auth, governance, root-inventory, and path queries return correct status-labeled results.
- [ ] Current blast-radius queries exclude planned, historical, and superseded edges by default.
- [ ] Representative queries complete in under one second locally.
- [ ] No runtime dependency or external service is added.
- [ ] The MySkills master skill and live installed copy are identical, and the operator is told to restart Devin CLI.
- [ ] No application source file in any scanned product repository is modified.
- [ ] Each phase touches six or fewer files, records observed verification output, commits only task files, and leaves unrelated work untouched.
