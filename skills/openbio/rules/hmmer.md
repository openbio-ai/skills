# HMMER Domain / Homology Search

Search for remote homologs and identify protein domains using HMMER profile Hidden Markov Models via the EBI HMMER API.

## When to Use

Use HMMER tools when:
1. Identifying protein domains and families in a sequence (hmmscan vs Pfam)
2. Detecting remote homologs that BLAST might miss (phmmer is more sensitive)
3. Classifying a protein into Pfam families and clans
4. Finding distant evolutionary relationships between proteins
5. Annotating domain architecture of an unknown protein

**HMMER vs BLAST**: Use HMMER when you need higher sensitivity for remote homology. BLAST is faster and better for close homologs (>30% identity). HMMER excels at finding distant relatives (<30% identity) and domain classification.

## Decision Tree

```
What do you need to find?
│
├─ Protein domains/families in a sequence?
│   └─ submit_hmmer_search with algorithm="hmmscan", database="pfam"
│       → Identifies Pfam domains, families, clans
│
├─ Remote homologs (more sensitive than BLAST)?
│   ├─ Search reviewed proteins?
│   │   └─ submit_hmmer_search with algorithm="phmmer", database="swissprot"
│   ├─ Search PDB structures?
│   │   └─ submit_hmmer_search with algorithm="phmmer", database="pdb"
│   └─ Broad search?
│       └─ submit_hmmer_search with algorithm="phmmer", database="refprot"
│
└─ Combine: find domains AND homologs?
    └─ Run both: hmmscan for domains, then phmmer for homologs
```

### Database Selection

| Database | Type | Algorithm | Contents | Best For |
|----------|------|-----------|----------|----------|
| pfam | HMM | hmmscan | 24,000+ Pfam families | Domain identification |
| refprot | Sequence | phmmer | UniProt Reference Proteomes | Balanced homology search |
| swissprot | Sequence | phmmer | Reviewed UniProt entries | High-quality annotations |
| pdb | Sequence | phmmer | PDB sequences | Structural homologs |
| uniprot | Sequence | phmmer | Full UniProt | Comprehensive (slower) |
| rp15–rp75 | Sequence | phmmer | Reduced proteome sets | Fast scanning |

## Tools Reference

### submit_hmmer_search — Submit an HMMER search

Submits a protein sequence to EBI HMMER and returns a job UUID for polling.

```bash
# Domain scan against Pfam
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_hmmer_search" \
  -F 'params={"sequence": "MTEYKLVVVGAGGVGKSALTIQLIQNHFVDEYDPTIEDSYRKQVVIDGETCLLDILDTAGQEEYSAMRDQYMRTGEGFLCVFAINNTKSFEDIHHQRQE", "algorithm": "hmmscan"}'
```

```bash
# Homology search against PDB
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_hmmer_search" \
  -F 'params={"sequence": "MVLSPADKTNVKAAWGKVGAHAGEYGAEALERMFLSFPTTKTYFPHFDLSH", "algorithm": "phmmer", "database": "pdb", "evalue": 0.001, "max_hits": 10}'
```

**Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| sequence | string | Yes | — | Protein sequence (raw or FASTA, 10–10,000 chars) |
| algorithm | string | Yes | — | `phmmer` (sequence vs sequence DB) or `hmmscan` (sequence vs Pfam) |
| database | string | No | refprot/pfam | Target database (auto-selected based on algorithm) |
| evalue | float | No | 0.01 | E-value inclusion threshold |
| max_hits | int | No | 20 | Maximum hits to return (1–100) |

**Returns**: `job_id` (UUID), `algorithm`, `database`, `message`

### get_hmmer_results — Retrieve search results

Fetch results once the search completes (typically <30 seconds).

```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_hmmer_results" \
  -F 'params={"job_id": "YOUR_JOB_UUID", "max_hits": 10}'
```

**Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| job_id | string | Yes | — | Job UUID from submit_hmmer_search |
| max_hits | int | No | 20 | Maximum hits to return (1–100) |

