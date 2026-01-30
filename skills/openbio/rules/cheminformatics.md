# Cheminformatics Tools

Molecular analysis: RDKit, PubChem, ChEMBL, SA Score.

## Tools

### RDKit

| Tool | Description |
|------|-------------|
| `calculate_molecular_properties` | MW, LogP, HBD, HBA, TPSA |
| `validate_and_canonicalize_smiles` | Validate/canonicalize SMILES |
| `calculate_fingerprint_similarity` | Tanimoto similarity |
| `search_substructure` | Substructure search |
| `calculate_molecular_descriptors` | 200+ descriptors |
| `standardize_molecule` | Neutralize, tautomerize |
| `generate_conformers` | Generate 3D conformers |
| `cluster_molecules` | Cluster by similarity |
| `search_similar_molecules` | Find similar molecules |
| `calculate_dipole_moment` | Calculate dipole |

### SA Score

| Tool | Description |
|------|-------------|
| `calculate_sa_score` | Synthetic Accessibility (1=easy, 10=hard) |
| `calculate_sa_score_batch` | Batch SA calculation |

### PubChem

| Tool | Description |
|------|-------------|
| `search_pubchem` | Search by name/SMILES |
| `get_pubchem_compound_by_cid` | Get compound by CID |

### ChEMBL

| Tool | Description |
|------|-------------|
| `search_chembl_by_name` | Search by name |
| `search_chembl_by_synonym` | Search by synonym |
| `get_chembl_molecule_by_id` | Get by ChEMBL ID |
| `get_SVG_by_chembl_id` | Get structure image |
| `get_chembl_molecule_by_inchi_key` | Get by InChI Key |
| `chembl_similarity_search_by_smiles` | Similarity search |
| `chembl_similarity_search_by_id` | Similarity by ID |
| `get_chembl_descriptors_from_smiles` | Calculate descriptors |
| `get_structural_alerts_from_smiles` | Check PAINS alerts |

## Examples

### Calculate Properties

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=calculate_molecular_properties" \
  -F 'params={"smiles": "CC(=O)Oc1ccccc1C(=O)O"}'
```

### Validate SMILES

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=validate_and_canonicalize_smiles" \
  -F 'params={"smiles": "c1ccccc1"}'
```

### Similarity Search

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=chembl_similarity_search_by_smiles" \
  -F 'params={"smiles": "CC(=O)Oc1ccccc1C(=O)O", "similarity": 70}'
```

### SA Score

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=calculate_sa_score" \
  -F 'params={"smiles": "CC(=O)Oc1ccccc1C(=O)O"}'
```

### PAINS Alerts

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_structural_alerts_from_smiles" \
  -F 'params={"smiles": "CC(=O)Oc1ccccc1C(=O)O"}'
```

### Search Databases

```bash
# PubChem
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_pubchem" \
  -F 'params={"query": "aspirin"}'

# ChEMBL
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_chembl_by_name" \
  -F 'params={"compound_name": "aspirin"}'
```

## Drug-likeness (Lipinski)

| Property | Range |
|----------|-------|
| MW | < 500 Da |
| LogP | -0.4 to 5.6 |
| HBD | ≤ 5 |
| HBA | ≤ 10 |
| TPSA | < 140 Å² |
| RotBonds | < 10 |
| SA Score | 1 (easy) - 10 (hard) |

## SMILES Tips

- Atoms: C, N, O, S, P, F, Cl, Br, I (lowercase = aromatic)
- Bonds: `-` single, `=` double, `#` triple
- Rings: Numbers (c1ccccc1 = benzene)
- Branches: Parentheses (CC(C)C = isobutane)
