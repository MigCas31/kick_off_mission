# Diagnostic CSV Structure Documentation

## Overview

The `diag_csv` folder contains CSV template files that define the structure and configuration for different diagnostic reports. Each diagnostic type (e.g., Lead CREP, Asbestos/Amiante) and administrative data has its own dedicated folder containing standardized CSV templates.

## Folder Structure

```
diag_csv/
├── Admin_data/          # Administrative data templates (shared across diagnostics)
├── lead_CREP/          # Lead diagnostic templates
├── Absestos_Amiante/   # Asbestos diagnostic templates
└── termite/            # Termite diagnostic templates (incomplete - not documented)
```

Each diagnostic folder contains a set of CSV template files that work together to define:
- **Tasks**: The workflow steps and actions required for the diagnostic
- **Variables**: Data fields collected during the diagnostic process
- **Enums**: Controlled value lists for dropdown selections
- **Catalogues**: Structured data collections (when applicable)
- **Generators**: Dynamic form generators (when applicable)
- **Note Types**: Observation capture templates (when applicable)

---

## CSV File Types

### 1. variables_template.csv

**Purpose**: Defines all data fields (variables) that can be collected during the diagnostic process. Each variable is associated with a specific task and defines its properties, validation rules, and data source.

**Key Columns**:
- `variable_key`: Unique identifier for the variable (e.g., `laboratory_name`, `calibration_measurement_value`)
- `label_en` / `label_fr`: English and French labels for UI display
- `description`: Detailed description of what the variable represents (must use semicolons, not commas)
- `task_key`: References a `task_id` from `tasks_template.csv` - indicates which task this variable belongs to
- `variable_kind`: Data type (`string`, `number`, `boolean`, `rich_text`, `enum_single`, `enum_multiple`, `tags`, `catalogue`, `generator`, `computed`)
- `value_source_type`: How the value is obtained (`user_input`, `system_provided`, `computed`, `inherited`, `static_default`, `derived`)
- `level_registered_at`: Hierarchical level where variable is stored (`case`, `report`, `building`, `storey`, `room`)
- `level_belongs_to`: The level the variable belongs to
- `required_default`: Default required status (`true`/`false`)
- `required_if`: Conditional requirement (e.g., `minors_present=true`)
- `enum_ref`: Reference to an enum in `enums_template.csv` (for enum-type variables)
- `catalogue_ref`: Reference to a catalogue (if applicable)
- `generator_ref`: Reference to a generator (if applicable)
- `unit`: Unit of measurement (e.g., `mg/cm²`, `date`, `file`)
- `status`: Status of the variable (`draft`, `final`, etc.)

**Example**:
```csv
variable_key,label_en,label_fr,description,task_key,variable_kind,value_source_type,level_registered_at,level_belongs_to,standard_key,required_default,required_if,enum_ref,catalogue_ref,generator_ref,unit,status
calibration_measurement_value,Calibration Measurement Value,Valeur de mesure d'étalonnage,Calibration measurement value taken at beginning of mission,lead_004,number,user_input,report,report,,true,,,mg/cm²,draft
```

**Relationships**:
- Variables reference tasks via `task_key` → `tasks_template.csv.task_id`
- Variables reference enums via `enum_ref` → `enums_template.csv.enum_key`
- Variables can reference catalogues and generators when applicable

---

### 2. tasks_template.csv

**Purpose**: Defines the workflow tasks/steps that make up the diagnostic process. Tasks represent discrete actions that users must perform, such as registering a device, entering measurements, or capturing observations.

