TALARIA — SysML 2.0 Concept Demo
=================================

HOW TO RUN
----------
Unzip anywhere, then double-click index.html. Any modern browser.
No server, no install, no internet connection required.

WHAT IT IS
----------
A clickable mock of the TALARIA Space Assurance Gateway showing how
SysML 2.0 would work inside the real product. The design tokens, app
shell, sidebar and page structures mirror the production codebase
(packages/styles/common.css and packages/ui) so it reads as TALARIA
rather than as a generic wireframe.

Every place SysML v2 does real work carries a purple marker. Click a
marker to jump to the explanation panel at the bottom of that page.
Each explanation states the construct, what it buys, and — labelled
"Today:" — what the real application does without it, so the delta is
visible rather than asserted.

SCREENS
-------
index.html                     Start here — screen index and reading notes

employee/bom.html              Product BOM. Three views of the same data:
                               table, SysML part-usage tree, generated
                               textual notation.
employee/run.html              Derating analysis run. Press "Start analysis"
                               to watch the LangGraph pipeline execute.
                               "Compare with today" shows the same rule as
                               a derating_rule row plus TypeScript.
employee/findings.html         Findings with the evaluated constraint shown.
                               "Request exception" opens the waiver flow.
employee/requirements.html     Requirements as formal constraints, with the
                               traceability matrix as a model query.
employee/phase-gate.html       Gate criteria as guarded transitions, a
                               Rev C → Rev D model diff, and baselines.

customer/overview.html         Customer view pinned to a named baseline.
customer/findings.html         Margin bars; "Show the rule that was applied"
                               reveals the actual constraint.
customer/deliverables.html     Documents as generated views, maturity-gated.
customer/model-exchange.html   Import from and publish to the customer's own
                               model repository over the standard REST API.
                               This screen has no equivalent in TALARIA today.

DEMO DATA
---------
Project J9Avionics — TIA Power Board Rev C. 69 part definitions across
142 usages, Phase 5 (Detailed Design), NASA-EEE-INST-002 Class B, 5-year
550 km LEO mission (TID 30 krad, RDM 2.0, -40…85 °C). Employee screens
are shown as Sarah Chen; customer screens as James Wilson at ProtoPD
Customer Corp. All figures are illustrative.

SCOPE NOTE
----------
SysML v2 supplies the language and the protocol. It supplies no derating
tables, no radiation models and no screening rules — that content remains
Jemba9's. What changes is that the content becomes expressible in a form
other tools can read and check.

FILES
-----
assets/talaria.css   Design system, adapted from the production tokens

Each page's JavaScript (tabs, modals, marker links, run simulation) is
inlined in the page itself. No separate .js files, no dependencies, no
CDN, no build step -- which also keeps the zip past mail-client filters.
