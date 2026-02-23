# Codon Optimization

Optimize DNA coding sequences or protein sequences for heterologous expression in a target host organism.

## When to Use

Use `optimize_codons` when:
1. Expressing a gene from one organism in a different host (e.g., human gene in E. coli)
2. Designing a synthetic gene from a protein sequence for maximum expression
3. Diagnosing poor recombinant protein yield — low CAI often explains the problem
4. Before ordering gene synthesis or before Gibson/Golden Gate assembly
5. Complex proteins needing folding-preserving codon harmonization (multidomain, membrane proteins)

Do NOT use this tool when:
- You only need sequence translation (no optimization needed)
- You're working with natural genomic sequences that won't be expressed heterologously

## Strategy Selection

Two optimization strategies are available:

| Strategy | When to Use | Mechanism | CAI Effect |
|----------|-------------|-----------|------------|
| `mfc` (default) | Short proteins, structural proteins, max yield goal | Picks the most frequent synonymous codon in the target | Maximizes to 1.0 |
| `harmonize` | Complex/multidomain proteins, membrane proteins, prone-to-aggregate proteins, distant-organism expression | Preserves native translation speed rank (Angov 2008) | CAI may decrease — intentional |

**Rule**: Use `harmonize` when expressing a eukaryotic gene (e.g., human) in a prokaryote (E. coli), or when pure MFC optimization leads to aggregation or insoluble protein. Harmonization preserves co-translational folding pauses.

## Decision Tree

```
Codon optimization task?
│
├─ I have a protein sequence → design a synthetic gene
│   └─ optimize_codons(sequence=protein, target_organism=X, sequence_type="protein")
│       → Always uses MFC; returns fully optimized synthetic CDS
│
├─ I have a DNA CDS → optimize for new host (simple protein / max yield)
│   └─ optimize_codons(sequence=CDS, target_organism=B, sequence_type="dna", strategy="mfc")
│       → Reports: codons_changed, cai_original → cai_optimized (should reach 1.0)
│
├─ I have a DNA CDS → complex/multidomain protein or prone to aggregate
│   └─ optimize_codons(sequence=CDS, target_organism=B, sequence_type="dna",
│                       strategy="harmonize", source_organism=A)
│       → Preserves translation speed profile; CAI may be lower than MFC
│
└─ I want to check if an existing CDS is well-adapted
    → optimize_codons(sequence=CDS, target_organism=X, sequence_type="dna")
    → Compare cai_original vs. cai_optimized
    → If cai_original > 0.8, sequence is already well-adapted
```

## Tools Reference

### optimize_codons

**When to use**: Any time you need to adapt codon usage for a new expression host.

```bash
# MFC: optimize a human gene for E. coli expression
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=optimize_codons" \
  -F 'params={
    "sequence": "ATGCAGCTGGGCCTGGTGGTGCTGCTGGCACTGGCCCAGCCGGCCGGCACC",
    "target_organism": "ecoli",
    "sequence_type": "dna",
    "strategy": "mfc"
  }'
```

```bash
# Harmonize: preserve folding pauses when moving human gene to E. coli
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=optimize_codons" \
  -F 'params={
    "sequence": "ATGCAGCTGGGCCTGGTGGTGCTGCTGGCACTGGCCCAGCCGGCCGGCACC",
    "target_organism": "ecoli",
    "sequence_type": "dna",
    "strategy": "harmonize",
    "source_organism": "human"
  }'
```

```bash
# Design synthetic gene from protein sequence for human expression
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=optimize_codons" \
  -F 'params={
    "sequence": "MKVLSGLRFLLVLFSFSRGVFRRDAHKSEVAHRFKDLGE",
    "target_organism": "human",
    "sequence_type": "protein"
  }'
```

