
Mike Berry <mb4540@gmail.com>
Attachments
Jul 25, 2026, 5:11 PM (7 days ago)
to Chad, David, Lawrence, Bob, Paul, Ryan, Chris

Hey all,

Super productive day.
Spent a good amount of time on the design of "Board Lab" (no i don't have a marketing degree) in the morning.
Produced the largest plan file in months (19 Phases)

Board Lab — Executive Summary
Board Lab automates PCB schematic analysis and worst-case corner analysis planning, with human approval gates at every critical step.

Schematic Copilot
Upload schematics, BOMs, and datasheets. The system reconstructs the netlist deterministically, then uses an LLM to draft device models from datasheets with per-field page citations. An engineer reviews and approves each model (Gate 0). Approved models enter an org-scoped library reusable across all future boards. A deterministic rule engine then walks the netlist to emit findings — the LLM never decides whether a finding exists, only drafts and explains.

WCCA
The pipeline divides the circuit into functional blocks (Gate 1), builds tolerance models across initial/tempco/aging/radiation axes using engineer-approved values (Gate 2), and plans worst-case corner permutations. All approvals are logged to an append-only audit trail.

CIRA Agent
A read-only chat agent with three sub-agents: Librarian (model/datasheet RAG with page citations), Analyst (circuit explanations), and Traceability Clerk (DO-254/AS9100 audit records). CIRA explains; it never mutates.

Status
Phases 0–4 approved. Phases 5–19 not yet started. 7 open engineering questions blocking downstream work. details in attached docs
 2 Attachments
  •  Scanned by Gmail

Ryan Swartzendruber
10:47 AM (20 hours ago)
+Lucas @Lucas Johnson, can you help with Q4 and Q7 questions below? Mike, I/we need to define the scope of what our tools are doing to answer Q6. Of those 1500

Mike Berry
11:29 AM (19 hours ago)
I will add this to this weekend work Sent from my iPhone On Jul 31, 2026, at 10:47 AM, Ryan Swartzendruber <RSwartze@jemba9.com> wrote: ﻿

Lucas Johnson
12:45 PM (18 hours ago)
to Ryan, me, Chad, David, Lawrence, Bob, Paul, Chris

That is a good question… I tried below.

 

Thanks,

Luke

 

From: Ryan Swartzendruber <RSwartze@jemba9.com>
Sent: Friday, July 31, 2026 9:48 AM
To: Mike Berry <mb4540@gmail.com>; Chad Fisher <cfisher@jemba9.com>; David Sedlock <DSedlock@jemba9.com>; Lawrence O'Donnell <lawrenceaodonnell@mailfence.com>; Bob Harr <bharr@jemba9.com>; Paul Rutt <prutt@jemba9.com>; Chris Long <clong@jemba9.com>
Cc: Lucas Johnson <ljohnson@jemba9.com>
Subject: Re: Weekend Status

 

+Lucas

@Lucas Johnson, can you help with Q4 and Q7 questions below?

 

 

Mike,

 

I/we need to define the scope of what our tools are doing to answer Q6.  Of those 1500 pins half will be power and ground, so it's not quite as daunting as it sounds.  This is another discussion item that I don't have a good feel for.

 

Answers to the questions in SUR-07-25-2026.html:

 

Open Questions — Awaiting Engineering Input

Raised by the Schematic Datasheet Mechanization thread of July 23–24 (Swartzendruber, Johnson). The solution design has been amended where the answer was not in doubt; these are not yet answerable and are deliberately not being guessed. Full analysis: Jemba9_Ecosystem/documents/plans/active/01-board-lab-engineering-feedback-review.md

Ryan Swartzendruber — HyperLynx Schematic Analysis (HLSA) / model templates

Q2For a datasheet covering ~200 orderable part numbers where the BOM uses three — should Board Lab model all 200 at import, or only the parts on a live BOM?Highest cost

Blocks: Phase 7 (bulk drafting), Phase 14 (trigger + permission). Why it matters: largest cost driver in the system — guessing "all" risks a ~100× token and Gate 0 review bill per datasheet. Ryan asked whether models can be created "for all orderable parts in the datasheets when they are first imported"; the plan currently drafts only unmodelled parts on a board.

 
Ryan:
Only the parts needed for a given design.  Sorry, I'm used to my AI usage being free.  Also I failed to recognize the need for human review.

 
Lucas
Completely agree.

 

Q1Can the HLSA part-model library be exported in bulk, or is it reachable only one part at a time through the application UI? What format?Blocking

Blocks: Phase 6 vendor-library reader; import precedence in Phase 9. Context: the solution design previously stated no HLSA library existed — corrected July 25 after Ryan confirmed one does, with incomplete part-number coverage. Matching is cheaper than drafting, so this should precede drafting. Octopart/Nexar is not a substitute — it carries no HyperLynx models.

 
Ryan:
This topic has multiple levels, I'll try to outline them from first import of a BOM into a project.

The application queries a Siemens cloud database based on Internal Part Number (as defined by our BOM) to look for existing models.  If matches are found with the cloud library (Manufacturer Part Number) the tool will use the existing model.  A link is made in the imported project BOM.

For parts on the BOM for which a matching part number is not found:

    The user can execute queries in the Siemens cloud database to look for existing models with a different part number that may be applicable/equivalent.

        The user can link existing models (Manufacturer Part Number) to Internal Part Numbers on the BOM through the AVL (approved vendor list, bad naming) by equating the BOM (internal part number) with the cloud library (manufacturer part number).  The tool AVL list can be exported or imported (only adds\replaces to AVL on import, doesn't delete in the tool).

            From what I see this links the model on the project BOM and then changes to the AVL don't affect the project BOM, it needs to be manually changed even if the AVL changes.

 

It appears to me that everything gets translated to the local tool AVL and project BOM.  Models from the cloud appear to get copied to the local library and assigned to parts in the project BOM.  New projects which have part numbers that match the local library or get matched through the AVL will have their models assigned in the new project BOM.

 

To access the existing HLSA library provided by Siemens would need to go through the application if it's even possible.  My reading found nothing in terms of another way to interface with their library.

 

This needs discussion, I'm not sure of the best path forward.

 
Lucas:
To add to this slightly and as an aside (Mike and Lawrence can skip this), the user can link models from a similar part number to the project and optionally also add them to the AVL, which then makes the link automatic for all projects. The AVL is functionally a component of our internal library. Additionally, models from the cloud are used for a project if the tool finds one, but that model is only pushed to our own library if we deliberately choose to make that model “ours” by ingesting it to our internal library.

 

We should eventually settle on whether we want our internal library to be greedy but stable/controlled, ingesting anything we use, or light weight, trusting in Siemens cloud models when available. I think for now, light weight is probably the right answer until we git gud.

 

 

Q3Does anyone actually author model templates in LibreOffice, or is .xlsx universal in practice?Low

Blocks: Phase 6 — nothing hard. Detect-and-refuse-clearly is correct either way; only the support effort differs. Note: .ots is OpenDocument, which the chosen parser (exceljs) cannot read at all, so support would be a new dependency rather than a detection tweak.

 
Ryan:
No, I think .xlsx can be considered universal.  I thought there could be some benefit to having the agent create the model in .ots.

 
Lucas:
Agree. .xls/.xlsx files can be imported to and exported from LibreOffice without trouble. No reason to not standardize on that file format I think.

 

Q5Can you send the Siemens instruction PDF, the two templates and the example models? And confirm the template version matches the V12 the 54 seeded pin types came from?Blocking

Blocks: Phase 6 entirely. These are the written specification for the model parsers — including the legal pin-name character set (Ryan noted + - % / and spaces are disallowed). Referenced as attached in the July 24 email but not yet in the repository; without them the column map would be reconstructed from guesswork.

 
Ryan:
I sent the PDF with two templates as attachments and a path to example models (below) already.  The example models have the version embedded and all indicate the version is 12.  I see other models that have been matched in the online database that have version 9.  Those are the only two versions I've seen referenced.

J:\Engineering\Tools and Software\Siemens\Schematic Analysis\HLSAWorkShopData\HLSAWorkShopData\Data\Datasheets and Models\Completed Models\

Lucas Johnson — datasheet content and retrieval expectations

Q4Of the parameters you need from graphs — which are genuinely load-bearing for worst-case analysis, versus nice to have?Scope

Affects: Phase 11 rule design. Context: reading values off plotted curves is out of v1 by decision — the text extractor returns no vector or raster graphics, so curves never reach the system. If the load-bearing list is short, engineer-supplied values with citations may close the gap without graph extraction. The planned mitigation is a rule that asks for the value rather than silently omitting the parameter.

 

For worst case analysis, graphs are used somewhere between occasionally and commonly, and the scope is limited to worst case component variability, which precedes circuit level analysis. Based on part type, we can identify which component variations we want to extract based on what analysis we need to perform on the circuit using that component. To keep it simple, we’ll probably want to just extract a standard set of parameters for a given part type. For each parameter, we (and this can be AI assisted) can develop a procedure for how this parameter should be derived from datasheets. Many require only reading 2 or 3 values out of a table in a datasheet, and a value or two from WCA guideline and some documented assumptions. We can procedurally identify when we expect to need a graph, and it can be as manual as necessary to get that data out.

 

On the design side, there is a lot of value in an AI assistant that at least knows what graphical data is present even if it never went through a complete curve extraction effort, and creating a catalog of what data is where. Even if it can’t simply tell me the answer to anything I might want to know about the part, a tool that can open the pdf, highlight the region where it understands the relevant information to be (or something effectively similar), is still huge. Or through this indexing, it can identify HOW a question should be answered, and what components necessary for that answer are or are not in the datasheet.

 

Others may disagree with me on this and interject: human-readable/understandable WCA is based on circuit or interface performance against requirements (internal and external), which inform the individual analyses which must be run against that circuit. “Walking the netlist to emit findings” requires an overhead structure of “what specific analysis am I doing against what requirements”, out of a list of analyses-type-for-requirement pairs it needs to analyze against specifically to know it has completed the job. Each of those analysis types are unique facets of the circuit where different component parameters are important and unimportant. What I’m trying to get at is I think a philosophical approach. What I hear in discussions sounds to my ears like “start with the pieces and then later get good at the structure”, and I think that’s backward. I think we should start with the structure, then get good at the pieces. Build in from the beginning the foundation or scaffolding that is sensitive to the full dimensionality even if it’s dramatically simplified for what we’re doing for a specific program or for what angles we take to automate it. Or how can I trust it? And this isn’t to say we aren’t already starting with the structure or that what I’m saying is even significantly applicable, in which case, all I’m saying is repetitive and for emphasis or just my two cents and feel free to nod with an eye roll. Highly reviewable, confidence-evoking WCA is highly tabular, structured, standardized, and documented and I just want to make sure that’s retained. Because to me, a tool that enforces or guides that structure is more powerful than a tool that automates the pieces, even if having both would be nice. Again, I could be totally off base since I don’t have much insight into this effort.

 

Q6For the FPGA and MCU cases — is a complete pin model the goal, or only the pins participating in the interfaces being analysed?Scope

Affects: Phase 9 extraction strategy and Gate 0 review design. Why it matters: a 1,500-pin device does not fit one prompt and its pin table spans dozens of pages; a 1,500-row approval screen is a formality, not a review. Answer determines how much extraction machinery is worth building.

Q7Can you share the component list and datasheets you referenced? They would become the evaluation corpus for model drafting.Helpful

Affects: Phase 9 test corpus — currently the closest thing we would have to acceptance criteria for extraction quality.

 

One thing that occurs to me is that we’ll want to extract the conditional formatting from the HSLA model template excel file to use as rules for what information must be populated. I’m still training myself on how to model, which is an important step in being able to review any automated model production. If what you need here isn’t what Ryan just provided, shoot me an email or Teams invite to talk it through.