---
aliases:
- Generator
- Generator Definition
- Generator Group
- Configuration Generator
- Generator Steps
country:
- all
created: '2026-02-10'
created_by: '[[anders]]'
description: Defines generators as a configuration-building pattern where surveyors
  walk through ordered enum steps to describe something with no pre-existing template
  database. Covers the step model, visibility rules, options filters, inherited fields,
  computed fields, and the relationship to catalogues and the Catalogue Registration
  task pattern.
doc_type: ontology
feature_area:
- platform
- technical-installations
keywords:
- generator
- generator definition
- generator group
- generator steps
- options filter
- visibility rule
- computed fields
- configuration builder
- installation set
lifecycle:
- onsite/building/annotation_mode
- onsite/building_element/annotation_mode
- onsite/property/annotation_mode
- onsite/room/annotation_mode
lifecycle_level:
- building
- building_element
- property
- room
lifecycle_moment:
- annotation_mode
lifecycle_phase:
- onsite
related:
- '[[catalogues]]'
- '[[variables]]'
- '[[note_types]]'
- '[[tasks]]'
- '[[catalogue_registration]]'
- '[[_architecture]]'
- '[[generator]]'
report_domain:
- epc
- electrical
status: active
steward: '[[anders]]'
summary: Ontology definition for Generators — a configuration-building pattern where
  surveyors walk through ordered enum steps to describe something that has no pre-existing
  template database. Peer concept to Catalogues. Covers step definition, visibility
  rules, options filters, inherited fields, computed fields, computation tables, and
  how generators power the Catalogue Registration task pattern alongside catalogues.
tags:
- ontology
- definition
- product
- generator
title: Generators
updated: '2026-02-19'
---

# Generators

## Simple Description

Generators are a configuration-building pattern. The surveyor walks through ordered enum steps to describe something that does not exist in a pre-built template database. Each step is an enum choice, and the combination of choices IS the output.

Think of a generator as a guided configuration form. Instead of searching a database for "Bosch Compress 5800i AW," the surveyor answers "what kind of generator? what fuel? what year?" and the answers together describe the installation. There is no product database behind it — the answers are the data.

The same pattern serves two domains today:
- **French DPE materials**: wall composition (material type → thickness → insulation → lining)
- **French DPE technical systems**: heating installation (generator type → distribution → emitters → regulation → intermittence)

## Why Generators Are Not Catalogues

| | Catalogue | Generator |
|---|-----------|-----------|
| **Has template database** | Yes (hundreds/thousands of records) | No |
| **Goal** | Find an existing thing | Describe a configuration |
| **Steps** | Cascading filters — each step narrows a shared result pool | Sparse dependency tree — some steps filter others, some are independent |
| **Output** | Template ID + pre-filled data from the matched record | Enum values + computed values (no template ID) |
| **Computed values** | No (data comes from the template) | Yes (derived from step selections + context) |
| **Context inheritance** | No | Yes (reads building-level data like climate zone) |
| **UI type** | [[wizard]] (bottom sheet over result list) | [[generator]] (full-screen stacked form) |
| **Variable kind** | `catalogue` with `catalogueRef` | `generator` with `generatorRef` |

They share infrastructure: both produce notes with structured fields, both use the note pipeline for storage and upload, and both can be grouped into domain groups with categories and types.

## Model

### Generator Definition

A generator definition is a reusable config that describes one type of configuration. It is the equivalent of a child catalogue.

| Field | What it means |
|---|---|
| `key` | Generator identifier (e.g., `chauffage`, `wall_material`) |
| `group_ref` | Reference to the parent generator group |
| `label_i18n` | Display name in supported languages |
| `inherited_fields` | Fields read from the parent level (read-only in the form) |
| `steps` | Ordered list of generator steps |
| `computed_fields` | Values derived from step selections + context |
| `computation_tables` | Lookup data used by computed fields |

### Generator Group

A generator group is a domain-level container that groups related generator definitions. It is the equivalent of a Domain Catalogue Group.

