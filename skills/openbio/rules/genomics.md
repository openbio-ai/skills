# Genomics Tools

Access genomics data: Ensembl, ENA, NCBI Gene, GWAS Catalog, GEO.

## Tools

### Ensembl (250+ species)

| Tool | Description |
|------|-------------|
| `lookup_gene` | Look up gene by ID/symbol |
| `get_sequence` | Get DNA/protein sequence |
| `vep_predict` | Variant Effect Predictor |
| `find_homologs` | Find orthologs/paralogs |
| `overlap_region` | Get features in region |
| `map_assembly` | Map between assemblies |
| `get_species_info` | Get species/assembly info |
| `get_cross_references` | Get cross-references |

### ENA (European Nucleotide Archive)

| Tool | Description |
|------|-------------|
| `search_ena_portal` | Search sequences/studies |
| `get_ena_file_report` | Get file report (FASTQ URLs) |
| `get_ena_record_metadata` | Get record metadata |
| `get_ena_sequence_fasta` | Get FASTA sequence |
| `get_ena_taxonomy` | Get taxonomic info |
| `get_ena_available_fields` | List search fields |

### NCBI Gene

| Tool | Description |
|------|-------------|
| `search_gene` | Search Gene database |
| `batch_gene_lookup` | Batch gene lookup |

### GWAS Catalog

| Tool | Description |
|------|-------------|
| `search_gwas_associations_by_trait` | Find associations by trait |
| `search_gwas_associations_by_variant` | Get associations for variant |
| `get_gwas_variant_details` | Get variant details |
| `search_gwas_variants_by_gene` | Find variants near gene |
| `search_gwas_variants_by_region` | Find variants in region |
| `get_gwas_study` | Get study details |
| `get_gwas_trait_info` | Get trait info (EFO) |

### GEO (Gene Expression Omnibus)

| Tool | Description |
|------|-------------|
| `search_geo_datasets` | Search expression datasets |
| `fetch_geo_summary` | Get dataset summary |
| `download_geo_series` | Download series matrix |
| `download_geo_file` | Download specific file |

## Examples

### Gene Lookup

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=lookup_gene" \
  -F 'params={"id": "BRCA1", "species": "human"}'
```

### Get Sequence

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_sequence" \
  -F 'params={"id": "ENSG00000139618", "type": "genomic"}'
```

### Variant Annotation (VEP)

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=vep_predict" \
  -F 'params={"variants": ["13:32936732:C:T"], "species": "human"}'
```

### GWAS by Trait

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_gwas_associations_by_trait" \
  -F 'params={"trait": "EFO_0000384"}'
```

### GWAS by Gene

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_gwas_variants_by_gene" \
  -F 'params={"gene": "APOE"}'
```

### Search GEO

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_geo_datasets" \
  -F 'params={"query": "breast cancer RNA-seq", "max_results": 10}'
```

### Search ENA

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_ena_portal" \
  -F 'params={"query": "breast cancer RNA-seq human", "result_type": "study"}'
```

## ID Formats

| Type | Format | Example |
|------|--------|---------|
| Ensembl Gene | ENSG + 11 digits | ENSG00000139618 |
| Ensembl Transcript | ENST + 11 digits | ENST00000357654 |
| NCBI Gene ID | Numeric | 672 |
| rsID (SNP) | rs + number | rs7329174 |
| EFO ID (trait) | EFO_xxxxxxx | EFO_0000384 |
| GEO Series | GSE + number | GSE12345 |
| ENA Accession | Various | SRR12345678 |
