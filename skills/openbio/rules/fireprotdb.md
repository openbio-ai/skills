# FireProtDB Protein Thermostability Tools

Query FireProtDB (https://loschmidt.chemi.muni.cz/fireprotdb/) for experimental thermostability data on protein variants. FireProtDB is the primary database for benchmarking protein stability engineering — it contains thousands of point mutations with measured ΔΔG and ΔTm values across well-studied proteins.

## When to Use

Use FireProtDB tools when:
1. Finding experimental ddG or Tm measurements for a protein of interest
2. Looking up which mutations stabilize or destabilize a specific protein
3. Benchmarking computational stability predictors against experimental data
4. Identifying wild-type thermal stability (Tm) for a protein
5. Finding PDB/AlphaFold structures associated with stability experiments
6. Retrieving publication references for thermostability datasets

## Decision Tree

```
Need thermostability data?
│
├─ Don't have a sequence/mutant ID yet?
│   └─ search_fireprotdb (by protein name, organism, UniProt accession)
│       → returns sequence_id, experiment measurements, publications
│
├─ Have a sequence_id and need all variants + full amino acid sequence?
│   └─ get_fireprotdb_sequence
│       → returns full AA sequence, structures, all experiments on that protein
│
└─ Have a mutant_id and need specific ddG/Tm measurements?
    └─ get_fireprotdb_mutant
        → returns substitution notation, ddG/DTM values, source/target seq IDs
```

## Measurement Types

| Type | Description | Unit |
|------|-------------|------|
| **TM** | Melting temperature (wild-type or mutant) | °C |
| **DTM** | ΔTm — change in melting temp vs wild-type | °C |
| **DDG** | ΔΔG — change in free energy of unfolding | kcal/mol |
| **DH** | Enthalpy of unfolding | kcal/mol |
| **REVERSIBILITY** | Whether unfolding is reversible | yes/no |

## Quality Thresholds

| Metric | Significantly Stabilizing | Neutral | Significantly Destabilizing |
|--------|--------------------------|---------|------------------------------|
| DDG (kcal/mol) | < −1.0 | −1.0 to +1.0 | > +1.0 |
| DTM (°C) | < −2.0 | −2.0 to +2.0 | > +2.0 |

**Note**: DDG sign convention in FireProtDB — **negative = stabilizing** (lowers free energy of folded state), positive = destabilizing.

## Tools Reference

### search_fireprotdb

Search FireProtDB by protein name, organism, UniProt accession, or keyword.

```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_fireprotdb" \
  -F 'params={"query": "barnase", "limit": 10}'
```

```bash
# Search by UniProt accession
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_fireprotdb" \
  -F 'params={"query": "P00720", "limit": 20}'
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| query | string | Yes | Protein name, organism, UniProt ID, PDB ID, or keyword |
| limit | int | No | Max results (1–100, default: 20) |
| offset | int | No | Pagination offset (default: 0) |

**Returns** per result:
- `sequence_id` — use with `get_fireprotdb_sequence`
- `proteins` — name, organism, UniProt accession, EC number
- `experiment` — measurements (TM, DH, etc.), dataset, publication (title, DOI, PMID)

---

### get_fireprotdb_sequence

Retrieve full sequence details and all experiments for a FireProtDB sequence ID.

```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_fireprotdb_sequence" \
  -F 'params={"sequence_id": 30}'
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sequence_id | int | Yes | FireProtDB sequence ID (from search_fireprotdb results) |

**Returns:**
- `sequence` — full amino acid sequence string
- `length` — sequence length
- `proteins` — protein name, organism, UniProt, EC number
- `structures` — list of `{pdb_id, alphafold_id, method, resolution_angstrom}`
- `features` — active sites, binding sites, domains (type, start, end)
- `experiments` — all experiments: measurements (TM, DDG, DTM), dataset, publication

---

### get_fireprotdb_mutant

Retrieve detailed experimental data for a specific FireProtDB mutant.

```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_fireprotdb_mutant" \
  -F 'params={"mutant_id": 1}'
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| mutant_id | int | Yes | FireProtDB mutant ID |

**Returns:**
- `substitutions` — list of `{notation: "P28S", position, wild_type, mutant}`
- `source_sequence_id` / `target_sequence_id` — IDs for WT and mutant sequences
- `structures` — PDB/AlphaFold structures for this variant
- `experiments` — measurements (TM, DTM, DDG, DH), dataset, publication

---

## Common Workflows

### Workflow 1: Find all stability data for a protein

```
1. Search by protein name or UniProt ID
   → search_fireprotdb with query="lysozyme" or query="P00720"
   → Note sequence_id values and wild-type Tm from experiments

2. Get full sequence details
   → get_fireprotdb_sequence with the sequence_id
   → Returns all experiments and structures

3. Get specific mutant details
   → get_fireprotdb_mutant for any mutant_id of interest
   → Get precise ddG/DTM values and publication DOI
```

### Workflow 2: Validate a computational stability prediction

```
1. You have a predicted ddG for mutation X in protein Y
2. Search for the protein: search_fireprotdb with protein name
3. Get sequence details: get_fireprotdb_sequence
4. Browse experiments to find if mutation X was tested experimentally
5. Compare predicted vs experimental ddG / DTM
   → |difference| < 1 kcal/mol = good prediction
```

### Workflow 3: Find stabilizing mutations for protein engineering

```
1. Search for the target protein: search_fireprotdb
2. Retrieve sequence experiments: get_fireprotdb_sequence
3. For each experiment, examine measurements:
   → DTM < −2°C (stabilizing) or DDG < −1 kcal/mol
4. Cross-reference with ThermoMPNN predictions:
   → Use run_thermompnn to predict ΔΔG computationally
   → Use FireProtDB data to validate the predictions
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No results for protein name | Name differs in DB | Try UniProt accession or organism name |
| mutant_id not found | ID out of range or deleted | Re-run search_fireprotdb to get current IDs |
| Missing DDG, only TM | Not all experiments measure ddG | Check `measurement_types` — DTM more common than DDG |
| Many duplicate entries | One sequence has many experiments | Each result = one experiment; use offset to paginate |

## Cross-References

After finding stability data in FireProtDB:
- **Predict stability computationally**: Use `run_thermompnn` (ThermoMPNN tool) for ΔΔG predictions
- **Get protein structures**: Use PDB tools with `pdb_id` from `structures` field
- **Find related papers**: Use `search_pubmed` with protein name + "thermostability"
- **Design stabilizing sequences**: Use `run_proteinmpnn` for sequence design

## External Resources

- **FireProtDB Web**: https://loschmidt.chemi.muni.cz/fireprotdb/
- **API Docs**: https://loschmidt.chemi.muni.cz/fireprotdb/api-docs/
- **Related tool**: FireProt (in silico stability design) — https://loschmidt.chemi.muni.cz/fireprot/
