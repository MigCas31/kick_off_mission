# Structure Context

This document explains the overall structure for organizing diagnostic (diag) data, tasks, UI types, and their relationships.

## Overview

The structure follows a hierarchical organization where:
1. **Diags** (diagnostic reports) are split into **Tasks**
2. Each **Task** has one **UI Type**
3. Tasks with `catalogue` or `generator` UI types require a **primary key** to join the task with its UI type definition
4. Tasks can be linked with **Note Types**
5. Each diag folder contains empty template CSVs that will be filled with screenshots and prompts

---

## Structure Hierarchy

```
diag_csv/
  └── {diag_name}/
      ├── tasks_template.csv          (tasks for this diag)
      ├── catalogues_template.csv     (catalogues used by tasks)
      ├── generators_template.csv      (generators used by tasks)
      └── enums_template.csv           (enums used by tasks/catalogues/generators)

diag_sav/
  └── {diag_name}/
      ├── tasks_template.csv          (empty copy, to be filled)
      ├── catalogues_template.csv     (empty copy, to be filled)
      ├── generators_template.csv     (empty copy, to be filled)
      └── enums_template.csv          (empty copy, to be filled)
```

---

## Entity Relationships

### Diag → Tasks

Each diagnostic report (diag) is decomposed into multiple tasks. All tasks for a diag are stored in `{diag_name}/tasks_template.csv`.

**Key field:** `task_id` (unique identifier per task)

### Task → UI Type

Each task has exactly one UI type. The UI type determines the interaction pattern:

| UI Type | Description | Requires Primary Key? |
|---------|-------------|----------------------|
| `form` | Flat list of fields | No |
| `checklist_with_answers` | Yes/No questions | No |
| `checklist_with_evidence` | Evidence-based checklist | No |
| `catalogue_registration` | Catalogue selection | **Yes** → `catalogue_key` |
| `observation_capture` | Space-driven discovery | No |
| `generator` | Step-by-step configuration | **Yes** → `generator_key` |

### Task → Catalogue/Generator (Primary Key Join)

When a task has UI type `catalogue_registration` or `generator`, it must reference the specific catalogue or generator definition using a primary key:

- **Catalogue tasks:** Use `catalogue_key` in `tasks_template.csv` to join with `catalogues_template.csv`
- **Generator tasks:** Use `generator_key` in `tasks_template.csv` to join with `generators_template.csv`

**Example:**
```
Task: "Register heating installation"
UI Type: catalogue_registration
catalogue_key: "danish_epc_heating_installations"  ← joins to catalogues_template.csv
```

### Task → Note Types

Tasks can reference note types (for evidence capture patterns). Multiple note types can be linked to one task.

**Key field:** `note_type_keys` (comma-separated list or JSON array)

---

## Template Files per Diag

Each diag folder (`diag_csv/{diag_name}/` and `diag_sav/{diag_name}/`) should contain:

### 1. `tasks_template.csv`
- One row per task
- Contains: task_id, task_name, pattern, ui_type, catalogue_key (if applicable), generator_key (if applicable), note_type_keys, etc.
- **Source:** `context/tasks_template.csv`

### 2. `catalogues_template.csv`
- Multiple rows per catalogue (see [csv_templates_logic.md](csv_templates_logic.md))
- Contains: catalogue_key, filter columns, data columns, templates
- **Source:** `context/catalogues_template.csv`
- **Only needed if tasks use `catalogue_registration` UI type**

### 3. `generators_template.csv`
- Multiple rows per generator (see [csv_templates_logic.md](csv_templates_logic.md))
- Contains: generator_key, steps, inherited fields, computed fields
- **Source:** `context/generators_template.csv`
- **Only needed if tasks use `generator` UI type**

### 4. `enums_template.csv`
- Multiple rows per enum (see [csv_templates_logic.md](csv_templates_logic.md))
- Contains: enum_key, item values with labels
- **Source:** `context/enums_template.csv`
- **Needed if tasks, catalogues, or generators reference enums**

---

## Primary Key Relationships

