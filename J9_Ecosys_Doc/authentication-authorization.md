# Authentication & Authorization (focus area)

**Sources:** `Ecosystem/docs/architecture/platform_identity_access.md`,
`Ecosystem/docs/architecture/j9-ecosystem-architecture.md` §4–§5,
`Ecosystem/docs/requirements/functional-requirements.md` FR-1/FR-2/FR-7,
`Ecosystem/docs/notes/security-baseline.md`, `Ecosystem/docs/run_notes.md`
(Sprints 02–03), `Ecosystem/infra/terraform/README.md`,
`Ecosystem/company.config.yaml`, `Applications/boardlab/CLAUDE.md`
("Non-negotiables AR-1…AR-8"), `Applications/*/CLAUDE.md`.

The single most important thing to understand about this area: **the design
is fully specified and the platform-side implementation is fully built and
tested — but none of it is running against real AWS Cognito, and no business
app has actually wired itself up to it yet.** Everything below separates
"designed," "implemented," and "actually running" on purpose.

## 1. The split: identity vs. authorization

| Question | Who answers | Where |
|---|---|---|
| Who are you? (authentication) | **AWS Cognito** — one user pool for the whole ecosystem | `Ecosystem/infra/terraform/` (Cognito), `Ecosystem/src/lib/auth/` |
| What may you do, everywhere? (authorization) | **Central authz service** — one Postgres-backed catalog of permissions/roles/grants | `Ecosystem/src/lib/authz/`, `Ecosystem/prisma/schema.prisma` |

Cognito tokens are deliberately kept lean (`sub`, email, name, MFA flag) —
**no permission ever rides in a JWT.** Permissions are computed server-side by
the authz service and delivered to apps via a session-baked fast path (default)
or a live decision call for sensitive actions. This split is locked platform
policy (`platform_identity_access.md` — "Direction (locked)").

## 2. Identity — designed vs. actual

