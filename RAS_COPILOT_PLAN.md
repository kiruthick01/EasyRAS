\# RAS Copilot — Project Plan

\*(working title — swap freely, see naming discussion)\*



\## ⚠️ Core Constraint (read first — applies to every phase, non-negotiable)

> Convenience never costs completeness. Every simplification must ship with a

> documented, ≤2-click escape hatch back to full HEC-RAS parameter access.

> A phase that cannot guarantee this does not ship.



\---



\## Instructions for AI Assistant (Claude or any LLM executing these phases)



Any assistant executing this plan — especially Phase 0 — must follow these

research rules. Violating them breaks the core constraint, since an

incomplete or hallucinated feature catalog makes every later "completeness"

claim false.



1\. \*\*Never hallucinate HEC-RAS fields, dialogs, or menu structures from

&#x20;  memory alone.\*\* Every catalog entry must be verified against a primary

&#x20;  source, not recalled and assumed correct.

2\. \*\*Primary sources, in priority order:\*\*

&#x20;  - The official HEC-RAS User's Manual (USACE Hydrologic Engineering Center — search for the current version, since it updates across releases)

&#x20;  - The official HEC-RAS Reference/Hydraulic Reference Manual (for what each field does, valid ranges, units)

&#x20;  - In-app Help documentation (HEC-RAS's own Help menu), if the assistant has direct UI or vision access to the software

&#x20;  - Screenshots or UI walkthroughs supplied directly by the user, when the assistant has no direct software access

3\. \*\*Web search is appropriate\*\* to locate current manual versions and release notes. Treat forums, blog posts, and third-party tutorials as supplementary only — never as the sole basis for asserting a field exists or how it behaves.

4\. \*\*When uncertain\*\* whether a field exists, which tier it belongs to (`common` / `situational` / `rare`), or what its valid range/units are — the assistant must flag this explicitly to the user and ask for confirmation from the live software, rather than guessing or quietly omitting it. Silent omission is a direct violation of the core constraint.

5\. \*\*Every catalog entry must record its source\*\* — manual name + section/page, or "confirmed via user screenshot, \[date]" — so completeness is auditable later, not just asserted.

6\. \*\*Before marking any phase's acceptance criteria as met\*\*, re-check the actual output against the Phase 0 catalog line by line — not from memory of having built it.



\---



\## Problem Statement

HEC-RAS's native UI suffers from complex, non-linear navigation across

Geometry Editor, Steady/Unsteady Flow Editor, 2D Flow Areas, Structures,

Sediment, Water Quality, RAS Mapper, and Plan/Run Manager. Users —

especially newcomers — struggle to locate the right tool for a given task

and to understand which fields matter for their project type. RAS Copilot

is a wrapper application that sits on top of HEC-RAS's engine (via COM

automation) and replaces the \*interaction layer only\* — never the feature

set.



\## Tech Stack (baseline — revisit after Phase 0)

\- \*\*Bridge:\*\* Python + `pywin32` / `pythonnet` → HEC-RASController (COM)

\- \*\*UI:\*\* PyQt/PySide (desktop) — or FastAPI + React if a web-based demo is preferred

\- \*\*Data:\*\* SQLite for project templates + a JSON-based "feature catalog" (Phase 0 output)

\- \*\*Search index:\*\* lightweight fuzzy search (e.g. `rapidfuzz`) over the feature catalog



\---



\## Phase 0 — Feature Audit (Foundation — do not skip)

\*\*Prompt:\*\*

> Following the "Instructions for AI Assistant" section above, catalog every

> dialog, tab, and field across HEC-RAS's modules: Geometry Editor, Steady

> Flow Editor, Unsteady Flow Editor, 2D Flow Areas, Structures (bridges,

> culverts, gates, weirs, pumps), Sediment Transport, Water Quality,

> Plan/Run Manager, and RAS Mapper. For each field, record: module, dialog

> path, field name, data type, valid range/units, source reference, and a

> tag — `common` / `situational` / `rare` — based on how frequently it's

> used across typical project types (urban drainage, riverine flood study,

> culvert design). Flag any field you cannot verify against a primary source

> instead of guessing. Output this as a structured JSON/CSV "feature

> catalog" — this file is the single source of truth for every later

> phase's completeness claims.



