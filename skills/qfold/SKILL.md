---

name	qfoldit-qfold
description	Hybrid quantum-classical protein folding: 2D HP-lattice model (Dill, 1985) with a QAOA-style circuit on Amazon Braket LocalSimulator, plus optional delegation to a full qFold MCP server (torsion-angle, quantum-walk, Psi4) when available. Use this skill whenever the user asks about protein folding, HP-model folding, lattice protein folding, hydrophobic-polar sequences, or "qfold"/"qFoldIT qfold". Trigger even if the user just gives an amino-acid or H/P sequence and asks how it folds. Do NOT use this for real 3D all-atom structure prediction unless the MCP server is explicitly available and requested.

qfoldit-qfold (2D HP-lattice folding + optional full qFold MCP path)

---

## Naming note — read first

This skill is named after the "QFold" project (P.A.M. Casares, R. Campos, M.A. Martin-Delgado, "QFold: Quantum Walks and Deep Learning to Solve Protein Folding," arXiv:2101.10279, reference implementation github.com/roberCO/QFold, and the AWS Braket notebook under healthcare-and-life-sciences/c-1-protein-folding-quantum-random-walk). That published algorithm uses torsion angles, Minifold deep-learning initialization, Psi4 quantum-chemistry energies, and a Szegedy quantum-walk Metropolis sampler, and was validated on real IBMQ hardware.

This skill provides **two paths**, depending on whether a full qFold MCP server is available:

1. **With MCP server** — delegates to the real QFold algorithm described above, running on AWS Braket (or a simulator), returning actual 3D structures with physical energies.
2. **Without MCP server** — falls back to the much simpler 2D HP-lattice model (Dill, 1985) with a QAOA-style circuit on Braket's free LocalSimulator and classical self-avoidance filtering.

If a user's question assumes this skill can do torsion-angle folding, Minifold/Psi4, or quantum-walk sampling, first check whether the MCP server is available. If it is, use it. If it is not, say so plainly and point them at the real QFold repo/paper, explaining that only the HP-lattice fallback is available here.

---

## MCP Server detection (run this check first)

At the beginning of every request, before deciding how to fold:

1. List all available MCP tools from the connected MCP servers.
2. Look for a tool whose name or description indicates it performs full qFold folding (likely named something like `qfold_server.run_qfold` or `qfold_full.fold_sequence` or similar).
3. If found:
   - Use this tool for **all folding requests**, regardless of sequence length or user wording about "HP-model" or "lattice". The MCP server provides real torsion-angle folding with quantum-walk sampling, Minifold initialization, and Psi4 energies, which is strictly superior.
   - The MCP server can handle sequences longer than ~15 residues and returns 3D all-atom structures — do not apply the HP-lattice limits when using it.
   - Report to the user that the full QFold algorithm (Casares et al.) is being used via the MCP server.
4. If NOT found:
   - Fall back to the 2D HP-lattice path described below.
   - Apply all its limits explicitly (~15 residues, 2D lattice, unitless contact score, classical self-avoidance filter).

---

## HP-lattice fallback (only when MCP server is unavailable)

### What the HP-lattice fallback actually does

Reduce the input to an H(ydrophobic)/P(olar) string (classify_hp if given a real amino-acid sequence).
Encode each of the (n-2) internal bond directions (L/R/U/D on a 2D lattice) with 2 qubits.
Run a QAOA-style circuit (RY rotations + CNOT entangling ladder) on Amazon Braket's free LocalSimulator to get a biased distribution over direction-bitstrings.
Classically decode every sampled bitstring into lattice coordinates, discard self-intersecting walks, score the valid ones by H-H contact count, and return the best.
Read references/model_documentation.md before answering — it has brute-force validation of the classical decode/scoring logic and the full side-by-side comparison against the real QFold algorithm.

### Critical scope limits — read before promising anything

- Not the published QFold algorithm. See the naming note above.
- 2D lattice only, not 3D, and not an all-atom structure. For real 3D structure prediction, point the user at the rosettafold3 tool, or, if the MCP server is available, use it.
- Short sequences only (~15 residues max). 2 qubits per internal bond means qubit count grows linearly, but the useful walk space still grows exponentially. Longer sequences need a proper constrained QUBO encoding, not implemented here.
- The circuit does not encode self-avoidance. Self-avoidance is enforced classically by discarding invalid bitstrings. Do not describe this circuit as encoding the folding constraints itself.
- The HP model's "energy" is a unitless contact count, not a physical energy in any real units, and not comparable to Psi4/force-field energies.
- Requires amazon-braket-sdk. fold_hp_sequence returns an explicit error if it isn't installed; LocalSimulator itself is free and needs no AWS account, but a real QPU device ARN would be billed by AWS (not used here).

---

## How to handle a request (unified procedure)

1. **Check MCP tools first** (as described in "MCP Server detection").
2. **If MCP qFold tool is available:**
   - Convert amino-acid sequences to the format required by the MCP tool (follow its input specification — it may accept a FASTA string, a list of residue codes, or a plain H/P string; infer from the tool's description and parameters).
   - Invoke the MCP tool with the sequence and any user-specified options (e.g., device ARN, simulator choice, number of iterations).
   - Report the result: 3D structure, torsion angles, physical energies (e.g., from Psi4), sampling method (quantum walk), and any initialization details (Minifold). Do NOT mention the HP-lattice fallback unless the user explicitly asks about alternative methods.
   - If the MCP call fails, fall back to the HP-lattice path and inform the user that the full qFold run encountered an error and the simpler model was used instead.
3. **If MCP qFold tool is NOT available (HP-lattice fallback):**
   - If given a real amino-acid sequence, run classify_hp() first to get the H/P string.
   - Run fold_hp_sequence(hp_sequence) from scripts/hp_lattice_folder.py. Report the H-H contact count and lattice coordinates, and be upfront that this is a 2D lattice toy, not a 3D structure.
   - If the user gives a sequence longer than ~15 residues, explain the limit rather than truncating silently.
   - If the user's question implies torsion angles, Minifold, Psi4, or a quantum-walk Metropolis sampler, explain the naming-note gap plainly instead of answering as if this skill covers that.
   - If asked to validate correctness, mention the brute-force cross-check in references/model_documentation.md rather than claiming the quantum-sampling path itself has been independently verified.

---

## Notes on hybrid operation (both paths available)

If both the MCP server and the HP-lattice fallback are available, prefer the MCP server for all requests. The HP-lattice fallback should only be used:

- When the MCP server is unavailable or fails.
- When the user explicitly requests "the HP-model toy" or "the lattice simulation" after being informed about the full qFold option.
- For educational demonstrations comparing the simple lattice model against the real quantum-walk algorithm (in which case you may run both and present a comparison).