```bash
# Optimize yeast gene for Pichia expression
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=optimize_codons" \
  -F 'params={
    "sequence": "ATGAGATTTTCTTTCATTTTCTTTTTTTTGATTTCGTTT",
    "target_organism": "pichia",
    "sequence_type": "dna"
  }'
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sequence | string | Yes | DNA CDS (nt, multiple of 3) or protein (1-letter codes) |
| target_organism | enum | Yes | ecoli, human, yeast, cho, mouse, pichia |
| sequence_type | enum | No (default: dna) | "dna" or "protein" |
| strategy | enum | No (default: mfc) | "mfc" or "harmonize" |
| source_organism | enum | Cond. required | Required when strategy="harmonize" and sequence_type="dna" |

### Returns

| Field | Description |
|-------|-------------|
| optimized_sequence | Codon-optimized DNA CDS including stop codon |
| original_sequence | Input as provided |
| sequence_type | "dna" or "protein" |
| strategy | Optimization strategy used ("mfc" or "harmonize") |
| target_organism | Display name of target host |
| protein_sequence | Amino acid translation |
| cds_length_nt | Length in nucleotides |
| protein_length_aa | Number of amino acids (coding only, not stop) |
| gc_content_original | GC% before optimization (DNA input) |
| gc_content_optimized | GC% after all post-processing |
| cai_original | CAI before optimization (DNA input) |
| cai_optimized | CAI after optimization and post-processing |
| cai_grade | Text interpretation: Excellent / Good / Moderate / Poor / Very poor |
| codon_count | Total coding codons (stop not included) |
| codons_changed | Coding codons replaced (DNA input) |
| percent_changed | Percentage of coding codons changed (DNA input) |
| gc_windows | GC% per 50 nt window; each entry has start, end, gc_percent, flag (True if outside 30–70%) |
| auto_fixes | Issues automatically corrected: SD sites resolved, restriction sites eliminated, GC windows smoothed |
| irreducible_issues | SD/restriction sites that **cannot** be eliminated without changing the amino acid sequence — structurally encoded by the protein (e.g., Glu-Lys-Arg pairs that spell GAGG). No action needed; inform synthesis provider if relevant |
| sequence_issues | Synthesis quality issues unrelated to amino acid constraints: AT/GC homopolymers |
| cpg_count | CpG dinucleotide count (mammalian hosts only: human, cho, mouse) |
| warnings | Processing warnings: internal stops, ambiguous bases, stop codon appended, strategy fallbacks |

## Quality Thresholds: CAI Interpretation

| CAI Range | Interpretation | Action |
|-----------|---------------|--------|
| 0.9 – 1.0 | Excellent | Ready for synthesis |
| 0.8 – 0.9 | Good | Usually sufficient |
| 0.6 – 0.8 | Moderate | Optimization recommended |
| 0.4 – 0.6 | Poor | Strong optimization needed |
| < 0.4 | Very poor | Expression may be severely limited |

**Rule**: After MFC optimization, `cai_optimized` should reach 1.0. After harmonization, CAI may be lower — this is expected and intentional. If `cai_original` > 0.8 already, the gene is well-adapted.

## Automatic Issue Resolution (`auto_fixes`)

The tool attempts to automatically correct problems via synonymous codon substitution:

| Issue | Auto-Resolution | Reports in |
|-------|----------------|------------|
| SD sites (E. coli) | Up to 3 passes of synonymous substitution to break the motif | `auto_fixes` if resolved; `irreducible_issues` if all synonyms still encode the motif |
| Restriction sites | Synonymous substitution to eliminate enzyme recognition site | `auto_fixes` if resolved; `irreducible_issues` if unavoidable |
| GC windows outside 30–70% | 2nd/3rd-ranked synonymous codons to shift GC toward target | `auto_fixes` with before/after GC% |
| AT homopolymers ≥10 nt | Not auto-resolved (too disruptive to coding sequence) | `sequence_issues` always |
| GC homopolymers ≥6 nt | Not auto-resolved | `sequence_issues` always |

**Three output fields, three severity levels:**
- `auto_fixes` — handled automatically, no action needed
- `irreducible_issues` — structurally encoded by the protein sequence (e.g., Glu-Lys-Arg runs produce GAGG); inform synthesis provider; cannot be silenced without changing the protein
- `sequence_issues` — synthesis quality risks (homopolymers); may require special synthesis protocol or manual redesign

**Restriction sites scanned**: NdeI, NcoI, EcoRI, BamHI, XhoI, HindIII, SalI, XbaI, NotI, SpeI, KpnI, SacI, AgeI, PstI, SmaI

## GC Windows (gc_windows)

Each entry covers 50 nt: `{"start": 1, "end": 50, "gc_percent": 52.0, "flag": false}`.

- `flag: true` = GC% outside 30–70% — problematic for gene synthesis
- Recommended range for most synthesis platforms: 35–65% GC
- Windows with `flag: true` in E. coli output may indicate homopolymer regions

## Organism Codon Preferences

| Organism | Notable Preferences | Avoid |
|----------|--------------------|----|
| E. coli | CCG (Pro), CTG (Leu), CGC/CGT (Arg), GCG (Ala), AAA (Lys) | AGA/AGG (rare Arg), CGA |
| Human | CTG (Leu), GCC (Ala), CAG (Gln), AAG (Lys), GAG (Glu) | CGA, TCG (rare) |
| Yeast | TTG (Leu), CCA (Pro), AGA (Arg), CAA (Gln), GAA (Glu) | CTG, CGG (very rare) |
| CHO | CTG (Leu), GCC (Ala), CAG (Gln) — similar to human | CGA, TCG |
| Mouse | CTG (Leu), GCC (Ala) — nearly identical to human | CGA, TCG |
| Pichia | TTG (Leu), CCA (Pro), AGA (Arg), AAG (Lys) | CTG, CGG |

## Common Workflows

### Workflow 1: Bacterial expression of simple mammalian protein (MFC)

```
1. Start with human protein sequence or CDS

