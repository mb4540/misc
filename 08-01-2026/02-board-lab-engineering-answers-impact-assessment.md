# Board Lab — Impact Assessment: Engineering Answers of 2026-07-31

**Date:** 2026-08-01
**Source:** `misc/08-01-2026/BoardLabEmai.md` — Ryan Swartzendruber 2026-07-31 09:47, Lucas Johnson 2026-07-31 12:45, replying to the status mail of 2026-07-25
**Assessed against:** `Jemba9_Ecosystem/documents/plans/active/00-board-lab-execution.md`, with reference to `00-board-lab-execution-review.md` (Phase 0–4 audit) and `01-board-lab-engineering-feedback-review.md` (E-1…E-12)
**Status:** Assessment only — no design or plan file changed.

---

## Summary

Four questions were blocking (`00-board-lab-execution.md` §"Open Decisions"). **Two are now closed, one is partly closed and materially reframed, one is partly closed with new work attached.** Two of the seven questions in the status mail (Q6, Q7) went unanswered and Ryan explicitly deferred Q6 to a scoping discussion.

The net effect on the plan is **more unblocking than new scope**, but that headline hides three things worth taking seriously:

1. **Q1 did not come back the way the plan assumed.** There is no bulk export of the Siemens model library. Phase 6 step 6 — "HLSA vendor-library reader" — is not buildable as written, and no amount of waiting will make it buildable. The one machine-readable artefact Ryan identified is the **AVL export/import**, which carries part-number *links*, not models. That is still useful, but it is a different tool solving a different problem.
2. **The AVL answer exposed an identity gap the plan does not model at all.** Their BOMs are keyed on **internal part number**; the Siemens library is keyed on **manufacturer part number**; the AVL is the mapping between them. Board Lab's schema has `component.mpn` and `device_model.mfg_pn` and no concept of an internal part number. Every match Board Lab performs is against the wrong key. This is a schema defect on the same footing as E-3, and Phase 4A — the migration that would carry the fix — has not started yet.
3. **Lucas's answer to Q4 is not really an answer to Q4.** He answered the graph question in one paragraph (out of scope for v1 stands, with a cheaper substitute available) and then made a structural argument that goes well beyond it: worst-case analysis is driven by *requirement → analysis type → relevant parameters*, and a tool that does not carry that structure cannot be trusted or shown to be complete. The plan has no representation of it. This is the largest genuinely new item in the thread.

Ryan's aside — *"Sorry, I'm used to my AI usage being free. Also I failed to recognize the need for human review."* — is worth recording as a shared constraint. Both engineers now price features in tokens **and** in Gate 0 review minutes. That is a better footing than the plan had a week ago.

| ID | Item | Effect on plan | Where it lands | Urgency |
|---|---|---|---|---|
| A-1 | **Q2 CLOSED** — model only the parts a design needs | Scope **reduction**; unblocks Phase 7 and Phase 14 | Plan §"Open Decisions", §7.0, §9, §14 | Now — it is free |
| A-2 | **Q3 CLOSED** — `.xlsx` is universal, standardise on it | Confirms Phase 6 step 5; kills the `.ots` writer idea | §6.1 step 5, §6.2 | Now — it is free |
| A-3 | **Q1 PARTLY ANSWERED and reframed** — no bulk export exists | Phase 6 step 6 is not buildable as written; retarget or drop | §6.1 step 6, §9 step 2, Follow-Up Items | Before Phase 6 |
| A-4 | **Q5 PARTLY CLOSED** — docs exist, are named, still not in the repo | Phase 6 remains blocked on physical files | §"Open Decisions" para 3, §6 | Before Phase 6 |
| A-5 | **Q4 PARTLY CLOSED** — graph extraction stays out; a cheaper substitute is what is actually wanted | Upgrades an existing Phase 11 rule; small new Phase 7 work | §7.1, §11.1, Follow-Up Items | Before Phase 7 |
| A-6 | **Q6 STILL OPEN** — Ryan deferred it; gave one useful datum (~50% power/ground) | Phase 9 stays as written; one deterministic mitigation now available | §9.1 step 3 | Before Phase 9 |
| A-7 | **Q7 STILL OPEN** — no corpus supplied | Phase 9 still has no acceptance corpus | §9.2, §19 | Before Phase 9 |
| N-1 | **Internal PN vs manufacturer PN is unmodelled** | Schema defect — matching keys are wrong | Phase 4A migration `0004`, Phase 5 BOM parser, Phase 9 coverage | **Now — while the DB is empty** |
| N-2 | **Template conditional formatting is a machine-readable spec** | New, cheap, de-risks A-4 | §6.1, §11.1 | With Phase 6 |
| N-3 | **V9 templates exist in the wild** | `sam-versions.ts` must name V9 explicitly | §6.1 step 1, §6.2 | With Phase 6 |
| N-4 | **Part-type parameter profiles + derivation procedures** | Replaces "additional rules" hand-waving with a data-driven catalog | Phase 11, possibly Phase 4A schema | Decide before Phase 10 |
| N-5 | **Requirement → analysis-type structure is absent** | Architectural gap; Lucas's central point | `04-architecture.html`, Phases 10/11/17 | **Scope decision now** |
| N-6 | **The "WCA guideline" is an ingest source nobody has mentioned** | New input class | Phase 4 registry, Phase 7 | Before Phase 10 |
| N-7 | **Greedy vs lightweight library — do not conflate the two libraries** | No change; a clarification worth writing down | `05-scope-and-files.html` | Low |