### Catalogue Join
```
tasks_template.csv
  ├── task_id: "task_001"
  ├── ui_type: "catalogue_registration"
  └── catalogue_key: "french_electrical_defects"  ← PRIMARY KEY

catalogues_template.csv
  ├── catalogue_key: "french_electrical_defects"  ← PRIMARY KEY
  ├── Row 1: Filter column "control_sheet"
  ├── Row 2: Filter column "severity"
  └── Row 3: Template "B.1.a"
```

### Generator Join
```
tasks_template.csv
  ├── task_id: "task_002"
  ├── ui_type: "generator"
  └── generator_key: "heating_installation"  ← PRIMARY KEY

generators_template.csv
  ├── generator_key: "heating_installation"  ← PRIMARY KEY
  ├── Row 1: Step "generator_type"
  ├── Row 2: Step "fuel_type"
  └── Row 3: Inherited field "climate_zone"
```

---

## Workflow

### 1. Extract Tasks from Diag
- Analyze the diagnostic report (diag)
- Split into individual tasks
- Fill `diag_csv/{diag_name}/tasks_template.csv`

### 2. Identify UI Types
- For each task, determine its UI type
- If `catalogue_registration` or `generator`, assign a primary key

### 3. Define Catalogues/Generators
- If tasks use catalogues, fill `catalogues_template.csv` (see [csv_templates_logic.md](csv_templates_logic.md))
- If tasks use generators, fill `generators_template.csv` (see [csv_templates_logic.md](csv_templates_logic.md))
- Ensure primary keys match between tasks and UI type definitions

### 4. Define Enums
- Identify enums needed by tasks, catalogues, or generators
- Fill `enums_template.csv` (see [csv_templates_logic.md](csv_templates_logic.md))

### 5. Link Note Types
- Identify note types for evidence capture tasks
- Add `note_type_keys` to relevant tasks

### 6. Create Empty Templates in diag_sav
- Copy empty templates to `diag_sav/{diag_name}/`
- These will be filled with screenshots and prompts during implementation

---

## Validation Rules

1. **Primary Key Consistency:**
   - Every task with `ui_type = "catalogue_registration"` must have a `catalogue_key` that exists in `catalogues_template.csv`
   - Every task with `ui_type = "generator"` must have a `generator_key` that exists in `generators_template.csv`

2. **Template Completeness:**
   - If a diag has catalogue tasks, `catalogues_template.csv` must exist
   - If a diag has generator tasks, `generators_template.csv` must exist
   - `enums_template.csv` should exist if any enums are referenced

3. **Note Type References:**
   - `note_type_keys` should reference valid note type identifiers
   - Tasks with evidence capture patterns should have note types

---

## Example Structure

```
diag_csv/termite/
  ├── tasks_template.csv
  │   ├── task_001: "Register termite evidence" (catalogue_registration, catalogue_key="termite_evidence")
  │   ├── task_002: "Describe treatment" (generator, generator_key="termite_treatment")
  │   └── task_003: "General observations" (observation_capture, note_type_keys="termite_observation")
  │
  ├── catalogues_template.csv
  │   └── catalogue_key="termite_evidence" (multiple rows for steps/templates)
  │
  ├── generators_template.csv
  │   └── generator_key="termite_treatment" (multiple rows for steps/fields)
  │
  └── enums_template.csv
      └── enum_key="termite_severity" (multiple rows for values)

diag_sav/termite/
  ├── tasks_template.csv (empty, for screenshots/prompts)
  ├── catalogues_template.csv (empty, for screenshots/prompts)
  ├── generators_template.csv (empty, for screenshots/prompts)
  └── enums_template.csv (empty, for screenshots/prompts)
```

---

## References

- [CSV Templates Logic](csv_templates_logic.md) - Detailed explanation of row structure for catalogues, generators, and enums
- [Tasks Template](../context/tasks_template.csv) - Task template structure
- [Catalogues Template](../context/catalogues_template.csv) - Catalogue template structure
- [Generators Template](../context/generators_template.csv) - Generator template structure
- [Enums Template](../context/enums_template.csv) - Enum template structure