**Key Columns**:
- `task_id`: Unique identifier (e.g., `lead_001`, `admin_002`)
- `task_name`: Human-readable task name in English
- `task_description`: Detailed description of what the task accomplishes
- `pattern`: Task pattern type (e.g., `form`, `generator`, `observation_capture`)
- `lifecycle_phase`: Phase in application lifecycle (`offsite-pre`, `onsite`, `offsite-post`)
- `lifecycle_level`: Hierarchical level where task operates (`case`, `report`, `building`, `storey`, `room`)
- `lifecycle_moment`: Moment in lifecycle (`before_arriving`, `on_creation`, `on_overview`, `on_completion`)
- `stage`: Stage identifier (e.g., `1.A`, `2.C`)
- `step`: Step identifier (e.g., `case.selection`, `room.overview`)
- `requiredness`: When task is required (`always`, `optional`, `always_if`)
- `condition`: Condition expression if `requiredness=always_if` (e.g., `report_type_selection includes french_lead`)
- `has_coverage_rules`: Boolean indicating if coverage rules apply
- `coverage_rules_description`: Description of coverage rules if applicable
- `ui_type`: UI component type (`form`, `checklist_with_answers`, `catalogue_registration`, `generator`, `observation_capture`)
- `note_type_keys`: Comma-separated list of note type keys (for `observation_capture` tasks)
- `catalogue_key`: Reference to catalogue if `ui_type=catalogue_registration`
- `generator_key`: Reference to generator if `ui_type=generator`

**Example**:
```csv
task_id,task_name,task_description,pattern,lifecycle_phase,lifecycle_level,lifecycle_moment,stage,step,requiredness,condition,has_coverage_rules,coverage_rules_description,ui_type,note_type_keys,catalogue_key,generator_key
lead_010,Element Sample Capture,Capture samples of building elements with properties substrate coating degradation nature type conservation state and measurements discovery task with pictures,observation_capture,onsite,room,on_overview,2.G,room.overview,always,,false,,observation_capture,lead_element_sample,,
```

**Relationships**:
- Tasks can reference catalogues via `catalogue_key` → `catalogues_template.csv.catalogue_key`
- Tasks can reference generators via `generator_key` → `generators_template.csv.generator_key`
- Tasks can reference note types via `note_type_keys` (comma-separated) → `note_types_template.csv.note_type_key`
- Variables reference tasks via `variables_template.csv.task_key` → `tasks_template.csv.task_id`

---

### 3. enums_template.csv

**Purpose**: Defines controlled value lists (enumerations) for dropdown selections and multi-select fields. Enums provide standardized options for variables of type `enum_single` or `enum_multiple`.

**Key Columns**:
- `enum_key`: Unique identifier for the enum (e.g., `lead_height`, `country`, `mission_analysis_scope`)
- `enum_title`: Human-readable title for the enum
- `item_value`: Value identifier (MUST be in English, lowercase with underscores, e.g., `ht_less_than_0_5m`)
- `item_label_en`: English label for the item
- `item_label_fr`: French label for the item

**Important Notes**:
- Multiple rows can have the same `enum_key` - each row represents one option in the enum
- The combination of `enum_key` + `item_value` must be unique
- `item_value` must follow the format: lowercase English with underscores (e.g., `private_parts`, not `Private Parts`)

**Example**:
```csv
enum_key,enum_title,item_value,item_label_en,item_label_fr
lead_height,Height,ht_less_than_0_5m,Height less than 0.5 m,Ht < 0,5 m
lead_height,Height,ht_0_5_to_1m,Height between 0.5 m and 1 m,0,5 m < Ht < 1 m
lead_height,Height,ht_1_to_1_5m,Height between 1 m and 1.5 m,1 m < Ht < 1,5 m
mission_analysis_scope,Mission Analysis Scope,private_parts,Private parts,Analyse des parties privatives
mission_analysis_scope,Mission Analysis Scope,occupied_parts,Occupied parts,Analyse des parties occupées
```

**Relationships**:
- Variables reference enums via `variables_template.csv.enum_ref` → `enums_template.csv.enum_key`

---

### 4. catalogues_template.csv

**Purpose**: Defines catalogue structures for catalogue-type UI components. Catalogues are used for structured data collections that require a wizard-like flow for registration.

**Key Columns**:
- `catalogue_key`: Unique identifier for the catalogue
- `catalogue_title`: Title of the catalogue
- `catalogue_description`: Description of the catalogue
- `catalogue_label_en`: English label for UI display
- `catalogue_label_fr`: French label for UI display
- `wizard_flow_step_label_en`: English label for wizard step
- `wizard_flow_step_label_fr`: French label for wizard step
- `wizard_flow_step`: Step identifier in wizard flow
- `step_visibility_rule`: Rule for step visibility

**Note**: Catalogues are typically used in `Admin_data` folder and may be empty in diagnostic-specific folders if not needed.