---

## A-1 — Q2 closed: model only what a design needs

**What was said.** Ryan: *"Only the parts needed for a given design. Sorry, I'm used to my AI usage being free. Also I failed to recognize the need for human review."* Lucas: *"Completely agree."*

**What this means.** The plan's existing behaviour was right. `model_draft` fires on unmodelled parts of a board revision. E-2's proposed second pipeline — *datasheet import → part enumeration → bulk draft → Gate 0* — is **not wanted** and should be withdrawn rather than deferred. Deferring it leaves a hook in Phase 9 and a permission decision in Phase 14 that now have no consumer.

**Suggested plan changes.**

- **§"Open Decisions" table** — strike Q2. Record the answer and its date in the same place, so a future session does not re-raise it.
- **§7.0 carry-in** — delete the "Partly blocked (E-2)" paragraph. Phase 7 is now unblocked.
- **§7.1 step 5** — the indexer's second responsibility changes shape. It was *"enumerate the orderable part numbers the datasheet covers"*. Under this answer, exhaustively enumerating a 200-row resistor table on import is work nobody asked for. Narrow it to **resolve on demand**: given an MPN that appears on a live BOM, locate the row/section of the datasheet that scopes that part's values and write **that** `datasheet_part` row, with `source_page` and `table_ref`. Same table, same schema (Phase 4A unchanged), a fraction of the work.
  - Worth noting the one thing eager enumeration bought that lazy resolution does not: a searchable index of which datasheet covers which part. If that turns out to matter, it can be added later against a table that already exists. It does not need to be decided now.
- **§9.1 step 3** — remove the E-2 requirement that `model_draft` be callable outside a board-revision context. It stays revision-scoped.
- **§14.1 step 1** — `POST /boardlab/api/models` keeps `boardlab.datasheet.manage`. The E-2 proposal to re-gate it on `boardlab.analysis.run` (because it would bill tokens in bulk) is moot.
- **§"Follow-Up Items"** — add "Bulk library building — considered and declined by engineering 2026-07-31, on cost and review-load grounds. Revisit only if review cost per model drops materially."

**Cost note.** This removes the ~100× worst case the plan flagged as its largest cost driver. Nothing else in the plan changes because of it, which is a good sign the question was correctly isolated.

---

## A-2 — Q3 closed: `.xlsx` only

**What was said.** Ryan: *"No, I think .xlsx can be considered universal. I thought there could be some benefit to having the agent create the model in .ots."* Lucas: *"No reason to not standardize on that file format."*

**What this means.** Two separate things, and only one of them is a question about reading.

- **Reading.** Phase 6 step 5 — detect `.ots`/`.ods` and refuse with a message naming the cause and the fix — stands unchanged. It was correct either way; it is now correct *and* cheap, because there is no follow-on support obligation. No new dependency, `exceljs` stays.
- **Writing.** Ryan floated `sam-writer.ts` emitting `.ots`. Lucas closed it. `sam-writer` emits `.xltx`/`.xlsx` only. Worth writing down explicitly in §6.1 step 4 so it does not resurface.

**Suggested plan changes.** Strike Q3 from §"Open Decisions". Add one clause to §6.1 step 4: *"Output is `.xltx`/`.xlsx` only — OpenDocument output was considered and declined 2026-07-31."* No verification changes; §6.2 already tests the refusal path.

---

## A-3 — Q1 reframed: there is no bulk export, and the AVL is the only interface

This is the answer that changes the plan the most, and it changes it by **removing** a planned capability.