**Returns** (phmmer): `status`, `stats`, `hits` array. Each hit includes:
- `accession`, `description`, `species`, `taxonomy_id`
- `evalue`, `score`, `bias`, `num_domains`
- `domain_architecture`, `uniprot_accession`, `external_link`
- `best_domain`: `{score, evalue, query_from, query_to}`

**Returns** (hmmscan): `status`, `stats`, `hits` array. Each hit includes:
- `accession` (Pfam ID), `identifier` (family name), `description`
- `type` (Domain/Family/Repeat), `clan`, `model_length`
- `evalue`, `score`, `bias`, `num_domains`
- `domain_locations`: `[{score, evalue, query_from, query_to, hmm_from, hmm_to}]`
- `interpro_link`, `clan_link`

**Status values**:
- `SUCCESS` — results ready
- `PENDING` — still running, wait 5–10 seconds and retry
- `FAILED` — search failed, resubmit

## Quality Thresholds

### E-value Interpretation

| E-value | Interpretation |
|---------|----------------|
| < 1e-50 | Excellent — near-identical or close homolog |
| 1e-50 to 1e-10 | Strong — likely homologous, conserved function |
| 1e-10 to 1e-5 | Good — probable remote homolog |
| 1e-5 to 0.01 | Marginal — possible remote homology, inspect carefully |
| > 0.01 | Weak — below significance threshold |

### Pfam Domain Significance (hmmscan)

Pfam uses **gathering thresholds** — hits above the threshold are considered true members. The HMMER API returns `is_included=true` for significant hits. All hits returned by get_hmmer_results are above the inclusion threshold.

### Score (Bit Score)

Higher scores indicate better matches. For hmmscan, scores above the Pfam gathering threshold indicate confident domain assignments.

## Common Workflow

```
1. Identify domains in a protein
   → submit_hmmer_search with algorithm="hmmscan"
   → get_hmmer_results → parse Pfam domains

2. Find remote homologs
   → submit_hmmer_search with algorithm="phmmer", database="swissprot"
   → get_hmmer_results → parse homologous sequences

3. Full annotation pipeline
   → hmmscan for domain architecture
   → phmmer for homologs with known function
   → Cross-reference with UniProt (search_uniprot) and PDB (search_pdb)
```

## Common Mistakes

### Wrong: Using phmmer with pfam database
```
❌ submit_hmmer_search with algorithm="phmmer" and database="pfam"
   → phmmer searches sequence databases, not HMM databases
```

```
✅ Use hmmscan for Pfam domain search:
   → algorithm="hmmscan" (automatically uses pfam)
```

### Wrong: Not waiting for completion
```
❌ Calling get_hmmer_results immediately
   → May get status=PENDING
```

```
✅ Check status and retry:
   → If status="PENDING", wait 5-10 seconds
   → Call get_hmmer_results again
```

### Wrong: Using HMMER for close homologs
```
❌ Using phmmer to find sequences >90% identical
   → HMMER is slower than BLAST for obvious matches
```

```
✅ Use BLAST for close homologs (>30% identity)
   Use HMMER for remote homologs (<30% identity) or domain classification
```

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Status stays PENDING | Large database or server load | Wait longer (up to 60s), poll every 5-10 seconds |
| No domains found (hmmscan) | Sequence too short or novel fold | Try full-length protein; check if domains are below threshold |
| No homologs found (phmmer) | Too stringent threshold | Increase evalue (e.g., 1.0); try broader database (uniprot) |
| "Sequence 1 is not valid" | Invalid characters or format issue | Ensure sequence contains only standard amino acid letters |
| Job not found (404) | Job expired or invalid UUID | Jobs expire after some time; resubmit the search |
| Too many hits | Permissive evalue | Lower evalue threshold (e.g., 0.001) or use smaller database |

---

**Tip**: For quick protein domain annotation, use `hmmscan` — it runs against Pfam in under 1 second and returns domain boundaries, family names, and InterPro links. Pair it with `phmmer` against `swissprot` for functional annotation of each domain.