| Field | What it means |
|---|---|
| `key` | Group identifier (e.g., `fr.dpe.systems`, `fr.dpe.materials`) |
| `label_i18n` | Display name |
| `taxonomy` | Categories and types, in display order |
| `rules` | Required types, cross-type constraints, cross-type references |
| `display` | Default presentation config for the domain list |

The taxonomy follows the same category → type structure as a Domain Catalogue Group. The UI reuses the same list-with-categories-and-completion-badges pattern from the Danish EPC Technical Installations list: the surveyor sees categories, checks which types are present, and walks through each generator in sequence.

## Steps

Steps are the core of a generator. Each step is one question the surveyor answers. Steps are ordered but not necessarily all dependent on each other.

### Step Definition

| Field | What it means |
|---|---|
| `key` | Step identifier |
| `field_ref` | Reference to a canonical variable in the field registry |
| `kind` | Variable kind — usually `enum_single`; numbers and booleans are exceptions |
| `required` | Whether the step must be answered |
| `computes` | Which computed field this step feeds (optional) |
| `visibility` | When this step is shown (conditional on prior step values) |
| `options_filter` | How prior steps filter this step's available options |

### Visibility

A step can be conditionally hidden based on prior step values. A step with no visibility rule is always shown. Visibility is show/hide only — it does not affect which options are available in the step. That is the job of options_filter.

Common visibility patterns:
- Show only when a prior step has a specific value (e.g., show "nominal power" only when the generator type is a boiler)
- Show only when a prior step does NOT have a specific value (e.g., show "insulation type" only when distribution is not "none")
- Always show after a specific step, regardless of its value (ordering hint only)

### Options Filter

Options filter controls which enum options a step offers based on prior step values. Each rule maps one parent value to a list of allowed child values, with a `_default` fallback for parent values that have no explicit rule. A step with no options_filter offers all its enum values regardless of other steps.

This is the sparse dependency tree that makes generators different from catalogue wizards. Step A's choice may filter step B's options while step C is completely independent. Only add an options_filter where a real dependency exists in the standard — do not create artificial chains.

### How This Differs from Wizard Filter Fields

| | Wizard `filter_fields` | Generator `steps` with `options_filter` |
|---|---|---|
| Shared pool | Yes — all steps filter one template list | No — each step has its own enum |
| Dependency shape | Chain: step 1 → step 2 → step 3 | Tree: step A → step B, step A → step D, step C independent |
| Auto-skip | Yes — if one option left, auto-select | Could apply, but less common |
| Independent steps | Not possible — every step filters the pool | Normal — steps without options_filter are free |

## Inherited Fields

Some generator steps need values from the parent level (building, case, room) that the surveyor does not re-enter. These are canonical variables in the [[variables|variable registry]] with `value_source.type: inherited`. The generator reads them at runtime and displays them read-only in the form.

Inherited fields are listed in the generator definition as `inherited_fields`. Each entry references a registry variable and declares whether to show it in the form (display-only) or use it silently for computation. If an inherited field is missing, the generator can still be filled but computed fields that depend on it show "?" until the parent-level value is available. The UI nudges the surveyor to complete the parent-level data first.

See [[variables]] for the `value_source.type: inherited` pattern.

## Computed Fields

Computed fields are derived values the system calculates from step selections and inherited fields. The surveyor never enters them directly. Each computed field references a canonical variable in the [[variables|variable registry]] with `value_source.type: computed`.

Two formula types are supported:
- **`lookup`**: table-based — the system looks up a value in a computation table using the specified step and context inputs as the key.
- **`expression`**: arithmetic — the system evaluates a formula referencing other computed or step fields.

Each computed field declares which step triggers its display in the form (a computed efficiency badge appearing as soon as the relevant step is answered, for example). Some computed fields are displayed only in the summary step.

### Computation DAG

