---
aliases:
- Note Type
- Annotation Capture
- Capture Modes
country:
- all
created: '2026-02-04'
created_by: '[[anders]]'
doc_type: ontology
feature_area:
- platform
keywords:
- note type
- annotation mode
- capture mode
- wizard mode
- multi-entry capture
- single-entry capture
- AR placement
- inspection observation
- report bundle
lifecycle:
- onsite/room/annotation_mode
- onsite/building_element/annotation_mode
lifecycle_level:
- building_element
- room
lifecycle_moment:
- annotation_mode
lifecycle_phase:
- onsite
related:
- '[[tasks]]'
- '[[_architecture]]'
- '[[_task-design-guide]]'
- '[[variables]]'
- '[[catalogues]]'
- '[[spatial_capture]]'
- '[[coverage_rules]]'
- '[[reports]]'
report_domain:
- epc
- electrical
- condition
- lead
- gas
- asbestos
- termites
status: active
steward: '[[anders]]'
summary: Defines note types as the composed capture unit used inside evidence-based
  and catalogue-based task patterns. A note type specifies how a surveyor collects
  one piece of evidence -- which capture mode to use, which catalogue item to match,
  which instance fields to fill, and whether spatial placement is required. Note types
  do not define task patterns, variables, catalogues, or coverage rules -- they reference
  those concepts.
tags:
- ontology
- definition
- product
title: Note Types
updated: '2026-02-19'
---

# Note Types

## Simple Description

A note type is a composed capture unit: a small form combined with a capture tool. It tells the surveyor what to record, how to capture it, and how it will appear in lists after capture. Note types are defined per report bundle and appear as options when the surveyor enters annotation mode.

Note types live **inside tasks**. They are not a task pattern themselves -- they are the thing three task patterns contain:

