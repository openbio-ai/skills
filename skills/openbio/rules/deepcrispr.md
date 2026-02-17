# CRISPR Efficiency Prediction (Azimuth)

## Overview

This tool uses Azimuth (Rule Set 2) for predicting CRISPR-Cas9 sgRNA on-target efficiency and Cas-OFFinder for genome-wide off-target detection. Azimuth is a gradient-boosted regression model trained on large-scale experimental datasets, and is used by the Broad Institute's CRISPick tool.

### When to Use CRISPR Prediction
- Score pre-designed sgRNA sequences for efficiency
- Design new guide RNAs for a target gene or genomic region
- Evaluate off-target specificity of guide candidates
- Rank guide RNAs for CRISPR knockouts or edits

### When NOT to Use CRISPR Prediction
- Need protein structure prediction → Use Boltz/Chai/SimpleFold
- Need molecular biology calculations → Use molecular biology tools
- Need variant annotation → Use VEP or genomics tools

## Decision Tree

```
What CRISPR task do you need?
│
├─ Score existing sgRNA sequences?
│   └─ submit_crispr_prediction
│       → Provide list of sgRNA sequences (20-23nt)
│       → Get on-target + off-target scores
│
└─ Design new guides for target?
    └─ submit_crispr_guide_design
        → Provide gene name or genomic region
        → Get ranked list of candidate guides
```

## Parameters

### submit_crispr_prediction (Score Existing Guides)

| Parameter | Type | Required | Default | Range | Description |
|-----------|------|----------|---------|-------|-------------|
| `sequences` | list[str] | Yes | — | 1-100 items | sgRNA sequences (20-23nt, ACGT only) |
| `pam` | string | No | NGG | — | PAM motif (NGG, NAG, NGA, NNGRRT) |
| `model_type` | string | No | azimuth | — | Prediction model (azimuth, ruleset2) |
| `include_off_target` | boolean | No | True | — | Compute off-target scores |
| `genome` | string | No | hg38 | — | Reference genome (hg38, hg19, mm10) |
| `max_off_targets` | integer | No | 10 | 1-100 | Max off-target sites per guide |
| `mismatch_threshold` | integer | No | 4 | 0-6 | Max mismatches for off-target search |
| `output_dir` | string | No | None | — | Custom output directory |

### submit_crispr_guide_design (Design New Guides)

| Parameter | Type | Required | Default | Range | Description |
|-----------|------|----------|---------|-------|-------------|
| `target` | string | Yes | — | — | Gene symbol, Ensembl ID, or chr:start-end |
| `species` | string | No | human | — | Target species (human, mouse) |
| `pam` | string | No | NGG | — | PAM motif (NGG, NAG, NGA, NNGRRT) |
| `guide_length` | integer | No | 20 | 17-25 | Guide RNA length |
| `model_type` | string | No | azimuth | — | Scoring model (azimuth, ruleset2) |
| `num_guides` | integer | No | 10 | 1-50 | Number of top guides to return |
| `include_off_target` | boolean | No | True | — | Compute off-target scores |
| `genome` | string | No | hg38 | — | Reference genome (hg38, hg19, mm10) |
| `max_off_targets` | integer | No | 10 | 1-100 | Max off-target sites per guide |
| `mismatch_threshold` | integer | No | 4 | 0-6 | Max mismatches |
| `exon_only` | boolean | No | True | — | Restrict guides to exonic regions |
| `output_dir` | string | No | None | — | Custom output directory |

## Quality Thresholds

### On-target Efficiency Score

| Score | Interpretation | Action |
|-------|---------------|--------|
| > 0.7 | High efficiency | Excellent candidate |
| 0.4 - 0.7 | Moderate efficiency | Acceptable, consider alternatives |
| < 0.4 | Low efficiency | Avoid if possible |

### Off-target Specificity Score

| Score | Interpretation | Action |
|-------|---------------|--------|
| > 0.9 | Very specific | Safe to use |
| 0.7 - 0.9 | Specific | Generally safe |
| < 0.7 | Low specificity | High off-target risk, use caution |

### Mismatch Levels

