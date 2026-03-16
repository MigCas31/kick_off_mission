---
aliases:
- Variables
- Variable Kinds
- Enums
country:
- all
created: '2026-02-04'
created_by: unknown
doc_type: ontology
feature_area:
- platform
keywords:
- variable
- enum
- data capture
- value source
- inheritance
- computation DAG
- catalogue
- generator
- spatial computed
- naming convention
lifecycle:
- offsite-pre/report
- onsite/building/on_creation
- onsite/storey/on_creation
- onsite/room/on_creation
- onsite/room/annotation_mode
- onsite/building_element/annotation_mode
- offsite-post/report/on_completion
lifecycle_level:
- building
- building_element
- report
- room
- storey
lifecycle_moment:
- annotation_mode
- on_completion
- on_creation
lifecycle_phase:
- offsite-post
- offsite-pre
- onsite
related:
- '[[_architecture]]'
- '[[app_lifecycle]]'
- '[[catalogues]]'
- '[[generators]]'
- '[[levels]]'
- '[[note_types]]'
- '[[spatial_capture]]'
- '[[tasks]]'
status: active
steward: '[[anders]]'
summary: Defines variables as the atomic unit of data capture in Plans. A variable
  is one question, one answer -- a boolean, a number, a choice from a list, a selection
  from a catalogue, or a value computed from other variables. Covers variable kinds,
  enums, value sources, cross-report inheritance, the computation DAG, naming conventions,
  and the full frontmatter schema for variable files.
tags:
- ontology
- definition
- product
title: Variables
updated: '2026-02-19'
---

# Variables

A variable is the atomic unit of data capture in Plans. One question, one answer.

Every piece of information the surveyor records, the system computes, or the report outputs is a variable. The building year is a variable. Whether a safety device is present is a variable. The energy efficiency rating is a variable. A wall material selected from a catalogue is a variable. The GPS coordinates auto-filled by the device are a variable.

Variables live inside [[tasks]]. The task pattern determines how variables are presented to the surveyor -- as form fields, checklist questions, annotation capture fields, or catalogue selections. See [[_architecture]] for the UI concepts and [[tasks]] for the 5 task patterns.

## What makes variables powerful

A variable is not just a form field. It is a unit of data that can have consequences:

- **A variable can trigger other variables.** A boolean `has_swimming_pool` = YES activates an entire set of pool safety variables. A `room_type` = bathroom activates wet-room checks across multiple reports.
- **A variable can inherit its value.** `building_year` is entered once at building level and inherited by every report and task below. The surveyor never enters it again.
- **A variable can be computed from other variables.** `overall_compliance` is derived from all checkpoint results. `energy_label` is computed from hundreds of input variables via a calculation engine.
- **A variable can be a choice from a catalogue.** When the surveyor selects a heat pump model from the [[catalogues|catalogue]], that selection IS a variable (kind: `catalogue`). The catalogue template then pre-fills other variables (efficiency, capacity, refrigerant type).
- **A variable can be a step-by-step configuration.** When no catalogue template matches, the surveyor builds a description through sequential choices using a [[generators|generator]]. The combination of choices IS the variable output.

## Variable kinds

The kind determines what type of data a variable holds and what input the surveyor sees.

| Kind | What it is | Example |
|------|-----------|---------|
| `boolean` | Yes/no toggle | Is the safety device present? |
| `number` | Numeric value, optionally with a unit | Earth resistance in ohms, height in cm |
| `string` | Short single-line text | Building name, reference number |
| `rich_text` | Multi-line formatted text | Summary comments, recommendations |
| `enum_single` | Single choice from a controlled list | Anomaly severity (DGI / A2 / A1) |
| `enum_multiple` | Multiple choices from a controlled list | Applicable compensatory measures |
| `tags` | Multiple freeform labels | Additional classification tags |
| `catalogue` | Selection from a template database via [[catalogues\|catalogue]] drill-down | Heat pump model, wall material, defect code |
| `generator` | Step-by-step configuration via a [[generators\|generator]] | Material layer composition, installation description |
| `computed` | Derived from other variables -- not interactive | Energy label, compliance status, spatial zone label |

