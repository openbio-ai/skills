---
name: openbio
description: OpenBio API for biological data access - protein structures, genomics, chemistry, literature, pathways, clinical data, and ML-based structure prediction. Use when working with biological databases or computational biology tools.
metadata:
  tags: biology, protein, genomics, chemistry, bioinformatics, drug-discovery
---

## When to use

Use this skill whenever working with:
- Biological databases (PDB, UniProt, ChEMBL, etc.)
- Scientific literature (PubMed, arXiv, bioRxiv)
- Genomics data (Ensembl, GWAS, GEO)
- Molecular biology workflows (PCR, cloning, assembly)
- Structure prediction (Boltz, Chai, ProteinMPNN)
- Pathway analysis (KEGG, Reactome, STRING)
- Clinical/drug data (ClinicalTrials, ClinVar, FDA)

## Authentication

**Required**: `OPENBIO_API_KEY` environment variable.

**Base URL**: `https://openbio-api.fly.dev/`

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F "tool_name=TOOL_NAME" \
  -F 'params={"param": "value"}'
```

## How to use

Read individual rule files for detailed documentation and examples:

### Core API
- [rules/api.md](rules/api.md) - API endpoints, tool invocation, job management

### Databases & Search
- [rules/protein-structure.md](rules/protein-structure.md) - PDB, PDBe, AlphaFold, UniProt for protein structures
- [rules/literature.md](rules/literature.md) - PubMed, arXiv, bioRxiv, OpenAlex for scientific literature
- [rules/genomics.md](rules/genomics.md) - Ensembl, ENA, Gene, GWAS, GEO for genomics data
- [rules/cheminformatics.md](rules/cheminformatics.md) - RDKit, PubChem, ChEMBL for molecular analysis
- [rules/pathway-analysis.md](rules/pathway-analysis.md) - KEGG, Reactome, STRING for pathways and interactions
- [rules/clinical-data.md](rules/clinical-data.md) - ClinicalTrials, ClinVar, FDA, Open Targets

### Computational Tools
- [rules/molecular-biology.md](rules/molecular-biology.md) - PCR, primers, restriction, Gibson/Golden Gate assembly
- [rules/structure-prediction.md](rules/structure-prediction.md) - Boltz, Chai, ProteinMPNN, LigandMPNN, ThermoMPNN

## Quick Reference

### List Tools
```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY"
```

### Search Tools
```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/tools/search?q=protein" \
  -H "Authorization: Bearer $OPENBIO_API_KEY"
```

### Get Tool Schema
```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/tools/{tool_name}" \
  -H "Authorization: Bearer $OPENBIO_API_KEY"
```

### Invoke Tool
```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=search_pubmed" \
  -F 'params={"query": "CRISPR", "max_results": 5}'
```

### Get Job Results with Download URLs
```bash
# For long-running jobs (submit_* tools), get output files:
curl -X GET "https://openbio-api.fly.dev/api/v1/jobs/{job_id}" \
  -H "Authorization: Bearer $OPENBIO_API_KEY"
# Returns signed URLs in output_files_signed_urls (valid 1 hour)
```

## Tool Categories

| Category | Tools | Description |
|----------|-------|-------------|
| `pdb_tools` | 12 | RCSB PDB structure queries |
| `pdbe_tools` | 8 | PDBe annotations and mappings |
| `alphafold` | 3 | AlphaFold predictions |
| `uniprot` | 2 | UniProt protein data |
| `pubmed` | 4 | PubMed literature |
| `arxiv` | 1 | arXiv preprints |
| `literature` | 5 | OpenAlex (240M+ works) |
| `biorxiv` | 4 | bioRxiv preprints |
| `ensembl` | 8 | Ensembl genome database |
| `ena` | 6 | European Nucleotide Archive |
| `gene` | 2 | NCBI Gene |
| `gwas` | 7 | GWAS Catalog |
| `geo` | 4 | Gene Expression Omnibus |
| `chembl` | 9 | ChEMBL bioactivity |
| `pubchem` | 2 | PubChem compounds |
| `rdkit` | 10+ | RDKit cheminformatics |
| `kegg` | 7 | KEGG pathways |
| `reactome` | 9 | Reactome pathways |
| `string` | 8 | STRING interactions |
| `clinicaltrials` | 2 | ClinicalTrials.gov |
| `clinvar` | 5 | ClinVar variants |
| `fda` | 8 | FDA databases |
| `opentargets` | 7 | Open Targets |
| `prediction_tools` | 15+ | ML structure prediction |
| `primer` | 2 | Primer design |
| `pcr` | 1 | PCR simulation |
| `restriction_site` | 7 | Restriction analysis |
| `gibson` | 1 | Gibson assembly |
| `golden_gate` | 1 | Golden Gate assembly |
