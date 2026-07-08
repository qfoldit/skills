# qFoldIT qFold Skill — 2D HP-Lattice Folding (Amazon Braket)

Claude Desktop Skill wrapping a hybrid quantum-classical protein folder
on the 2D HP (Hydrophobic/Polar) lattice model, sampled via a QAOA-style
circuit on Amazon Braket's free `LocalSimulator`.

## What's inside

```
qfoldit-qfold/
├── SKILL.md                          — triggers, instructions, naming note, scope limits
├── README.md                         — this file
├── scripts/
│   └── hp_lattice_folder.py          — decode/scoring logic + circuit + brute-force validator
└── references/
    └── model_documentation.md        — brute-force validation + full comparison vs. real QFold
```

## Please read the naming note before anything else

This skill is named after -- but does **not** implement -- the
published QFold algorithm (Casares, Campos & Martin-Delgado,
arXiv:2101.10279; github.com/roberCO/QFold; also shipped as an AWS
Braket notebook in `qfoldit/AWS-Deploy` under
`healthcare-and-life-sciences/c-1-protein-folding-quantum-random-walk`).
That algorithm uses torsion angles, a Minifold deep-learning
initializer, real Psi4 quantum-chemistry energies, and a genuine
Szegedy quantum-walk Metropolis sampler, and was validated on real IBMQ
hardware. What's actually here is a much smaller toy: the well-known 2D
HP-lattice model (Dill, 1985) sampled with a fixed-depth QAOA-style
circuit, with self-avoidance enforced entirely by classical
post-filtering rather than encoded in the circuit. See
`SKILL.md` and `references/model_documentation.md` for the full
side-by-side comparison. If you want the real published algorithm, go
to the roberCO repo or the AWS notebook directly -- this skill is not a
substitute for it, and shouldn't be described as one.

## Scientific basis

**Model:** HP lattice folding (Dill, 1985) -- proteins reduced to an
H(ydrophobic)/P(olar) string, folded as a self-avoiding walk on a 2D
square lattice, scored by non-sequential H-H contact count. A standard,
long-established teaching/prototyping simplification in the
lattice-protein literature, independent of and decades older than the
QFold paper.

**Quantum component:** a QAOA-style ansatz (Hadamard layer, then
alternating CNOT-entangling ladders and RY-rotation layers) run on
Braket's `LocalSimulator`, producing a distribution over
direction-bitstrings that's biased away from uniform random. This is a
single fixed-depth sampling pass, not an iterative quantum-walk process.

**Classical component:** every sampled bitstring is decoded into
lattice coordinates; self-intersecting or immediately-backtracking
walks are discarded (self-avoidance is a purely classical filter, not a
circuit constraint); valid walks are scored by H-H contact count; the
best is returned.

## Validation — read this before trusting outputs

- **Decode/scoring logic** (`decode_walk`, `hp_energy`): validated by
  exhaustive brute-force enumeration (`brute_force_best_fold`) against
  several HP sequences up to 13 residues -- every case produces a
  well-defined, internally-consistent best fold. See
  `references/model_documentation.md` for the full table.
- **External context**: the standard 20-mer HP benchmark
  (`HPHPPHHPHPPHPHHPPHPH`) has a published optimal energy of -9
  (9 contacts) -- cited as scale context only; it exceeds this tool's
  ~15-residue limit and was not run as a match check.
- **Not validated here**: the quantum-sampling path itself, because
  this skill was authored in an environment without network access to
  install `amazon-braket-sdk`. Anyone running this in the real
  container (where the SDK is installed) should confirm
  `fold_hp_sequence` converges toward the same optimum
  `brute_force_best_fold` finds, as a recommended follow-up check.

## Scope limits — important, not optional reading

- **Not the published QFold algorithm** (see naming note above).
- **2D lattice only, not 3D** -- not an all-atom structure. Use
  `rosettafold3` for real 3D structure prediction.
- **~15 residues max** -- qubit count grows linearly, but useful walk
  space grows exponentially; longer sequences need a proper constrained
  QUBO encoding, not implemented here.
- **Self-avoidance is a classical post-filter**, not encoded in the
  circuit -- a materially weaker use of "quantum" than a true
  constrained-sampling or quantum-walk approach.
- **HP-model "energy" is a unitless contact count**, not a physical
  energy comparable to Psi4/force-field results.
- **Requires `amazon-braket-sdk`.** `LocalSimulator` is free and needs
  no AWS account; a real QPU device ARN would be billed by AWS (not
  used here).

## Using this Skill in Claude Desktop

1. Copy the `qfoldit-qfold/` folder into your Claude Desktop skills
   directory (or install via the `.skill` package if provided).
2. Ask Claude things like: *"Fold the sequence HPHPPHHPHPPH on the HP
   lattice"* or give a real amino-acid sequence for `classify_hp()` to
   convert first.
3. Claude will run the fold, report H-H contacts and lattice
   coordinates, and -- per the skill instructions -- will say plainly if
   you ask about torsion angles, Minifold, Psi4, or the real QFold
   quantum-walk algorithm, rather than answering as if this smaller
   HP-lattice code covers that.
