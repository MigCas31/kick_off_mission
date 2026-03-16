# Value Source Types: inherited, derived, and computed

## Overview

Every variable in the diagnostic CSV templates has a `value_source_type` that describes how it gets its initial value. This document explains the differences between three related types: `inherited`, `derived`, and `computed`.

---

## Quick Comparison

| Aspect | `inherited` | `derived` | `computed` |
|--------|-------------|-----------|------------|
| **Source** | Parent level in hierarchy | Sibling variables at same level | Other variables via formula/lookup |
| **Where assigned** | Set at parent level during setup | Computed automatically at runtime | Computed via formula/lookup table |
| **Scope** | Cross-level (parent → child) | Same level only | Can be cross-level |
| **Complexity** | Direct value copy | Simple logic/comparison | Complex formulas/tables |
| **Editable** | May be editable or read-only | Always read-only | Always read-only |
| **Use when** | Value captured once, reused | Simple sibling comparison | External tables/formulas needed |

---

## inherited

### Definition

Read from a parent level at runtime. The value is set at a higher level in the hierarchy and flows down to child levels.

### Characteristics

- **Source**: Parent level (building, storey, case, room)
- **Assignment**: Set during level setup at the parent level
- **Scope**: Cross-level inheritance (parent → child)
- **Editable**: May be read-only or editable (surveyor can override)
- **Complexity**: Direct value copy, no calculation

### Where Values Are Assigned

Values are assigned at parent levels during level setup:

- **Building level**: `construction_year`, `building_type`, `heating_system_type`
- **Storey level**: `floor_separation_thickness`, default materials
- **Room level**: `room_type`, `room_area`
- **Case level**: `climate_zone`, property address

### How It Works

1. A variable is set at a parent level (e.g., building setup in EPC report)
2. The value becomes available to ALL reports and tasks at that level and below
3. Cross-report inheritance: values captured in one report are available to other reports
4. Source is specified by source level and source field: "read `climate_zone` from `case` level"

### Examples

**Example 1: Building Year**
```csv
variable_key: construction_year
value_source_type: inherited
level_registered_at: building
level_belongs_to: building
description: Building construction year inherited from building setup
```

- Set at: Building setup (EPC report)
- Inherited by: Electrical, Gas, Lead, Condition reports at building level

**Example 2: Room Type**
```csv
variable_key: room_type
value_source_type: inherited
level_registered_at: room
level_belongs_to: room
description: Room type set during room creation
```

- Set at: Room creation (form at on_creation)
- Inherited by: Conditional task activation across ALL active reports (bathroom triggers wet-room checks for Electrical, zone measurements for Lead)

**Example 3: Wall Substrate**
```csv
variable_key: wall_substrate
value_source_type: inherited
level_registered_at: building_element
level_belongs_to: building_element
description: Material substrate inherited from EPC material assignment
```

- Set at: EPC material assignment
- Inherited by: Lead report for pre-filling substrate in XRF measurement notes

### When to Use

Use `inherited` when:
- Value is captured once at a parent level and reused
- Value comes from building/storey/case data
- You want to avoid re-entering the same information
- The value should be available across multiple reports

### Key Principle

**"Capture once, inherit everywhere"** - The surveyor should never enter the same information twice.

---

## derived

### Definition

Simple logic over sibling variables at the same level. Non-interactive (surveyor cannot edit).

### Characteristics

- **Source**: Sibling variables at the same level
- **Assignment**: Computed automatically at runtime
- **Scope**: Same level only (no cross-level)
- **Editable**: Always read-only
- **Complexity**: Simple comparisons or boolean logic

### Where Values Are Assigned

- Computed automatically at runtime based on sibling variables
- Uses simple expressions or comparisons
- Recomputes when input variables change

### How It Works

1. Operates on variables at the same level (siblings)
2. Uses simple expressions or boolean logic
3. Recomputes reactively when input variables change
4. No external tables or complex formulas

### Examples

**Example 1: Overall Compliance Status**
```csv
variable_key: overall_compliance_status
value_source_type: derived
level_registered_at: room
level_belongs_to: room
description: Overall compliance derived from individual checkpoint results
# Expression: all(checkpoint_1_pass, checkpoint_2_pass, checkpoint_3_pass)
# If all sibling checkpoints pass → "compliant", else "non_compliant"
```

**Example 2: All Checkpoints Pass**
```csv
variable_key: all_checkpoints_pass
value_source_type: derived
level_registered_at: room
level_belongs_to: room
description: Boolean indicating if all checkpoints at this level pass
# Expression: checkpoint_1_pass AND checkpoint_2_pass AND checkpoint_3_pass
```

### When to Use

Use `derived` when:
- Simple comparison/logic over sibling variables at the same level
- Example: "all checkpoints pass → compliant"
- No external tables or complex formulas needed
- Logic can be expressed as a simple boolean expression

### Key Principle

**Simple logic only** - Use for straightforward comparisons, not complex calculations.

---

## computed

### Definition

Derived from other variables via formula, lookup table, or cross-level aggregation. Non-interactive (surveyor cannot edit).

### Characteristics

- **Source**: Other variables via formula, lookup table, or cross-level aggregation
- **Assignment**: Computed via formula/lookup table at runtime
- **Scope**: Can be cross-level (aggregates from multiple levels)
- **Editable**: Always read-only
- **Complexity**: Complex formulas, lookup tables, regulatory calculations

