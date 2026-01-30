# Pathway Analysis Tools

Pathways and interactions: KEGG, Reactome, STRING.

## Tools

### KEGG

| Tool | Description |
|------|-------------|
| `kegg_get_info` | Get database info |
| `kegg_list_entries` | List database entries |
| `kegg_search` | Search by keyword |
| `kegg_get_entry` | Get entry details |
| `kegg_convert_ids` | Convert between IDs |
| `kegg_link_entries` | Find linked entries |
| `kegg_drug_interactions` | Drug-drug interactions |

### Reactome

| Tool | Description |
|------|-------------|
| `query_reactome_pathway` | Get pathway details |
| `search_reactome_pathways` | Search pathways |
| `get_pathway_entities` | Get pathway entities |
| `get_contained_events` | Get sub-pathways |
| `analyze_gene_list` | Enrichment analysis |
| `analyze_expression_data` | Expression analysis |
| `get_analysis_results` | Get results |
| `get_pathway_browser_url` | Get visualization URL |
| `download_reactome_results` | Download results |

### STRING

| Tool | Description |
|------|-------------|
| `map_string_ids` | Map names to STRING IDs |
| `get_string_network` | Get interaction network |
| `get_interaction_partners` | Get partners |
| `analyze_string_enrichment` | GO/KEGG enrichment |
| `check_ppi_enrichment` | Check network enrichment |
| `get_string_network_image` | Get network image |
| `get_string_homology` | Get homologs |
| `get_string_version` | Get database version |

## Examples

### KEGG Search

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=kegg_search" \
  -F 'params={"database": "pathway", "query": "cancer"}'
```

### KEGG Pathway Details

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=kegg_get_entry" \
  -F 'params={"entry_id": "hsa05200"}'
```

### Reactome Enrichment

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=analyze_gene_list" \
  -F 'params={
    "genes": ["TP53", "BRCA1", "BRCA2", "ATM", "CHEK2"],
    "species": "Homo sapiens"
  }'
```

### STRING Network

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_string_network" \
  -F 'params={
    "identifiers": ["TP53", "BRCA1", "MDM2", "ATM"],
    "species": 9606,
    "required_score": 400
  }'
```

### STRING Enrichment

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=analyze_string_enrichment" \
  -F 'params={
    "identifiers": ["TP53", "BRCA1", "BRCA2", "ATM", "CHEK2"],
    "species": 9606
  }'
```

### Network Image

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_string_network_image" \
  -F 'params={"identifiers": ["TP53", "BRCA1", "MDM2"], "species": 9606}'
```

## KEGG Codes

| Code | Database |
|------|----------|
| `pathway` | Pathways |
| `ko` | Orthology |
| `compound` | Compounds |
| `drug` | Drugs |
| `disease` | Diseases |

| Organism | Code |
|----------|------|
| Human | hsa |
| Mouse | mmu |
| E. coli | eco |

## STRING Species

| Taxon ID | Species |
|----------|---------|
| 9606 | Human |
| 10090 | Mouse |
| 10116 | Rat |

## STRING Scores

| Score | Confidence |
|-------|------------|
| 900+ | Highest |
| 700-900 | High |
| 400-700 | Medium |
| 150-400 | Low |

## Tips

- KEGG: Academic use only
- Reactome: Works with gene symbols, UniProt, Ensembl IDs
- STRING: Use NCBI taxonomy IDs
- Provide 5-10+ genes for meaningful enrichment