\*\*Deliverable:\*\* `feature\_catalog.json` (or `.csv`), with a `source` column per entry

\*\*Acceptance criteria:\*\* Every field visible in native HEC-RAS menus has a corresponding, sourced catalog entry. No field is invented or guessed.



\---



\## Phase 1 — Core Engine Bridge

\*\*Prompt:\*\*

> Build a Python bridge to HEC-RASController via COM automation that can

> read AND write every field listed in `feature\_catalog.json` — not just the

> common ones. Validate correctness by round-tripping real `.prj` files:

> open in native RAS → read all fields via the bridge → write them back

> unchanged → confirm RAS reads the result identically (diff the files).



\*\*Deliverable:\*\* `ras\_bridge` module with full read/write coverage + round-trip test suite

\*\*Acceptance criteria:\*\* 100% of Phase 0's cataloged fields are readable and writable through the bridge, proven by passing round-trip tests — before any UI work begins.



\---



\## Phase 2 — Common-Case Guided Flow + Tool Search (Convenience Layer)

\*\*Prompt:\*\*

> Build a wizard-style guided flow: project type selection → geometry import

> → boundary conditions → plan setup → run. Show only `common`-tagged fields

> (per the Phase 0 catalog) by default. Every screen must have a visible,

> always-present "Advanced" expand control that reveals `situational` and

> `rare` fields for that same section inline — never on a separate hidden

> page. Alongside this, build a global tool-search / command palette

> (keyboard-triggerable, e.g. Ctrl+K) that fuzzy-searches across ALL fields

> in the catalog — common, situational, and rare — by name, module, and

> synonym, and jumps the user directly to that field's location regardless

> of which screen they're currently on.



\*\*Deliverable:\*\* Guided wizard UI + global command-palette search

\*\*Acceptance criteria:\*\*

\- Every `common` field reachable within the wizard with zero menu-hunting

\- Every field in the entire catalog reachable via search in ≤3 keystrokes + selection

\- Search results ranked by relevance (exact name match > synonym > module match)

\- No field is exclusive to search-only access if it's also reachable via Advanced expand



\---



\## Phase 3 — Full Parameter Access Layer (Completeness Guarantee)

\*\*Prompt:\*\*

> Build a raw/full parameter view per module (Geometry, Flow, Structures,

> Sediment, Water Quality, Plan Manager) that exposes 100% of the fields in

> `feature\_catalog.json`, structurally mirroring HEC-RAS's own completeness

> but styled consistently with the rest of the app (tooltips, unit labels,

> inline validation). From any screen in the app, a user must be able to

> reach any field's full raw view in ≤2 clicks.



\*\*Deliverable:\*\* Full raw parameter views, one per module

\*\*Acceptance criteria:\*\* Manual audit against Phase 0 catalog confirms zero missing fields. Escape-hatch click-path measured and documented per module.



\---



\## Phase 4 — RAS Mapper Simplification

\*\*Prompt:\*\*

> Build a lightweight map view (Leaflet or Mapbox) covering the 80%

> geospatial case: terrain/DEM overlay display, cross-section placement by

> click, flood-extent visualization. Add an explicit "Open in RAS Mapper"

> button on every geospatial screen as the escape hatch for anything the

> lightweight view doesn't yet support. Use real usage data from Phases 2–3

> to prioritize what this lightweight view covers first.



\*\*Deliverable:\*\* Lightweight map module + native RAS Mapper handoff

\*\*Acceptance criteria:\*\* Every RAS Mapper capability is either replicated or has a working one-click handoff to native RAS Mapper — none silently dropped.



\---



\## Phase 5 — Validation, Testing, Parity Audit

\*\*Prompt:\*\*

> Run identical projects through native HEC-RAS and through RAS Copilot,

> diff every output field. Cross-check final coverage against the Phase 0

> feature catalog and mark each field as verified-covered. Produce a parity

> report.



\*\*Deliverable:\*\* `parity\_audit\_report.md` with per-field coverage status

\*\*Acceptance criteria:\*\* 100% of Phase 0 catalog entries marked verified; any gap is either fixed or explicitly documented with its escape-hatch path.



\---



\## Cross-Phase Rule

No phase is "done" until its acceptance criteria are checked against the

Phase 0 feature catalog. The catalog is the contract — "convenience never

costs completeness" is enforced by measurement and sourcing, not by intention.

