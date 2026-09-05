# Upgrade Notes — Port the RoadmapAlternate Mock into the Real Roadmap App

**Audience:** an engineer/agent with zero prior context on this work.
**Goal:** upgrade the real app at `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/Jemba9_QMS/app/roadmap` so it matches the IA, visual design, and features demonstrated in this directory's static mock (`index.html`, `backlog.html`, `architecture.html`, `readiness.html`, `forecast.html`, `runbooks.html`).

**Read this whole document before writing code.** Several of the "obvious" implementation choices you'd reach for by default are wrong for this codebase — there is already infrastructure in place (a shared design-token system with a working dark theme, an existing `ThemeContext` pattern used by four sibling apps, and existing logo assets) that must be reused, not reinvented. Sections C and D below exist specifically to stop you from redoing work that's already done elsewhere in this monorepo family.

---

## 0. What the mock is, and what it is not

The mock is a **static, inline-style, zero-JavaScript HTML prototype**. Those two constraints (no JS, prefer inline style) were requirements specific to *building the mock quickly as a throwaway visual reference* — they are **not** requirements for the real app. The real app is a React 19 + Vite + react-router-dom SPA with an existing CSS design-token system. Do not:

- Port the mock's inline `style="..."` attributes verbatim into JSX. Use the real app's existing semantic CSS classes (`badge-impl`, `alert-warning`, `stat-card`, etc. — full list in §B) wherever an equivalent already exists.
- Port the mock's CSS-only checkbox/`:has()` dark-mode toggle. The real app has React state and an established `ThemeContext` pattern used by 4 other apps in this project family — use that instead (§C).
- Treat the mock's specific data (which backlog items are Done/In Progress/Missing, specific dates) as ground truth forever. It is a snapshot from **2026-07-19**. Before you copy any status, re-derive it from the real source of truth (§G) — it has almost certainly changed since this mock was built.

Use the mock for: information architecture (which pages exist and what each contains), visual layout/hierarchy, the specific copy/wording for sections that are still accurate, the new architecture diagram, and the general "how much should be on one page" judgment calls it embodies.

---

## A. Information architecture: 9 documents → 4

### A.1 Current state (`src/Shell.tsx`, `src/main.tsx`)

`NAV_ITEMS` in `Shell.tsx` currently renders 10 items (00–09 + the Forecast arrow item), each routed in `main.tsx` to a `<StaticPage html={pXX} />` backed by a template-literal HTML string in `src/pages/pXX-*.ts`:

| # | Route | File | 
|---|---|---|
| 00 | `/` | `p00-overview.ts` |
| 01 | `/01` | `p01-executiveSummary.ts` |
| 02 | `/02` | `p02-backlog.ts` |
| 03 | `/03` | `p03-roadmap.ts` |
| 04 | `/04` | `p04-architecture.ts` |
| 05 | `/05` | `p05-observability.ts` |
| 06 | `/06` | `p06-opsSupport.ts` |
| 07 | `/07` | `p07-csoReview.ts` |
| 08 | `/08` | `p08-top25.ts` |
| 09 | `/09` | `p09-gantt.ts` (+ `roadmap-gantt.css.ts`) |

The problem this upgrade fixes: the same P0 backlog status (e.g. "secrets rotation done", "tenant boundary tests done", "DNS/TLS missing") gets **independently re-explained in prose on 5–6 of these pages**, and they drift out of sync with each other (this was audited and partially fixed once already this session — see the conversation history / git blame on these files for the specific corrections made on 2026-07-19).

### A.2 Target state: 4 documents

Collapse to 4 routes. Suggested mapping (content sourcing, not literal copy — see §0):

