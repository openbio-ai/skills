# Clinical Data Tools

Clinical and drug data: ClinicalTrials.gov, ClinVar, FDA, Open Targets.

## Tools

### ClinicalTrials.gov

| Tool | Description |
|------|-------------|
| `search_clinical_trials` | Search by condition/intervention |
| `get_clinical_trial_details` | Get trial details |

### ClinVar

| Tool | Description |
|------|-------------|
| `search_clinvar` | Search variants |
| `get_variant_summaries` | Get variant summaries |
| `get_variant_details` | Get detailed info |
| `link_clinvar_to_database` | Link to NCBI databases |
| `search_by_position` | Search by genomic position |

### FDA

| Tool | Description |
|------|-------------|
| `search_drug_adverse_events` | Search FAERS |
| `get_drug_label` | Get drug labeling |
| `search_drug_recalls` | Search recalls |
| `search_device_adverse_events` | Device events |
| `search_device_510k` | 510(k) clearances |
| `search_device_recalls` | Device recalls |
| `search_food_recalls` | Food recalls |
| `get_substance_info` | Substance info (UNII, CAS) |

### Open Targets

| Tool | Description |
|------|-------------|
| `search_opentargets` | Search targets/diseases |
| `get_target_info` | Get target details |
| `get_disease_info` | Get disease details |
| `get_target_disease_evidence` | Get association evidence |
| `get_known_drugs_for_disease` | Get drugs for disease |
| `get_drug_info` | Get drug mechanism |
| `get_target_associations` | Get all associations |

## Examples

### Search Clinical Trials

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=search_clinical_trials" \
  -F 'params={
    "condition": "breast cancer",
    "status": ["RECRUITING"],
    "max_results": 20
  }'
```

### Get Trial Details

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=get_clinical_trial_details" \
  -F 'params={"nct_id": "NCT04379596"}'
```

### Search ClinVar

```bash
# By gene
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=search_clinvar" \
  -F 'params={"query": "BRCA1[gene]", "max_results": 20}'

# By position
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=search_by_position" \
  -F 'params={"chromosome": "17", "start": 43044295, "stop": 43170245, "assembly": "GRCh38"}'
```

### FDA Adverse Events

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=search_drug_adverse_events" \
  -F 'params={"drug_name": "ibuprofen", "reaction": "liver injury", "limit": 20}'
```

### Drug Label

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=get_drug_label" \
  -F 'params={"drug_name": "aspirin"}'
```

### Open Targets Search

```bash
# Target
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=search_opentargets" \
  -F 'params={"query": "EGFR", "entity_type": "target"}'

# Disease
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=search_opentargets" \
  -F 'params={"query": "lung cancer", "entity_type": "disease"}'
```

### Target-Disease Evidence

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=get_target_disease_evidence" \
  -F 'params={"target_id": "ENSG00000146648", "disease_id": "EFO_0001071"}'
```

## Clinical Trial Status

| Status | Description |
|--------|-------------|
| `RECRUITING` | Currently recruiting |
| `NOT_YET_RECRUITING` | Not yet open |
| `ACTIVE_NOT_RECRUITING` | Ongoing |
| `COMPLETED` | Completed |
| `TERMINATED` | Ended early |

## Trial Phases

| Phase | Description |
|-------|-------------|
| `PHASE1` | Safety/dosing |
| `PHASE2` | Efficacy |
| `PHASE3` | Large-scale |
| `PHASE4` | Post-marketing |

## ClinVar Significance

| Classification | Description |
|---------------|-------------|
| Pathogenic | Disease-causing |
| Likely pathogenic | Probably disease-causing |
| Uncertain significance | Unknown |
| Likely benign | Probably harmless |
| Benign | Harmless |

## Open Targets IDs

- Targets: Ensembl Gene IDs (ENSG...)
- Diseases: EFO IDs (EFO_...) or MONDO
- Drugs: ChEMBL IDs

## Tips

- ClinicalTrials: Use NCT IDs for specific trials
- ClinVar: Search by gene[gene] or condition
- FDA FAERS: Drug names may vary (brand vs generic)
- Open Targets: Ensembl IDs for targets, EFO for diseases