**What was said.** Ryan walked the whole flow: HLSA queries a Siemens **cloud** database by internal part number on BOM import; matches are copied into the local library and linked in the project BOM; unmatched parts can be linked manually through the **AVL**, which maps internal part number to manufacturer part number and **can be exported and imported** (import adds/replaces, never deletes). And then: *"To access the existing HLSA library provided by Siemens would need to go through the application if it's even possible. My reading found nothing in terms of another way to interface with their library. This needs discussion, I'm not sure of the best path forward."*

Lucas added that a cloud model is only pushed into their internal library if they deliberately ingest it, and that the AVL *"is functionally a component of our internal library."*

**What the plan assumed.** §6.1 step 6: *"HLSA vendor-library reader — BLOCKED, do not start… The export format is unknown."* The plan was waiting for a format. **There isn't one.** E-1's recommendation — *"Importing it is cheaper than drafting and should precede drafting in precedence order"* — was sound reasoning from a premise that has now failed.

**Consequences, stated plainly.**

- Phase 6 step 6 cannot be built. Not "not yet" — not at all, on the interfaces Ryan can see.
- Phase 9's fourth coverage bucket (`vendorMatch`) loses its data source. The plan already hedged this correctly (*"leave the bucket present but always empty"*), and that hedge is now the permanent state unless something below is adopted.
- `device_model.source = 'vendor'` stays unused. No enum change needed; it costs nothing to leave.

**The one thing that is actually reachable — and what it is worth.** The AVL export is a real, machine-readable, exportable file. It does not contain models. It contains **the mapping from their internal part numbers to manufacturer part numbers, for the parts HLSA has models for.** That is precisely the input `model-coverage` needs to answer "does a vendor model already exist for this part?" without ever touching the model itself.

So the retarget is:

- **Read** the AVL export → populate a coverage signal → `model-coverage` can bucket a part as "HLSA already has this; do not spend tokens or a Gate 0 approval on it."
- **Write** an AVL import file from Board Lab's approved models → the engineer loads it into HLSA to link Board Lab's work back to their internal part numbers. This is the same file-handoff posture as `sam-writer.ts`, and it respects the constraint already recorded in Follow-Up Items: *"Board Lab owns the model library and must never block on HLSA being reachable."*

**I would not build this yet.** The AVL export format is unknown, and the plan has just been bitten once by designing against an unknown format. Ask for a sample file first (see Questions).

**Suggested plan changes.**

- **§6.1 step 6** — rewrite from "BLOCKED, do not start (format unknown)" to "**Withdrawn as specified.** No bulk export of the Siemens library exists; access is through the HLSA application only (Ryan, 2026-07-31). Replaced by an optional AVL reader/writer, pending a sample AVL export."
- **§9.1 step 2** — keep the `vendorMatch` bucket. Change its stated source from "HLSA vendor model import" to "AVL coverage signal, when available; otherwise always empty."
- **§"Open Decisions"** — Q1 does not close. It narrows to: *what is the AVL export format, and can we have a sample?* Keep it blocking Phase 6's optional step only, not Phase 6 as a whole.
- **§"Follow-Up Items"** — amend the existing "HLSA import interfaces" bullet. It currently reads *"Ryan believes automation interfaces exist."* Ryan has now looked and found none beyond the application UI. Record that, so nobody re-investigates.

---

## A-4 — Q5 partly closed: the documents are identified but still not in the repository

