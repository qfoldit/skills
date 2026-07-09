---
name: qfoldit-bionemo-agent-toolkit
description: >
  Provides protein structure prediction via both AlphaFold2 (cloud API, NVIDIA BioNeMo NIM)
  and AlphaFold3 (local Docker container using alphafold3_mcp), plus molecular docking
  (DiffDock) and small-molecule generation (MolMIM). Use whenever the user asks to predict
  a 3D protein or complex structure (including ligands, nucleic acids, ions), dock a small
  molecule, or generate new molecules. This skill is a compatible client, not an official
  NVIDIA or Google DeepMind integration.
---

# qFoldIT BioNeMo Agent Toolkit (Hy‑AlphaFold Edition)

A unified client that gives you access to **two complementary structure prediction
backends** — the fast, cloud‑based AlphaFold2 NIM **and** the full‑featured local
AlphaFold3 — plus cloud docking and molecule generation.

## What is implemented

| Component | Function | How it runs |
|---|---|---|
| AlphaFold2 | `predict_protein_structure_af2(sequence)` | Cloud API (BioNeMo NIM) – quick single‑chain prediction |
| AlphaFold3 | `predict_protein_structure_af3(sequence, …)` | Local Docker container (alphafold3_mcp) – full complex prediction |
| DiffDock | `dock_ligand(protein_pdb, ligand)` | Cloud API (BioNeMo NIM) |
| MolMIM | `generate_molecules(seed_smiles)` | Cloud API (BioNeMo NIM) |

You can choose the right tool for the task:
- For fast predictions of single protein chains → **AlphaFold2**.
- For predictions of complexes (multiple chains, DNA/RNA, ligands, ions) → **AlphaFold3**.

Both methods return standard PDB structures and can feed directly into DiffDock for
downstream docking.

## What is NOT implemented

- **OpenFold3**, **ESMFold**, **Boltz‑2**, **RFdiffusion**, **ProteinMPNN**, **Evo 2**,
  and other BioNeMo NIMs are not wrapped — their schemas were not independently verified.
  See `references/api_reference.md` for how to extend the client safely.
- The local AlphaFold3 integration relies on the third‑party
  [alphafold3_mcp](https://github.com/MacromNex/alphafold3_mcp) project; this skill
  does not implement its own AlphaFold3 runner.

## AlphaFold2: quick, cloud‑based single‑chain folding

### Requirements
- An NVIDIA API key (`nvapi-...`). Get a free evaluation key at
  [build.nvidia.com](https://build.nvidia.com). This is an evaluation tier with rate
  limits and finite credits; for production use, consider a paid plan or self‑hosting.

### Usage
```python
predict_protein_structure_af2(sequence: str) -> str  # returns PDB text
```
- Validates the amino‑acid sequence before submission.
- Expects latency on the order of minutes; the skill will warn the user accordingly.
- If the API call fails (missing key, rate limit, server error), the real error
  message is surfaced — no fabricated structures.

## AlphaFold3: full complex prediction on your own GPU

Because AlphaFold3 is **not** available as a cloud service, this skill manages a local
Docker container (`alphafold3_mcp`) that wraps the genuine DeepMind model.

### Prerequisites
1. **Docker** with NVIDIA GPU support (`nvidia-docker` or `--gpus` runtime).
2. **AlphaFold3 model weights** – request a license at
   [https://github.com/google-deepmind/alphafold3](https://github.com/google-deepmind/alphafold3)
   and place the `model/` directory somewhere accessible (e.g., `/data/af3/model`).
3. **AlphaFold3 genetic databases** – also from DeepMind; place them at a known
   location (e.g., `/data/af3/db`).
4. **Sufficient GPU memory** – an NVIDIA A100 (40/80 GB) or equivalent is recommended.
   Large complexes may require substantial VRAM.
5. (Optional) The Docker image will be pulled automatically on first use if not present
   locally (`ghcr.io/macromnex/alphafold3_mcp:latest`).

### Configuration
Set these environment variables (or a `.env` file):
```
AF3_MODEL_DIR=/path/to/alphafold3/model
AF3_DB_DIR=/path/to/alphafold3/databases
# Optional custom image tag
AF3_DOCKER_IMAGE=ghcr.io/macromnex/alphafold3_mcp:latest
```

The skill will mount these directories into the container and verify GPU access before
submitting any job.

### Usage
```python
predict_protein_structure_af3(
    sequence: str = None,
    fasta_path: str = None,
    config_json_path: str = None,
    output_dir: str = "./af3_results",
    wait: bool = True,
    timeout: int = 3600  # seconds
) -> dict  # {job_id, pdb_path, …}
```
- Accepts an amino‑acid string, a FASTA file path, or a full AlphaFold3 input JSON.
- Submits the job to the local container and polls until completion (if `wait=True`).
- Warns the user about potentially long runtimes (hours for large complexes).
- Provides job status updates when running interactively.

### Important notes
- The container is started on first use and kept alive for subsequent calls; it is
  automatically stopped when the skill session ends.
- All computation runs on **your local hardware**; no sequence data leaves your machine.
- The AlphaFold3 model is provided by Google DeepMind under a CC‑BY‑NC‑SA 4.0 license.
  Respect its terms.

## DiffDock & MolMIM (cloud NIMs)

These work exactly as before, requiring only an NVIDIA API key.

- **DiffDock**: dock a ligand (SMILES or SDF) against a protein structure (PDB).
  The protein PDB typically comes from an AlphaFold2/AlphaFold3 prediction.
- **MolMIM**: generate candidate molecules similar to a given seed SMILES.

Authentication and error handling are identical to the AlphaFold2 cloud service.

## How to handle a request

1. **Determine the right structure prediction tool**:
   - If the user asks for a simple protein chain and wants speed → **AlphaFold2**.
   - If they need a complex (multiple entities, ligands, DNA, ions) → **AlphaFold3**.
   - The skill can also fall back to AlphaFold2 if AlphaFold3 prerequisites are not met.
2. **Check prerequisites** for the chosen backend before attempting a call.
   - Missing NVIDIA API key → point the user to build.nvidia.com.
   - Missing AlphaFold3 model weights / databases / Docker GPU → explain exactly what
     is needed and do not fabricate results.
3. **Set expectations**:
   - AlphaFold2: minutes.
   - AlphaFold3: minutes to hours, depending on complex size.
   - Docking/generation: seconds to minutes.
4. **Common workflow**: Fold with AlphaFold2/AlphaFold3 → obtain PDB → dock with
   DiffDock → generate optimized molecules with MolMIM.
5. **Never fake results**: if any API or local call fails, propagate the real error
   message. Be transparent that AlphaFold2 runs on NVIDIA’s cloud and AlphaFold3
   runs locally via the alphafold3_mcp container (not an official Google service).

## Testing status

- **AlphaFold2, DiffDock, MolMIM**: offline‑verified against OpenAPI schemas.
  Run `scripts/smoke_test.py` with an NVIDIA API key for live validation.
- **AlphaFold3 local client**: tested in environments with Docker + GPU;
  `scripts/smoke_test_af3.py` verifies the container lifecycle and a simple
  single‑chain prediction.
- **Pipeline test**: `scripts/smoke_test_pipeline.py` exercises the full
  AF3 → DiffDock flow (requires both local GPU and an API key).

If any prerequisites are missing, the skill will clearly inform the user and
refuse to continue.

## References

- `references/api_reference.md` — cloud NIM endpoint documentation.
- `references/alphafold3_mcp.md` — MCP tools and parameters used by the AF3 client.
- [alphafold3_mcp GitHub](https://github.com/MacromNex/alphafold3_mcp)
