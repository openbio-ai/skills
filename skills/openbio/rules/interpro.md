# InterPro Protein Domain Tools

Query protein domains and families from EBI InterPro, Pfam, SMART, PROSITE, CDD, and other member databases via OpenBio API.

## When to Use

Use InterPro tools when:
1. Identifying protein domains and functional regions
2. Classifying proteins into families
3. Finding all proteins containing a specific domain
4. Analyzing protein domain architecture
5. Discovering GO terms and functions associated with domains
6. Mapping between domain databases (Pfam, SMART, PROSITE, etc.)

## Decision Tree

```
Need domain information?
│
├─ Have domain name or keyword?
│   └─ search_interpro_entries (search across all databases)
│
├─ Have domain accession (PF00001, IPR000001)?
│   └─ get_interpro_entry (get full domain details)
│
├─ Have UniProt ID and want domains?
│   └─ get_protein_domains (get domain architecture)
│
└─ Have domain and want all proteins with it?
    └─ search_proteins_by_domain (find domain members)
```

## Supported Databases

| Database | Accession Prefix | Type |
|----------|-----------------|------|
| **InterPro** | IPRxxxxxx | Integrated signatures |
| **Pfam** | PFxxxxx | Protein families |
| **SMART** | SMxxxxx | Domain architecture |
| **PROSITE** | PSxxxxx | Patterns and profiles |
| **CDD** | cdxxxxx | Conserved domains |
| **PANTHER** | PTHRxxxxx | Protein families |
| **TIGRFAM** | TIGRxxxxx | Protein families |

## Quality Thresholds

### Domain Significance

| Metric | Significant | Marginal | Not Significant |
|--------|-------------|----------|-----------------|
| E-value (HMM) | < 0.001 | 0.001-0.01 | > 0.01 |
| Coverage | > 80% | 50-80% | < 50% |

**Rule**: Use domains with E-value < 0.001 for confident annotations. Always check coverage - partial matches may indicate incomplete domains or insertions.

### Domain Architecture Interpretation

| Observation | Interpretation |
|-------------|----------------|
| Multiple related domains | Gene duplication or modular evolution |
| Missing expected domain | Possible annotation error or splice variant |
| Domain atypical position | Unusual architecture, verify with literature |

## Common Mistakes

### Wrong: Searching without specifying database
```
❌ search_interpro_entries with query="kinase" and no database filter
```
**Why wrong**: Returns results from all databases, may be overwhelming.

```
✅ Start with specific database:
   search_interpro_entries with database="pfam", query="kinase"
   Pfam is most comprehensive for families
```

### Wrong: Using partial accession numbers
```
❌ search_interpro_entries with query="PF0001"
```
**Why wrong**: Pfam accessions are PF + 5 digits (e.g., PF00001).

```
✅ Use full accession:
   get_interpro_entry with accession="PF00001"
   or search with query="7tm_1" (short name)
```

### Wrong: Assuming single domain = single function
```
❌ Annotating protein function based on one domain alone
```
**Why wrong**: Domain context matters. A kinase domain in a receptor vs. cytoplasmic protein has different regulation.

```
✅ Consider domain architecture:
   get_protein_domains to see all domains and their arrangement
   Check if domains form known supra-domains
```

## Tools Reference

### search_interpro_entries

Search domain entries across InterPro and member databases.

```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_interpro_entries" \
  -F 'params={"query": "kinase", "database": "pfam", "max_results": 10}'
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| query | string | Yes | Domain name, keyword, or partial accession |
| database | string | No | Database: "interpro", "pfam", "smart", "prosite", "cdd", "panther", "tigrfam", or "all" (default: "all") |
| max_results | int | No | Maximum results (1-50, default: 10) |

**Returns:**
```json
{
  "query": "kinase",
  "database": "pfam",
  "count": 10,
  "results": [
    {
      "accession": "PF00069",
      "name": "Protein kinase domain",
      "short_name": "Pkinase",
      "type": "domain",
      "source_database": "pfam",
      "integrated": "IPR000719",
      "description": "Protein kinase domain..."
    }
  ]
}
```

### get_interpro_entry

Get detailed information for a specific domain entry.

```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_interpro_entry" \
  -F 'params={"accession": "PF00069"}'
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| accession | string | Yes | Domain accession (PF00001, IPR000001, SM00001, etc.) |