### Where Values Are Assigned

- Computed via formula or lookup table
- May aggregate data from multiple levels
- Uses external computation tables or regulatory formulas
- Recomputes when input variables change

### How It Works

1. Uses formulas, lookup tables, or cross-level aggregation
2. Can reference variables from multiple levels
3. Two formula types supported:
   - **`lookup`**: table-based — system looks up value in computation table
   - **`expression`**: arithmetic — system evaluates formula referencing other fields
4. Part of computation DAG (Directed Acyclic Graph)

### Examples

**Example 1: Energy Efficiency**
```csv
variable_key: energy_efficiency
value_source_type: computed
level_registered_at: building
level_belongs_to: building
description: Energy efficiency calculated from regulatory formula
# Uses lookup table from regulatory standard
# May aggregate data from multiple levels (building, storey, room)
```

**Example 2: Element Classification**
```csv
variable_key: lab_result_classification
value_source_type: computed
level_registered_at: report
level_belongs_to: report
description: Element classification computed from measurement value and other factors
# Uses regulatory formula or lookup table
```

**Example 3: Minimum Conductor Cross-Section**
```csv
variable_key: minimum_conductor_cross_section
value_source_type: computed
level_registered_at: room
level_belongs_to: room
description: Minimum conductor cross-section from NF C 15-100 Table 52E
# Uses external lookup table (NF C 15-100 Table 52E)
```

### When to Use

Use `computed` when:
- External lookup tables are involved
- Cross-level aggregation is needed
- Regulatory formulas are required
- More complex than simple sibling comparison
- Formula or table-based calculation is needed

### Key Principle

**Complex calculations** - Use when simple derived logic is insufficient.

---

## Decision Guide

### When to Use Each Type

```
Is the value set at a parent level?
├─ Yes → Use `inherited`
│   └─ Value flows from parent to child levels
│
└─ No → Is it calculated from other variables?
    ├─ Simple logic over siblings at same level?
    │   └─ Yes → Use `derived`
    │       └─ Example: all(checkpoints) → compliant
    │
    └─ Complex formula/lookup table/cross-level?
        └─ Yes → Use `computed`
            └─ Example: energy efficiency from regulatory formula
```

### Comparison: derived vs computed

| Criteria | `derived` | `computed` |
|----------|-----------|------------|
| **Complexity** | Simple | Complex |
| **Scope** | Same level only | Can be cross-level |
| **Method** | Simple boolean/logic | Formula/lookup table |
| **Example** | "all checkpoints pass" | "energy efficiency from regulatory calc" |
| **External data** | No | Yes (tables, formulas) |

**Rule of thumb:**
- Use `derived` for simple comparisons over siblings (e.g., "all checkpoints pass → compliant")
- Use `computed` when external lookup tables, cross-level aggregation, or regulatory formulas are involved (e.g., "minimum conductor cross-section from NF C 15-100 Table 52E")

---

## Examples in Context

### Example: Building Inspection Report

**inherited:**
- `construction_year` - Set at building setup, inherited by all reports
- `room_type` - Set at room creation, inherited by all room-level tasks
- `climate_zone` - Set at case level, inherited by building-level calculations

**derived:**
- `all_electrical_checkpoints_pass` - Derived from sibling checkpoint variables at room level
- `overall_compliance` - Derived from individual compliance flags at same level

**computed:**
- `energy_efficiency_rating` - Computed from building data using regulatory formula
- `minimum_wire_size` - Computed from lookup table (NF C 15-100)
- `lab_result_classification` - Computed from measurement value and degradation factors

---

## Implementation Notes

### For inherited Variables

- Specify source level and source field: "read `climate_zone` from `case` level"
- Variable can be editable (surveyor can override) or read-only
- In generators, inherited fields are listed in `inherited_fields` section
- If inherited field is missing, dependent computed fields show "?" until parent value is available

### For derived Variables

- Specify inputs (sibling variables at same level)
- Define expression (simple boolean/logic)
- Always read-only
- Recomputes reactively when inputs change

### For computed Variables

- Specify inputs (can be from multiple levels)
- Define computation method:
  - `lookup`: table-based computation
  - `expression`: arithmetic formula
- Reference computation tables if needed
- Part of computation DAG (Directed Acyclic Graph)
- Always read-only
- Recomputes reactively when inputs change

---

## Related Concepts

### Computation DAG (Directed Acyclic Graph)

Variables can depend on other variables, forming a DAG:
- `inputs_from`: This variable reads from these other variables
- `outputs_to`: This variable feeds into these other variables

Used for:
- **Reactive computation**: when a variable changes, all downstream variables recompute
- **Validation ordering**: ensure inputs are available before computed variables evaluate
- **Impact analysis**: trace which variables are affected by a change

### Cross-Report Inheritance

The `inherited` value source enables the shared data layer principle:
- When a variable is set at a level, it becomes available to ALL reports and tasks at that level and below
- Example: Building year set in EPC is available to Electrical, Gas, Lead, Condition reports
- Prevents duplicate data entry across reports

---

## References

- [Variables Documentation](../md_kick-off/variables.md) - Full variable specification
- [Tasks Documentation](../md_kick-off/tasks.md) - Inheritance and auto-computation patterns
- [Generators Documentation](../md_kick-off/generators.md) - Inherited and computed fields in generators
- [CSV Templates Logic](csv_templates_logic.md) - CSV structure and relationships
