---
name: qfoldit-qfold
description: Hybrid quantum-classical protein folding on the 2D HP (Hydrophobic/Polar) lattice model (Dill, 1985) -- a QAOA-style circuit run on Amazon Braket's free LocalSimulator proposes candidate self-avoiding lattice walks, classical code filters and scores them by H-H contact count. Use this skill whenever the user asks about protein folding, HP-model folding, lattice protein folding, hydrophobic-polar sequences, or "qfold"/"qFoldIT qfold" by name. Trigger even if the user just gives an amino-acid or H/P sequence and asks how it folds, without naming the model explicitly. Do NOT use this for real 3D all-atom structure prediction (use the rosettafold3 skill/tool) or for the published QFold algorithm by Casares/Campos/Martin-Delgado (torsion-angle + quantum-walk Metropolis + Minifold + Psi4) -- that algorithm is NOT implemented here; see the naming-note section below before answering any question that assumes this skill IS that paper's method.
---

# qfoldit-qfold (2D HP-lattice folding)

**Read this section before answering anything that assumes this skill
implements the published "QFold" paper -- it doesn't, and saying so
plainly is more useful than letting the name imply otherwise.**

## Naming note -- read first

This skill is named after the "QFold" project (P.A.M. Casares, R. Campos,
M.A. Martin-Delgado, *"QFold: Quantum Walks and Deep Learning to Solve
Protein Folding,"* arXiv:2101.10279, and the reference implementation at
github.com/roberCO/QFold, also packaged as an AWS Braket notebook under
`healthcare-and-life-sciences/c-1-protein-folding-quantum-random-walk`
in `qfoldit/AWS-Deploy`). That published algorithm is a real, fully
scalable hybrid quantum algorithm that:

- parameterizes the protein by **torsion (dihedral) angles**, explicitly
  avoiding a lattice simplification;
- initializes the search with **Minifold**, a lightweight deep-learning
  structure predictor (a stand-in for AlphaFold);
- computes conformation energies with **Psi4**, real ab-initio quantum
  chemistry;
- samples conformations with a **Szegedy-style quantized quantum walk**
  applied to a Metropolis Markov chain (a genuine quantum-walk sampler,
  not a fixed-depth variational circuit), giving a proven polynomial
  speedup over classical Metropolis for this sampling task;
- was validated on real IBMQ hardware (IBMQ Casablanca) for small
  peptides.

**None of that is implemented in this skill.** What's implemented here
is a much smaller, self-contained toy that only borrows the general
"quantum-biased sampling + classical scoring" spirit and applies it to
the well-known **2D HP-lattice folding model** (Dill, 1985) -- a
standard simplification in the lattice-protein literature that predates
and is independent of the QFold paper. If a user's question assumes
this skill can do torsion-angle folding, use Minifold/Psi4, or run a
quantum-walk Metropolis sampler, say so plainly and point them at the
real QFold repo/paper rather than answering as if this skill's much
smaller HP-lattice code applies.

## What this skill actually does

1. Reduce the input to an H(ydrophobic)/P(olar) string (`classify_hp` if
   given a real amino-acid sequence).
2. Encode each of the (n-2) internal bond directions (L/R/U/D on a 2D
   lattice) with 2 qubits.
3. Run a QAOA-style circuit (RY rotations + CNOT entangling ladder) on
   Amazon Braket's free `LocalSimulator` to get a biased distribution
   over direction-bitstrings.
4. Classically decode every sampled bitstring into lattice coordinates,
   **discard self-intersecting (physically invalid) walks**, score the
   valid ones by H-H contact count, and return the best.

**Read `references/model_documentation.md` before answering** -- it has
the brute-force validation of the classical decode/scoring logic and the
full side-by-side comparison against the real QFold algorithm.

## Critical scope limits -- read before promising anything

1. **Not the published QFold algorithm.** See the naming note above.
2. **2D lattice only, not 3D**, and not an all-atom structure. For real
   3D structure prediction, point the user at the `rosettafold3` tool.
3. **Short sequences only (~15 residues max).** 2 qubits per internal
   bond means qubit count grows linearly, but the useful walk space
   still grows exponentially, and the shot budget needed for good
   coverage grows with it. Longer sequences need a proper constrained
   QUBO encoding, not implemented here.
4. **The circuit does not encode self-avoidance.** It only biases the
   sampled distribution away from uniform random; the self-avoidance
   constraint is enforced entirely classically, after sampling, by
   discarding invalid bitstrings. This is a materially weaker use of
   "quantum" than QFold's quantum-walk Metropolis sampler -- do not
   describe this circuit as encoding the folding constraints itself.
5. **The HP model's "energy" is a unitless contact count**, a teaching
   abstraction of hydrophobic collapse -- not a physical energy in any
   real units, and not comparable to Psi4/force-field energies.
6. **Requires `amazon-braket-sdk`.** `fold_hp_sequence` returns an
   explicit error if it isn't installed; `LocalSimulator` itself is free
   and needs no AWS account, but a real QPU device ARN would be billed
   by AWS (not used here).

## How to handle a request

1. If given a real amino-acid sequence, run `classify_hp()` first to get
   the H/P string.
2. Run `fold_hp_sequence(hp_sequence)` from
   `scripts/hp_lattice_folder.py`. Report the H-H contact count and
   lattice coordinates, and be upfront that this is a 2D lattice toy,
   not a 3D structure.
3. If the user gives a sequence longer than ~15 residues, explain the
   limit rather than truncating silently or fabricating a plausible-
   looking result.
4. If the user's question implies torsion angles, Minifold, Psi4, or a
   quantum-walk Metropolis sampler, explain the naming-note gap plainly
   (point 1 above) instead of answering as if this skill covers that.
5. If asked to validate correctness, mention the brute-force
   cross-check in `references/model_documentation.md` rather than
   claiming the quantum-sampling path itself has been independently
   verified (it requires the Braket runtime, which this skill's
   authoring environment did not have network access to install).
