# OpenBio API Reference

Core API endpoints for tool discovery, invocation, and job management.

## Authentication

All requests require `OPENBIO_API_KEY` in the `X-API-Key` header:

```bash
-H "X-API-Key: $OPENBIO_API_KEY"
```

**Base URL**: `https://openbio-api.fly.dev/`

## Endpoints

### List All Tools

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

Response:
```json
{
  "tools": [
    {
      "name": "search_pubmed",
      "simple_name": "Search PubMed",
      "description": "Search PubMed literature database",
      "category": "pubmed",
      "is_long_running": false
    }
  ],
  "total": 60
}
```

### Get Tool Schema

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/tools/{tool_name}" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

Response includes JSON Schema for parameters:
```json
{
  "name": "search_pubmed",
  "description": "...",
  "parameters": {
    "type": "object",
    "properties": {
      "query": {"type": "string"},
      "max_results": {"type": "integer", "default": 10}
    },
    "required": ["query"]
  }
}
```

### Invoke a Tool

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F "tool_name=TOOL_NAME" \
  -F 'params={"param": "value"}'
```

Response:
```json
{
  "success": true,
  "tool_name": "search_pubmed",
  "data": { ... },
  "execution_time_ms": 245
}
```

### Search Tools by Capability

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/tools/search?q=protein+structure" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

### List Categories

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/tools/categories" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

### Get Category Tools

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/tools/categories/{category_name}" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

## File Uploads

Tools accepting files use multipart form data:

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=analyze_pdb_file" \
  -F 'params={}' \
  -F "pdb_file=@/path/to/structure.pdb"
```

Maximum file size: 50MB.

## Long-Running Jobs

Tools prefixed with `submit_` are long-running:

```bash
# Submit job
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_boltz_prediction" \
  -F 'params={"sequence": "MVLSPADKTNVK..."}'
```

Response:
```json
{
  "success": true,
  "data": {
    "job_id": "job_abc123",
    "status": "submitted"
  }
}
```

### Check Job Status (Quick)

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/jobs/{job_id}/status" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

Response:
```json
{
  "success": true,
  "job_id": "job_abc123",
  "status": "completed",
  "error_message": null
}
```

### Get Job Details with Download URLs

When a job completes, retrieve full details including **signed URLs to download output files**:

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/jobs/{job_id}" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

Response:
```json
{
  "success": true,
  "job": {
    "job_id": "job_abc123",
    "status": "completed",
    "tool_name": "submit_boltz_prediction",
    "output_files": {
      "structure": "outputs/boltz/job_abc123/prediction.pdb",
      "confidence": "outputs/boltz/job_abc123/confidence.json"
    },
    "submitted_at": "2024-01-15T10:30:00Z",
    "completed_at": "2024-01-15T10:35:00Z"
  },
  "input_files_signed_urls": {
    "input_fasta": "https://s3.amazonaws.com/...signed-url..."
  },
  "output_files_signed_urls": {
    "structure": "https://s3.amazonaws.com/...signed-url...",
    "confidence": "https://s3.amazonaws.com/...signed-url..."
  }
}
```

**Download files** using the signed URLs (valid for 1 hour):

```bash
# Download structure file
curl -o prediction.pdb "https://s3.amazonaws.com/...signed-url..."

# Download confidence scores
curl -o confidence.json "https://s3.amazonaws.com/...signed-url..."
```

### List Your Jobs

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/jobs" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

Query parameters:
- `limit`: Max results (default: 50, max: 100)
- `offset`: Pagination offset
- `status`: Filter by status (submitted, running, completed, failed)
- `tool`: Filter by tool name

Response:
```json
{
  "success": true,
  "jobs": [...],
  "total_count": 42,
  "pagination": {
    "limit": 50,
    "offset": 0,
    "has_more": false
  }
}
```

## Error Handling

| Status | Meaning | Action |
|--------|---------|--------|
| 401 | Invalid/missing API key | Check `OPENBIO_API_KEY` |
| 404 | Tool not found | Use `/api/v1/tools` to list available |
| 400 | Invalid parameters | Check tool schema |
| 429 | Rate limited | Wait and retry |
| 500 | Server error | Retry or contact support |

## Rate Limits

- Tool listing: 60/minute
- Tool invocation: 30/minute
- Search: 60/minute