**Designed (production target):**
- One Cognito User Pool, OIDC Authorization Code + PKCE, custom J9-branded
  login page (not the Cognito Hosted UI, which can't meet the brand bar).
- MFA (TOTP) required for all internal accounts and any admin-class account in
  any app.
- Federation-ready: external corporate IdPs (Google, Entra, Okta, SAML/OIDC)
  can federate into Cognito later without app changes.
- Every app maps the Cognito `sub` to its own thin local profile on first
  login (JIT provisioning) — no password/MFA-secret column anywhere outside
  Cognito.

**Actually implemented and running today (Sprint 02, `[sprint-02-auth]`):**
- A **provider abstraction** (`src/lib/auth/`) with two interchangeable
  backends: a real local **mock OIDC issuer** (RS256 keypair, JWKS, TOTP
  challenges) used for dev/CI, and a **Cognito** implementation
  (`provider/cognito.ts`) that is code-complete but only loads when
  `AUTH_PROVIDER=cognito` is set.
- **The mock issuer is the working backend right now.** Per
  `Ecosystem/docs/run_notes.md` (Sprint 02 deviations): *"Local mock OIDC issuer
  is the working backend; Cognito is apply-ready but not applied... left as an
  explicit user decision because it creates real, billable cloud
  infrastructure."*
- Terraform for the Cognito user pool + PKCE app client exists
  (`Ecosystem/infra/terraform/main.tf`) and is described in its own README as
  **"apply-ready but not yet applied."** `terraform apply` has never been run
  against AWS account `573631993254`.
- Switching from mock to Cognito is designed to be **env-only** — no app code
  change, just `AUTH_PROVIDER=cognito` + the pool/client IDs from the Terraform
  outputs — but that switch has not happened.

**Practical implication:** anyone testing or building against "the platform's
auth" today is testing against the mock issuer, not AWS Cognito. That's fine
for development, but treat any claim of "Cognito login works" as **unverified
in production** until someone runs `terraform apply` and flips the env var.

## 3. Authorization — designed vs. actual

**Designed (production target, `platform_identity_access.md` §4):**
- One `platform` Postgres schema is the only permission store in the
  ecosystem — no app keeps its own permission table.
- Core model: `organization`, `identity`, `app`, `permission`, `role`,
  `role_permission`, `app_access`, `role_grant`, `permission_grant`.
- **Default deny** everywhere. Effective permissions = union of role grants +
  additive permission grants − deny grants, always scoped by `org_id`.
- **Append-only.** Grants are revoked via `revoked_at`, never hard-deleted —
  full audit history survives.
- Decision delivery is **hybrid**: session-baked (fast, short TTL) for normal
  actions; a live `POST /api/authz/decision` call for permissions marked
  `sensitive` (e.g. admin write actions, document export), so a just-revoked
  grant takes effect immediately rather than waiting out the session TTL.

**Actually implemented and running today (Sprint 03, `[sprint-03-authz]`,
and hardened through Sprint 10):**
- `Ecosystem/src/lib/authz/`: `algebra.ts` (pure merge: roles ∪ allows − denies,
  deny wins), `effective.ts` (`getEffectivePermissions`, `can()`,
  `getEntitlements()`), `cache.ts` (60s in-process cache + Redis bake-version
  invalidation), `bake.ts` (`requirePermission(key)` — sensitive keys always go
  live), `grants.ts` (grant/revoke, audit + event on every write), `sweep.ts`
  (BullMQ job revoking expired grants every `grant_expiry_sweep_minutes`, 15 by
  default from `company.config.yaml`).
- Decision endpoint `POST /api/authz/decision` — session or per-app API key
  (`Authorization: Bearer`) auth, backed by the 60s cache, benchmarked
  (`npm run bench:authz`) at p95 ≈ 0.001ms warm / ≈ 1.7ms cold.
- Row-level security (RLS) on every tenant-scoped table
  (`app_access`, `role_grant`, `permission_grant`, `platform_event`), enforced
  against a **non-superuser** `j9_app` runtime role — the owner/superuser
  connection intentionally bypasses RLS and is used only for migrations/seed.
- This part of the stack has real integration-test coverage: deny-wins, org
  scoping, expiry sweep idempotency, immediate revocation for sensitive keys,
  fail-closed on query error, RLS cross-org isolation.

**Practical implication:** unlike identity, the authorization engine is not a
stub — it's a real, tested, running subsystem against local Postgres/Redis.
The gap here isn't "not built," it's "not yet exercised by any real business
app" (see §4).

## 4. The App Contract — what every business app must do, and where each stands

Every app in `Applications/` is required (by its own `CLAUDE.md`, mirroring
the platform's `app-onboarding-requirements.md` AR-1…AR-8) to:

1. Have **no auth of its own** — identity comes from the platform via
   `GET /api/session`.
2. **Delegate every decision server-side** via `can()` — UI hiding is courtesy
   only.
3. **Fail closed** — any non-allow (unreachable, timeout, malformed) is
   denied.
4. Use **standard PostgreSQL only**, own its own schema, `org_id` + RLS on
   every tenant-scoped table.
5. Declare permissions as **manifest data**, not code.
6. Emit audit/usage events and expose `/healthz`.
7. Hold **one secret** — its `J9_APP_API_KEY` — server-side only.

Status per app (`Applications/README.md`):

| App | Status | Auth/authz integration |
|---|---|---|
| Board Lab | Feature-complete; datasheet ingestion unwired | Its own `CLAUDE.md` documents the App Contract as binding, but Board Lab currently runs its own local session/authz plumbing for dev — it has not gone through manifest registration against a live platform instance. |
| Data Room | Migrating, **"not yet integrated"** per `Applications/README.md` | No platform session/authz wiring yet. |
| OKR | Migrating | Currently on WorkOS + django-allauth (its own ADR-004); the identity doc explicitly supersedes that ADR and calls for a full rewrite onto Cognito + the central authz service — not yet done. |
| QMS | Planned | Not started; today has a homegrown scrypt/TOTP auth stack that the identity doc says must be fully removed and replaced. |

So the honest end-to-end state is: **the platform can authenticate and
authorize, but no business app currently asks it to.** Each app still owns
some form of its own auth/session logic in the interim.

## 5. Security posture already verified (Sprint 10 baseline)

From `Ecosystem/docs/notes/security-baseline.md` (reviewed 2026-07-14), already
true of the platform code as it stands, independent of the Cognito-vs-mock
question:

- CSP, X-Content-Type-Options, X-Frame-Options, Referrer-Policy,
  Permissions-Policy, and HSTS are applied to all routes.
- Sessions are opaque, httpOnly, `SameSite=Lax` (Secure in prod), stored
  server-side in Redis with real revocation (`revokeAllSessions`) — not a JWT
  the app trusts blindly.
- Tokens are validated against JWKS (issuer/audience/expiry/signature) for
  both the mock and Cognito providers identically.
- Progressive lockout on login/MFA; rate limiting on event ingest.
- Zod validation at every external boundary.
- Audit log and event stream reject UPDATE/DELETE at the database level via
  triggers; only the sanctioned retention sweep can prune events (never the
  audit log).
- **Open item, owner "platform":** CSP still uses `'unsafe-inline'` for
  scripts/styles because of Next App Router's inline bootstrap; a nonce +
  `strict-dynamic` upgrade is tracked but not done.

## 6. What would need to happen to make this real

1. Decide to spend the money: `terraform apply` the Cognito module in
   `Ecosystem/infra/terraform/` against account `573631993254`.
2. Set `AUTH_PROVIDER=cognito` + the pool/client IDs from the Terraform output
   — no app code change required.
3. Pick one app (Board Lab is closest, per its own status) and actually run it
   through manifest registration against a live platform instance, then
   refactor its local session checks into calls against `GET /api/session` /
   `can()`.
4. Resolve the DNS blocker (see `deployment-pipeline.md` §4) so Cognito
   callback URLs and the session cookie domain can point at `jemba9.com`
   instead of a CloudFront dev URL.