| Mismatches | Risk Level | Recommendation |
|------------|------------|----------------|
| 0 | Very high | Essential site, avoid unless intentional |
| 1-2 | High | Verify off-target is safe |
| 3-4 | Moderate | Acceptable for non-essential targets |
| 5-6 | Low | Generally acceptable |

## Output Format

### Scoring Mode Output Files

**scores.csv**:
- `guide_sequence`: sgRNA sequence (20-23nt)
- `on_target_score`: 0-1, higher = better cutting efficiency
- `off_target_count`: Number of off-target sites found
- `specificity_score`: 0-1, higher = more specific
- `gc_content`: 0-1, GC fraction
- `self_complementarity`: Score for secondary structure risk

**off_targets.csv** (if `include_off_target=True`):
- `guide_sequence`: Query sgRNA
- `off_target_sequence`: Genomic match
- `chromosome`: Chromosome
- `position`: Genomic position
- `strand`: + or -
- `mismatches`: Number of mismatches
- `off_target_score`: 0-1, risk score
- `gene`: Nearby gene (if any)

**summary.json**: Aggregate statistics

### Design Mode Output Files

**designed_guides.csv**:
- `rank`: Rank (1 = best)
- `guide_sequence`: Designed sgRNA
- `target_region`: Genomic coordinates
- `exon`: Exon number (if `exon_only=True`)
- `on_target_score`: Predicted efficiency (0-1)
- `specificity_score`: Off-target specificity (0-1)
- `composite_score`: Weighted combined score (0-1)
- `gc_content`: GC fraction (0-1)
- `off_target_count`: Number of off-target sites

Plus `off_targets.csv` and `summary.json` as above.

## API Usage

### Score Existing Guides
```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_crispr_prediction" \
  -F 'params={
    "sequences": ["GACCGGTTGACGGTTTCGAA", "TTCGATCGTACGTACGTCGA"],
    "pam": "NGG",
    "model_type": "azimuth",
    "include_off_target": true,
    "genome": "hg38"
  }'
```

### Design Guides for Gene
```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_crispr_guide_design" \
  -F 'params={
    "target": "BRCA1",
    "species": "human",
    "pam": "NGG",
    "guide_length": 20,
    "num_guides": 20,
    "exon_only": true
  }'
```

### Get Tool Info
```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_crispr_tool_info" \
  -F 'params={}'
```

## Expected Runtime

| Task | Sequences/Guides | Off-targets | Runtime |
|------|-----------------|-------------|---------|
| Quick scoring | 1-10 | No | 2-5 min |
| Standard scoring | 10-50 | Yes | 5-15 min |
| Large batch | 50-100 | Yes | 15-30 min |
| Guide design | Gene | Yes | 10-25 min |

## Rate Limits

- **Per minute**: 2 jobs maximum
- **Per day**: 10 jobs maximum
- **Max sequences**: 100 per prediction request
- **Max guides**: 50 per design request
- **Timeout**: 30 minutes

## Common Mistakes

### Wrong: Invalid sequence format
```
❌ sequences: ">guide1\nGACCGGTTGACGGTTTCGAA"
```
```
✅ sequences: ["GACCGGTTGACGGTTTCGAA"]
```

### Wrong: Non-standard nucleotides
```
❌ sequences: ["GACCGGTTGACGGTTTCN"]
```
```
✅ sequences: ["GACCGGTTGACGGTTTCG"]
```

### Wrong: Wrong genome for species
```
❌ species: "human", genome: "mm10"
```
```
✅ species: "human", genome: "hg38"
```

### Wrong: Ignoring off-targets
```
❌ include_off_target: false
   → Missing critical specificity information
```
```
✅ include_off_target: true
   → Full safety assessment
```

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Invalid sequence format | FASTA headers or whitespace | Use plain sequences list |
| No guides found | Invalid gene symbol | Check spelling or use Ensembl ID |
| Timeout | Too many sequences | Reduce batch size |
| Low efficiency scores | Difficult target | Try different PAM or region |
| High off-target risk | Common sequence | Increase mismatch_threshold to filter |

## Workflows

