# Literature Search Tools

Search scientific literature: PubMed, arXiv, bioRxiv, OpenAlex.

## Tools

### PubMed

| Tool | Description |
|------|-------------|
| `search_pubmed` | Search with query, returns abstracts |
| `fetch_abstract` | Get abstract for PubMed ID |
| `fetch_full_text` | Get full text link (PMC) |
| `pubmed_query_helper` | Help construct queries |

### arXiv

| Tool | Description |
|------|-------------|
| `arxiv_search` | Search arXiv preprints |

### bioRxiv

| Tool | Description |
|------|-------------|
| `biorxiv_search_keywords` | Search by keywords |
| `biorxiv_search_author` | Search by author |
| `biorxiv_get_paper_details` | Get paper by DOI |
| `biorxiv_recent_papers` | Get recent preprints |

### OpenAlex (240M+ works)

| Tool | Description |
|------|-------------|
| `search_literature` | Search scholarly works |
| `get_paper` | Get paper by DOI/ID |
| `search_authors` | Search authors |
| `get_author_works` | Get author's publications |
| `search_institutions` | Search institutions |

## Examples

### Search PubMed

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_pubmed" \
  -F 'params={"query": "CRISPR gene editing", "max_results": 10}'
```

### Fetch Abstract

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=fetch_abstract" \
  -F 'params={"pubmed_id": "36812345"}'
```

### Search bioRxiv

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=biorxiv_search_keywords" \
  -F 'params={"keywords": "protein folding", "max_results": 10}'
```

### Recent bioRxiv Papers

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=biorxiv_recent_papers" \
  -F 'params={"server": "biorxiv", "days": 7}'
```

### Search arXiv

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=arxiv_search" \
  -F 'params={"query": "machine learning protein structure", "max_results": 10}'
```

### OpenAlex Search

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=search_literature" \
  -F 'params={"query": "deep learning drug discovery", "per_page": 25}'
```

### Get Paper by DOI

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_paper" \
  -F 'params={"identifier": "10.1038/s41586-021-03819-2"}'
```

## PubMed Query Tips

- MeSH terms: `"diabetes mellitus"[MeSH]`
- Date filter: `2023[pdat]` or `2020:2024[pdat]`
- Article type: `review[pt]`, `clinical trial[pt]`
- Boolean: `(CRISPR OR "gene editing") AND cancer`
- Author: `Smith J[Author]`
- Journal: `Nature[Journal]`

## Response Data

PubMed results include:
- `pubmed_id`, `title`, `abstract`
- `authors`, `journal`, `publication_date`
- `doi`, `pmc_id`, `keywords`, `mesh_terms`

bioRxiv/arXiv results include:
- `doi`, `title`, `authors`, `abstract`
- `category`, `posted_date`, `pdf_url`