| New page | Route | Replaces | Content sourced from |
|---|---|---|---|
| **Overview & Verdict** | `/` | `p00` + summary parts of `p01` + the LBGUPS scorecard (dedup'd out of `p07`) | Mock: `index.html`. One-paragraph verdict (as a bullet list — see recent edit in this conversation), priority stat cards, **one** LBGUPS scorecard, a "Phase 0 snapshot" linking into the Backlog page instead of restating status. |
| **Backlog & Timeline** | `/01` | `p02` + `p03` (phase groupings) + `p09` (gantt) + `p08` (top-25 ranking) | Mock: `backlog.html`. This becomes the **single source of truth** for every backlog item's status/evidence/effort/dependencies. Every other page must link to `#p0-02` style anchors here instead of restating status. Keep the CSS-timeline-bar concept (phase-colored proportional bar + a "Today" marker) — recompute the Today marker position from the actual current date at implementation time, don't hardcode 07/19. |
| **Architecture & Observability** | `/02` | `p04` + `p05`, trimmed of restated backlog status | Mock: `architecture.html`. Keep the architecture diagram (§E), auth/identity model, security posture table, full observability section (OTel plan, metrics table, alerts table, agent-workflow-monitoring bullets, audit-trail gap). The "Production Readiness at a Glance" table should link to Backlog anchors, one-line state only — no restated evidence paragraphs. |
| **Commercial & Ops Readiness** | `/03` | `p06` + `p07`, minus the duplicate LBGUPS scorecard and duplicate Gate-2 checklist | Mock: `readiness.html`. Commercial gates 1–4, value prop/narrative, credibility risks, support tooling roadmap (linked to Backlog anchors), admin console requirements, incident management flow, SLA/SLO table, runbooks-required list, release management. |

Leave **`MVP Roadmap Forecast`** (`/forecast`) and **Runbooks** (`/runbooks`, `/runbooks/:id`) exactly where they are in the nav — they are not part of the 9→4 consolidation, they're separate features with their own real functionality. See §F — do **not** rebuild them from the mock's `forecast.html`/`runbooks.html`, those were static visual stand-ins only.

### A.3 Migration mechanics

1. Update `NAV_ITEMS` in `Shell.tsx` to 4 entries (numbered 00–03) + the Forecast arrow item, matching the mock's sidebar order.
2. Update the route table in `main.tsx` to 4 routes instead of 10.
3. Decide whether old routes `/04` through `/09` should redirect (react-router `<Navigate>`) to the new consolidated pages or 404. Recommend redirecting `/04`→`/02`, `/05`→`/02`, `/06`→`/03`, `/07`→`/03`, `/08`→`/01`, `/09`→`/01`, in case anything has the old URLs bookmarked.
4. Delete `p05-observability.ts`, `p06-opsSupport.ts`, `p07-csoReview.ts`, `p08-top25.ts`, `p09-gantt.ts`, `roadmap-gantt.css.ts` once their content is merged in, and their content/tests are gone. Check `roadmap-pages.css.ts` for any classes that were *only* used by the deleted pages before removing them.
5. Rename/rewrite `p01-executiveSummary.ts` → the new Overview content or fold entirely into `p00`; rewrite `p02-backlog.ts` → Backlog & Timeline; rewrite `p03-roadmap.ts` → fold into Backlog & Timeline; rewrite `p04-architecture.ts` → Architecture & Observability.
6. There are existing tests referencing some of these pages/routes (check `*.test.tsx` files under `src/` and `src/components/`) — update or remove as routes change.

---

## B. Visual design system — reuse what exists, don't invent

The real app already has a proper CSS design-token system. **Do not introduce new hex colors or a parallel variable system.**

### B.1 Where the tokens come from

`src/main.tsx` imports `'../../src/styles/common.css'`, which resolves to:

```
/Users/michaelberry/Documents/Jemba9/CasecadeProjects/Jemba9_QMS/app/src/styles/tokens.css
```

This file is explicitly marked:
> `Vendored verbatim. On SAG monorepo integration, replace this file with: import "@j9ccgit/shared/styles/common.css". Do NOT add or redefine any token or class already present here.`

It defines `--bg-page`, `--bg-card`, `--text-primary`, `--text-secondary`, `--text-muted`, `--border-color`, `--brand-primary`, `--brand-accent`, `--surface-success`/`--surface-success-text`, `--surface-danger`/`-text`, `--surface-warning`/`-text`, `--surface-info`/`-text`, `--surface-accent`/`-text`, `--surface-neutral`/`-text`, spacing scale (`--space-1`…`--space-8`), radii, shadows, etc. **`roadmap.css` already consumes these correctly** — 89 `var(--...)` usages vs. exactly 1 stray hardcoded hex (`border: 1px solid #ccc;` at `roadmap.css:739` — fix that one while you're in there).

### B.2 Dark theme already exists — you don't need to build a color system

`tokens.css` line 109 already has a complete `[data-theme="dark"]` override block with dark values for every token above. This is real, working, and used by the other apps in this project. Your entire dark-mode task is:

1. Get `data-theme="dark"|"light"` set on `document.documentElement` from a toggle (§C).
2. Make sure content that currently uses hardcoded hex instead of tokens gets converted, so it actually responds to the attribute (§B.3).

Do not write a second set of dark-mode variables like the static mock did (the mock had no choice — no JS meant no way to read `tokens.css`'s existing dark values via a real theme system, so it improvised its own). The real app has no such excuse.

### B.3 Existing semantic classes (from `roadmap.css` / `pages/roadmap-pages.css.ts`) — use these, not inline styles

```
.badge, .badge-impl, .badge-miss, .badge-mock, .badge-part, .badge-unk,
.badge-p0, .badge-p1, .badge-p2, .badge-p3,
.badge-phase0 .. .badge-phase4
.alert, .alert-info, .alert-warning, .alert-danger, .alert-success
.card, .card-title
.stats-grid, .stat-card, .stat-value, .stat-p0 .. .stat-p3, .stat-label
.table-wrap
.page-header, .eyebrow, .meta
.prose
.svg-wrap, .svg-caption
.backlog-item, .backlog-item-header, .backlog-item-body, .backlog-item-meta, .item-id, .item-title, .meta-tag
.phase-block, .phase-header, .phase-body, .phase-0 .. .phase-4
```

When you author the 4 new consolidated pages, map every visual element in the mock to one of these existing classes first. Only fall back to a new class (added to `roadmap.css`, using `var(--...)` tokens, never raw hex) if nothing above fits — e.g. the Backlog page's new timeline bar and the Architecture page's numbered-connector diagram legend are genuinely new UI that will need new classes.

### B.4 Known gaps to clean up while you're in this code

- `p09-gantt.ts` has 18 inline `style="...#hex..."` attributes (it's the most bespoke/least token-compliant page in the deck) — irrelevant if you delete it per §A.3, but if any of its bar-chart concept survives into the new Backlog & Timeline page, rebuild it with tokens, not copy its inline styles.
- `p00-overview.ts` (5), `p01-executiveSummary.ts` (4), `p07-csoReview.ts` (3), `p08-top25.ts` (6) have a handful of inline-hex spans (mostly ✅ checkmarks and ad hoc status-colored text) — convert to the `.badge-*`/`.alert-*` classes or `.stat-p0`-style utility classes where equivalent, or add small new token-based utility classes if truly one-off.
- `shell.css` has 9 hardcoded hex values alongside 21 `var()` usages — audit and convert before you add the theme toggle to it, so the sidebar itself is fully theme-aware too.

---

## C. Light / Dark mode toggle — vendor the existing pattern, don't invent one

### C.1 Reference implementation (copy this pattern, don't import the package)

The real pattern lives in the sibling monorepo at:

```
/Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit/packages/ui/src/contexts/ThemeContext.tsx
```

Full contents (as of this writing):

```tsx
import {
  createContext,
  useContext,
  useState,
  useCallback,
  useEffect,
  type ReactNode,
} from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextValue {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

const STORAGE_KEY = 'sag-theme';

function getInitialTheme(): Theme {
  const stored = localStorage.getItem(STORAGE_KEY);
  if (stored === 'dark' || stored === 'light') {
    return stored;
  }
  return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
}

interface ThemeProviderProps {
  children: ReactNode;
}

export function ThemeProvider({ children }: ThemeProviderProps) {
  const [theme, setTheme] = useState<Theme>(getInitialTheme);

  useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem(STORAGE_KEY, theme);
  }, [theme]);

  const toggleTheme = useCallback(() => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  }, []);

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme(): ThemeContextValue {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}
```

`Jemba9_QMS/app/roadmap` has **no dependency on `@j9ccgit/ui`** (checked `package.json` — not present), so you cannot `import` this. **Vendor a copy** at `src/ThemeContext.tsx` (same pattern as this repo already vendors `tokens.css` from the same upstream package, per the comment in that file).

Keep `STORAGE_KEY = 'sag-theme'` (don't rename it to something roadmap-specific) — `tokens.css`'s own header comment says this app is expected to eventually merge into the real SAG monorepo, at which point sharing the same localStorage key means a user's theme preference carries over seamlessly across apps instead of resetting. This is a deliberate forward-compatibility choice, not an oversight.

### C.2 Where to wire it up

- Wrap the router in `main.tsx` with `<ThemeProvider>` (same level as `<SignInGate>` — decide ordering based on whether the sign-in screen itself should be themeable; recommend `ThemeProvider` outermost so the sign-in screen also respects the theme).
- Add the toggle UI to `Shell.tsx`'s `.sidebar-footer` div (currently just plain text `"Source: j9ccgit · June 2026"`). The established cross-app convention, from `j9ccgit/packages/ui/src/components/Sidebar.tsx`:

```tsx
<div className="nav-item" onClick={toggleTheme}>
  <span className="icon">
    {theme === 'dark' ? <Sun size={16} /> : <Moon size={16} />}
  </span>
  {theme === 'dark' ? 'Light Mode' : 'Dark Mode'}
</div>
```

`lucide-react` (which provides `Sun`/`Moon`) is already a dependency of `app/roadmap` — no new package needed.

- For the actual toggle **control** styling/interaction pattern (not the icon/label — the click-to-toggle mechanics and accessibility), the closest in-repo precedent is `src/components/ModeToggle.tsx` in this very app (button group, `role="group"`, `aria-pressed`). A single toggle doesn't need a button group, but do follow its accessibility pattern: a real `<button>` (not a `<div onClick>` — the packages/ui reference above uses a div, but prefer a semantic button here for a cleaner a11y story than the reference), `aria-pressed={theme === 'dark'}`, visible focus state.
- Add `src/ThemeContext.test.tsx` following the pattern of `src/components/ModeToggle.test.tsx`, covering: default theme resolution (localStorage empty → `prefers-color-scheme`), toggle flips both React state and the DOM attribute, and persists to localStorage.

### C.3 What NOT to do

Do not reintroduce the mock's `<style>` block with a second `:root` / `:root:has(#theme-toggle:checked)` variable set. That was a workaround for the mock having no JavaScript. The real tokens already have a complete dark palette at `tokens.css:109`.

---

## D. Logos — the assets already exist in this repo, don't fetch from Desktop

Do **not** copy anything from `~/Desktop/J9Logos` (that's where the mock's assets came from, because the mock lives outside this repo). The exact same PNGs are **already vendored inside `Jemba9_QMS`**:

```
Jemba9_QMS/app/public/jemba9-logo.png
Jemba9_QMS/app/public/jemba9-logo-orange.png
Jemba9_QMS/app/public/jemba9-logo-white.png
Jemba9_QMS/app/shell/public/jemba9-logo-orange.png
Jemba9_QMS/app/shell/public/jemba9-logo-white.png
```

`app/roadmap` is a separate Vite app with its own `public/` directory (`app/roadmap/public/` — currently holds `seed-forecast.json`), so its dev server won't serve files from `app/public/`. **Copy** (don't symlink) `jemba9-logo-white.png` and `jemba9-logo-orange.png` into `app/roadmap/public/`.

### D.1 Established usage convention (follow this, not the mock's choice)

From `app/shell/src/App.tsx` and `app/src/App.tsx` (the main QMS shell, which already has working logo placement):

```tsx
<img src="/jemba9-logo-white.png" alt="Jemba9" height={24} />       // on dark surfaces (topbar)
<img src="/jemba9-logo-orange.png" alt="Jemba9" className="..." />  // on light surfaces (hero, sign-in)
```

**The mock used the orange logo on the dark sidebar** (it reads fine, orange pops against navy) — but the established convention elsewhere in this project is **white logo on dark surfaces, orange on light surfaces**. For consistency with the rest of the QMS app family, use `jemba9-logo-white.png` in the roadmap sidebar (which is dark), not orange. This is a minor visual call — flag it to a human if you want, but default to matching the existing convention rather than the mock.

Replace the current `Shell.tsx` `.sidebar-brand` block (plain `"Jemba9 Works"` / `"Roadmap"` text) with the logo image plus the existing `"Roadmap"` subtitle line kept as-is underneath.

---

## E. Architecture diagram — port the enhanced AWS-network-style SVG

The mock's `architecture.html` (System Diagram section, first `<svg>`) replaced the original simple layer diagram with:

1. **All 4 real client apps** (from `j9ccgit/apps/`: `employee-console`, `customer-portal`, `control-center`, `stakeholder-portal`) as separate client boxes — the original `p04-architecture.ts` diagram only showed 2 (Employee Console, Customer Portal).
2. A proper nested **Region → VPC → Availability Zone → Subnet** boundary structure following standard AWS diagramming convention (region dashed, VPC solid, AZ dashed, subnets solid colored by public/private), grounded in the actual CDK config at `j9ccgit/infra/lib/network-stack.ts` (`10.0.0.0/16`, `maxAzs: 2`, `natGateways: 1` — note the asymmetry this implies: AZ-B's public subnet has **no** NAT Gateway of its own and egresses through AZ-A's, which the diagram calls out explicitly).
3. **Numbered off-page connectors** instead of long diagonal lines crossing unrelated boxes. This was added after user feedback mid-session specifically because literal lines from ECS to Secrets Manager/CloudWatch/S3/Cognito, and from the ALB down to each AZ's ECS task, were crossing straight through the *other* Availability Zone's box and looking messy. The fix: each such connection gets a small numbered circle (①⑥) with a short arrow stub at both the source and destination, plus a legend mapping number → connection. **Reuse this exact pattern** for any future connection that would otherwise cross an unrelated box — don't reintroduce crossing lines.

**Action:** copy the diagram's *content and structure* (box layout, nesting, the 6 numbered connectors, the legend) from `RoadmapAlternate/architecture.html`'s first `<svg>` block into the new Architecture & Observability page. You can keep it as inline SVG in the template-literal page content (that's already how `p04-architecture.ts` and friends work — no established "diagram component" convention to conform to here, plain embedded SVG is consistent with existing practice). Update the account ID / region / CIDR values if the CDK config in `network-stack.ts` has changed since this was written (`824999955649` / `us-west-2` / `10.0.0.0/16` as of 2026-07-19).

The second `<svg>` on the mock's Architecture page (the "Target Observability Signal Stack" diagram) is unchanged from the original `p05-observability.ts` — no diagram work needed there, just relocate it under the consolidated page per §A.2.

---

## F. MVP Roadmap Forecast & Runbooks — visual polish only, no rebuild

These two already have full, working functionality in the real app (`src/App.tsx` + `ForecastBoard`/`PresentView`/`SinglePageView`, backed by `src/lib/forecast-api.ts` and `public/seed-forecast.json`; and `RunbooksIndex.tsx`/`RunbookPage.tsx` backed by `src/runbooks/registry.ts`). The mock's `forecast.html` and `runbooks.html` were **static, non-functional visual stand-ins** built only so the mock's sidebar/IA looked complete for review purposes — per the user's explicit instruction at the time ("current capabilities do not need to be added to this mock, I just want them represented again for visual requirements sake").

**Do not** replace or rebuild these features from the mock's HTML. The only changes these two areas need from this upgrade:

1. Sidebar/logo/theme-toggle changes from §C/§D apply here too (they share `Shell.tsx`).
2. If you want visual consistency with the new 4-page IA's styling (card borders, badge colors, etc.), that's a nice-to-have, not a requirement — these pages already work and are already reasonably polished. Don't spend time here until §A–§E are done.
3. Do not add a 5th/6th nav entry for these — they already have their nav entries (the `→ MVP Roadmap Forecast` item and the `Runbooks` section in `Shell.tsx`); nothing about their nav position changes.

---

## G. Data accuracy — re-verify, don't trust the mock's snapshot

Every status badge, date, and "in progress" note in the mock (`Backlog & Timeline` page especially) reflects the state of the project **as of 2026-07-19**. Known facts that were true then and may have changed by the time you implement this:

- P0-02 (staging CDK stack) and P0-03 (GitHub Actions CI/CD) were "In Progress" — check `j9ccgit/documents/plans/active/25-staging-cdk-stack-and-deploy-pipeline-execution.md` and `j9ccgit/documents/deploy-runbooks/active/staging-deploy.md` for current phase status before writing any status into the new Backlog page. (As of the last check during this session, that plan's Phases 1–8 were approved and Phase 10 — the first real staging deploy — was the next step; this will have moved on.)
- P0-05 (DNS/TLS/custom domain) was confirmed **not** done as of 2026-07-19, despite an earlier draft of this mock briefly (incorrectly) marking it done — re-verify against `infra/lib/real-environment.ts` (`domainName`/`certificateArn` wiring) before trusting either claim.
- The tenant isolation test suite (P0-04) was done but **not yet enforced in CI** (that's what P0-03 delivers) — don't copy "runs in CI on every PR" language anywhere; several of the original 9 pages had this exact inaccuracy and it was corrected during this session.

**Process:** for every backlog item's status in the new consolidated pages, trace it back to the actual `j9ccgit` source (backlog docs, plan files, runbooks, or direct code/infra inspection) rather than copying the mock's text. Use the mock only for structure (what a row/section should contain), not as a data source.

---

## H. Suggested order of work

1. §B.4 cleanup (cheap, isolated, unblocks nothing but reduces risk of the later steps compounding technical debt).
2. §C (ThemeContext + toggle) — do this early since it's infrastructure everything else benefits from, and it's fully spec'd/de-risked above.
3. §D (logo swap) — quick, isolated.
4. §A (IA consolidation: write the 4 new pages, sourcing fresh data per §G, using existing classes per §B.3) — the bulk of the work.
5. §E (diagram port) — part of the Architecture page from step 4, called out separately here because it's the most novel/detailed piece.
6. §F — confirm no regressions to Forecast/Runbooks, apply cosmetic polish only if time remains.
7. Run the existing test suite (`npm test` in `app/roadmap`) after each step, not just at the end — there are existing `*.test.tsx` files that will break as routes/components change and are much easier to fix incrementally.
8. Visual verification: actually run the dev server and click through both themes and all 4 pages before calling this done — this whole upgrade is UI/IA work, and static analysis won't catch layout regressions, dark-mode contrast issues, or broken anchors.

---

## Appendix: file inventory referenced above

**Mock (this directory):** `index.html`, `backlog.html`, `architecture.html`, `readiness.html`, `forecast.html`, `runbooks.html`, `assets/jemba9-logo-{orange,white}.png`

**Real app root:** `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/Jemba9_QMS/app/roadmap/`

**Key real-app files:** `src/Shell.tsx`, `src/main.tsx`, `src/App.tsx`, `src/shell.css`, `src/roadmap.css`, `src/pages/p00…p09-*.ts`, `src/pages/roadmap-pages.css.ts`, `src/pages/roadmap-gantt.css.ts`, `src/components/ModeToggle.tsx`, `src/runbooks/registry.ts`, `src/components/RunbooksIndex.tsx`, `src/components/RunbookPage.tsx`

**Shared tokens (vendored, do not duplicate):** `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/Jemba9_QMS/app/src/styles/tokens.css`, `common.css`

**Reference implementations in the sibling monorepo (copy patterns from here, do not add as a dependency):** `/Users/michaelberry/Documents/Jemba9/CasecadeProjects/j9ccgit/packages/ui/src/contexts/ThemeContext.tsx`, `j9ccgit/packages/ui/src/components/Sidebar.tsx`, `j9ccgit/apps/employee-console/src/components/layout/Layout.tsx`

**Existing logo assets (copy from here, not from Desktop):** `Jemba9_QMS/app/public/jemba9-logo-{white,orange}.png`

**Ground-truth data sources for §G:** `j9ccgit/documents/plans/active/25-staging-cdk-stack-and-deploy-pipeline-execution.md`, `j9ccgit/documents/deploy-runbooks/active/staging-deploy.md`, `j9ccgit/infra/lib/*.ts`
