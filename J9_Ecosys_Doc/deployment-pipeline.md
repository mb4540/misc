# Deployment pipeline (focus area)

**Sources:** `README.md`, `Ecosystem/docs/architecture/j9-ecosystem-architecture.md`
§3/§10, `Ecosystem/docs/deployment/{aws,dns,replication}.md`,
`Ecosystem/docs/adr/0002-compute.md`, `Ecosystem/docs/notes/deviation_codecommit.md`,
`Ecosystem/company.config.yaml`, `Ecosystem/infra/terraform/`,
`.github/workflows/ci.yml`, `Applications/dataroom/.github/workflows/`,
`Applications/boardlab/CLAUDE.md`, live `git remote -v` / `git log` on the repo.

Headline finding: **there is no production deployment today.** Nothing has
been provisioned in AWS beyond, at most, an unapplied Terraform plan for
Cognito. What exists is (a) a working local/CI dev loop, (b) a written design
for a future AWS deploy, and (c) a documented, self-acknowledged conflict
between the git host the design assumes and the git host actually in use.

## 1. Source control — what's actually in use

```
$ git remote -v
origin  codecommit::us-east-2://Jemba9_Ecosystem   (fetch/push)
```

The whole monorepo (`Ecosystem/` + `Applications/*`) lives in **AWS
CodeCommit**, repo `Jemba9_Ecosystem`, account `824999955649` (the `onej9` SSO
org), region `us-east-2`. Branches in active use: `main`, `feat/board-lab`,
`feat/component-master`, `okr-app`, `audit/2026-08-07-hardening-and-perf`.
Commits are tagged by sprint, e.g. `[sprint-10-hardening] …`. Pull requests, if
used, go through `aws codecommit create-pull-request` or the console — there
is no GitHub remote to open a `gh pr` against.

This is a **documented deviation**
(`Ecosystem/docs/notes/deviation_codecommit.md`, owner-approved 2026-07-15,
status "adopted"), because the locked platform decision it overrides
explicitly says the opposite (see §3 below).

## 2. CI — what actually runs

- **`Ecosystem/.github/workflows/ci.yml`** (repo-root `.github/`, i.e.
  `J9Eco/.github/workflows/ci.yml`): on every push/PR, two jobs —
  `build` (install → typecheck → lint → format:check → unit test → build) and
  `integration` (spins up a Postgres 16 service container, provisions the
  shadow DB + least-privilege `j9_app` role, migrates, seeds, runs integration
  tests). Node 20. **No deploy job.**
- The file's own header comment says: *"Deploy-to-AWS via GitHub Actions OIDC
  is added in Sprint 10."* It was not — Sprint 10's actual deliverables
  (`docs/run_notes.md`) were settings/hardening/security-headers/docs, not a
  deploy job; no deploy step exists in `ci.yml` today.
- **This is a GitHub Actions file living in a repo whose remote is
  CodeCommit.** GitHub Actions do not run against a CodeCommit-hosted repo
  without an explicit mirror/bridge, and no such mirror is configured anywhere
  in this repo. Unless there's a GitHub mirror set up outside this checkout,
  `ci.yml` is not currently executing anywhere. Sprint 00's run notes even flag
  this as a known caveat: *"GitHub only runs workflows from the repo root
  `.github/`. Today J9Ecosystem is a subfolder of the `Jemba9` monorepo with no
  remote; the workflow is authored assuming this project becomes its own
  repo."* That assumption never resolved.
- `Applications/dataroom/.github/workflows/{deploy,deploy-staging}.yml` **do**
  exist and target GitHub Actions OIDC → AWS, but they deploy to an S3 bucket
  named `cloudcowboy-us-prod-site` / CloudFront distribution for a domain
  `cloudcowboy.us`, with Supabase env vars (`VITE_SUPABASE_URL`, etc.) baked in
  as build-time secrets. That is **not a Jemba9 target** — it reads like
  leftover boilerplate from a different project copied into the app folder,
  not a working Data Room pipeline. Do not treat it as a reference for how
  Jemba9 apps deploy.
- `Applications/boardlab/` has **no** `.github/workflows/`, no
  `deploy-constitution.md`, and no CI/CD config of any kind.

## 3. The locked design vs. the deviation (the core contradiction)

The architecture doc's decision table is explicit and still reads this way
today:

> "Source control / CI | **GitHub + GitHub Actions** (OIDC role-assumption
> into AWS) | **AWS CodeCommit stopped accepting new customers in July 2024**
> — it is not a viable target." — `j9-ecosystem-architecture.md` §3

> "Do not target AWS CodeCommit — it stopped accepting new customers in July
> 2024. CI/CD uses GitHub Actions with OIDC into AWS." —
> `docs/deployment/aws.md` (top of file, called out as a warning)

But the actual git host adopted a month later (`deviation_codecommit.md`,
2026-07-15) **is** CodeCommit — reachable because the *repo* account
(`824999955649`) has CodeCommit access, even though the original blocker (the
platform's *deploy* account, `573631993254`, has zero CodeCommit repos and
cannot create them) is still true and unrelated. The deviation note's own
"Consequences / follow-ups" section admits this was never fully resolved:

