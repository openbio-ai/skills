# Sequence Analysis Tools

Find and characterise Open Reading Frames (ORFs) in DNA sequences.

## When to Use

Use sequence analysis tools when:
1. Identifying all possible protein-coding regions in a DNA sequence
2. Annotating a novel sequence or contig for ORFs before downstream analysis
3. Checking both strands for coding potential
4. Finding nested or alternative start-codon ORFs
5. Using non-standard genetic codes (mitochondrial, bacterial, etc.)

## Decision Tree

```
What sequence analysis task?
│
└─ Find protein-coding regions (ORFs)?
    ├─ Standard genetic code → sequence_find_orfs (table=1, default)
    ├─ Non-standard code    → sequence_find_orfs (table=<NCBI id>)
    ├─ Include nested ORFs  → sequence_find_orfs (find_nested=true)
    └─ Long sequence (FASTA file) → sequence_find_orfs (sequence_file_path=...)
```

## Tool Reference

### sequence_find_orfs — Find Open Reading Frames

Scans all 6 reading frames (3 forward + 3 reverse-complement) and returns every
complete ORF (start codon → first in-frame stop codon) that meets the minimum
length threshold.

```bash
# Inline sequence
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=sequence_find_orfs" \
  -F 'params={
    "sequence": "ATGCCATAA",
    "min_length": 30,
    "find_nested": false,
    "table": 1
  }'

# FASTA file already uploaded to your project
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=sequence_find_orfs" \
  -F 'params={
    "sequence_file_path": "/my-project/genome_contig.fa",
    "min_length": 100,
    "find_nested": false,
    "table": 11
  }'
```

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sequence` | string | — | DNA string (IUPAC; N and – allowed). Use for sequences ≤ 50 bp or quick checks. |
| `sequence_file_path` | string | — | Path to a FASTA file in your project. Recommended for sequences > 50 bp. |
| `min_length` | integer | 30 | Minimum ORF length **in codons** (not nucleotides). Default = 30 codons = 90 nt. |
| `find_nested` | boolean | false | When true, also returns ORFs whose coordinates lie entirely within a longer same-frame ORF. |
| `table` | integer | 1 | NCBI translation table ID. See table below. |

Provide **exactly one** of `sequence` or `sequence_file_path`.

#### Response Fields

| Field | Description |
|-------|-------------|
| `success` | true/false |
| `sequence_length` | Length of the input sequence in nucleotides |
| `orf_count` | Number of ORFs returned |
| `orfs` | List of ORF objects (see below) |
| `analysis_summary` | Human-readable summary string |

Each ORF object:

| Field | Description |
|-------|-------------|
| `start` | 1-based genomic start (always ≤ end, regardless of strand) |
| `end` | 1-based genomic end, inclusive |
| `strand` | 1 = forward, -1 = reverse complement |
| `frame` | +1/+2/+3 (forward) or -1/-2/-3 (reverse) |
| `length` | ORF length in nucleotides, **including** the stop codon |
| `protein_sequence` | Translated amino-acid sequence (stop codon excluded) |

## Key Concepts

### Coordinate Convention

Coordinates are always reported as genomic positions on the forward strand
(low number = 5′ end of forward strand), regardless of which strand the ORF is on.
Use the `strand` field to determine orientation.

```
Forward ORF:   start=10, end=30, strand= 1  →  fwd[10..30]
Reverse ORF:   start=10, end=30, strand=-1  →  RC of fwd[10..30]
```

### min_length is in Codons, not Nucleotides

```
min_length=30  →  ORF must be ≥ 90 nt  (30 codons × 3)
min_length=100 →  ORF must be ≥ 300 nt (100 codons × 3)
```

The length **includes** the stop codon in the nucleotide count, but the stop codon
amino acid is **not** included in `protein_sequence`.

### Nested ORFs

When `find_nested=false` (default), if a shorter ORF (same frame, same strand) is
completely contained within a longer ORF, only the longer one is returned.

```
find_nested=false → return MPMG... (outer ORF wins)
find_nested=true  → return MPMG... AND MG... (both reported)
```

Use `find_nested=true` only when specifically studying internal ribosome entry sites
(IRES) or alternative translation initiation.

### Alternative Start Codons

Table 1 (Standard) recognises TTG and CTG as valid start codons in addition to ATG.
The tool will find and report these, but note that the `protein_sequence` will show
the amino acid encoded by that codon (L or V), not M — this is a known limitation.
For most gene-finding purposes, filter results by `protein_sequence.startswith("M")`
if you only want canonical ATG-initiated ORFs.

## Common NCBI Translation Tables

| ID | Name | Use When |
|----|------|----------|
| 1 | Standard | Default; bacteria, eukaryotic nuclear |
| 2 | Vertebrate Mitochondrial | Animal mtDNA (TGA = Trp, ATA = Met) |
| 4 | Mold/Protozoan/Coelenterate Mito + Mycoplasma | TGA = Trp |
| 6 | Ciliate, Dasycladacean, Hexamita Nuclear | TAA/TAG = Gln |
| 11 | Bacterial, Archaeal + Plant Plastid | Bacteria / phage / plastid |

Always check the NCBI translation table list for edge cases: https://www.ncbi.nlm.nih.gov/Taxonomy/Utils/wprintgc.cgi

## Common Mistakes

### Wrong: Using nucleotide count for min_length
```
❌ min_length=90  →  ORF must be ≥ 270 nt (way too long for most searches)
```
**Why wrong**: `min_length` is in **codons**, not nucleotides.
```
✅ min_length=30  →  ORF must be ≥ 90 nt (default, sensible for most cases)
```

### Wrong: Providing both sequence and sequence_file_path
```
❌ {"sequence": "ATG...", "sequence_file_path": "/file.fa"}
```
**Why wrong**: Providing both is rejected. Use one or the other.
```
✅ {"sequence": "ATG..."}          ← short sequences
✅ {"sequence_file_path": "/..."}  ← FASTA files
```

### Wrong: Expecting protein_sequence to include stop codon
```
❌ Counting * at end of protein_sequence
```
**Why wrong**: The tool calls `translate(to_stop=True)` — the stop codon is dropped.
```
✅ protein_sequence = "MPMG"  (not "MPMG*")
   orf.length = len(orf) in nt including stop codon nucleotides
