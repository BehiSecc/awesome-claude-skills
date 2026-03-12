---
name: ontology-mapper
description: >
  Map materials science terms, crystal structures, and sample descriptions to ontology classes
  and properties. Supports any ontology registered in ontology_registry.json. Use when translating
  natural-language material descriptions to ontology terms, annotating simulation inputs with
  ontology metadata, or mapping crystal parameters (space group, Bravais lattice, lattice constants)
  to standardized ontology representations.
allowed-tools: Read, Bash
---

# Ontology Mapper

## Goal

Translate real-world materials science descriptions into standardized ontology annotations. Given terms like "FCC copper" or structured data like `{"material": "iron", "structure": "BCC", "lattice_a": 2.87}`, produce the corresponding ontology classes and properties for any registered ontology.

## Requirements

- Python 3.8+
- No external dependencies (Python standard library only)
- Requires ontology-explorer's summary JSON and `ontology_registry.json`
- Per-ontology mapping config (`<name>_mappings.json`) for ontology-specific synonyms and labels

## Inputs to Gather

| Input | Description | Example |
|-------|-------------|---------|
| Ontology | Ontology name from registry | `cmso`, `asmo` |
| Term(s) | Natural-language materials concept(s) | `"unit cell"`, `"FCC,copper,lattice"` |
| Crystal system | One of the 7 crystal systems | `cubic`, `hexagonal` |
| Bravais lattice | Lattice type (symbol or common name) | `FCC`, `cF`, `BCC` |
| Space group | Space group number (1-230) | `225` |
| Lattice parameters | a, b, c in angstroms; alpha, beta, gamma in degrees | `a=3.615` |
| Sample description | JSON dict with material properties | `{"material":"copper","structure":"FCC"}` |

## Decision Guidance

```
What do you need to map?
├── A concept or term to find its ontology class
│   └── concept_mapper.py --ontology <name> --term "<term>"
├── Crystal structure parameters to ontology terms
│   └── crystal_mapper.py --ontology <name> --bravais <type> --space-group <N> --a <val>
├── A full sample description to ontology annotations
│   └── sample_annotator.py --ontology <name> --sample '<json>'
└── Multiple terms at once
    └── concept_mapper.py --ontology <name> --terms "term1,term2,term3"
```

## Script Outputs (JSON Fields)

| Script | Key Outputs |
|--------|-------------|
| `scripts/concept_mapper.py` | `results.matches`, `results.unmatched`, `results.suggestions` |
| `scripts/crystal_mapper.py` | `results.ontology_classes`, `results.ontology_properties`, `results.effective_system`, `results.bravais_lattice`, `results.validation_warnings` |
| `scripts/sample_annotator.py` | `results.annotations`, `results.sample_type`, `results.material_type`, `results.unmapped_fields`, `results.suggested_properties` |

## Workflow

1. If the user provides natural-language terms, use `concept_mapper.py` to find matching ontology classes.
2. If the user describes crystal structure parameters, use `crystal_mapper.py` to map them and validate constraints.
3. For a complete sample description, use `sample_annotator.py` to produce full ontology annotations.
4. Review any validation warnings (e.g., lattice parameter mismatches for the crystal system).
5. Check `unmapped_fields` and `suggested_properties` for completeness.

## Conversational Workflow Example

```
User: I'm setting up an MD simulation of BCC iron with lattice parameter 2.87 angstroms.
      What CMSO terms should I use?

Agent: Let me map your iron sample to CMSO ontology terms.

[Runs: sample_annotator.py --ontology cmso --sample '{"material":"iron","structure":"BCC","lattice_a":2.87,"space_group":229}' --json]

Your BCC iron simulation maps to these CMSO annotations:

- **Sample**: Atomic Scale Sample (subclass of Computational Sample)
- **Material**: Crystalline Material
- **Unit Cell**: Bravais lattice = "cI" (body-centered cubic)
- **Space Group**: number = 229 (Im-3m)
- **Lattice**: a = 2.87 Å
- **Element**: Fe

Suggested additions:
- Number of atoms in the simulation cell
- Simulation cell vectors and angles
```

## CLI Examples

```bash
# Map a single concept
python3 skills/ontology/ontology-mapper/scripts/concept_mapper.py \
  --ontology cmso --term "space group" --json

# Map multiple terms
python3 skills/ontology/ontology-mapper/scripts/concept_mapper.py \
  --ontology cmso --terms "FCC,copper,lattice constant" --json

# Map crystal parameters (with ontology-specific labels)
python3 skills/ontology/ontology-mapper/scripts/crystal_mapper.py \
  --ontology cmso --bravais FCC --space-group 225 --a 3.615 --json

# Map crystal parameters (generic labels, no ontology specified)
python3 skills/ontology/ontology-mapper/scripts/crystal_mapper.py \
  --bravais FCC --space-group 225 --a 3.615 --json

# Annotate a full sample
python3 skills/ontology/ontology-mapper/scripts/sample_annotator.py \
  --ontology cmso \
  --sample '{"material":"copper","structure":"FCC","space_group":225,"lattice_a":3.615}' \
  --json
```

## Adding a New Ontology

To support a new ontology (e.g., ASMO), create a `<name>_mappings.json` in `references/`:

```json
{
  "ontology": "asmo",
  "synonyms": { "simulation method": "Simulation Method", ... },
  "property_synonyms": { "timestep": "has timestep", ... },
  "material_type_rules": { "keyword_rules": [...], "default": "Material" },
  "sample_schema": { "sample_class": "Simulation", ... },
  "crystal_output": { "base_classes": [...], "property_map": {...} },
  "annotation_routing": { "unit_cell_indicators": [...], ... }
}
```

Then add `"mappings_file": "asmo_mappings.json"` to the ontology's entry in `ontology_registry.json`. No code changes needed.

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `space_group must be between 1 and 230` | Invalid space group number | Use a valid space group number |
| `a must be positive` | Non-positive lattice parameter | Provide positive values in angstroms |
| `Sample must be a non-empty dict` | Empty or missing sample data | Provide a valid JSON sample dict |
| Validation warnings | Lattice parameters inconsistent with crystal system | Check that a=b=c for cubic, etc. |

## Interpretation Guidance

- **Confidence scores**: 1.0 = exact match, 0.9 = synonym match, 0.7 = substring match, 0.5 = description match
- **Validation warnings**: indicate potential mistakes (e.g., specifying a!=b for cubic). These are warnings, not errors — the mapping still proceeds.
- **Unmapped fields**: input keys that the annotator doesn't recognize. These may need manual mapping.
- **Suggested properties**: additional ontology properties that would make the annotation more complete.

## Limitations

- Concept mapping uses string matching and a per-ontology synonym table; it does not understand arbitrary natural language
- Crystal system validation checks basic constraints only (not all crystallographic rules)
- The element resolver recognizes common element names and symbols but may miss unusual spellings
- Bravais lattice aliases cover common usage (FCC, BCC, HCP) but not all crystallographic notation variants

## References

- [Mapping Patterns](references/mapping_patterns.md) — common mapping examples
- [Crystal Systems](references/crystal_systems.json) — crystal system definitions and Bravais lattices
- [Element Data](references/element_data.json) — periodic table data
- [CMSO Mappings](references/cmso_mappings.json) — CMSO-specific synonym tables and annotation config
- [CMSO Guide](../ontology-explorer/references/cmso_guide.md) — CMSO ontology overview

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-02-25 | 1.1 | Refactored for multi-ontology support: externalized CMSO-specific knowledge to config |
| 2026-02-25 | 1.0 | Initial release with CMSO mapping support |