> "Cross-account CI/CD. Code lives in CodeCommit in `824999955649`; the
> platform deploys into `573631993254`... A pipeline (CodePipeline/CodeBuild)
> must be granted cross-account access to the CodeCommit source, or a mirror
> set up. **Confirm the CI wiring before relying on it.**"

Nobody has confirmed that wiring. `company.config.yaml` still declares
`hosting.ci: "github-actions-oidc"`, and the deployment docs still describe a
GitHub Actions pipeline, while the only git host actually in use is
CodeCommit. This is the single biggest open question in the deployment story
— see `gaps-and-risks.md`.

## 4. The planned AWS deploy path (design only — nothing applied)

`Ecosystem/docs/deployment/aws.md`, `dns.md`, `replication.md`, and
`docs/adr/0002-compute.md` lay out a coherent target design. None of it has
been built in AWS yet beyond an unapplied Cognito Terraform plan.

**Target topology** (account `573631993254`, region `us-west-2` per
`company.config.yaml`):

```
Route 53 (jemba9.com — different AWS account, not yet accessible)
   │ ALIAS
CloudFront distribution (single front door; ACM cert in us-east-1)
   ├─ /                → platform origin (Ecosystem)
   ├─ /admin/*         → platform origin
   └─ /okr/* /qms/* /dataroom/* /boardlab/*  → each app's own origin (join later)
Origins: ECS Fargate services behind an internal ALB
Data:    RDS/Aurora PostgreSQL (migrated from Lakebase) · ElastiCache Redis · S3
Secrets: SSM Parameter Store / Secrets Manager
```

**Compute choice (ADR 0002, accepted):** AWS **ECS Fargate** behind an
internal ALB, chosen over App Runner and Amplify Hosting specifically because
the platform ships two long-running processes — the Next.js app and a BullMQ
**worker** (grant-expiry sweep, health polling, rollups, retention) — that
Fargate runs as sibling services cleanly.

**Planned deploy job** (never implemented): on push to `main`, after CI —
`configure-aws-credentials` via OIDC (no long-lived keys) → build + push a
Docker image to ECR (Next `output: 'standalone'`) → `prisma migrate deploy`
against RDS from a one-off task → `aws ecs update-service --force-new-deployment`
→ smoke-test `/healthz` behind CloudFront.

**Terraform reality check:** `Ecosystem/infra/terraform/` today contains
*only* the Cognito module (`main.tf`, `variables.tf`, `outputs.tf`), documented
in its own README as **"apply-ready but not yet applied"** — deliberately left
as a human go/no-go because it provisions real, billable AWS resources. The
`aws.md` plan calls for five more modules (network, data/RDS, secrets,
compute/ECS, cdn/CloudFront) plus an S3+DynamoDB Terraform state backend
migration (currently local, gitignored state) — **none of these exist yet**,
only the plan to add them.

**DNS is a tracked, separate blocker**: the `jemba9.com` Route 53 zone lives
in a different AWS account than the deploy account. Until that's resolved
(either get zone access to add an ALIAS record, or re-delegate the registrar
to a new zone in `573631993254`), the platform can only ever be reached at a
`*.cloudfront.net` URL — which is fine for interim operation but blocks public
launch (`docs/deployment/dns.md`).

## 5. Replication playbook (as designed, for a *new* deployment)

`docs/deployment/replication.md` describes standing up a fresh, differently
-branded deployment "in under a day": copy the repo → fill
`company.config.yaml` → `terraform apply` (Cognito + the not-yet-written
network/RDS/CloudFront/secrets/CI modules) → migrate + seed → push to `main`
so the (not-yet-written) GitHub Actions OIDC pipeline builds/deploys → smoke
test → attach DNS. This is aspirational until the missing Terraform modules
and the CodeCommit/GitHub CI question are resolved — it has never actually
been executed end to end.

## 6. Per-app deployment status

| App | Deploy pipeline | Notes |
|---|---|---|
| Ecosystem (platform) | None applied. CI-only (`ci.yml`), no deploy job, unclear whether CI even runs given the CodeCommit remote. | See §2–§3. |
| Board Lab | None. No `.github/workflows/`, no `deploy-constitution.md`. Runs locally via `StartBoardLab.sh` → `docker compose` (Postgres) → `prisma migrate deploy` → `npm run seed` → `next dev -p 3002`. | Its own `CLAUDE.md` documents local dev commands only; production deploy is undefined. |
| Data Room | Has `.github/workflows/deploy.yml` / `deploy-staging.yml`, but they target an unrelated project (`cloudcowboy.us`, Supabase) — not usable as-is for Jemba9. | Treat as leftover template content, not a real pipeline. |
| OKR | Not investigated in depth here (out of scope of this pass) — no evidence of a working deploy pipeline was found. | — |

## 7. Practical takeaway

If you need to "commit and push to production" for any app in this
ecosystem today, there is no such command — because there is no production
environment to push to, and the CI/CD story that would eventually build one is
internally contradictory (CodeCommit repo, GitHub-Actions-shaped design). The
actionable next step is a decision, not a command: pick GitHub Actions (and
either move the repo or bridge CodeCommit → GitHub) or commit to
CodePipeline/CodeBuild against CodeCommit directly, then write the Terraform
modules `aws.md` describes and apply them. See `gaps-and-risks.md` item 1.