**Returns:**
```json
{
  "accession": "PF00069",
  "name": "Protein kinase domain",
  "short_name": "Pkinase",
  "type": "domain",
  "source_database": "pfam",
  "integrated": "IPR000719",
  "description": "...",
  "go_terms": [...],
  "literature": {...},
  "member_databases": [...]
}
```

### get_protein_domains

Get domain architecture for a UniProt protein.

```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_protein_domains" \
  -F 'params={"uniprot_id": "P53_HUMAN", "database_filter": "pfam"}'
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| uniprot_id | string | Yes | UniProt ID or accession (e.g., "P53_HUMAN", "P04637") |
| database_filter | string | No | Filter by database: "pfam", "smart", "prosite", etc. |

**Returns:**
```json
{
  "uniprot_id": "P53_HUMAN",
  "protein_info": {
    "accession": "P04637",
    "name": "P53_HUMAN",
    "organism": "Homo sapiens",
    "length": 393
  },
  "domain_count": 4,
  "domains": [
    {
      "accession": "PF00870",
      "name": "p53 DNA-binding domain",
      "database": "pfam",
      "start": 102,
      "end": 292,
      "evalue": 1e-120,
      "match_status": "full"
    }
  ]
}
```

### search_proteins_by_domain

Find all proteins containing a specific domain.

```bash
curl -X POST "https://api.openbio.tech/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_proteins_by_domain" \
  -F 'params={"domain_accession": "PF00069", "max_results": 10}'
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| domain_accession | string | Yes | Domain accession (PF00001, IPR000001, etc.) |
| max_results | int | No | Maximum results (1-50, default: 10) |

**Returns:**
```json
{
  "domain_accession": "PF00069",
  "protein_count": 10,
  "proteins": [
    {
      "uniprot_id": "EGFR_HUMAN",
      "uniprot_accession": "P00533",
      "protein_name": "Epidermal growth factor receptor",
      "organism": "Homo sapiens",
      "domain_start": 688,
      "domain_end": 966
    }
  ]
}
```

## Common Workflows

### Workflow 1: Annotate a novel protein

```
1. Get UniProt ID for protein
   → Use UniProt search or have user provide it

2. Get domain architecture
   → get_protein_domains with uniprot_id
   
3. For each interesting domain:
   → get_interpro_entry to get full description
   → Note GO terms and literature
   
4. Interpret architecture:
   - List all domains with positions
   - Identify key functional domains
   - Flag any unusual arrangements
```

### Workflow 2: Find proteins with specific domain

```
1. Search for domain by keyword
   → search_interpro_entries with query and database="pfam"

2. Get top hit accession
   → Note the "accession" field from results

3. Get detailed domain info
   → get_interpro_entry to confirm it's the right domain

4. Find all proteins with this domain
   → search_proteins_by_domain with the accession

5. Filter results as needed:
   - By organism
   - By review status (if using UniProt lookup)
```

### Workflow 3: Compare domain architectures

```
1. Get domains for protein A
   → get_protein_domains with uniprot_id_A

2. Get domains for protein B
   → get_protein_domains with uniprot_id_B

3. Compare:
   - Shared domains (homology)
   - Domain order (orthologs vs paralogs)
   - Missing domains (functional divergence)
   - Domain boundaries (alternative splicing)
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "No results found" | Query too specific | Try broader keywords or wildcards |
| Domain not integrated | Not yet curated in InterPro | Check member database directly (e.g., Pfam) |
| Partial domain matches | Domain fragments or atypical architectures | Check coverage % and E-values |
| Unexpected domain count | Isoforms or fragments in UniProt | Verify which isoform ID you're using |
| Slow response | Large protein or many domains | Try with database_filter to narrow results |
| Accession not found | Wrong prefix or outdated accession | Check current accessions at interpro.ac.uk |

## Cross-References

After identifying domains, you may want to:
- **Get structures**: Use PDB tools with domain boundaries
- **Find pathways**: Use KEGG/Reactome with protein IDs
- **Analyze sequences**: Use BLAST with domain sequences
- **Literature search**: Use PubMed with domain names

## External Resources

- **InterPro Web**: https://www.ebi.ac.uk/interpro
- **Pfam**: https://www.ebi.ac.uk/interpro/set/pfam/
- **Help**: Contact support@openbio.ai or visit https://docs.openbio.ai
