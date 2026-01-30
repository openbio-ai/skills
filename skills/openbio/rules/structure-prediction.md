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
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=get_boltz_tool_info" \
  -F 'params={}'

# Submit
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
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
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=submit_chai_prediction" \
  -F 'params={
    "fasta_string": ">protein\nMVLSPADKTNVK...",
    "ligand_smiles": "CC(=O)Oc1ccccc1C(=O)O"
  }'
```

### ProteinMPNN Design

```bash
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
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
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
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
curl -X POST "https://openbio-api.fly.dev/api/v1/tools" \
  -H "Authorization: Bearer $OPENBIO_API_KEY" \
  -F "tool_name=fetch_job_status" \
  -F 'params={"job_id": "job_abc123"}'
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

## Tips

- **Always** use info tool first to understand parameters
- `temperature`: Lower = more conservative (0.1 typical)
- Use `fixed_positions` to preserve important residues
- Poll with exponential backoff
