# CSV Templates Logic

This document explains the row structure and relationships for the CSV templates used to describe catalogues, generators, and enums.

## General Principle

Each CSV template uses a **denormalized structure** where multiple rows can belong to the same entity. The grouping is done by a key column that identifies which entity the row belongs to.

---

## Catalogue Template (`catalogues_template.csv`)

### Logic: 1 row = 1 step/column of a catalogue

**Key column:** `catalogue_key`

**Structure:**
- Multiple rows can share the same `catalogue_key` (they belong to the same catalogue)
- Each row represents either:
  - A **filter column** (used in wizard steps to narrow the result set)
  - A **data column** (stored on templates, not used for filtering)
  - A **template** (a specific entry in the catalogue)

**Example:**
```
catalogue_key: "french_electrical_defects"
Row 1: Filter column "control_sheet" (order: 1)
Row 2: Filter column "severity" (order: 2)
Row 3: Data column "anomaly_description"
Row 4: Template "B.1.a" with values
Row 5: Template "B.1.b" with values
```

**Wizard flow:**
- Step 1: Filter by `control_sheet` → narrows templates
- Step 2: Filter by `severity` → further narrows templates
- Final: Select one template from the filtered set

**Key fields:**
- `is_filter_column` / `is_data_column` - identifies column type
- `filter_sequence_order` - determines wizard step order
- `template_key` - identifies individual templates (rows with same catalogue_key but different template_key)

---

## Generator Template (`generators_template.csv`)

### Logic: 1 row = 1 step/field of a generator

**Key column:** `generator_key`

**Structure:**
- Multiple rows can share the same `generator_key` (they belong to the same generator definition)
- Each row represents either:
  - A **step** (one question in the generator flow)
  - An **inherited field** (read from parent level)
  - A **computed field** (derived from step selections)

**Example:**
```
generator_key: "heating_installation"
Row 1: Step "generator_type" (order: 1, enum_single)
Row 2: Step "fuel_type" (order: 2, filtered by generator_type)
Row 3: Step "distribution" (order: 3, filtered by generator_type)
Row 4: Step "installation_year" (order: 4, number, independent)
Row 5: Inherited field "climate_zone" (from building level)
Row 6: Computed field "efficiency" (from lookup table)
```

**Generator flow:**
- Surveyor answers Step 1 → may filter Step 2 and Step 3 options
- Surveyor answers Step 2 → independent of other steps
- Surveyor answers Step 3 → may filter other steps
- System computes derived values from all step selections

**Key fields:**
- `step_key` - identifies individual steps
- `step_order` - determines sequence in the form
- `step_options_filter_json` - defines how prior steps filter this step's options
- `inherited_field_ref` / `computed_field_ref` - identifies non-step fields

**Note:** Unlike catalogues (which filter to find one template), generators build configurations. Each combination of step answers produces a unique configuration.

---

## Enum Template (`enums_template.csv`)

### Logic: 1 row = 1 item value of an enum

**Key column:** `enum_key`

**Structure:**
- Multiple rows can share the same `enum_key` (they belong to the same enum)
- Each row represents **one possible value** that the enum can take
- All rows with the same `enum_key` together form the complete value list

**Example:**
```
enum_key: "checkpoint_result"
Row 1: value="compliant", label_en="Compliant", label_fr="Conforme"
Row 2: value="non_compliant", label_en="Non-compliant", label_fr="Non conforme"
Row 3: value="not_verifiable", label_en="Not verifiable", label_fr="Non vérifiable"
Row 4: value="not_applicable", label_en="Not applicable", label_fr="Non applicable"
```

**Usage:**
- Variables of kind `enum_single` or `enum_multiple` reference this enum via `enum_ref`
- The surveyor selects from the complete list of values defined by all rows with the same `enum_key`
- Can also be used as filter columns in catalogues

**Key fields:**
- `item_value` - machine-readable value (English snake_case, unique per enum)
- `item_label_*` - localized human-readable labels (en, fr, da, de)

---

## Summary Table

| Template | Key Column | Row Represents | Multiple Rows = |
|----------|------------|----------------|-----------------|
| **Catalogue** | `catalogue_key` | 1 step/column/template | Different steps/columns/templates of the same catalogue |
| **Generator** | `generator_key` | 1 step/field | Different steps/fields of the same generator |
| **Enum** | `enum_key` | 1 item value | Different possible values of the same enum |

---

## Working with the Templates

### To find all parts of an entity:
Filter rows where the key column matches the entity identifier.

**Example - Get all steps of a generator:**
```sql
SELECT * FROM generators_template 
WHERE generator_key = 'heating_installation'
ORDER BY step_order
```

### To understand relationships:
- **Catalogue → Templates:** Filter by `catalogue_key` and `template_key IS NOT NULL`
- **Catalogue → Filter columns:** Filter by `catalogue_key` and `is_filter_column = TRUE`, order by `filter_sequence_order`
- **Generator → Steps:** Filter by `generator_key` and `step_key IS NOT NULL`, order by `step_order`
- **Enum → Items:** Filter by `enum_key`, all rows together form the value list

---

## Notes

- **Catalogue and Generator are treated equally** in terms of structure: both use multiple rows per entity to represent their components (steps/columns/fields)
- **Enum is simpler**: multiple rows = multiple possible values, all belonging to one enum
- The denormalized structure allows easy editing in spreadsheet tools while maintaining clear entity relationships
