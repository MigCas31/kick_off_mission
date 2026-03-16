---
aliases:
- Task Types
- Task Patterns
- Task Requiredness
country:
- all
created: '2026-02-04'
created_by: '[[anders]]'
doc_type: ontology
feature_area:
- platform
- technical-installations
keywords:
- task pattern
- form
- checklist with answers
- checklist with evidence
- catalogue registration
- observation capture
- task requiredness
- note type composition
- coverage rules
- inheritance
- auto-computation
lifecycle:
- offsite-pre/report/before_arriving
- onsite/report/on_creation
- onsite/report/on_overview
- onsite/building/on_overview
- onsite/building/on_creation
- onsite/storey/on_creation
- onsite/room/on_creation
- onsite/room/on_overview
- onsite/room/annotation_mode
- onsite/building_element/annotation_mode
- onsite/property/annotation_mode
- onsite/report/on_completion
- offsite-post/report/off_site
lifecycle_level:
- building
- building_element
- property
- report
- room
- storey
lifecycle_moment:
- annotation_mode
- before_arriving
- off_site
- on_completion
- on_creation
- on_overview
lifecycle_phase:
- offsite-post
- offsite-pre
- onsite
related:
- '[[_architecture]]'
- '[[app_lifecycle]]'
- '[[catalogues]]'
- '[[coverage_rules]]'
- '[[evidence_and_access]]'
- '[[levels]]'
- '[[note_types]]'
- '[[phase_2]]'
- '[[spatial_capture]]'
- '[[variables]]'
report_domain:
- epc
- electrical
- condition
- lead
status: active
steward: '[[anders]]'
summary: 'Ontology for tasks in the Plans app. Defines the 5 task patterns (form,
  checklist with answers, checklist with evidence, catalogue registration, observation
  capture), task and field requiredness, inheritance and auto-computation rules, and
  coverage rules. References _architecture for UI concepts, app_lifecycle for timing,
  and levels for the spatial hierarchy.

  '
tags:
- ontology
- definition
- product
title: Tasks
updated: '2026-02-19'
---

# Simple Description

For each report we describe what the surveyor has to do, at each level. A task is a single unit of work at a specific level in the inspection hierarchy. Each task follows one of five patterns that match the surveyor's physical situation and cognitive mode.

## Example: Danish EPC

The Danish surveyor comes to a home with the goal of registering data about the house and giving the home an energy label -- a comparable rating of energy efficiency.

What the surveyor has to do:

1. Measure every surface facing unheated areas and register what material that surface has, so they can calculate heat loss.
2. Register what technical installations the home has, across 26 different categories, to measure consumption and production of energy.
3. Based on what they register, make recommendations for upgrades the home owner should do, based on the return on investment (costs vs savings).

| Level | Tasks | Pattern |
|-------|-------|---------|
| Report | Summary comments, Recommendations, Front-of-house reference photo | Form, Note capture |
| Property | Technical Installations (Outside) | Catalogue Registration |
| Building | Heat Loss config, Technical Installations (Inside), Building Year, Name | Form, Catalogue Registration, Form (on_creation) |
| Storey | Name, Floor separation thickness, Material defaults per element type | Form (on_creation) |
| Room | Create building elements, Room name, Heating system, Whether addition | Form (on_creation), Form |
| Building Element | Type, Material, Area | Form |
| Object | Select specific installations/datapoints from templated values | Catalogue Registration (via note types) |

# Surveyor Reality Context

Before reading the task model, understand the physical and cognitive situation tasks are designed for.

## The surveyor doesn't know the property in advance

The surveyor arrives at an unfamiliar building. They have never been inside. They discover what exists -- walls, rooms, installations, materials, defects -- as they walk through the space. Tasks must support progressive discovery, not predetermined checklists of known items.

## Multiple reports run simultaneously

A single inspection visit produces multiple report types (EPC + Condition + Electrical + Lead + ...). Tasks from different reports coexist at the same level. When the surveyor enters a room, they see tasks from ALL active reports merged together. The UI must make it clear which tasks belong to which report, and completion tracking works per-report.

## The main cognitive strain is remembering to capture everything

