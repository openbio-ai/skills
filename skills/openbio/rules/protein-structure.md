# Protein Structure Tools

Query protein structure databases: PDB, PDBe, AlphaFold, UniProt.

## Tools

### PDB Tools (RCSB)

| Tool | Description |
|------|-------------|
| `fetch_pdb_metadata` | Get metadata for a PDB entry |
| `get_pdb_overview` | Get structural overview |
| `run_text_query` | Search PDB by text |
| `search_for_attributes` | Search by specific attributes |
| `perform_data_search` | Advanced multi-criteria search |
| `run_seq_similarity_query` | Find similar sequences (BLAST) |
| `run_struct_similarity_query` | Find structurally similar proteins |
| `run_chem_similarity_query` | Find structures with similar ligands |
| `run_seq_motif_query` | Search for sequence motifs |
| `run_attribute_query` | Query specific PDB attributes |
| `extract_pdb_sequences` | Extract sequences from PDB files |
| `get_interactions` | Get protein-ligand interactions |

### PDBe Tools

| Tool | Description |
|------|-------------|
| `get_protein_ligand_annotation` | Get ligand annotations |
| `get_binding_site_residues` | Get binding site info |
| `get_summary_of_ligand` | Get ligand summary |
| `get_proteins_with_ligand` | Find proteins with specific ligand |
| `find_similar_ligands` | Find similar ligands |
| `get_mappings_of_ligand` | Get ligand-to-database mappings |
| `get_protein_database_mappings` | Map PDB to other databases |
| `get_best_structures_for_uniprot_id` | Find best PDB for UniProt ID |

### AlphaFold Tools

| Tool | Description |
|------|-------------|
| `get_alphafold_prediction` | Get predicted structure |
| `get_alphafold_summary` | Get prediction summary |
| `get_alphafold_annotations` | Get confidence annotations (pLDDT, PAE) |

### UniProt Tools

| Tool | Description |
|------|-------------|
| `search_uniprot` | Search UniProt database |
| `get_protein_info_from_uniprot` | Get detailed protein info |

## Examples

### Get Structure by PDB ID

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=fetch_pdb_metadata" \
  -F 'params={"pdb_id": "1MBO"}'
```

### Find Best PDB for UniProt

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_best_structures_for_uniprot_id" \
  -F 'params={"uniprot_id": "P00533"}'
```

### AlphaFold Prediction

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_alphafold_prediction" \
  -F 'params={"uniprot_id": "P00533"}'
```

### Sequence Similarity Search

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=run_seq_similarity_query" \
  -F 'params={"sequence": "MVLSPADKTNVKAAWGKVGAHAGEYGAEALERMFL...", "identity_cutoff": 0.7}'
```

### Text Search

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=run_text_query" \
  -F 'params={"query": "kinase inhibitor", "return_type": "entry", "max_results": 10}'
```

### Binding Site Analysis

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_binding_site_residues" \
  -F 'params={"pdb_id": "3HTB"}'
```

## ID Formats

| Type | Format | Example |
|------|--------|---------|
| PDB ID | 4 characters | 1MBO, 3HTB |
| UniProt | Accession | P00533, Q9Y6K9 |

## Tips

- **Resolution**: Lower is better (< 2.0 Å is high quality)
- **pLDDT scores**: AlphaFold confidence (> 90 = high, < 50 = low)
- Use AlphaFold when no experimental structure exists
