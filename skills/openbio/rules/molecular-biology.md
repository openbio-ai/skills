# Molecular Biology Tools

DNA manipulation: PCR, primers, restriction, Gibson/Golden Gate assembly.

## Tools

### Primer Design

| Tool | Description |
|------|-------------|
| `design_primers` | Design PCR primers |
| `evaluate_primers` | Evaluate Tm, GC%, dimers |

### PCR

| Tool | Description |
|------|-------------|
| `run_pcr` | Simulate PCR amplification |

### Restriction Analysis

| Tool | Description |
|------|-------------|
| `restriction_find_sites` | Find restriction sites |
| `restriction_digest` | Simulate digest |
| `restriction_batch_analyze` | Analyze multiple sequences |
| `restriction_enzyme_info` | Get enzyme info |
| `restriction_suggest_cutters` | Suggest enzymes |
| `restriction_compare_enzymes` | Compare cutting patterns |
| `restriction_map_output` | Generate restriction map |

### Ligation & Assembly

| Tool | Description |
|------|-------------|
| `simulate_ligation` | Simulate fragment ligation |
| `assemble_gibson` | Gibson assembly |
| `assemble_golden_gate` | Golden Gate assembly |

### Gel Electrophoresis

| Tool | Description |
|------|-------------|
| `simulate_gel` | Simulate agarose gel |

### Recombination

| Tool | Description |
|------|-------------|
| `check_recombination` | Check for recombination sites |
| `assemble_recombination` | Assemble via recombination |

## Examples

### Design Primers

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=design_primers" \
  -F 'params={
    "sequence": "ATGCGATCGATCGATCG...",
    "target_start": 100,
    "target_end": 500,
    "primer_length_range": [18, 25],
    "tm_range": [55, 65]
  }'
```

### PCR Simulation

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=run_pcr" \
  -F 'params={
    "template": "ATGCGATCG...CGATCGCAT",
    "forward_primer": "ATGCGATCG",
    "reverse_primer": "ATGCGATCG"
  }'
```

### Find Restriction Sites

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=restriction_find_sites" \
  -F 'params={
    "sequence": "ATGAATTCGATCGGATCCGATC",
    "enzymes": ["EcoRI", "BamHI", "HindIII"]
  }'
```

### Restriction Digest

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=restriction_digest" \
  -F 'params={
    "sequence": "ATGAATTCGATCGGATCCGATC",
    "enzymes": ["EcoRI", "BamHI"],
    "circular": false
  }'
```

### Gibson Assembly

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=assemble_gibson" \
  -F 'params={
    "fragments": ["ATGCGATCGATCG...", "CGATCGCATAGCT..."],
    "overlap_length": 20
  }'
```

### Golden Gate Assembly

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=assemble_golden_gate" \
  -F 'params={
    "parts": [
      {"sequence": "...", "entry_overhang": "GGAG", "exit_overhang": "AATG"},
      {"sequence": "...", "entry_overhang": "AATG", "exit_overhang": "GCTT"}
    ],
    "enzyme": "BsaI"
  }'
```

### Gel Simulation

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=simulate_gel" \
  -F 'params={"fragments": [500, 1000, 2000, 5000], "ladder": "1kb", "gel_percent": 1.0}'
```

## Assembly Methods

| Method | Overlap | Scarless | Best For |
|--------|---------|----------|----------|
| Gibson | 15-40 bp homology | Yes | General cloning |
| Golden Gate | 4 bp overhangs | Yes | Combinatorial assembly |
| Ligation | Compatible ends | No | Simple inserts |

## Common Enzymes

| Enzyme | Recognition | Overhang |
|--------|-------------|----------|
| EcoRI | G^AATTC | 5' AATT |
| BamHI | G^GATCC | 5' GATC |
| HindIII | A^AGCTT | 5' AGCT |
| NotI | GC^GGCCGC | 5' GGCC |
| BsaI | GGTCTC(1/5) | Type IIS |
| BbsI | GAAGAC(2/6) | Type IIS |

## Tips

- Gibson: 20-40 bp overlaps for reliable assembly
- Golden Gate: Plan 4 bp overhangs to avoid re-ligation
- Primers: Target Tm 55-65°C, GC 40-60%