```

### Wrong: Using table=1 for bacterial sequences expecting ATG-only starts
```
❌ Surprised by TTG/CTG-initiated ORFs in bacterial analysis
```
**Why**: Table 11 (Bacterial) has the same stop codons as Table 1 but is the
conventionally correct choice for bacteria; use it for clarity.
```
✅ table=11 for bacteria/archaea
   table=2  for vertebrate mitochondria
   table=1  for eukaryotic nuclear (default)
```

## Common Workflows

### Workflow 1: Quick ORF scan on a short sequence

```
1. Call sequence_find_orfs with inline sequence
   → min_length=1 for short test sequences, 30+ for real analysis

2. Review orf_count and orfs list
   → Check strand, frame, start/end coordinates
   → protein_sequence gives you the translated product

3. Filter by strand or frame if needed
   → strand=1 for forward, strand=-1 for reverse complement
```

### Workflow 2: Annotate a novel contig (FASTA file)

```
1. Upload FASTA file to your project (filesystem tools)

2. Call sequence_find_orfs with sequence_file_path
   → min_length=100 (300 nt minimum — filters noise)
   → table=11 for bacterial contigs
   → find_nested=false (default)

3. Sort results by length (longest ORFs = most likely real genes)

4. Cross-reference with restriction_find_sites or design_primers
   for the ORFs of interest
```

### Workflow 3: Check both strands for missed ORFs

The tool always scans all 6 frames automatically — no separate reverse-complement
call needed. Filter by `strand` in the response to separate results:

```
strand= 1  →  ORFs on the sense strand you provided
strand=-1  →  ORFs on the antisense / reverse-complement strand
```

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| `orf_count=0` on a sequence with an obvious ATG | `min_length` too high | Lower `min_length` (try 1 for testing) |
| Cryptic error in `errors` field when no source provided | Model validation bug with both fields `None` | Always provide `sequence` or `sequence_file_path` |
| TTG/CTG ORFs with non-M first amino acid | Alt start codon translation limitation | Filter `protein_sequence.startswith("M")` for ATG-only |
| Unexpected ORFs on reverse strand | All 6 frames are always scanned | Filter by `strand=1` if only forward is wanted |
| Many overlapping ORFs | `find_nested=true` set | Switch to `find_nested=false` (default) |

## Related Tools

- **restriction_find_sites** — find restriction enzyme sites within an ORF region
- **design_primers** — design PCR primers to amplify a specific ORF
- **run_pcr** — simulate amplification of an ORF with designed primers
- **parse_plasmid_file** — extract sequence from a GenBank/SnapGene file, then pass to sequence_find_orfs

---

**Tip**: For real gene-finding on genomic sequences, combine `sequence_find_orfs` results
with literature search (`search_pubmed`) for homology context, or use BLAST
(`submit_blast`) to check if the protein sequence matches known proteins.