### Workflow 1: Score Existing sgRNAs
```
1. Submit sequences
   → submit_crispr_prediction with sgRNA list

2. Poll for completion
   → GET /api/v1/jobs/{job_id}/status

3. Download results
   → GET /api/v1/jobs/{job_id}
   → Review scores.csv for efficiency

4. Filter by quality
   → Keep guides with on_target_score > 0.5
   → Check specificity_score > 0.7
```

### Workflow 2: Design Guides for Gene Knockout
```
1. Submit gene target
   → submit_crispr_guide_design with gene symbol
   → exon_only=True for coding regions

2. Poll for completion
   → GET /api/v1/jobs/{job_id}/status

3. Download results
   → GET /api/v1/jobs/{job_id}
   → Review designed_guides.csv

4. Select top candidates
   → Rank by composite_score
   → Verify exon targeting
   → Check off-target risk
```

### Workflow 3: Full CRISPR Experiment Design
```
1. Design guides
   → submit_crispr_guide_design for target gene

2. Score candidates
   → submit_crispr_prediction for top guides
   → Verify predictions

3. Cross-reference with genomics
   → lookup_gene for target information
   → vep_predict for variant annotation

4. Design primers
   → design_primers for cloning
```

## Best Practices

1. **Check sequence format**: Use plain nucleotide sequences, no headers
2. **Match genome to species**: hg38/hg19 for human, mm10 for mouse
3. **Review off-targets**: Always check specificity scores before experiments
4. **Request multiple guides**: Get 20-30 candidates for options
5. **Prioritize composite scores**: Balance efficiency and specificity
6. **Verify exon targeting**: Use exon_only=True for knockouts
7. **Consider PAM alternatives**: Try NAG/NGA if NGG results are poor

## Sample Output

### Job Submission
```json
{
  "success": true,
  "job_id": "crispr_abc123",
  "message": "CRISPR prediction job submitted successfully. Job ID: crispr_abc123 (Direct Processing)",
  "estimated_runtime": "5-10 minutes",
  "num_sequences": 10
}
```

### Completed Job
```json
{
  "success": true,
  "job": {
    "job_id": "crispr_abc123",
    "status": "completed",
    "created_at": "2025-02-14T10:00:00Z",
    "completed_at": "2025-02-14T10:08:32Z"
  },
  "output_files_signed_urls": {
    "scores.csv": "https://s3.../scores.csv?...",
    "off_targets.csv": "https://s3.../off_targets.csv?...",
    "summary.json": "https://s3.../summary.json?..."
  }
}
```

### What Good Output Looks Like
- **High on_target_score**: > 0.7 for best guides
- **Low off-target_count**: < 5 for specific guides
- **Balanced scores**: Both efficiency and specificity > 0.6
- **CSV file size**: ~1-50 KB depending on input

## Verify Success

```bash
# Check job completed
curl -s "https://api.openbio.tech/api/v1/jobs/{job_id}/status" \
  -H "X-API-Key: $OPENBIO_API_KEY" | jq '.status'

# Verify output files exist
curl -s "https://api.openbio.tech/api/v1/jobs/{job_id}" \
  -H "X-API-Key: $OPENBIO_API_KEY" | jq '.output_files_signed_urls | keys'
```

## Model Comparison

| Feature | Azimuth (Current) | Alternative Models |
|---------|-------------------|--------------------|
| On-target accuracy | High | Varies |
| Off-target prediction | Yes (Cas-OFFinder) | Varies |
| Training data | Large-scale (Doench 2016) | Varies |
| Speed | Fast (no GPU needed) | Varies |
| Best for | Production use | Research/comparison |

**Currently Implemented**: Azimuth for on-target scoring + Cas-OFFinder for off-target detection
**Note**: Model parameter is available for future integration of additional models

## Rate Limits

- **Per minute**: 2 jobs maximum
- **Per day**: 10 jobs maximum
- **Max sequences**: 100 per request
- **Max guides**: 50 per design request
- **Timeout**: 30 minutes

---

**Next**: For primer design → Use `design_primers`. For variant annotation → Use `vep_predict`. For gene info → Use `lookup_gene`.
