# Sequence Logo Tools

Generate Position Weight Matrix (PWM) and Information Content (IC) data for
visualising conservation patterns across a set of aligned biological sequences.

## When to Use

Use the sequence logo tool when:
1. Visualising conserved positions in a multiple sequence alignment (MSA)
2. Analysing transcription factor binding site (TFBS) motifs
3. Characterising consensus sequences from aligned promoter regions, UTRs, or exons
4. Displaying amino-acid conservation in a protein family alignment
5. Showing RNA secondary-structure consensus from aligned RNA sequences

## Decision Tree

```
What do you need?
│
└─ Visualise conservation across aligned sequences?
    ├─ DNA sequences (A/C/G/T)     → visualization_sequence_logo (alphabet_type="dna")
    ├─ RNA sequences (A/C/G/U)     → visualization_sequence_logo (alphabet_type="rna")
    └─ Protein sequences (20 AA)   → visualization_sequence_logo (alphabet_type="protein")
```

## Tool Reference

### visualization_sequence_logo — Generate Sequence Logo Data

Accepts a list of aligned sequences of equal length and returns per-position
probability distributions and Information Content values suitable for rendering
a sequence logo.

```bash
# DNA sequences
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=visualization_sequence_logo" \
  -F 'params={
    "sequences": ["ATGCTA", "ATGCGA", "ATGCTA", "TTGCTA"],
    "alphabet_type": "dna",
    "pseudocounts": 0.0
  }'

# RNA sequences
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=visualization_sequence_logo" \
  -F 'params={
    "sequences": ["AUGCUA", "AUGCGA", "AUGCUA"],
    "alphabet_type": "rna",
    "pseudocounts": 0.5
  }'

# Protein sequences
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=visualization_sequence_logo" \
  -F 'params={
    "sequences": ["ACDEFG", "ACDEFG", "ACKEFG"],
    "alphabet_type": "protein",
    "pseudocounts": 0.0
  }'
```

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sequences` | list of strings | required | Aligned sequences, **all the same length**. Lowercase is accepted and normalised to uppercase. |
| `alphabet_type` | string | `"dna"` | `"dna"` (A/C/G/T), `"rna"` (A/C/G/U), or `"protein"` (20 standard amino acids). Must match the actual characters in your sequences. |
| `pseudocounts` | float | `0.0` | Added to every letter count before normalising. Use `0.5` or `1.0` to avoid zero-probability letters at sparse positions. |

#### Response Fields

| Field | Description |
|-------|-------------|
| `success` | true/false |
| `motif_length` | Length of the aligned sequences (= number of positions) |
| `sequence_count` | Number of sequences analysed |
| `alphabet` | The alphabet used (`"dna"`, `"rna"`, or `"protein"`) |
| `consensus_sequence` | Highest-frequency character at each position, concatenated |
| `logo_data` | List of `PWMData` objects, one per position (see below) |
| `analysis_summary` | Human-readable summary string |

Each `PWMData` object:

| Field | Description |
|-------|-------------|
| `position` | 0-based position index |
| `frequencies` | Dict mapping each letter to its probability (0–1) at this position. Keys are the alphabet letters only (e.g., A/C/G/T for DNA). Always sums to 1.0. |
| `information_content` | IC in bits at this position. Range: 0 (uniform) to log₂(alphabet size) (perfectly conserved). |

## Key Concepts

### Information Content (IC)

IC measures conservation at a position:

```
IC(pos) = log₂(N) − H(pos)

where:
  N    = alphabet size (4 for DNA/RNA, 20 for protein)
  H    = Shannon entropy = −∑ p(letter) × log₂(p(letter))
```

| Scenario | IC (DNA) | IC (Protein) |
|----------|----------|--------------|
| Perfectly conserved (one letter, p=1.0) | 2.0 bits | 4.32 bits |
| Completely uniform | 0.0 bits | 0.0 bits |
| Partially conserved | 0–2 bits | 0–4.32 bits |

The total IC across all positions is reported as `sum(pd.information_content for pd in logo_data)`.
A high total IC indicates a tightly conserved motif.

### Pseudocounts

Without pseudocounts (`pseudocounts=0.0`), a letter absent from all sequences at
a given position gets probability 0. This is mathematically fine but can produce
misleading logos for small datasets.

```
pseudocounts=0.0   → absent letters: p=0  (strict, ok for large N)
pseudocounts=0.5   → absent letters: p = 0.5/(N_seqs + 0.5×alphabet_size)
pseudocounts=1.0   → heavier smoothing, useful for very few sequences (< 10)
```

Rule of thumb: use `pseudocounts=0.5` when you have fewer than ~20 sequences.

### All Sequences Must Be Pre-Aligned and Equal Length

The tool does **not** perform alignment. Feed it already-aligned sequences
(output from MUSCLE, MAFFT, Clustal, etc.). Sequences of different lengths
are rejected by the validator.

```
✅ ["ATGCTA", "ATGCGA", "TTGCTA"]   ← all 6 chars, aligned
❌ ["ATGCTA", "ATGC"]               ← different lengths → ValidationError
```

## Common Mistakes

### Wrong: Passing unaligned sequences
```
❌ ["ATGGCATACGAT", "GCATA", "ATGCTAGCGAT"]
```
**Why wrong**: Sequences must be the same length. The validator will reject mismatched lengths.
```
✅ Align with an MSA tool first, then pass the aligned sequences.
```

### Wrong: alphabet_type mismatch
```
❌ alphabet_type="dna" with sequences containing U
   → U is not in the DNA alphabet (A/C/G/T); U counts are silently zero
   → ZeroDivisionError in PWM normalisation when pseudocounts=0
