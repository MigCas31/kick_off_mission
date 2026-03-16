---
aliases:
- Catalogue
- Templates
- Catalogue Group
- Domain Catalogue Group
- Question-Answer Catalogue Group
country:
- all
created: '2026-02-04'
created_by: '[[anders]]'
description: Defines catalogues as curated databases of templates used for structured
  data capture in Plans. Covers template structure, filter and data columns, the two
  group types (Question-Answer and Domain), and how catalogues connect to task patterns,
  note types, variables, and generators.
doc_type: ontology
feature_area:
- platform
- technical-installations
keywords:
- catalogue
- template
- filter column
- wizard
- catalogue group
- question-answer catalogue
- domain catalogue
- catalogue registration
- note type
- template selection
lifecycle:
- onsite/room/annotation_mode
- onsite/building_element/annotation_mode
- onsite/building/annotation_mode
- onsite/property/annotation_mode
lifecycle_level:
- building_element
- room
- building
- property
lifecycle_moment:
- annotation_mode
lifecycle_phase:
- onsite
related:
- '[[tasks]]'
- '[[variables]]'
- '[[note_types]]'
- '[[generators]]'
- '[[catalogue_registration]]'
- '[[_architecture]]'
report_domain:
- electrical
- epc
- condition
- lead
- gas
- asbestos
status: active
steward: '[[anders]]'
summary: Ontology definition for Catalogues — curated databases of templates that
  surveyors select from rather than fill in manually. Covers template structure, filter
  vs data columns, the two group patterns (Question-Answer for checklists, Domain
  for registration tasks), and the relationships to task patterns, note types, variables,
  and generators.
tags:
- ontology
- definition
- product
title: Catalogues
updated: '2026-02-19'
---

# Catalogues

## Simple Description

A catalogue is a curated database of templates. Each template is a structured record — a row with named fields — that the surveyor selects from during capture rather than fills in manually. When a template is matched, its data fields are injected into the captured note automatically.

Catalogues exist because surveyors recognize things by parameters but rarely recall exact codes or model numbers. A surveyor knows they are looking at a "Bosch condensing gas boiler, installed around 2015" — not the exact product string. A catalogue encodes that knowledge as filter steps, letting the surveyor drill down to the right template by answering one parameter at a time.

Catalogues are distinct from [[variables|enums]] (simple closed value lists with no per-item metadata) and from [[generators]] (step-by-step configuration builders with no pre-existing template database). If an item has multiple structured fields per option, it is a catalogue. If it is a single value from a fixed list, it is an enum. If the user builds the configuration from scratch by answering questions, it is a generator.

## Structure

### Templates

A template is a single entry in a catalogue. It carries a set of typed fields defined by the catalogue schema. Examples:

- **Defect code template** (French Electrical, NF C 16-600 Annexe B): anomaly code, description, severity class (DGI / A2 / A1), compensatory measure flag.
- **Technical installation template** (Danish EPC): brand, model, rated output, energy class, applicable room types.
- **Material template** (EPC): material type, thermal conductivity, typical thickness range.

Templates are authored data. Surveyors select them; they do not create them in the field.

### Filter Columns and Data Columns

Every catalogue column is one of two kinds:

| Column type | Role | Example |
|---|---|---|
| Filter column | Drives wizard steps; each answer narrows the result set | Control sheet (B.1–B.11), Brand, Severity |
| Data column | Stored on the matched template; not used for selection | Anomaly description, model number, compensation flag |

Filter columns define the wizard sequence. The order matters: broader attributes (control sheet, brand) come before narrower ones (specific code, model variant). Data columns carry the reference data that the matched template injects into the note.

### Catalogue Groups

Catalogues can be organized into a group. The group holds the taxonomy (categories and types), display configuration, and cross-type constraints. Two group patterns are used.

#### Question-Answer Catalogue Groups

