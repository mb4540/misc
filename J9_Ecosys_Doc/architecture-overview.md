# Architecture overview

**Sources:** `README.md`, `Ecosystem/README.md`, `Ecosystem/CLAUDE.md`,
`Ecosystem/docs/architecture/j9-ecosystem-architecture.md`,
`Ecosystem/docs/architecture/platform_identity_access.md`,
`Applications/README.md`

## The concept

One front door at `jemba9.com`. AWS Cognito answers "who are you," a central
authorization service answers "what may you do," a portal launcher shows each
person only the apps they hold keys to, and every business app is a thin
**enforcement point** — it never owns login or permission rules, only the
`can()` decision the platform hands it.

> "authentication is decided in exactly one place (Cognito), authorization in
> exactly one place (the central authz service), and apps own only their
> business logic." — `j9-ecosystem-architecture.md` §1

## Repo layout (two-part monorepo)

```
J9Eco/                              (git: codecommit::us-east-2://Jemba9_Ecosystem)
├── Ecosystem/                      ← the platform. Run all platform commands from here.
│   ├── src/                        Next.js 16 App Router app
│   ├── prisma/                     schema + migrations (platform DB)
│   ├── infra/terraform/            Cognito only, today
│   ├── packages/j9-app-sdk/        the client every business app vendors
│   ├── docs/                       architecture, requirements, ADRs, deployment, notes
│   └── company.config.yaml         the only company-specific file
└── Applications/                   ← one folder per business app, each independent
    ├── OKR/                        migrating (Django + React, WorkOS being dropped)
    ├── dataroom/                   migrating, not yet integrated with the platform
    ├── boardlab/                   feature-complete; datasheet ingestion unwired
    └── _template/                  scaffold for new apps
```

Apps are physically separate from the platform: the platform's quality gates
run only inside `Ecosystem/`, and an app's build never touches the platform
repo tree. Apps talk to the platform **only over the network** (session,
authorization decisions, event ingest) — they never read platform database
tables directly. This is a deliberate reversal of an earlier "apps in their
own separate repos" design; see `Ecosystem/docs/notes/deviation_monorepo_apps.md`.

## Layers

```
Entry      jemba9.com (Route 53 → CloudFront front door; dev: CloudFront URL)
Identity   AWS Cognito — one user pool, OIDC code+PKCE, MFA
Portal     Launcher at / — tiles from live app_access grants
Apps       /admin (in this repo) · /qms · /okr · /dataroom (join later via contract)
Shared     Central authz service · platform events/telemetry · audit log
Data       Lakebase Postgres (standard-Postgres-only) · S3 for objects
```

Integration pattern: **path-based apps under one domain** — one cookie domain,
one session; CloudFront path behaviors route `/app/*` to each app's own
origin, so a broken app never takes the whole building down.

## How an app joins (the App Contract)

Any app joins with **zero core platform code changes**:

1. Accept the platform session — resolve identity via `GET /api/session`.
2. Ship a **permission manifest** (JSON) declaring its permission keys, default
   roles, and optional custom metrics — imported by an admin at
   Admin → Apps → Register.
3. Delegate every authorization decision to `POST /api/authz/decision` (or the
   session-baked fast path) — never invent local permission logic.
4. Own its own data on standard PostgreSQL — never read platform tables.
5. Emit usage/audit events via `POST /api/events` and expose `/healthz`.

Full spec: `Ecosystem/docs/architecture/app-manifest.md` (manifest shape,
register/re-sync semantics) and `Ecosystem/docs/requirements/functional-requirements.md`
FR-7 (the App Contract) / `app-onboarding-requirements.md` AR-1…AR-8 (the
per-app checklist each `Applications/<key>/CLAUDE.md` restates).

Registered apps today: **none in production.** The Admin app itself is the
only "app" seeded (`company.config.yaml` `apps.enabled: ["admin"]`). Board Lab,
Data Room, and OKR are built or being built but have not yet gone through
manifest registration against a live platform instance — see
`deployment-pipeline.md` and `authentication-authorization.md` for what that
implies.

## Roadmap as originally scoped

Sprints 00–10 (`Ecosystem/prompts/`) built: scaffold → schema → auth → authz →
portal → admin (users, grants, registry, audit, dashboards) → hardening +
deploy docs. All ten are marked done in `Ecosystem/docs/run_notes.md`. What
comes next per that roadmap: migrate Data Room, QMS, and OKR onto the platform,
each via a manifest registration plus an enforcement-point refactor in its own
app folder.
