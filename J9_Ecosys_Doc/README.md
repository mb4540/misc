# Jemba9 Ecosystem — Documentation Snapshot

**Generated:** 2026-08-11 · **Method:** `Consulting/MySkills/.devin/workflows/document.md`
applied as a point-in-time audit (not an incremental diff — there was no code
change driving this; it's a full-repo read of `Jemba9_Ecosystem` with focused
depth on the deployment pipeline and authentication/authorization).

**Source repo:** `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/Jemba9_Ecosystem`
(`J9Eco`) — git remote `codecommit::us-east-2://Jemba9_Ecosystem`, branch `main`
(platform work also tracked on `feat/board-lab`, `feat/component-master`).

This snapshot exists **outside** the target repo (`misc/J9_Ecosys_Doc/`) rather
than under `Ecosystem/docs/`, so it does not become a second source of truth the
team has to keep in sync. Treat every claim below as traceable back to the cited
file path in the repo — if the repo changes, re-run the audit rather than editing
this copy by hand.

## Contents

| Doc | Covers |
|---|---|
| [`architecture-overview.md`](architecture-overview.md) | What the ecosystem is, the repo layout, the two-repo relationship (platform vs. business apps), and how apps join |
| [`authentication-authorization.md`](authentication-authorization.md) | **Focus area.** Identity (Cognito) vs. authorization (central `can()` service) — designed model, what's actually implemented and running today, and the App Contract every business app must satisfy |
| [`deployment-pipeline.md`](deployment-pipeline.md) | **Focus area.** Git hosting, CI, the planned AWS deploy path (Terraform/ECS/CloudFront), what has and hasn't actually been applied, and per-app deploy status (Board Lab, Data Room, OKR) |
| [`gaps-and-risks.md`](gaps-and-risks.md) | Consolidated list of contradictions, unresolved decisions, and blockers found while producing this doc — the part most worth a human decision |

## One-paragraph summary

The Jemba9 Ecosystem is a monorepo (`J9Eco`) containing a platform
(`Ecosystem/`, Next.js 16) that provides single sign-on (AWS Cognito) and
central, default-deny authorization for a family of business apps
(`Applications/OKR`, `Applications/dataroom`, `Applications/boardlab`, future
QMS). The platform's identity/authz/admin functionality (Sprints 00–10) is
**fully built and tested locally** — but it runs today against a **local mock
OIDC issuer**, not real Cognito, and **no infrastructure has actually been
provisioned or deployed to AWS**: Terraform for Cognito is "apply-ready but not
applied," there is no ECS/CloudFront/RDS Terraform at all yet (only an ADR
recommending it), and the repo's CI workflows are GitHub Actions files sitting
in a repo whose actual git remote is AWS CodeCommit — a mismatch the project's
own deviation notes flag as unresolved. See `gaps-and-risks.md` for the full list.