**Relationships**:
- Tasks reference catalogues via `tasks_template.csv.catalogue_key` → `catalogues_template.csv.catalogue_key`

---

### 5. generators_template.csv

**Purpose**: Defines generator structures for generator-type UI components. Generators create dynamic forms with multiple steps, where each step can have different fields and validation rules.

**Key Columns**:
- `generator_key`: Unique identifier for the generator
- `generator_title`: Title of the generator
- `generator_description`: Description of the generator
- `generator_label_en`: English label for UI display
- `generator_label_fr`: French label for UI display
- `step_visibility_rule`: Rule for step visibility
- `step_key`: Identifier for a step within the generator
- `step_order`: Order of the step in the generator flow
- `variable_key`: Variable that appears in this step
- `variable_kind`: Type of the variable
- `field_type`: UI field type (`text`, `enum`, `date`, `file`, etc.)
- `is_step`: Boolean indicating if this is a step separator
- `is_inherited_field`: Boolean indicating if field is inherited
- `is_computed_field`: Boolean indicating if field is computed
- `enum_ref`: Reference to enum if applicable

**Example**:
```csv
generator_key,generator_title,generator_description,generator_label_en,generator_label_fr,step_visibility_rule,step_key,step_order,variable_key,variable_kind,field_type,is_step,is_inherited_field,is_computed_field,enum_ref
lead_measurement_device,Lead Measurement Device,Register measurement device with manufacturer model serial number calibration info and ASN authorization details,Lead Measurement Device,Appareil de mesure plomb,,device_name,1,device_name,string,text,true,false,false,
lead_measurement_device,Lead Measurement Device,Register measurement device with manufacturer model serial number calibration info and ASN authorization details,Lead Measurement Device,Appareil de mesure plomb,,manufacturer,4,manufacturer,enum_single,enum,true,false,false,lead_device_manufacturer
```

**Relationships**:
- Tasks reference generators via `tasks_template.csv.generator_key` → `generators_template.csv.generator_key`
- Generators can reference enums via `enum_ref` → `enums_template.csv.enum_key`

---

### 6. note_types_template.csv

**Purpose**: Defines note type structures for observation capture tasks. Note types are used when tasks have `ui_type=observation_capture` and define the fields that can be captured during on-site observations (e.g., photos, location, measurements, descriptions).

**Key Columns**:
- `note_type_key`: Unique identifier for the note type (primary key, links to `tasks.note_type_keys`)
- `note_type_title`: Human-readable title for the note type
- `note_type_description`: Detailed description of the note type
- `note_type_label_en`: English label for the note type
- `note_type_label_fr`: French label for the note type
- `variable_key`: Unique identifier for the variable within the note type
- `variable_kind`: Type of variable (`image`, `location`, `text`, `rich_text`, `number`, `string`, `enum_single`, etc.)
- `field_order`: Order of the field in the note type (integer, must be sequential and unique within a note type)
- `label_en`: English label for the variable
- `label_fr`: French label for the variable
- `required`: Whether the field is required (`true`/`false`)
- `description`: Description of the variable (must use semicolons, not commas)
- `enum_ref`: Reference to enum if `variable_kind` is `enum_single` or `enum_multiple`

**Important Notes**:
- Multiple rows can have the same `note_type_key` - each row represents one field in the note type
- The combination of `note_type_key` + `variable_key` must be unique
- `field_order` values must be sequential and unique within the same `note_type_key`
- Description fields must not contain commas (use semicolons instead)

**Example**:
```csv
note_type_key,note_type_title,note_type_description,note_type_label_en,note_type_label_fr,variable_key,variable_kind,field_order,label_en,label_fr,required,description,enum_ref
lead_element_sample,Lead Element Sample,Observation note for capturing lead element samples with location substrate coating measurements and degradation information,Lead Element Sample,Échantillon d'élément plomb,picture,image,0,Picture,Photo,true,Photo of the sampled element,
lead_element_sample,Lead Element Sample,Observation note for capturing lead element samples with location substrate coating measurements and degradation information,Lead Element Sample,Échantillon d'élément plomb,position,location,1,Position,Position,true,Spatial location of the element sample,
lead_element_sample,Lead Element Sample,Observation note for capturing lead element samples with location substrate coating measurements and degradation information,Lead Element Sample,Échantillon d'élément plomb,hauteur,enum_single,3,Height,Hauteur,true,Height range of the element,lead_height
```