The first 7 kinds (`boolean` through `tags`) are direct data entry -- the surveyor types, selects, or toggles. The last 3 (`catalogue`, `generator`, `computed`) involve complex interactions or derivation logic defined in their respective ontology documents.

## Enums

An enum is a controlled value list used by `enum_single` and `enum_multiple` variables. It defines the allowed options the surveyor can choose from.

Each enum has a `key`, a set of items, and each item has a `value` (machine-readable, English snake_case) and `label` (human-readable, localized).

**When two or more variables share the same value list, they reference a shared enum.** For example, multiple checkpoint variables across different control sheets all use the same `checkpoint_result` enum (yes / no / not_verifiable / not_applicable). The enum is defined once and referenced by each variable via `enum_ref`.

Enums can also serve as filters in [[catalogues]] -- the same value list that drives a form dropdown can filter catalogue entries.

### Enum vs catalogue

| Question | Enum | Catalogue |
|----------|------|-----------|
| How many items? | Small, closed list (3-30 options) | Large, hierarchical database (tens to hundreds) |
| Structure? | Flat list of values | Multi-level with categories, types, templates |
| What comes back? | A single value | A template that pre-fills multiple variables |
| User interaction? | Dropdown or segmented control | Multi-step [[wizard]] drill-down |

If the surveyor picks from a short list and gets one value back, it's an enum. If they drill through a hierarchical database and get a template with pre-filled fields back, it's a catalogue.

## Value sources

Every variable has a value source that describes how it gets its initial value before the surveyor interacts with it.

| Value source | What it means | Example |
|-------------|--------------|---------|
| `user_input` | Surveyor enters the value directly. No prefill. | A measurement reading, a free-text observation |
| `static_default` | Hardcoded default value. Surveyor can change it. | A numeric field defaulting to 0 |
| `inherited` | Read from a parent level at runtime. May be read-only or editable. | Building year inherited from building setup into room-level tasks |
| `system_provided` | Platform provides automatically (GPS, timestamp, device ID, logged-in user). | Inspection date, surveyor name, GPS coordinates |
| `computed` | Derived from other variables via formula, lookup table, or cross-level aggregation. Non-interactive. | Energy efficiency from a regulatory calculation, compliance status from checkpoint results |
| `derived` | Simple logic over sibling variables at the same level. Non-interactive. | Overall pass/fail from a set of individual checkpoint results |

**`derived` vs `computed`:** Use `derived` for simple comparisons over siblings (e.g., "all checkpoints pass → compliant"). Use `computed` when external lookup tables, cross-level aggregation, or regulatory formulas are involved (e.g., "minimum conductor cross-section from NF C 15-100 Table 52E").

For `inherited` variables, the source is specified by a source level and source field: "read `climate_zone` from `case` level." The variable can be editable (surveyor can override) or read-only.

## Cross-report inheritance