2. optimize_codons(sequence=..., target_organism="ecoli", strategy="mfc")
   → Check: codons_changed, cai_original → cai_optimized (should reach 1.0)
   → Check warnings for internal stop codons
   → Check sequence_issues for SD sites and restriction sites

3. Review optimized_sequence
   → GC content 50–65% is typical for E. coli
   → CAI should reach 1.0

4. Proceed to gene synthesis or Gibson/Golden Gate assembly
```

### Workflow 2: Bacterial expression of complex multidomain protein (Harmonize)

```
1. Have human CDS (e.g., kinase, GPCR, membrane protein)

2. First try MFC: optimize_codons(sequence=CDS, target_organism="ecoli", strategy="mfc")
   → If protein aggregates or is insoluble in practice:

3. Use harmonize: optimize_codons(sequence=CDS, target_organism="ecoli",
                                    strategy="harmonize", source_organism="human")
   → Preserves co-translational folding pauses
   → cai_optimized will be lower than MFC — this is intentional
   → Check sequence_issues for SD sites in the harmonized output
```

### Workflow 3: Secreted protein expression in Pichia

```
1. Obtain protein sequence (e.g., from UniProt)
   → fetch_uniprot_entry to get canonical sequence

2. optimize_codons(sequence=protein, target_organism="pichia", sequence_type="protein")
   → Generates fully optimized synthetic CDS

3. Add signal peptide codons upstream (α-factor pre-pro or other signal)
   → optimize_codons for signal peptide separately if needed

4. assemble_gibson or assemble_golden_gate to join CDS with vector
```

### Workflow 4: Diagnosing poor expression

```
1. Have existing CDS with poor yield

2. optimize_codons(sequence=CDS, target_organism=host, sequence_type="dna")
   → Check cai_original
   → If cai_original < 0.6: codon usage is a likely problem
   → Review codons_changed and percent_changed
   → Check sequence_issues for synthesis problems

3. If many rare codons:
   → Use optimized_sequence for gene re-synthesis
   → Or introduce targeted synonymous mutations for worst codons
```

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| "length not a multiple of 3" | Partial CDS or wrong frame | Provide complete in-frame CDS from start ATG |
| Internal stop codon warning | Frameshift or non-CDS input | Verify sequence is correct CDS; check reading frame |
| Ambiguous nucleotide (N) warning | Degenerate positions | Replace N with specific base before final synthesis |
| cai_original already 1.0 | Sequence already fully optimized | No action needed |
| protein_sequence contains X | Unknown amino acids in input | Replace X with correct AA before synthesis |
| "source_organism required" | harmonize without source_organism | Add source_organism parameter |
| "harmonize requires sequence_type=dna" | Harmonize with protein input | Use strategy="mfc" for protein input (no source) |
| Items in irreducible_issues | SD/restriction site structurally encoded by amino acid sequence | No fix possible without protein engineering; inform synthesis provider; most vendors handle these cases with sequence flagging |
| SD site in irreducible_issues (E. coli) | e.g., Glu(GAG)-Lys(AAG)-Arg pattern = GAGGAAG contains GAGG | Expected for natural sequences; does not prevent expression in practice, just adds minor risk of rare truncation |
| gc_windows flag=true after smoothing | Homopolymer or extreme composition window | Check sequence_issues for homopolymer runs; these require gene synthesis with special protocols |
| auto_fixes is empty but irreducible_issues has items | Auto-resolution failed for all instances | All synonymous alternatives for overlapping codons encode the same motif — structurally unavoidable |

## Related Tools

- **After optimization**: `assemble_gibson` or `assemble_golden_gate` for cloning the optimized sequence into a vector
- **Before optimization**: `fetch_uniprot_entry` to get protein sequence, `submit_blast` to verify sequence identity
- **Restriction check**: `restriction_find_sites` — after optimization, verify no unwanted restriction sites conflict with cloning
- **Stability check**: After expression, use `submit_thermompnn_prediction` to predict stability-improving mutations