**Relationships**:
- Tasks reference note types via `tasks_template.csv.note_type_keys` (comma-separated) → `note_types_template.csv.note_type_key`
- Note types can reference enums via `enum_ref` → `enums_template.csv.enum_key`

---

## Data Relationships

The CSV files are interconnected through reference keys:

```
tasks_template.csv (task_id)
    ↑
    │ (task_key)
    │
variables_template.csv
    │
    ├─→ (enum_ref) → enums_template.csv (enum_key)
    ├─→ (catalogue_ref) → catalogues_template.csv (catalogue_key)
    └─→ (generator_ref) → generators_template.csv (generator_key)

tasks_template.csv
    ├─→ (catalogue_key) → catalogues_template.csv (catalogue_key)
    ├─→ (generator_key) → generators_template.csv (generator_key)
    └─→ (note_type_keys) → note_types_template.csv (note_type_key)
```

---

## Folder-Specific Notes

### Admin_data/
Contains shared administrative data templates used across all diagnostics:
- Property information
- Owner and client details
- Surveyor information
- Room designations and materials
- Common enums (country, property types, etc.)

### lead_CREP/
Contains Lead diagnostic-specific templates:
- Device registration and calibration
- Element sampling and measurement
- Laboratory results entry
- Lead-specific enums (substrates, coatings, degradation types, etc.)
- Note types for element sample capture

### Absestos_Amiante/
Contains Asbestos diagnostic templates (structure to be documented when populated).

### termite/
Contains Termite diagnostic templates (incomplete - not documented in this file).

---

## Validation Rules

When working with these CSV files, ensure:

1. **CSV Format**: Commas must only be used as column separators. Use semicolons (`;`) instead of commas within column values (especially in `description` fields).

2. **Reference Integrity**: 
   - All `task_key` values in variables must exist as `task_id` in tasks
   - All `enum_ref` values must exist as `enum_key` in enums
   - All `catalogue_key` values in tasks must exist in catalogues
   - All `generator_key` values in tasks must exist in generators
   - All `note_type_keys` in tasks must exist in note_types

3. **Enum Completeness**: Enum variables should reference enums with at least 2 items (unless boolean-like).

4. **Field Uniqueness**:
   - `task_id` must be unique in tasks
   - `variable_key` must be unique in variables
   - `enum_key` + `item_value` combination must be unique in enums
   - `note_type_key` + `variable_key` combination must be unique in note_types

5. **Field Order**: In note types, `field_order` values must be sequential and unique within the same `note_type_key`.

6. **Status Fields**: All variables must have a `status` field populated (typically `draft` for work in progress).

---

## Usage Workflow

1. **Define Tasks**: Start by defining tasks in `tasks_template.csv` that represent the workflow steps.

2. **Define Variables**: Create variables in `variables_template.csv` that belong to each task, specifying their types and validation rules.

3. **Define Enums**: Create enums in `enums_template.csv` for any dropdown or multi-select fields, then reference them in variables via `enum_ref`.

4. **Define Note Types** (if needed): For observation capture tasks, define note types in `note_types_template.csv` with their fields, then reference them in tasks via `note_type_keys`.

5. **Define Generators** (if needed): For complex dynamic forms, define generators in `generators_template.csv`, then reference them in tasks via `generator_key`.

6. **Define Catalogues** (if needed): For structured data collections, define catalogues in `catalogues_template.csv`, then reference them in tasks via `catalogue_key`.

7. **Validate**: Ensure all references are valid and all required fields are populated.

---

## Best Practices

- Use descriptive, consistent naming conventions for keys (e.g., `lead_` prefix for lead diagnostic)
- Keep descriptions clear and concise, using semicolons instead of commas
- Ensure enum `item_value` fields are in English, lowercase, with underscores
- Maintain sequential `field_order` values in note types
- Always set `status` fields appropriately
- Validate cross-references after making changes
- Use `required_if` conditions for conditional field requirements
- Document any special validation rules in task descriptions