The surveyor's hardest job is not the capture itself -- it is remembering what needs capturing while physically moving through unfamiliar space. The app must:
- Show what has been captured vs what is still expected (coverage tracking)
- Surface gaps as reminders, not blockers (informational warnings)
- Support both proactive workflows (known lists to work through) and reactive workflows (scan and capture what you find)

## Tasks activate when the surveyor discovers the relevant context

A task to register a heat pump is irrelevant until the surveyor finds the utility room. A task to measure lead on walls is irrelevant until the surveyor enters a room and sees the walls. Tasks are defined in the report bundle configuration, but they activate in the UI when their level context exists and their conditions are met.

# 5 Task Patterns

Each task follows exactly one pattern. The pattern determines the interaction mode and how the surveyor engages with the task's fields and notes.

See [[_architecture]] for how patterns map to UI concepts, and [[_task-design-guide]] for the selection process.

## Form

A flat list of input fields. Fixed location, eyes on screen, entering known data. See [[_architecture#2. Form]] for the UI concept.

**When to use:** The task is a set of fields to fill with no evidence capture, no checklist semantics, and no catalogue drill-down.

**Examples:** Building name/year/type, summary comments (offsite), room-level heating system, report-level self-regulation questions, single-field prompts at level creation (`lifecycle_when: on_creation` with 1 field).

## Checklist with Answers

A list of yes/no/NA questions the surveyor works through sequentially. Fixed location, known control points. See [[_architecture#4. Checklist]] (answers-only configuration).

**When to use:** The standard defines a fixed list of control points requiring yes/no/NA answers without evidence capture. If evidence IS needed for failed checks, use Checklist with Evidence instead.

**Examples:** French Electrical control sheet B.1 (Emergency Shutoff Assessment), French Electrical B.3.3 (Earthing Installation), Danish Condition self-regulation questions.

## Checklist with Evidence

A dashboard of control points populated by captured [[note_types]] as evidence. The surveyor moves through a space discovering problems; evidence files under matching control points. See [[_architecture#4. Checklist]] (references-observations configuration).

**When to use:** The surveyor scans for problems and captures evidence matched against standard-defined control points. The list shows coverage progress but findings are discovered reactively.

**Examples:** French Electrical room scan (B.7 + B.8), French Lead (CREP) room coating assessment, Danish Condition defect capture per room.

## Catalogue Registration

A dashboard of categories and types the surveyor registers proactively. Each item uses a [[note_types|note type]] (by_selection) with either a [[wizard]] (drill down to match a template) or a [[generators|generator]] (build a configuration step by step). See [[_architecture#3. Note Type (annotation)]] and [[catalogues]].

**When to use:** The standard requires systematic registration of items from a catalogue taxonomy. The system shows a fixed list; the surveyor works through it registering what is present.

**Examples:** Danish EPC Technical Installations (26 categories), hot water pipe registration, building element material registration.

## Observation Capture

Space-driven assessment where findings are discovered reactively. No predefined control points -- the room geometry and building elements define what to assess. Uses [[note_types]] (by_discovery). Completion is assessed via [[coverage_rules]] (spatial completeness), not checklist status. See [[_architecture#3. Note Type (annotation)]].

**When to use:** The standard requires assessing a space without defining specific control points. The surveyor walks the space and captures what they find. Zero findings is valid.

**Differs from Checklist with Evidence:** Both are reactive, but checklist_with_evidence is organized by standard-defined control points; observation capture is organized by the physical space.

**Examples:** Danish Condition defect capture per room, French Lead (CREP) room measurement (walls per zone, ceiling, doors, windows), general condition assessment of building elements.

## Pattern summary

| Pattern | Physical situation | UI concepts |
|---------|-------------------|-------------|
| Form | Fixed location, entering data | [[_architecture#2. Form\|Form]] |
| Checklist with Answers | Fixed location, sequential yes/no checks | [[_architecture#4. Checklist\|Checklist]] (answers-only) |
| Checklist with Evidence | Moving through space, scanning for problems | [[_architecture#4. Checklist\|Checklist]] (references observations) |
| Catalogue Registration | Location with things to register | [[_architecture#3. Note Type (annotation)\|Note Type]] (by_selection) + [[catalogues\|catalogue]] |
| Observation Capture | Assessing a space, capturing findings as discovered | [[_architecture#3. Note Type (annotation)\|Note Type]] (by_discovery) |

> [!note] Standalone photo capture is NOT a task pattern
> Standalone photo requirements (reference photo, attic access photo) are handled via note type requiredness rules on the report bundle, or as a form task with a single `image` variable.

# Task Composition with Note Types

Tasks contain [[note_types]]. A note type is the composed capture unit: capture mode + template selection + instance fields + placement. See [[note_types]] for the full specification and [[_architecture#3. Note Type (annotation)]] for the UI concept.

## Which patterns contain note types

| Pattern | Contains note types? | How |
|---------|---------------------|-----|
| Form | No | Pure field entry, no evidence capture |
| Checklist with Answers | No | Yes/No answers without evidence |
| Checklist with Evidence | Yes | Each captured finding is a note type instance filed under a control point |
| Catalogue Registration | Yes | Each registered item is a note type instance with wizard template match |
| Observation Capture | Yes | Each finding captured during the assessment is a note type instance |

## Standalone note capture IS a valid task

A task can be "capture at least 1 note of type X." This is neither a checklist nor a catalogue -- it is a simple requirement to produce one or more note instances. The task pattern is a lightweight wrapper around a single note type.

**Examples:**
- Front-of-house reference photo (always required, 1 photo)
- General condition overview photo per room
- Attic access photo

## How note types compose inside tasks

```
Task (pattern)
  └── Note Type (composed capture unit)
        ├── Capture mode: image | multiImage | video | ...
        ├── Template selection: wizard drill-down into catalogue
        ├── Instance fields: dropdowns, generators, number inputs
        └── Placement: placement_prompt (AI + spatial context)
```

Note types are defined per report bundle and referenced by tasks. The same note type definition can be used by multiple tasks. Note type capture cardinality (single-entry vs multi-entry) is configured on the note type, not on the task pattern. See [[note_types#Capture Cardinality]].

# Related concepts

Tasks operate within a broader system. See these for the full definitions:

- **[[reports]]** -- what a report is, the report bundle structure, multi-report inspections
- **[[levels]]** -- the inspection hierarchy where tasks live (case → building → storey → room → building element)
- **[[app_lifecycle]]** -- when tasks appear (phases, stages, moments) and the full surveyor flow
- **[[variables]]** -- the data fields tasks capture, including value sources and inheritance
- **[[note_types]]** -- the composed capture units inside evidence and catalogue patterns
- **[[catalogues]]** -- structured reference datasets for template selection
- **[[coverage_rules]]** -- spatial completeness validation for evidence patterns
- **[[report-configuration]]** -- how report bundles are technically configured

# Task Requiredness

Task requiredness defines whether a task must be completed. It is set per task in the report bundle configuration.

| Requiredness | Meaning | Example |
|-------------|---------|---------|
| `always` | Must be done for every instance of the level. | Room name, building year |
| `always_once` | Must be done once per report, at any instance of the level. | Front-of-house photo (once per report, at property level) |
| `always_if` | Must be done if a condition is met (field value, level property). | Crawlspace inspection (only if storey type = crawlspace) |
| `confirm_default` | A default value exists; the surveyor must confirm or change it. | Material defaults inherited from storey primaries |
| `optional_with_default` | A default value exists; the surveyor can leave it as-is. | Calculated values that auto-fill but can be overridden |
| `optional` | Task is available but not required. | Additional notes, supplementary photos |

Completion tracking is per-task, per-level-instance, per-report. A room is "complete" for a report when all `always` and `always_if` (where condition is met) tasks are done at that room level for that report.

# Field Requiredness

Within a task, each field instance has its own requiredness. This is separate from task requiredness -- a task can be required while some of its fields are optional.

| Field requiredness | Meaning | Example |
|-------------------|---------|---------|
| `required` | Must have a value before the task is considered complete. | Anomaly code on an electrical defect note |
| `required_if` | Required when a condition on another field is met. | Compensatory measure, required only when the anomaly code has a compensation option |
| `optional` | The surveyor can leave it blank. | Additional comments, supplementary observations |

Field validation (type, range, allowed values) comes from the canonical variable definition. See [[variables]]. The task's field instance references the variable and adds requiredness context.

# Lifecycle placement

Every task is placed at a specific point in the [[app_lifecycle]] using a three-part coordinate: `{phase}/{level}/{moment}` (e.g. `onsite/room/annotation_mode`). The lifecycle defines 14 stages, 6 moments, and the valid combinations. See [[app_lifecycle#Stage assignment]] for how to assign tasks to stages.

Tasks from multiple reports share the same moments. At `annotation_mode` in a room, the surveyor sees annotation tasks from ALL active reports.

# Inheritance and Auto-Computation

Before grouping variables into tasks, check what can be inherited from parent levels or auto-computed from the spatial pipeline. The surveyor should never enter the same information twice.

See [[_architecture#Capture once, inherit everywhere]].

## Inheritance from parent levels

When a variable is set during level setup (building year, heating type, wall materials, room type, floor construction), it becomes available to every report and task at that level and below. This is the `value_source.type: inherited` pattern from [[variables]].

Before adding a field to a task, check:
1. Does another task at this level already capture it?
2. Does a parent level (building, storey) provide it?
3. Does another report's setup flow set it as a level property?
4. Could this be a level default set once and inherited by all reports?

If yes to any of these, use inheritance rather than re-capture. The new task should receive the value as a pre-filled default that the surveyor can confirm or override.

### Examples of shared level properties

| Property | Set at | Available to |
|----------|--------|-------------|
| Building year, address, building type | Building creation (form at on_creation) | All reports at building level and below |
| Wall materials (substrate, coating) | EPC material assignment or storey primaries | Lead (substrate), Condition (materials), Asbestos (material type) |
| Heating system type | EPC building setup | Gas, Electrical, TI reports |
| Room type (kitchen, bathroom, bedroom) | Room creation (form at on_creation) | Conditional task activation across all reports |
| Floor separation thickness | Storey creation (form at on_creation) | EPC heat loss calculations |

## Auto-computation from the spatial pipeline

When a variable CAN be inferred from the spatial pipeline, it SHOULD be. The surveyor places an AR pin; the system computes zone_label, height_above_floor, surface_type_at_point, nearest_wall, and more. These computed values pre-fill note type fields, removing manual selection.

See [[spatial_capture]] for the full list of computed spatial variables and their derivation status.

### Fields that should be auto-computed, not manually entered

| Field | Computed from | Manual alternative avoided |
|-------|--------------|---------------------------|
| `zone_label` (A/B/C/D) | AR pin position + wall_zone_assignment | Surveyor selecting "which wall" from a dropdown |
| `height_above_floor` | AR pin Y coordinate vs floor plane | Surveyor estimating height |
| `height_bucket` (below_1m / above_1m) | Derived from height_above_floor | Surveyor selecting height range |
| `surface_type` (wall/door/window/ceiling) | RoomPlan surface detection | Surveyor selecting surface type from enum |
| `nearest_wall` | Plane projection + clamping | Surveyor selecting wall from list |
| `cardinal_direction` | Wall normal + device azimuth | Surveyor guessing compass direction |
| `near_window` | Window proximity check (0.75m threshold) | Surveyor noting "near window" |
| `ai_placement_description` | Vision model + all spatial vars + photo | Surveyor writing free-text description |

The surveyor's manual input should be limited to what the system genuinely cannot infer: measurement readings, degradation assessments, subjective observations. Everything about WHERE something is should come from the spatial pipeline.

# Coverage Rules

Tasks can have coverage rules that evaluate spatial completeness at level completion. Coverage rules answer: "Has the surveyor captured everything they should have at this level?"

See [[coverage_rules]] for the full specification.

## Key properties

- Coverage rules are **informational** -- warnings, never blockers. The surveyor can always acknowledge a gap and move on.
- They are **dynamic** -- they adapt as the surveyor discovers diagnostic units (walls from RoomPlan, doors/windows from the scan).
- They are **conditional** -- measurement requirements can escalate based on findings (e.g., positive lead on one wall triggers additional measurements on other walls).
- They are evaluated **at level completion** and can be displayed as progress indicators during annotation mode.

## Relationship to task patterns

| Pattern | Coverage rules? | How |
|---------|----------------|-----|
| Checklist with Evidence | Yes, the primary use case | Evaluates which control points have evidence, which zones/positions are measured |
| Catalogue Registration | Possible | Can check that expected categories have at least one registered item |
| Form | No | Static fields, no spatial coverage concept |
| Observation Capture | Yes | Spatial coverage: did you assess all walls, elements, surfaces at this level? |
| Checklist with Answers | Possible | Can verify all required questions are answered, though this is simpler than spatial coverage |

## Example: French Lead room coverage

```yaml
coverage_rules:
  scope: room
  validation_mode: informational
  unit_types:
    - type: wall
      discovery: per_zone           # one per wall zone (A, B, C, D)
      measurement_positions:
        - key: below_1m
          required: always
        - key: above_1m
          required: if_first_negative
    - type: ceiling
      discovery: per_room
      min_measurements: 1
    - type: door
      discovery: dynamic            # discovered from RoomPlan scan
      measurement_positions:
        - key: frame
          required: always
        - key: leaf
          required: if_first_negative
  escalation_rules:
    - trigger: any_unit_positive_in_room
      effect: add_measurement
      applies_to: same_type
```

# Glossary

| Term | Definition |
|------|-----------|
| Task | A single unit of work at a specific level in the [[levels\|inspection hierarchy]]. |
| Task pattern | The interaction mode for a task. One of five: form, checklist_with_answers, checklist_with_evidence, catalogue_registration, observation_capture. See [[_architecture]] for how patterns map to UI concepts. |
| Task requiredness | Whether a task must be completed: always, always_once, always_if, confirm_default, optional_with_default, optional. |
| Field requiredness | Whether a field within a task must have a value: required, required_if, optional. |
| Coverage rule | A soft validation rule for spatial completeness. See [[coverage_rules]]. |
| Inheritance | A field value set at a parent level that flows down to child levels. See [[variables#Cross-report inheritance]]. |
| Auto-computation | A field value derived from the spatial pipeline. See [[spatial_capture]]. |
## Agent Decision Guide

<!-- INJECTED INTO: Phase 2 task design, report bundle authoring, coverage rule design -->
<!-- LAST REVIEWED: 2026-02-19 -->

### Pick the task pattern

```
What is the surveyor doing?

1. Filling fields with known data?
   → FORM (proactive)

2. Working through a yes/no checklist?
   → CHECKLIST WITH ANSWERS (proactive)

3. Scanning a space and capturing evidence of problems?
   → Does the standard define specific control points?
   ├─ Yes → CHECKLIST WITH EVIDENCE (reactive, organized by control points)
   └─ No  → OBSERVATION CAPTURE (reactive, organized by physical space)

4. Registering items from a taxonomy?
   → CATALOGUE REGISTRATION (proactive)
```

For how each pattern maps to UI concepts, see [[_architecture]] and [[_task-design-guide]].

### Requiredness decision

| Condition | Use |
|-----------|-----|
| Every instance of the level must complete it | `always` |
| Once per report, any instance | `always_once` |
| Only when a field/property condition is met | `always_if` |
| Default exists, surveyor must acknowledge | `confirm_default` |
| Default exists, no acknowledgement needed | `optional_with_default` |
| Available but never blocking | `optional` |

### Lifecycle placement

Use the stage-first workflow from [[app_lifecycle#Stage assignment]]:

1. **Identify the stage** — which of the 14 working episodes? (e.g. 2.J Building Overview)
2. **Find the step** — which atomic step within the stage? (e.g. `building.overview`)
3. **Derive the coordinate** — read the step's YAML block → `{phase}/{level}/{moment}`

Set `stage`, `step`, and `lifecycle_when` on the task.

### Before adding a field to a task

1. Already captured by another task at this level? → inherit.
2. Set at a parent level (building, storey)? → inherit.
3. Spatial pipeline can compute it? → auto-computation (see [[spatial_capture]]).
4. None of the above → add as a manual field.

### Note type composition check

Only `checklist_with_evidence`, `catalogue_registration`, and `observation_capture` contain [[note_types]]. If no evidence capture or catalogue drill-down, the task does NOT need `note_type_keys`.