**What was said.** Ryan: *"I sent the PDF with two templates as attachments and a path to example models… The example models have the version embedded and all indicate the version is 12. I see other models that have been matched in the online database that have version 9."* Path: `J:\Engineering\Tools and Software\Siemens\Schematic Analysis\HLSAWorkShopData\...\Completed Models\`.

**What this changes.** The question of *whether the specification exists* is answered — it does, and V12 is confirmed. The question of *whether Phase 6 can be executed* is not, because:

- The PDF and templates are attachments on an email thread, not files in the repository. §"Open Decisions" paragraph 3 says Phase 6 *"cannot be executed faithfully from guesswork"* — that remains true today.
- `J:\` is a Windows engineering share. It is not reachable from the development environment. The example models must be copied out by someone with access.
- §6.2 requires *"sam-parser reads V12 template fixture correctly"* and a round-trip test. Both need real files. There is no V12 fixture in the repo.
- **E-5 is still open on the fact that matters most.** The legal pin-name character set is *"template-version-dependent and must be confirmed against the Siemens document, not guessed"* (§6.1 step 1). Ryan has told us `+ - % /` and spaces are disallowed. Whether that list is complete, and whether it differs between V9 and V12, is in the PDF.

**Suggested plan changes.** No structural change — this is an inputs problem, not a design problem. Amend §"Open Decisions" paragraph 3 to record that the documents are confirmed to exist, are named, and are pending transfer into the repository, rather than implying they may not exist. Add a concrete pre-Phase-6 action item: land the PDF, both templates, and a handful of completed models under a fixtures path in `Applications/boardlab/`, and check whether any of them carry NDA or export-control markings before they are committed.

---

## A-5 — Q4 on graphs: the answer relaxes the requirement, and points at something cheaper and better

**What was said.** Lucas: graphs are used *"between occasionally and commonly"*, scoped to worst-case **component variability**, which *precedes* circuit-level analysis. Per part type, a standard set of parameters; per parameter, a documented derivation procedure — *"many require only reading 2 or 3 values out of a table… and a value or two from WCA guideline and some documented assumptions."* And: *"We can procedurally identify when we expect to need a graph, and it can be as manual as necessary to get that data out."*

Then the part that is worth more than the answer: *"there is a lot of value in an AI assistant that at least knows what graphical data is present even if it never went through a complete curve extraction effort, and creating a catalog of what data is where… a tool that can open the pdf, highlight the region where it understands the relevant information to be… is still huge."*

**What this confirms.** Graph/curve extraction stays out of v1. That decision (`04-architecture.html`, "Not in v1") is now endorsed by the engineer who raised the concern, not merely tolerated. The Phase 11 rule `parameter-only-available-from-plot` is the right mitigation and Lucas's *"as manual as necessary"* is exactly the behaviour it produces.

**What this adds, cheaply.** A **figure index** is buildable in Phase 7 with no new dependency. Figure captions are text — `pdf-parse` already returns them, and page numbers are already tracked because chunking is per page. A `datasheet_figure` row of `{ page, caption, figure_number }` turns the Phase 11 rule from *"this parameter needs a value from a plot"* into *"this parameter needs a value from Figure 7, page 12 — please supply it with a citation."* That is a large usability difference for one small table and a regex over chunk text.

Region highlighting — Lucas's *"highlight the region"* — needs coordinates, which `pdf-parse` does not provide. `pdfjs-dist` would, but that is a dependency change and a viewer surface. Follow-up, not v1.

**Suggested plan changes.**

- **§7.1 step 5** — add: extract figure captions and their page numbers into a `datasheet_figure` index while chunking. Deterministic, no LLM.
- **§4A.1** — add `datasheet_figure` to the `0004` migration (org-scoped, RLS'd, FK to `datasheet`). Phase 4A has not started; this is free now and a separate migration later.
- **§11.1** — `parameter-only-available-from-plot` cites the figure index when it can resolve one.
- **§"Follow-Up Items"** — amend the graph-extraction bullet: record that engineering endorsed deferral, and that the near-term ask is *cataloguing and locating* graphical data rather than reading values off it. Add region highlighting as the next increment.

---

## A-6 — Q6 still open, with one useful datum

**What was said.** Ryan: *"I/we need to define the scope of what our tools are doing to answer Q6. Of those 1500 pins half will be power and ground, so it's not quite as daunting as it sounds. This is another discussion item that I don't have a good feel for."*

**Assessment.** Q6 stays open and stays as written in §"Open Decisions" Q4 (second clause). Ryan is right that it depends on a scoping decision they have not made, and it should not be guessed.

But the datum is useful and it enables a mitigation that is correct **under either answer**: Board Lab does not need the datasheet to identify power and ground pins. It has the netlist. A pin connected to a power or ground net is knowable deterministically from ODB++ and the BOM, which Phase 5 already parses. So:

- Classify power/ground pins from **net membership**, deterministically, before `model_draft` runs.
- Exclude them from LLM extraction — roughly halving prompt volume on large devices.
- Exclude them from the Gate 0 review surface, or collapse them to one row per rail. A 1,500-row approval screen is a formality; a 700-row one is still a formality, but a screen showing *"742 power/ground pins collapsed to 6 rails — expand"* plus the signal pins is a review a human can actually perform.

This is worth adding to Phase 9 now. It reduces cost and improves Gate 0 whether the eventual answer is "complete pin models" or "interface pins only."

**Suggested plan changes.** §9.1 step 3 — add deterministic power/ground pin classification from net membership, ahead of LLM extraction, with the classification recorded so Gate 0 can collapse it. Keep the existing completeness-accounting requirement and keep the "do not over-build before the answer" caution.

---

## A-7 — Q7 still open

Lucas did not supply the component list or datasheets. Phase 9 still has no evaluation corpus and therefore no acceptance criteria for extraction quality beyond "it type-checks." §9.2 and §19 both assume fixtures that do not exist.

His closing offer — *"If what you need here isn't what Ryan just provided, shoot me an email or Teams invite to talk it through"* — is the opening. A 30-minute call producing five or six real datasheets across distinct part classes (passive, diode, regulator, connector, MCU) is worth more to Phase 9 than any further written exchange. Note this overlaps with the Phase 6 fixture need in A-4 — one collection trip should serve both.

---

## N-1 — Internal part number vs manufacturer part number (schema defect)

**Why this surfaced.** Nobody asked about it. It fell out of Ryan's Q1 walkthrough: *"The application queries a Siemens cloud database based on Internal Part Number (as defined by our BOM)… by equating the BOM (internal part number) with the cloud library (manufacturer part number)."*

**What is built.** `component.mpn`, `device_model.mfg_pn`, `datasheet_part.mpn` — all manufacturer part numbers, matched by string equality. `device_model` carries a partial unique index on `(org_id, mpn)` for approved models (Phase 19 tests it). There is no internal part number anywhere in the schema.

**Why this matters, concretely.**

- **Phase 5's BOM parser reads their BOM.** Their BOM's identity column is the internal part number. The parser's column auto-detection will find *something*, and whatever it finds will be written to `component.mpn`. If that is the internal PN, every downstream match against a manufacturer-keyed datasheet or model silently fails. If the BOM also carries an MPN column and the parser picks that instead, the internal PN — the key their whole toolchain runs on — is discarded.
- **`model-coverage` (Phase 9) matches on `mpn`.** With one identity column it cannot express "this internal PN is satisfied by a model for this manufacturer PN", which is the *entire content* of the AVL and the exact shape of the A-3 coverage signal.
- **One internal PN maps to many manufacturer PNs** (that is what an approved vendor list is *for*). A single column cannot hold it, and the unique index on `(org_id, mpn)` is defined over the wrong key.
- **Gate 0 and the audit trail.** An approval record that says "approved a model for `RC0402FR-07100RL`" is less defensible than one that also says which internal part number it satisfies on which BOM. Traceability is the point of the application.

This is the same class of defect as E-3 (one `mpn` per datasheet, when a datasheet covers a family) and it was found the same way — by an engineer describing their actual workflow.

**Suggested change — fold into Phase 4A's `0004` migration.**

- `component.internal_pn String?` — the BOM's own identity column, kept alongside `mpn`.
- A mapping table, `part_alias` (org-scoped, RLS'd): `internal_pn`, `mfg_pn`, `source` (`bom | avl | engineer`), `approved_by`, `approved_at`. This is Board Lab's AVL. It is also where an imported AVL export lands (A-3) and what an exported one is generated from.
- `device_model` gains nothing — it stays manufacturer-keyed, which is correct. Resolution happens through `part_alias`.
- Revisit the `(org_id, mpn)` approved-model unique index in light of the above before Phase 19 tests it.

**Timing.** Phase 4A is `NOT STARTED`, both databases are empty, and `0004` is already going to be written. This is the cheap window and it is the *last* one — §4A.0 makes the same argument for the E-3/E-4/E-5 changes, and the Phase 2 `@@map` rename paid for the lesson. After Phase 5 produces fixtures this becomes a data migration.

**Confidence caveat.** This rests on Ryan's description of their BOM, not on a BOM. Get one before writing the migration — see Questions. If their BOM turns out to carry both columns cleanly, the change is small; if internal PN is the only identity, it is load-bearing for Phase 5 onward.

---

## N-2 — The template's conditional formatting is a machine-readable specification

**What was said.** Lucas, unprompted, at the end: *"we'll want to extract the conditional formatting from the HSLA model template excel file to use as rules for what information must be populated."*

**Why this is a good idea and not just a nice one.** The plan's §6.1 step 1 has `sam-versions.ts` as a hand-written `Record<version, ColumnMap>`, reconstructed by reading a Siemens PDF — the single most guesswork-prone artefact in Phase 6, and the reason Phase 6 is blocked on A-4. Conditional formatting rules in the template encode *which cells must be populated and under what conditions*, in a form `exceljs` can read directly. That is a slice of the specification available from a file rather than from prose.

It also feeds Phase 11 for free: "field X is required for pin type Y and is empty" is a deterministic, provenance-carrying finding — the exact shape the rule engine exists to produce — and it is derived from Siemens' own definition of a valid model rather than from our reading of it.

**Suggested plan changes.**

- **§6.1 step 1** — derive the required-field rules from the template's conditional formatting where possible; hand-code only what cannot be read. Keep the PDF as the authority for anything the two disagree on, and record disagreements rather than silently preferring one.
- **§11.1** — add a rule `model-field-required-by-template`: a field the template marks as required for a given pin type is empty on an approved model.
- **§6.2** — add a verification line: the extracted required-field set is asserted against the V12 template fixture.

**Caveat.** Conditional formatting is a presentation feature; teams use it inconsistently and it may encode warnings as well as hard requirements. Treat what is extracted as a *draft* rule set to be reviewed once, not as gospel — the same posture the design takes toward everything else the machine proposes.

---

## N-3 — V9 templates exist

**What was said.** Ryan: *"The example models have the version embedded and all indicate the version is 12. I see other models that have been matched in the online database that have version 9. Those are the only two versions I've seen referenced."*

**Impact.** §6.1 step 1 says unknown version → throw. That is still right. But V9 is not "unknown" — it is *known to exist and known to arrive from the Siemens cloud library*. A model pulled from the cloud, exported, and handed to Board Lab may well be V9. Throwing a generic "unrecognised version" on a version we have been told about is a poor experience and a support ticket.

**Suggested plan changes.** §6.1 step 1 — the version map carries V12 as supported and V9 as **known but unsupported**, with a refusal message that names it and says what to do. Add to §6.2: a V9-marked file is refused by name, not as "unknown version." Add a Follow-Up Item: obtain a V9 template if V9 models turn out to be common in the cloud library; the delta between V9 and V12 column maps is unknown and may be small.

---

## N-4 — Part-type parameter profiles and derivation procedures

**What was said.** Lucas: *"Based on part type, we can identify which component variations we want to extract based on what analysis we need to perform… we'll probably want to just extract a standard set of parameters for a given part type. For each parameter, we (and this can be AI assisted) can develop a procedure for how this parameter should be derived from datasheets."*

**What is built.** §11.1 lists four named rules and *"additional rules."* There is no notion of part type driving a required parameter set, and no representation of a derivation procedure.

**Why it fits the architecture unusually well.** This is the design's central inversion applied one level up: a human (AI-assisted) authors the procedure once per part-type/parameter pair; deterministic code applies it per part; the LLM's job is retrieval and drafting, never deciding. It also gives `model_draft` something the plan currently lacks — a definition of *done*. Today "the model is complete" means "the LLM stopped." Under this, it means "every parameter the profile requires for this part type has a value or a work item."

**Suggested approach.** Reference data, not tenant data — seeded like `pin_type`:

- `parameter_profile`: `part_type`, `parameter_key`, `required` (bool), `derivation_note`, `expected_source` (`table | graph | wca_guideline | assumption`).
- Phase 11 gains one generic rule, `required-parameter-missing`, driven by the profile — replacing an open-ended list of hand-written rules with a table an engineer can extend without a code change. That is a much better fit for a system whose users are the domain experts.
- `expected_source = 'graph'` is precisely Lucas's *"procedurally identify when we expect to need a graph"* — and it makes A-5's figure index actionable rather than decorative.

**Sizing honestly.** Seeding this needs engineering input Board Lab does not have: the parameter list per part type. It is not buildable from the repository. Start with five or six part types from Lucas, or defer the seed and build the mechanism. **Do not treat this as a small addition to Phase 11** — the mechanism is small, the content is a sustained engineering effort.

---

## N-5 — The requirement → analysis-type structure is missing (scope decision)

**What was said.** This is the heart of Lucas's reply and it deserves to be quoted rather than summarised: *"'Walking the netlist to emit findings' requires an overhead structure of 'what specific analysis am I doing against what requirements', out of a list of analyses-type-for-requirement pairs it needs to analyze against specifically to know it has completed the job… I think we should start with the structure, then get good at the pieces… Or how can I trust it?… Highly reviewable, confidence-evoking WCA is highly tabular, structured, standardized, and documented."*

**What is built.** `netlist → functional_block → tolerance_row → analysis_job`, where `analysis_job` is method × corner (EVA/RSS/MC × cold/ambient/hot), generated combinatorially by the permutation planner. There is **no requirement entity, no analysis-type entity, no link from a job to the thing it exists to demonstrate, and no completeness criterion.** Board Lab can currently say "here are 27 planned jobs." It cannot say "these 27 jobs discharge these 9 requirements, and requirement R-14 is not covered by any of them."

**Assessment — he is right about the gap, and partly right about the remedy.** Right, that a WCA whose completeness cannot be demonstrated against requirements is not a reviewable WCA, and that the structure is what buys trust. Right, that retrofitting structure onto a pile of generated findings is harder than building the structure first — that is the same argument §4A.0 makes about schema, and it has been correct twice already on this project.

Where I would push back: a full requirements-management surface is a large feature with **no ingest path in the design**. Requirements live in customer specs, ICDs and internal standards. Nothing in the plan reads any of those, and building that is a bigger job than the analysis pipeline it would serve. Adopting N-5 wholesale would restructure Phases 10, 11 and 17 and add an input class from nowhere.

**Recommended middle path — the skeleton, not the feature.** Add the structure so the dimensionality is present and honest, and let it be sparsely populated in v1:

- `requirement` — org- and revision-scoped, minimally `{ key, title, source_doc, text }`. Populated manually or by CSV import in v1. No parsing, no extraction.
- `analysis_type` — reference data: the list of analysis types (worst-case voltage margin, timing margin, power dissipation, thermal derating, …), seeded with whatever list Lucas can provide.
- `analysis_requirement` — the *"analyses-type-for-requirement pair"* Lucas names, linking a requirement to the analysis types that discharge it for a given block.
- `analysis_job.requirement_id` / `analysis_type_id` — every planned job cites what it is for. Jobs with no citation are visible as such rather than invisible.
- A **coverage view** in Phase 17: requirements × analysis types × jobs, showing what is covered, what is planned, what is unaddressed. This single screen is most of the trust Lucas is asking for, and it is mostly a query.

That is a few tables, one screen, and a citation column. It makes the tool *sensitive to the full dimensionality* — his words — without committing to requirements management. If the structure stays empty on the first program, nothing is lost; if it fills up, the pieces get better inside a frame that was right from the start.

**This is a scope decision, not a defect.** It should be taken deliberately, priced, and either adopted or explicitly declined in writing — not folded in quietly. If adopted, the schema half belongs in Phase 4A alongside N-1, for the same reason. If declined, say so to Lucas with the reasoning, because he raised it as a matter of whether the output can be trusted.

---

## N-6 — The "WCA guideline" is an input class nobody has mentioned

Lucas, in passing: *"Many require only reading 2 or 3 values out of a table in a datasheet, and a value or two from WCA guideline and some documented assumptions."*

There is an internal WCA guideline document that supplies parameter values, and Board Lab does not know it exists. The ingest registry (Phase 4) has eight artifact kinds; none is a standards or guideline document. The schema has a `reference_doc` enum value — and, per the Phase 0–4 review, no `reference_doc` table behind it. Interestingly, the original design apparently anticipated something like this and it was never built.

Two consequences: a model or tolerance value derived partly from the guideline has no citable source in the audit trail, and "documented assumptions" — which Lucas names as a first-class input — have nowhere to live except free text.

**Suggested plan changes.** Add `wca_guideline` (or a general `reference_doc` kind) to the Phase 4 ingest registry. Resolve the missing `reference_doc` / `reference_chunk` tables already noted as an undocumented gap in the Phase 0–4 review — this is the use case that justifies them. Ask for the guideline document. Modest work; it closes a real hole in traceability.

---

## N-7 — Greedy vs lightweight library: two different libraries, do not conflate

Lucas: *"We should eventually settle on whether we want our internal library to be greedy but stable/controlled, ingesting anything we use, or light weight, trusting in Siemens cloud models when available. I think for now, light weight is probably the right answer until we git gud."*

This is a decision about **their HLSA local library**, not about Board Lab's `device_model` table, and the two should not be merged in anyone's head:

- **HLSA's local library** is a tool-side cache of models sourced from Siemens. Lightweight is a reasonable call — Siemens' copy is authoritative and re-fetchable.
- **Board Lab's `device_model`** is the audit record for models *Board Lab drafted and an engineer approved at Gate 0*. It cannot be lightweight. It is the evidence that the approval happened, and it must survive the cloud copy changing underneath it. That is not a preference; it is what the append-only audit trail is for.

No plan change. Worth one sentence in `05-scope-and-files.html` so the distinction is written down before someone proposes "trusting the cloud" for Board Lab's library too.

---

## Net effect on the plan

**Unblocked:** Phase 7 (Q2 closed — A-1). Phase 6 partly (Q3 closed, Q1 resolved into a smaller optional step) — but still blocked on the physical Siemens files (A-4).

**Removed:** the E-2 second pipeline; Phase 9's datasheet-scoped `model_draft` entry point; the Phase 14 permission re-gate; Phase 6's vendor-library reader as specified; `.ots` output.

**Added, small and cheap:** `datasheet_figure` index (A-5); deterministic power/ground pin classification (A-6); conditional-formatting rule extraction (N-2); V9 named refusal (N-3); AVL reader/writer as an optional step pending a sample (A-3).

**Added, needing a decision:** `internal_pn` + `part_alias` (N-1 — **recommend adopting into Phase 4A now**); parameter profiles (N-4 — mechanism small, content is an engineering effort); requirement/analysis-type skeleton (N-5 — **the real decision in this assessment**); reference-doc ingest (N-6).

**Schedule.** A-1's removals roughly offset the small additions. N-1 adds perhaps an hour to Phase 4A. N-5's skeleton is a day or so across schema, planner and one page — and that is the skeleton, not the feature. Phase 4A's estimate of 1–2 hours should go to 2–3 if N-1 is adopted, more if N-5's schema comes with it.

**Should the plan change?** Yes — but only in phases that have not started, so under the stop-and-review protocol these are plan edits, not deviations. Phases 0–4 are `APPROVED` and untouched by everything above. The one item with a closing window is **N-1**: it wants to ride in `0004` alongside the E-3/E-4/E-5 changes, and Phase 4A is the last phase where that is free.

**Recommended order:**

1. **Now** — record the Q2/Q3 answers and strike them from §"Open Decisions". Free, and it stops the questions being re-asked.
2. **Before Phase 4A** — decide N-1 (recommend yes) and N-5's schema half (recommend the skeleton). Get a real BOM first for N-1.
3. **Before Phase 6** — land the Siemens PDF, templates and example models in the repository. Rewrite §6.1 step 6 per A-3.
4. **Before Phase 7** — nothing blocking. Add the figure index and narrow the indexer to lazy resolution.
5. **Before Phase 9** — Q6, Q7, and the N-4 parameter-profile content.

---

## Questions to send back

**Ryan**

1. **Can you export a sample AVL and send the file?** Any format. This is now the only machine-readable interface to HLSA's coverage, and the plan will not design against another unknown format (A-3).
2. **Can you send a real BOM** — one you would actually load into HLSA? Specifically: which column is the identity column, and does the BOM also carry manufacturer part numbers? Phase 5's parser and the `internal_pn` schema decision both hang on this (N-1).
3. **The Siemens PDF, the two templates, and three or four completed models** need to be files in the repository, not attachments. Can you copy them off the `J:` share? And are any of them NDA or export-controlled — do they need to stay out of version control (A-4)?
4. **Is a V9 template obtainable**, or should Board Lab simply refuse V9 by name and ask the engineer to re-save from V12 (N-3)?
5. **Q6 remains yours and Lucas's to scope.** When you do: the useful framing is not "1,500 pins" but "how many pins does a reviewer have to look at." Board Lab can classify power/ground pins from the netlist without the datasheet, which removes roughly half before anyone reviews anything (A-6).

**Lucas**

6. **Q7 — the component list and datasheets.** Five or six across distinct part classes (passive, diode, regulator, connector, MCU) would serve as both the Phase 9 evaluation corpus and the Phase 6 fixtures. Your offer of a call is the fastest route; one collection covers both.
7. **A first cut at part-type parameter profiles** — for, say, resistor, capacitor, diode, LDO and op-amp: which parameters do you extract as standard, and where does each normally come from (table / graph / WCA guideline / assumption)? This is the content behind N-4 and it cannot be derived from the repository.
8. **Can we have the WCA guideline document?** You named it as a source of values and Board Lab currently has no ingest path for it, which means anything derived from it has no citable provenance (N-6).
9. **On your structure-first argument** — it is well taken and the plan does not currently carry it. The proposal is a *skeleton*: requirements, analysis types, the analysis-type-for-requirement pairing, every planned job citing what it discharges, and a coverage screen showing what is unaddressed. Sparsely populated in v1, hand-entered, no requirements parsing. Does that meet what you mean by being sensitive to the full dimensionality, or is the substance of your point that the pairing list itself has to be authoritative before any of it is worth doing (N-5)?
10. **Conditional formatting** — good catch, and it may be the most reliable part of the model specification we can get, since it comes from Siemens' own file rather than our reading of their PDF. Do you know whether the template uses it for hard requirements only, or also for softer warnings? It changes whether the extracted rules can be trusted as-is (N-2).