Every variable in the registry can declare `dag.inputs_from` and `dag.outputs_to` in its frontmatter. This creates a directed acyclic graph across the registry — making energy models machine-readable. An agent can load the variable index, traverse the DAG, and understand what inputs are needed to compute any output.

See [[variables]] for the full DAG specification.

## Computation Tables

Computation tables are bundled datasets (JSON files) used by computed fields. They embed official regulatory lookup tables — for example, the 3CL-DPE 2021 generation efficiency factors indexed by generator type, installation year, climate zone, and emitter type.

Not all generators need computation tables. French DPE materials use pure enum composition with no derived values, so they have no computation tables. Only system characterization generators (heating, DHW, ventilation, cooling) need regulatory lookups. The engine handles both cases — the complexity lives in the config, not in code.

## Grouping and the Installation Set Pattern

Generator groups support the same batch-selection pattern as Domain Catalogue Groups. The surveyor sees a domain list organized by categories and types, checks which types are present, and walks through each generator in sequence. This is the same category dashboard pattern as the Danish EPC Technical Installations list.

For DPE systems, the surveyor opens "Technical Systems," sees categories (Heating, DHW, Ventilation, Cooling, Renewables), checks which are present, then works through each generator form in sequence. The summary shows all completed systems with computed efficiencies.

For DPE materials, the surveyor opens a material category (e.g., External Walls), taps "Add material," and walks through the generator form for that building element type. Multiple materials per category are possible (e.g., two different wall constructions).

## Relationship Between Materials and Systems

Both French DPE materials and French DPE systems use generators, but with different characteristics:

| | Materials | Systems |
|---|---|---|
| Generator group | `fr.dpe.materials` | `fr.dpe.systems` |
| Categories | Wall types, Window types, Floor types, Ceiling types | Heating, DHW, Ventilation, Cooling, Renewables |
| Steps per type | 4–9 | 7–10 |
| Computed fields | No (pure enum composition) | Yes (3CL efficiency lookups) |
| Inherited fields | Building element type (from parent) | Climate zone, construction period, inertia |
| Binding level | Building element (wall, window) | Building / case |
| Multiple instances | Yes (multiple wall materials per wall) | Yes (primary + secondary heating) |
| Dependency complexity | Low (mostly independent steps) | High (generator type filters distribution, emitter, etc.) |

## How Generators Relate to Other Concepts

**Catalogues.** Generators are the complement to [[catalogues]]: a catalogue finds an existing pre-built thing; a generator describes a new configuration. Both can power a [[catalogue_registration|Catalogue Registration]] task with a category dashboard and batch selection. The generator group / generator definition split mirrors the catalogue group / child catalogue split exactly. See [[catalogues]] for the full comparison.