- [[tasks#Checklist with Evidence]] -- each captured finding is a note type instance filed under a control point
- [[tasks#Catalogue Registration]] -- each registered item is a note type instance with a wizard template match
- [[tasks#Observation Capture]] -- each discovered finding is a note type instance

See [[tasks#Task Composition with Note Types]] for the full composition model, and [[_architecture#3. Note Type (annotation)]] for the UI concept.

> [!note] What a note type is NOT
> A note type is not a task pattern, not a variable, and not a catalogue. It does not define what data means (that is [[variables]]), what templates exist (that is [[catalogues]]), or when a task must be done (that is [[tasks#Task Requiredness]]). A note type is the bridge between those concepts: it assembles them into one capture interaction.

**In human terms:** when a surveyor spots a defect, they tap the note type for that defect category. The app shows a capture tool (camera, optional AR pin), then a small form. When they save, the note appears in a list with a short summary. That full interaction is a note type.

## Anatomy of a Note Type

Every note type is composed of four components:

| Component | What it provides | Canonical definition |
|-----------|-----------------|----------------------|
| Capture mode | How evidence is collected: photo, multi-photo, video, polygon, AR drawing | [[spatial_capture]] |
| Template selection | Which catalogue item this note records -- via wizard drill-down or search | [[catalogues]] |
| Instance fields | Details for this specific capture: enums, generators, free text, spatial computed values | [[variables]] |
| Placement | Where in the room the note is, via AR pin raycasted to the 3D model (optional or required) | [[spatial_capture]] |

Not every note type uses all four. A photo-only note type has capture mode and fields but no template selection. An observation with no spatial requirement omits placement.

## Capture Modes

The capture mode determines what kind of evidence the surveyor records.

| Mode | What it captures | Spatial? | Status |
|------|-----------------|----------|--------|
| `image` | Single photo | Optional AR point | Production |
| `multiImage` | Multiple photos for one note | Optional AR point | Production |
| `video` | Video clip | No | Designed |
| `polygon` | Drawn area on the camera view | Yes | Designed |
| `arDrawing` | Freeform drawing on the AR view | Yes | Designed |
| `measurementTape` | Two AR points measuring a distance | Yes | Concept |
| `frames` | Pose-adjusted images with depth data during RoomPlan scan; not annotation-bound | Inherits scan pose | Research complete |

Only `image` and `multiImage` are shipped in production report bundles. Do not use others in new bundles without confirming their implementation status.

**Spatial placement:** when an AR session is active, notes can carry a 3D coordinate via raycasting. Some note types require placement (e.g. a lead measurement must have a wall location); others treat it as optional. This is controlled by `requires_spatial` on the note type. See [[spatial_capture]] for the full spatial pipeline and the computed variables that AR placement unlocks.

## Wizard Modes

When a note type involves template selection from a catalogue, `wizardMode` controls the step order.

| Mode | What it means | When to use |
|------|--------------|-------------|
| `catalogueFirst` | Surveyor picks the catalogue item before filling instance fields | When the catalogue choice determines which fields appear |
| `combinedOnly` | Template selection and fields shown together in one step | When the template and fields are tightly coupled and simple |

Rule: if selecting the catalogue item changes which fields are visible or required, use `catalogueFirst`. See [[catalogues]] for the full wizard and search UX specification.

## Capture Cardinality

Note types can be used in two cardinality modes. These are not separate types -- they are a property of how the note type is configured within a task.

### Single-entry capture

The surveyor captures one note instance at a time. They identify one item (via wizard or search), fill instance fields, capture evidence, and save.

Use when:

- Findings are discovered one at a time during scanning
- Items are diverse (different categories each capture)
- The report needs findings recorded individually

Flow: identify → capture evidence → fill fields → save.

### Multi-entry capture

The surveyor selects multiple templates first (building a queue), then iterates through each to fill per-instance details and capture evidence.

Use when:

- Multiple similar items are expected and speed matters
- The standard expects multiple findings of the same domain
- The surveyor can see several items at once before capturing each

Flow: select many → build queue → iterate per item (fields + evidence) → review summary.

Multi-entry requires specific config on the note type (`multiEntryTemplateCapture`) linking the template field, template ID, and catalogue reference pattern. This cardinality is configured on the note type, not on the task pattern.

## Presentation Modes

A note type operates in one of two presentation modes that determine how the task organizes captures:

| Mode | Items known in advance? | Completion model | Example |
|------|------------------------|-----------------|---------|
| `by_selection` | Yes, selected before capture | Trackable: done / not done per item | Technical installations: select which are present, fill each |
| `by_discovery` | No, discovered as you go | Open-ended: done when the surveyor says done | Electrical anomalies, condition defects |

See [[_architecture#3. Note Type (annotation)]] for the full UI concept.

**List presentation:** every note type must declare a presentation configuration so captured notes show up meaningfully in lists -- a title line and optional pills for key distinguishing fields. Without this, note rows appear blank. The configuration references field paths, not raw values; it is specified alongside the note type in the report bundle.

## Coverage Rules

Note types captured under observation-based and checklist-with-evidence tasks can be subject to coverage rules that evaluate whether the surveyor captured enough notes at a level (e.g. one measurement per wall zone). Coverage rules are informational -- warnings at level completion, never blockers.

See [[coverage_rules]] for the full specification, including zone-based validation, conditional escalation, and diagnostic unit types.

## Related Concepts

- **[[tasks]]** -- the task patterns that contain note types (checklist_with_evidence, catalogue_registration, observation_capture) and their requiredness model
- **[[_architecture]]** -- the seven UI concepts; Note Type is concept #3; this document defines the UI composition rules
- **[[_task-design-guide]]** -- step-by-step selection tree for choosing the right concept and config when designing a task
- **[[variables]]** -- the variable kinds used as fields inside note types; canonical vs instance rules; value sources
- **[[catalogues]]** -- the template databases used in wizard-driven note types; the wizard and search UX
- **[[spatial_capture]]** -- the full spatial data pipeline; capture modes; computed spatial variables unlocked by AR placement
- **[[coverage_rules]]** -- spatial completeness rules that evaluate note type coverage at level completion
- **[[reports]]** -- report bundle structure; how note types are declared inside a bundle; the noteCaptureLoop flow pattern

## Glossary

| Term | Definition |
|------|-----------|
| Note type | A composed capture unit inside a task: capture mode + template selection + instance fields + placement. |
| Capture mode | The method used to collect evidence for a note (image, multiImage, video, polygon, etc.). |
| Wizard mode | The step order for catalogue-driven capture: `catalogueFirst` or `combinedOnly`. |
| Single-entry capture | Capture of one note instance at a time; findings identified and recorded individually. |
| Multi-entry capture | Capture where multiple templates are queued first, then iterated through for per-instance details. |
| Note presentation | The UI summary configuration for how a note appears in lists: title line and pills. |
| `by_selection` | Presentation mode where items are selected before capture; completion is trackable per item. |
| `by_discovery` | Presentation mode where items are discovered during scanning; completion is open-ended. |
| `requires_spatial` | Property declaring whether a note type must have an AR-placed 3D coordinate. |

## Agent Decision Guide

<!-- INJECTED INTO: Phase 2 note type design, report bundle authoring, capture flow design -->
<!-- LAST REVIEWED: 2026-02-19 -->

### Reuse vs new note type

Before defining a new note type, check whether an existing one in the report bundle covers the same capture:

- Same catalogue domain + same fields + same capture mode → reuse with a different task reference
- Different catalogue domain, or different required fields → define a new note type
- Only cardinality differs (single vs multi-entry) → same note type, configure `multiEntryTemplateCapture`

### Capture mode selection

```text
Is spatial placement needed?
├─ Required (e.g. lead on a specific wall) → requires_spatial: true
└─ Optional or none                        → requires_spatial: false

How many photos per note?
├─ One  → mode: image
└─ Many → mode: multiImage

(video, polygon, arDrawing, measurementTape, frames → not production-ready; do not use in new bundles)
```

### Wizard mode selection

| Situation | Mode |
|-----------|------|
| Catalogue choice determines which fields appear | `catalogueFirst` |
| Template and fields are tightly coupled and shown together | `combinedOnly` |

### Cardinality decision

| Situation | Cardinality |
|-----------|-------------|
| Findings discovered one at a time while scanning | Single-entry |
| Multiple similar items expected; speed matters; surveyor can batch-select | Multi-entry (configure `multiEntryTemplateCapture`) |

### Presentation mode

| Items known before capture starts? | Mode |
|------------------------------------|------|
| Yes (systematic registration) | `by_selection` |
| No (reactive scanning) | `by_discovery` |

### Presentation config checklist

Every note type must have a presentation block with:

- At least one `fieldPath` for the list title line (surveyor can identify the note at a glance)
- At least one `pills` entry for the most important distinguishing field (severity, type, zone)

Missing presentation = blank rows in the note list UI.

## Common Mistakes

| Mistake | Why it's wrong | How to fix |
|---------|---------------|------------|
| Defining a note type outside a task | Note types only exist in context of a task; they have no meaning standalone | Always assign to a task with the right pattern |
| Creating a separate note type when only cardinality differs | Duplicates the definition unnecessarily | Use `multiEntryTemplateCapture` config on the same note type |
| Using `by_discovery` when items are known in advance | Breaks completion tracking | Use `by_selection` when the surveyor works through a fixed list |
| Omitting `requires_spatial` for defects that need localization | Note has no position; can't trace to a wall or zone | Set `requires_spatial: true` for any defect the report needs located |
| Wrong `wizardMode` | Confusing step order for the surveyor | Use `catalogueFirst` when the catalogue choice drives field visibility |
| Missing presentation config | Note appears blank in list view | Add `fieldPath` for title and `pills` for key distinguishing fields |