The `inherited` value source is how the shared data layer principle works in practice. See [[levels#Levels as a shared data layer]] and [[_architecture#Capture once, inherit everywhere]].

When a variable is set at a level (building year at building setup, wall materials at storey primaries, room type at room creation), it becomes available to ALL reports and tasks at that level and below -- not just the report that captured it.

**Examples:**

| Variable | Set at | Inherited by |
|----------|--------|-------------|
| `construction_year` | Building setup (EPC) | Electrical, Gas, Lead, Condition reports at building level |
| `wall_substrate` | EPC material assignment | Lead report for pre-filling substrate in XRF measurement notes |
| `heating_system_type` | Building setup (EPC) | Gas and Electrical reports |
| `room_type` | Room creation | Conditional task activation across ALL active reports (bathroom triggers wet-room checks for Electrical, zone measurements for Lead) |

When designing variables, always ask: could this value be captured once and inherited, rather than re-entered per report?

## The computation DAG

Variables can depend on other variables, forming a directed acyclic graph (DAG). The DAG has two edges:

| Edge | Meaning |
|------|---------|
| `inputs_from` | This variable reads from these other variables |
| `outputs_to` | This variable feeds into these other variables |

The DAG is used for:
- **Reactive computation:** when a variable changes, all downstream variables recompute
- **Validation ordering:** ensure inputs are available before computed variables evaluate
- **Impact analysis:** trace which variables are affected by a change

See [[generators]] for how the DAG works in complex computation chains.

## Computed spatial variables

The spatial capture system derives variables from RoomPlan geometry and AR pin placement. These are `computed` variables -- not entered by the surveyor. They are defined in detail in [[spatial_capture]].

Key spatial variables: `zone_label` (which wall), `height_above_floor`, `surface_type`, `nearest_wall`, `cardinal_direction`, `near_window`, `ai_placement_description`. Everything about WHERE an annotation is should come from the spatial pipeline, not manual entry.

## Where variables live in the hierarchy

Every variable belongs to a [[levels|level]] in the inspection hierarchy. Assign to the highest level where the data makes sense.

| Level | What lives here | Examples |
|-------|----------------|---------|
| `case` | Job-wide data | Inspection date, surveyor, client info |
| `report` | Report-specific metadata | Report type settings, scope selections, assessor info |
| `property` | Outdoor / site-level data | Outdoor installations, garden structures |
| `building` | Whole-building data | Year, type, address, earth resistance, main panel |
| `storey` | Floor-specific data | Floor separation type, material primaries, ceiling height |
| `room` | Room-specific data | Room type, bathroom zone compliance, socket counts |
| `building_element` | Room geometry surfaces (from RoomPlan) | Wall material, window condition, floor type |
| `object` | Inspectable things that are not room geometry | Heat pump, radiator, fuse box, electrical board |

Objects have flexible parents -- they can belong to a room, property, building, building element, or another object. See [[levels#Object architecture]].

## Naming conventions

All machine-readable identifiers must be English `snake_case`. Source-language terms live only in labels and external references.

| Field | Rule | Good | Bad |
|-------|------|------|-----|
| Variable `key` | English snake_case, no abbreviations | `general_control_protection_device_present` | `agcp_present`, `presence_agcp` |
| Enum item `value` | English snake_case | `heat_pump_geothermal` | `pac_geothermie` |
| Variable `label` | Localized i18n map (`en`, `fr`, `da`, `de`) | `{ en: "...", fr: "..." }` | English-only |
| Enum item `label` | Localized, minimum `en` + market primary | `{ en: "Heat pump air/water", fr: "PAC Air/Eau" }` | -- |

**No abbreviations** in keys or enum values. Spell out all terms. Abbreviations may appear in `description_i18n` to explain terminology (e.g., "RCD — Residual Current Device"). Prefix with domain when ambiguous: `heating_generator_type`, not `type`.

## External references

External references map variables to source standards, regulatory systems, and export formats.

| Field | What it means | Example |
|-------|--------------|---------|
| `system` | External system identifier | `nf_c_16_600`, `ademe_dpe` |
| `ref_key` | Field name in external system | `supplementary_equipotential_bonding_present` |
| `source_term` | Original term in source language | `Présence d'une LES` |
| `source_document` | Reference to source standard document | `[[nf-c-16-600-en]]` |

External refs are critical for traceability: they let you trace from an English variable key back to the French regulatory term and the exact standard section that defines it.

## Validation

Validation determines what values are valid. This is separate from requiredness (see [[tasks#Field Requiredness]]).

| Rule | What it means | Example |
|------|--------------|---------|
| `min` | Minimum numeric value | Grounding resistance min: 0 |
| `max` | Maximum numeric value | Grounding resistance max: 1000 |
| `pattern` | Regex for string variables | Email format |
| `required_default` | Whether the variable requires a value by default | `true` for mandatory checkpoints |

Requiredness is set on the task or note type, not on the variable. The same variable can be required in one task and optional in another.

## Variable frontmatter schema

Every variable is a markdown file with YAML frontmatter. The frontmatter fields below are the machine-readable definition. This schema applies to all variable files (e.g., files in `spec_fr_*/phase-1-extraction/variables/`).

### Identity

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `key` | string | Yes | Unique identifier. English `snake_case`, no abbreviations. |
| `title` | string | Yes | Human-readable title. |
| `aliases` | string[] | Yes | Alternative names for linking. |
| `doc_type` | enum | Yes | Always `variable`. |
| `kind` | enum | Yes | One of: `boolean`, `number`, `string`, `rich_text`, `enum_single`, `enum_multiple`, `tags`, `catalogue`, `generator`, `computed`. |
| `unit` | string | If number | Unit of measurement (e.g., `cm`, `ohm`, `kWh`). Null for non-numeric kinds. |

### Classification

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `level` | enum | Yes | Which [[levels\|hierarchy level]]: `case`, `report`, `property`, `building`, `storey`, `room`, `building_element`, `object`. |
| `report_domain` | string[] | Yes | Which report types: `epc`, `electrical`, `gas`, `lead`, `asbestos`, `termites`, `carrez`, `boutin`, `condition`. |
| `country` | string[] | Yes | Which markets: `fr`, `dk`, `de`, `all`. |
| `status` | enum | Yes | `draft`, `active`, `reference`, `deprecated`. |
| `tags` | string[] | Yes | Always includes `ontology`, `variable`. |

### Labels and descriptions

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `label` | i18n map | Yes | Display labels per language: `{ en: "...", fr: "...", da: "...", de: "..." }`. |
| `summary` | string | Yes | 1-3 sentence description. |
| `description_i18n` | i18n map | Recommended | Detailed description per language. May include abbreviation explanations. |

### Enum wiring

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `enum_ref` | wiki-link | If enum kind | Link to the shared enum definition file (e.g., `[[checkpoint_result]]`). |
| `enum_items` | array | If inline enum | Inline enum items when no shared enum exists. Each item has `value`, `label`, and optional `external_refs`. |

### Value source and computation

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `value_source` | object | Yes | How the variable gets its value. Has `type` (one of: `user_input`, `static_default`, `inherited`, `system_provided`, `computed`, `derived`), plus type-specific fields (`inherit_from`, `default`, etc.). |
| `dag` | object | Recommended | Computation DAG edges: `inputs_from` (variables this reads from) and `outputs_to` (variables that read from this). |
| `required_if` | string | If conditional | Extraction-time metadata for conditional requiredness (e.g., "only when pool present"). Used by task designers, not runtime. |
| `relevant_context` | string | If scoped | Physical or logical context where this variable applies (e.g., "bathrooms only", "swimming pools"). |

### Validation

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `validation` | object | Recommended | Validation rules: `min`, `max`, `pattern`, `required_default`. |

### Provenance and standards

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `standards` | array | Yes | Regulatory standard references. Each entry has `ref` (wiki-link to standard), `section`, and `markets`. |
| `source_refs` | array | Recommended | Provenance tracing: `doc` (wiki-link), `section`, `heading_link`, `line_range`, `original_term`. |
| `external_refs` | array | Recommended | External system mappings: `system`, `ref_key`, `ref_value`, `source_term`, `source_document`. |
| `control_sheet_ref` | string | If applicable | Section or control sheet ID from the standard (e.g., `B.5`). Used for task grouping. |

### Relationships

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `related` | wiki-link[] | Recommended | Links to related variables (e.g., sibling checkpoints in the same control sheet). |
| `used_by` | object | Recommended | Tracking: which `catalogues`, `generators`, `note_types`, and `prds` reference this variable. |

### Governance

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `created` | date | Yes | Creation date. |
| `updated` | date | Yes | Last meaningful edit. |
| `created_by` | wiki-link | Yes | Person or agent who created the file. |
| `steward` | wiki-link | Yes | Person or agent responsible for maintenance. |

## Frontmatter for other ontology entities

The frontmatter schema for other ontology objects is documented in their respective definition files:

| Entity | Definition | Key frontmatter fields |
|--------|-----------|----------------------|
| Task | [[tasks]] | `key`, `task_pattern`, `level`, `lifecycle_when`, `stage`, `step`, `requiredness`, `variables`, `note_type_keys`, `catalogue_keys` |
| Enum | [[variables]] (this doc) | `key`, `items` (each with `value`, `label`, `external_refs`) |
| Catalogue | [[catalogues]] | `key`, `description`, `wizard_steps` |
| Note type | [[note_types]] | `key`, `capture_modes`, `wizard_mode`, `supports_ar`, `catalogue_refs` |
| Generator | [[generators]] | `key`, `group_ref` |
| Standard | (per schema) | `key`, `type`, `issuing_body`, `effective_date`, `version`, `url` |

See the [[obsidian-markdown|frontmatter schema]] for the full field registry including required, recommended, and conditional fields for each `doc_type`.

# Glossary

| Term | Definition |
|------|-----------|
| Variable | The atomic unit of data capture. One question, one answer. Lives inside [[tasks]]. |
| Variable kind | The data type: boolean, number, string, rich_text, enum_single, enum_multiple, tags, catalogue, generator, computed. |
| Enum | A controlled value list referenced by enum_single and enum_multiple variables. Defined once, shared across variables. |
| Enum item | One option in an enum: `value` (machine-readable) + `label` (human-readable, localized). |
| Value source | How a variable gets its initial value: user_input, static_default, inherited, system_provided, computed, derived. |
| Inheritance | A value set at a parent [[levels\|level]] that flows down to child levels and tasks. See [[#Cross-report inheritance]]. |
| Computation DAG | The directed graph of variable dependencies (`inputs_from` / `outputs_to`). |
| External ref | A mapping from a variable to an external standard or regulatory system. |

---

## Agent Decision Guide

<!-- INJECTED INTO: extract_variables.py, cross_link_variables.py, review_entities.py -->
<!-- LAST REVIEWED: 2026-02-19 -->

### Kind decision tree

```
Is it yes/no? ────────────────────────────────── boolean
Single choice from a closed list? ────────────── enum_single
Multi-select from a closed list? ─────────────── enum_multiple
Numeric measurement with a unit? ─────────────── number  (set unit)
Free text, one line? ─────────────────────────── string
Free text, multi-line? ───────────────────────── rich_text
Multiple freeform labels? ────────────────────── tags
Pick from a template database? ───────────────── catalogue (set catalogue ref)
Walk ordered steps to build a config? ────────── generator (set generator ref)
Derived from other variables? ────────────────── computed
```

### Naming rules

- `key`: always English `snake_case`. Never source-language terms. No abbreviations.
- `enum item value`: always English `snake_case`.
- Source-language terms go ONLY in `label.{lang}` and `external_refs.source_term`.
- Prefix with domain when ambiguous: `heating_generator_type`, not `type`.

### Value source assignment

| Signal in standard | Value source |
|---|---|
| "enter the value" / no prefill mentioned | `user_input` |
| "defaults to X" / fixed starting value | `static_default` |
| Value comes from building/storey/case data | `inherited` (set source level + source field) |
| GPS, timestamp, device sensor, logged-in user | `system_provided` |
| Formula, lookup table, cross-level aggregation | `computed` (set inputs + table/expression) |
| Simple comparison or logic over sibling variables at same level | `derived` (set inputs + expression) |

### Level assignment

```
Whole job (client, address, report selection)? ───── case
Report-specific metadata/output? ─────────────────── report
Whole building (year, type, energy source)? ──────── building
One floor (separation type, default materials)? ──── storey
One room (room type, area, heating)? ─────────────── room
A surface/component detected by RoomPlan? ────────── building_element
Outdoor/site-level? ──────────────────────────────── property
An inspectable thing NOT part of room geometry? ──── object (specify parent)
```

Assign to the HIGHEST level where the data makes sense.

### Shared enum rule

If two or more variables from different reports share the same value list, extract one shared enum and reference it via `enum_ref`. Do not duplicate enum definitions.

### Extraction granularity

One variable per verifiable checkpoint. Never merge two separately assessable checks into a single variable.

**Split** when:
- The standard lists items a), b), c) -- each is its own variable
- A paragraph describes both a measurement AND a compliance check
- A section covers both "presence of X" AND "condition of X"
- Two checks could independently be compliant or non-compliant

**Keep as one** when:
- A single yes/no question with no sub-items
- A measurement that is always a single number
- A text field capturing a single observation

When in doubt, split. Merging later is easy; splitting after tasks are designed is painful.

### Relationship rules

Every variable must declare:
- `report_domain`: which report types it belongs to (at least one)
- `country`: which markets
- `standards`: which regulatory standard defines it (section ref + market)
- `external_refs`: source-language term + standard section

### Control sheet ref rule

If the standard organizes checkpoints into control sheets (B.1, B.2, ...) or sections (3.1, 3.2, ...), capture the identifier in `control_sheet_ref`. This is critical for task grouping in Phase 2.

### Conditionality rule

The variable definition is always unconditional -- the task decides when to show it. During extraction, capture conditions in `required_if` as metadata for task designers (e.g., "only when pool present").
