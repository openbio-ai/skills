# Structure Prediction Tools

ML-based prediction: Boltz, Chai, GeoDock, ProteinMPNN, LigandMPNN, ThermoMPNN, Pinal.

## Important

- All tools are **long-running** (`submit_*` prefix)
- Returns `job_id` to track status
- Poll `fetch_job_status` until completion

## Tools

### Structure Prediction

| Tool | Description |
|------|-------------|
| `submit_boltz_prediction` | Boltz structure prediction |
| `get_boltz_tool_info` | Get Boltz parameters |
| `submit_chai_prediction` | Chai-1 prediction |
| `get_chai_tool_info` | Get Chai parameters |

### Docking

| Tool | Description |
|------|-------------|
| `submit_geodock_prediction` | GeoDock protein-ligand docking |
| `get_geodock_tool_info` | Get GeoDock parameters |

### Sequence Design

| Tool | Description |
|------|-------------|
| `submit_proteinmpnn_prediction` | ProteinMPNN design |
| `get_proteinmpnn_tool_info` | Get parameters |
| `submit_ligandmpnn_prediction` | LigandMPNN design |
| `submit_ligandmpnn_scoring` | Score sequences |
| `get_ligandmpnn_tool_info` | Get parameters |
| `submit_thermompnn_prediction` | ThermoMPNN stability design |
| `get_thermompnn_tool_info` | Get parameters |

### Text-to-Protein

| Tool | Description |
|------|-------------|
| `submit_pinal_text_design` | Text-guided design |
| `submit_pinal_structure_design` | Structure-guided design |
| `get_pinal_tool_info` | Get parameters |

### Job Management

| Tool | Description |
|------|-------------|
| `fetch_job_status` | Check job status |
| `fetch_user_jobs` | List your jobs |
| `fetch_chat_jobs` | List chat jobs |

## Examples

### Boltz Prediction

```bash
# Get info first
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_boltz_tool_info" \
  -F 'params={}'

# Submit
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_boltz_prediction" \
  -F 'params={
    "sequences": [{"type": "protein", "sequence": "MVLSPADKTNVK..."}],
    "recycling_steps": 3,
    "sampling_steps": 200
  }'
```

### Chai Prediction

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_chai_prediction" \
  -F 'params={
    "fasta_string": ">protein\nMVLSPADKTNVK...",
    "ligand_smiles": "CC(=O)Oc1ccccc1C(=O)O"
  }'
```

### ProteinMPNN Design

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_proteinmpnn_prediction" \
  -F 'params={
    "pdb_path": "path/to/structure.pdb",
    "num_sequences": 8,
    "temperature": 0.1,
    "fixed_positions": "A:1-10,A:50-60"
  }'
```

### LigandMPNN Design

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_ligandmpnn_prediction" \
  -F 'params={
    "pdb_path": "path/to/complex.pdb",
    "num_sequences": 8,
    "design_chains": ["A"],
    "ligand_chain": "X"
  }'
```

### Check Job Status

```bash
# Quick status check
curl -X GET "https://openbio-api.fly.dev/api/v1/jobs/job_abc123/status" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

Response:
```json
{
  "success": true,
  "job_id": "job_abc123",
  "status": "completed"
}
```

### Download Results After Completion

Once status is `completed`, get full job details with **signed download URLs**:

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/jobs/job_abc123" \
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
    }
  },
  "output_files_signed_urls": {
    "structure": "https://s3.amazonaws.com/...signed-url...",
    "confidence": "https://s3.amazonaws.com/...signed-url..."
  }
}
```

**Download the files** using signed URLs (valid for 1 hour):

```bash
# Download predicted structure
curl -o prediction.pdb "https://s3.amazonaws.com/...signed-url..."

# Download confidence scores  
curl -o confidence.json "https://s3.amazonaws.com/...signed-url..."
```

### List All Jobs

```bash
curl -X GET "https://openbio-api.fly.dev/api/v1/jobs?status=completed&tool=submit_boltz_prediction" \
  -H "X-API-Key: $OPENBIO_API_KEY"
```

## Tool Comparison

| Tool | Input | Output | Use Case |
|------|-------|--------|----------|
| Boltz | Sequences | Structure | General prediction |
| Chai | Seq + ligand | Complex | Protein-ligand |
| GeoDock | Structure + ligand | Poses | Docking |
| ProteinMPNN | Backbone | Sequences | Fixed-backbone design |
| LigandMPNN | Complex | Sequences | Ligand-aware design |
| ThermoMPNN | Structure | Sequences | Thermostability |
| Pinal | Text/structure | Seq + struct | De novo design |

## Job Status Values

| Status | Meaning |
|--------|---------|
| `submitted` | Queued |
| `running` | Processing |
| `completed` | Finished |
| `failed` | Error |
| `cancelled` | Cancelled |

## Complete Workflow Example

```bash
# 1. Get tool info
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=get_boltz_tool_info" -F 'params={}'

# 2. Submit prediction job
RESPONSE=$(curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "X-API-Key: $OPENBIO_API_KEY" \
  -F "tool_name=submit_boltz_prediction" \
  -F 'params={"sequences": [{"type": "protein", "sequence": "MVLSPADKTNVK..."}]}')
JOB_ID=$(echo $RESPONSE | jq -r '.data.job_id')

# 3. Poll for completion (with backoff)
while true; do
  STATUS=$(curl -s "https://openbio-api.fly.dev/api/v1/jobs/$JOB_ID/status" \
    -H "X-API-Key: $OPENBIO_API_KEY" | jq -r '.status')
  
  if [ "$STATUS" = "completed" ]; then break; fi
  if [ "$STATUS" = "failed" ]; then echo "Job failed"; exit 1; fi
  
  sleep 30  # Wait 30 seconds between checks
done

# 4. Get download URLs
RESULT=$(curl -s "https://openbio-api.fly.dev/api/v1/jobs/$JOB_ID" \
  -H "X-API-Key: $OPENBIO_API_KEY")

# 5. Download output files
STRUCTURE_URL=$(echo $RESULT | jq -r '.output_files_signed_urls.structure')
curl -o prediction.pdb "$STRUCTURE_URL"
```

## Tips

- **Always** use info tool first to understand parameters
- `temperature`: Lower = more conservative (0.1 typical)
- Use `fixed_positions` to preserve important residues
- Poll with exponential backoff (start 10s, max 60s)
- Signed URLs expire after 1 hour - regenerate if needed