```
```
✅ alphabet_type="rna"  for sequences with U
   alphabet_type="dna"  for sequences with T
   alphabet_type="protein" for amino-acid sequences
```

### Wrong: No pseudocounts with very few sequences
```
❌ 2 sequences, pseudocounts=0.0
   → Any letter not seen gets p=0 and is invisible in the logo
   → IC at a 50/50 position = 2 − 1 = 1 bit (correct, but visually misleading at small N)
```
```
✅ pseudocounts=0.5 for small datasets (< 20 sequences)
```

### Wrong: Expecting a rendered image in the response
```
❌ Waiting for a PNG/SVG logo in the response
```
**Why wrong**: The tool returns **data** (PWM + IC), not a rendered image. Use the
`logo_data` to render with a charting library (e.g., Logomaker in Python, or d3-sequence-logo in JS).
```
✅ Use logo_data[pos].frequencies and logo_data[pos].information_content
   to drive your chosen rendering library.
```

## Interpreting Results

### Reading the logo_data

```python
# Example response snippet
{
  "position": 2,
  "frequencies": {"A": 0.75, "C": 0.0, "G": 0.25, "T": 0.0},
  "information_content": 1.186   # bits — position is moderately conserved
}
```

- **High IC (close to max)** → strong conservation; the tallest column in a sequence logo
- **Low IC (close to 0)** → high variability; short column, all letters roughly equal height
- **frequencies** → letter heights within a column are proportional to `p × IC`

### Thresholds

| IC (DNA/RNA bits) | Interpretation |
|-------------------|---------------|
| > 1.8 | Highly conserved (essentially invariant) |
| 1.0–1.8 | Moderately conserved |
| 0.5–1.0 | Weakly conserved |
| < 0.5 | Essentially variable |

| IC (Protein bits) | Interpretation |
|-------------------|---------------|
| > 3.5 | Highly conserved |
| 2.0–3.5 | Moderately conserved |
| < 2.0 | Variable |

## Common Workflows

### Workflow 1: TFBS motif from ChIP-seq peaks

```
1. Collect bound sequences (e.g., from MEME motif analysis or JASPAR)
   → Ensure all sequences are the same length (e.g., ±15 bp around peak summit)

2. Call visualization_sequence_logo
   → alphabet_type="dna"
   → pseudocounts=0.5 (typical for 10–50 sequences)

3. Read logo_data
   → Sort positions by information_content to find the most conserved core
   → consensus_sequence gives the highest-frequency letter at each position

4. Render with Logomaker (Python) or equivalent:
   import logomaker
   df = logomaker.Logo(pd.DataFrame([pd['frequencies'] for pd in logo_data]))
```

### Workflow 2: Protein family conservation

```
1. Fetch aligned sequences from UniProt family / Pfam
   → Use get_uniprot_sequences or BLAST hits (submit_blast → get_blast_results)

2. Strip gap columns ('-') if desired

3. Call visualization_sequence_logo
   → alphabet_type="protein"
   → pseudocounts=1.0 for diverse families

4. Identify positions with IC > 3.0
   → These are structurally / functionally critical residues
   → Cross-reference with structure via fetch_pdb_metadata
```

### Workflow 3: RNA consensus from aligned UTRs

```
1. Align 5' UTR sequences from homologous genes

2. Call visualization_sequence_logo
   → alphabet_type="rna"  ← important: use "rna" not "dna"
   → pseudocounts=0.5

3. High-IC positions indicate conserved RNA secondary structure elements
```

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| `ValidationError`: sequences different lengths | Unaligned input | Align sequences first |
| `ValidationError`: empty sequences list | `sequences=[]` | Provide at least one sequence |
| `success=False`, `ZeroDivisionError` in errors | `alphabet_type` mismatch (e.g., U in DNA mode) with `pseudocounts=0` | Set correct `alphabet_type` or use `pseudocounts > 0` |
| `success=False`, `alphabet_type must be...` | Invalid `alphabet_type` value | Use `"dna"`, `"rna"`, or `"protein"` |
| All IC values = 0 | Completely uniform alignment | Expected — no conservation signal present |
| `frequencies` keys don't include T (DNA) | `alphabet_type="rna"` was used | Change to `alphabet_type="dna"` |

## Related Tools

- **submit_blast / get_blast_results** — find homologous sequences to build an MSA
- **search_pubmed** — find literature on a motif or binding site of interest
- **fetch_pdb_metadata** / **get_alphafold_prediction** — map conserved positions onto a 3D structure

---

**Tip**: The tool returns raw PWM data, not a rendered image. Pair it with a
charting library such as [Logomaker](https://logomaker.readthedocs.io/) (Python)
or [d3-sequence-logo](https://github.com/nicgirault/d3-sequence-logo) (JavaScript)
to produce the visual logo for the user.
