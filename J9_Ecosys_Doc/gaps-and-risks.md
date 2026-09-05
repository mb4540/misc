# Gaps, contradictions, and open decisions

Consolidated from `deployment-pipeline.md` and `authentication-authorization.md`.
Ordered roughly by how much a real decision is blocked on each.

## 1. CI/CD host mismatch — unresolved (deployment)

- The locked architecture decision and `docs/deployment/aws.md` both say:
  git host = GitHub, CI/CD = GitHub Actions with OIDC, *because* "AWS
  CodeCommit stopped accepting new customers in July 2024."
- The repo's actual git remote is **AWS CodeCommit**
  (`codecommit::us-east-2://Jemba9_Ecosystem`), adopted via a dated,
  owner-approved deviation (`docs/notes/deviation_codecommit.md`) that itself
  says: *"Confirm the CI wiring before relying on it"* — and nobody has.
- `.github/workflows/ci.yml` exists and looks correct, but GitHub Actions does
  not execute against a CodeCommit-hosted repo without a mirror/bridge, and
  none is configured. It's unclear whether this CI has ever actually run in
  the last several sprints, or whether "gates pass" claims in `run_notes.md`
  were verified by running these commands locally rather than by CI.
- `company.config.yaml` still declares `hosting.ci: "github-actions-oidc"`.
- **Needs a decision:** either (a) move/mirror the repo to GitHub and keep the
  GitHub Actions design, or (b) commit to CodePipeline/CodeBuild reading from
  CodeCommit directly and rewrite the deployment docs to match, or (c) set up
  a CodeCommit→GitHub mirror so both the existing `ci.yml` and the CodeCommit
  git workflow keep working.

## 2. No AWS infrastructure has actually been provisioned

- Terraform exists only for Cognito, and its own README says "apply-ready but
  not yet applied."
- The five additional modules `aws.md` calls for (network, RDS, secrets, ECS
  compute, CloudFront) do not exist in `infra/terraform/` at all — only the
  ADR recommending ECS Fargate (0002) has been written.
- Terraform state is local and gitignored; the S3+DynamoDB backend is only a
  stub in `main.tf`, not migrated to.
- **Consequence:** there is no staging or production environment anywhere.
  Everything that has been "verified" (Sprints 00–10 acceptance criteria) was
  verified against local Docker Postgres/Redis + the mock OIDC issuer.

## 3. Auth is running on a mock identity provider, not Cognito

- `AUTH_PROVIDER=cognito` has never been set outside of it being "apply-ready."
- Anyone reasoning about "how login works in production" should read Sprint 02
  acceptance criteria with that caveat in mind — the *design* and the *code
  path* for Cognito exist and are meant to be a pure env-var flip, but that
  flip is unverified against a real Cognito pool.

## 4. No business app has completed the App Contract

- Central authz is fully implemented and tested — but zero business apps
  (Board Lab, Data Room, OKR, QMS) have actually registered a manifest against
  a live platform instance or refactored their own session logic to call
  `GET /api/session` / `can()`.
- Board Lab, Data Room, and OKR each still run some form of their own
  local/independent session or auth handling in the interim (Data Room
  explicitly flagged "not yet integrated" in `Applications/README.md`; OKR is
  still on WorkOS/django-allauth pending the Cognito rewrite the identity doc
  mandates).
- This means "the platform enforces access for the ecosystem" is true of the
  Admin app only, not of any of the actual business apps yet.

## 5. `Applications/dataroom`'s existing deploy workflows look like leftover boilerplate

- `deploy.yml` / `deploy-staging.yml` target `cloudcowboy.us` (S3 + CloudFront)
  with Supabase secrets — unrelated to Jemba9's Cognito/Lakebase/ECS design.
- Risk: if someone runs these thinking they deploy the Jemba9 Data Room, they
  will push to the wrong AWS account/bucket/domain, or fail outright on
  missing secrets. Worth deleting or clearly marking as stale if not actively
  maintained.

## 6. DNS blocker (lower urgency, but blocks public launch)

- `jemba9.com`'s Route 53 zone lives in a different AWS account than the
  deploy account (`573631993254`). Two unblock paths are documented
  (`docs/deployment/dns.md`): get zone access, or re-delegate the registrar to
  a new zone in the deploy account. Neither has been actioned. Interim
  operation at a `*.cloudfront.net` URL is explicitly fine per that doc, so
  this only blocks the *vanity domain*, not functionality.

## 7. CSP `'unsafe-inline'` (minor, tracked, not urgent)

- `docs/notes/security-baseline.md` already tracks this as an open item owned
  by "platform" — a nonce + `strict-dynamic` upgrade before public launch.
  Flagged here only for completeness; not a new finding.

## Suggested next step

Given how much of the deployment story hinges on item 1, that's the decision
worth forcing first: **pick one CI/CD host and make the docs, `ci.yml`, and
`company.config.yaml` agree with it** before writing any of the missing
Terraform modules. Everything else (applying Cognito, standing up ECS,
onboarding the first real app) is downstream of knowing where "push to
production" actually pushes to.