**Tasks and task patterns.** The [[tasks#Catalogue Registration|Catalogue Registration]] task pattern supports both Domain Catalogue Groups and generator groups. When the task is backed by a generator group, each item in the dashboard enters a generator flow instead of a wizard flow. See [[catalogue_registration]] for the full task pattern and [[tasks]] for all five patterns.

**Note types.** A note type references a generator via a field of kind `generator`. The `generatorRef` can point to a generator group (the surveyor picks the type from the taxonomy, then enters the generator flow) or a specific generator definition (the surveyor goes directly to one flow). The output is a note with structured fields — step selections and computed values stored together — exactly like a catalogue capture. See [[note_types]] for how generator fields bind to note types.

**Variables.** Generator steps reference canonical variables via `field_ref`. Computed fields reference canonical variables with `value_source.type: computed`. Inherited fields reference canonical variables with `value_source.type: inherited`. The variable registry is the shared definition layer for all three. See [[variables]].

**Entry points in the app.** A generator form can be opened from three places: a domain list at building or property level (the installation set pattern), annotation mode at room or building element level (via a note type that references the generator), or a form field at any level (a field of kind `generator` that opens the form when tapped). All three produce the same output.

## Common Mistakes

| Mistake | Why it is wrong | How to fix |
|---|---|---|
| Modelling generators as catalogue child configs | Generators have no template database; the concepts are different | Use generator definitions, not catalogue child configs |
| Putting building-level data in generator steps | The surveyor re-enters data that already exists at case/building level | Use `inherited_fields` to read from the parent level |
| Making all steps dependent on all prior steps | Creates an unnecessarily rigid chain; most steps are independent | Only add `options_filter` where a real dependency exists in the standard |
| Missing computed fields for regulated domains | 3CL efficiency values must be auto-calculated, not manually entered | Add `computed_fields` with lookup tables for every regulatory output |
| No computation tables for lookup-based generators | Computed fields cannot evaluate without table data | Bundle JSON lookup tables in `computation_tables` |
| Using a generator where a catalogue exists | If a product database exists (brand, model, serial number), use a catalogue | Reserve generators for configurations with no matching product template |

## Glossary

| Term | Definition |
|---|---|
| Generator | A configuration-building pattern where the surveyor walks through ordered enum steps with no pre-existing template database. |
| Generator definition | The reusable config for one type of configuration (e.g., heating installation, wall material). Equivalent to a child catalogue. |
| Generator group | A domain-level container grouping related generator definitions with a taxonomy. Equivalent to a Domain Catalogue Group. |
| Generator step | One question in the generator flow, referencing a canonical variable. |
| Visibility rule | Controls whether a step is shown based on prior step values. Does not affect available options — that is options_filter. |
| Options filter | Controls which enum options are available in a step based on prior step values (sparse dependency tree). |
| Inherited field | A value read from the parent level (case, building), displayed read-only in the generator form. |
| Computed field | A derived value calculated from step selections, inherited fields, and lookup tables. Never entered manually. |
| Computation table | Bundled JSON lookup data used by computed fields (e.g., 3CL-DPE efficiency tables). |

---

## Agent Decision Guide

<!-- INJECTED INTO: extract_variables.py, cross_link_variables.py, review_entities.py -->
<!-- LAST REVIEWED: 2026-02-19 -->

### When generator vs catalogue

```
Does a pre-built template database of items exist
  (product models, defect codes, control points)? ── CATALOGUE
Does the surveyor DESCRIBE a configuration by answering
  ordered questions, with NO template DB behind it? ─ GENERATOR
```

**Generator signals in a standard:**
- "Describe the heating installation: type → fuel → distribution → emitters → regulation"
- "Characterize the wall: material → thickness → insulation → lining"
- Step answers feed into regulatory lookup tables for computed outputs
- No product database — the combination of answers IS the data

### Step structure rules

For each generator step extracted from a standard:

1. **One step = one canonical variable.** Set `field_ref` to the variable's registry key.
2. **Most steps are `enum_single`.** Numbers (e.g. installation year) and booleans are exceptions.
3. **Only add `options_filter` where a real dependency exists.** If step B's options do not change based on step A's value, do NOT add a filter.
4. **Add `visibility` only when a step should be hidden** for certain prior values. No visibility rule = always shown.
5. **Order steps** in the sequence the standard presents them or the surveyor encounters them.

### Computed fields rule

If the standard provides a lookup table or formula that derives a value from step selections and context:
- Create a `computed_number` variable in the registry with `value_source.type: computed`.
- Reference the lookup table in `computation_tables`.
- List exact input variable keys in `inputs`.

If the generator has no derived values (pure enum composition, e.g. DPE materials), leave `computed_fields` empty and omit `computation_tables`.

### Inherited fields rule

If a generator step needs a value from a parent level (climate zone, construction period, building inertia):
- Do NOT add it as a generator step.
- List it in `inherited_fields` with `field_ref` pointing to a registry variable that has `value_source.type: inherited`.

### Group type check

If the generator domain groups multiple types under categories (heating / DHW / ventilation, or walls / windows / floors):
- Create a **generator group** with a `taxonomy` matching the standard's category structure.
- Each type in the taxonomy gets its own **generator definition**.
- The group powers a [[catalogue_registration|Catalogue Registration]] task with a category dashboard.