A Question-Answer Catalogue Group pairs two catalogues used together by a [[tasks#Checklist with Answers|Checklist with Answers]] task:

1. **Questions catalogue** — the set of control points, with filter columns for context (room type, building type, etc.)
2. **Answers catalogue** — the answer options per control point, filtered by question ID

The pairing allows each control point to have its own set of valid answers. The questions catalogue determines which control points appear; the answers catalogue provides the options for each.

**Example:** French Electrical control sheets (NF C 16-600 Annexe B). The questions catalogue holds control points B.1.a, B.1.b, B.2.a, etc., filtered by room type. The answers catalogue holds Compliant / Non-compliant / Non-verifiable per control point, filtered by question ID.

#### Domain Catalogue Groups

A Domain Catalogue Group is a set of related catalogues organized by category and type. The group powers a [[catalogue_registration|Catalogue Registration]] task: the surveyor sees a category dashboard and works through types systematically, registering what is present.

The group holds the taxonomy (categories → types) and display rules. Each child catalogue holds the templates and filter sequence for one type.

**Examples:**

- Danish EPC Technical Installations: categories include heating, domestic hot water, ventilation, solar; each backed by a child catalogue of product templates.
- EPC materials: one child catalogue per building element type (outer walls, windows, floors, ceilings).

## How Catalogues Relate to Other Concepts

**Tasks and task patterns.** The [[tasks#Catalogue Registration|Catalogue Registration]] task pattern is built around a Domain Catalogue Group. The category dashboard IS the task screen; the wizard and parameter form are the per-item capture sub-flow. The [[tasks#Checklist with Answers|Checklist with Answers]] task pattern uses a Question-Answer Catalogue Group when each control point has data-bearing answer options. See [[catalogue_registration]] for the full Catalogue Registration flow and [[tasks]] for all five task patterns.

**Note types.** A note type references a catalogue for template selection. Completing the wizard match pre-fills the note type's data fields from the matched template. How a note type binds to a catalogue — via `catalogueRef`, `wizardMode`, and multi-entry capture configuration — is defined in [[note_types]].

**Variables and enums.** A variable of kind `catalogue` references a catalogue for its value. When the data is a simple closed list (a value plus a label, nothing more), use an enum instead. See [[variables]] for the boundary between the `catalogue` and `enum_single` variable kinds, and for how catalogue fields appear in forms.

**Generators.** Generators are the complement to catalogues: a catalogue finds an existing pre-built thing; a generator describes a configuration the user builds step by step with no template database behind it. Both can power a [[catalogue_registration|Catalogue Registration]] task. See [[generators]] for the full comparison, including the contrast between wizard filter fields (shared pool, cascading) and generator steps (sparse dependency tree, no shared pool).

## Common Mistakes

| Mistake | Why it's wrong | How to fix |
|---|---|---|
| Using an enum where a catalogue is needed | Enums carry no per-item metadata; anything with multiple structured fields per option (code + description + severity + flag) needs a catalogue | Create a catalogue with those fields as columns |
| No filter columns defined | The wizard has nothing to filter on; users face the full unnarrowed list | Add filter columns for the attributes the standard uses to organize items |
| Pairing answers to wrong catalogue | A Q-A group only works if the answers catalogue filters by question ID | Ensure the answers catalogue has the question ID as a filter column |
| Putting configuration logic in a catalogue | Catalogues hold pre-built templates, not user-built configurations | Use a [[generators\|generator]] for step-by-step configuration where no template database exists |
| Modelling a Q-A group as a single flat catalogue | A single catalogue cannot cleanly serve both as question list and per-question answer set | Use two catalogues linked by question ID |

## Glossary

| Term | Definition |
|---|---|
| Catalogue | A curated database of templates that the surveyor selects from during capture. |
| Template | A single entry in a catalogue; a structured record with filter and data fields. |
| Filter column | A catalogue field used to narrow the result set in the wizard. |
| Data column | A catalogue field stored on the matched template; not used for filtering. |
| Catalogue group | A domain container grouping related catalogues under a shared taxonomy. |
| Question-Answer Catalogue Group | A catalogue group pairing a questions catalogue with an answers catalogue; used by Checklist with Answers tasks. |
| Domain Catalogue Group | A catalogue group organizing items by category and type; used by Catalogue Registration tasks. |

---

## Agent Decision Guide

<!-- INJECTED INTO: extract_variables.py, cross_link_variables.py, review_entities.py -->
<!-- LAST REVIEWED: 2026-02-19 -->

### When to create a catalogue (vs enum vs generator)

```
Does the standard define a list of items with MULTIPLE fields per item
  (code + description + severity + flags)? ────────── CATALOGUE
Does it define a simple closed list of values
  with no per-item metadata beyond a label? ───────── ENUM
Does it describe a configuration the user builds
  step-by-step with NO pre-existing template DB? ──── GENERATOR (see [[generators]])
```

**Catalogue signals in a standard:**
- A reference table with columns (code, description, severity, applicability, compensation flag)
- "Select from the following defects / materials / control points" where each option carries structured data
- Paired question + answer sets where answer options differ per question

### Catalogue vs enum boundary

| Standard says | Entity |
|---|---|
| "Choose: DGI, A2, A1" (just values, no per-item metadata) | Enum on a variable |
| "Anomaly B.1.a: Missing AGCP, severity DGI, no compensation" (structured row) | Catalogue template |
| "Control point B.1.a → answers: Compliant, Non-compliant" (paired per question) | Q-A Catalogue Group |

### Group type assignment

| Signal in the standard | Group type |
|---|---|
| Questions paired with per-question answer sets | Question-Answer Catalogue Group |
| Items organized by category/type (materials by element, installations by system) | Domain Catalogue Group |

### Task pattern implied by group type

| Group type | Task pattern | Why |
|---|---|---|
| Question-Answer Catalogue Group | Checklist with Answers | The task works through control points; answers are selected from the answers catalogue |
| Domain Catalogue Group | Catalogue Registration | The task shows a category dashboard; each item is registered via wizard + parameter form |
