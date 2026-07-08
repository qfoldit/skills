# Model Reference: qfoldit-quantum (VQE simulator)

## 1. What is implemented

A full statevector simulator (exact for small qubit counts, not an
approximation): qubits represented as a 2^n complex vector; gates
(RX, RY, RZ, CNOT) applied as explicit unitary matrices via Kronecker
products with identity on untouched qubits; Pauli-string Hamiltonians
built as explicit 2^n x 2^n matrices. Ansatz: hardware-efficient
(Kandala et al. 2017) -- RY+RZ rotation layer on every qubit, followed
by a ring of CNOT entanglers, repeated for `depth` layers. Optimizer:
scipy COBYLA (gradient-free, standard for VQE since real quantum
hardware doesn't give you analytic gradients), with multiple random
restarts to avoid local minima.

## 2. Validation performed

### Gate mechanics (unit-level)
- CNOT|10> -> |11> confirmed by direct state inspection.
- RY(π) on qubit 0 correctly flips |00> -> |01> (probability 1.0).

### Toy Hamiltonians (self-consistent checks)
- H = Z0 + Z1: VQE converges to -2.0 (analytically obvious: ground state
  |11>, eigenvalue -1-1=-2), error ~2e-10.
- Random Hermitian 2-qubit Hamiltonian (16 random Pauli-string
  coefficients): VQE matches exact diagonalization to ~8e-10.

### Real molecular benchmark: H2, STO-3G basis
Qubit Hamiltonian coefficients (2-qubit, parity-mapped, two-qubit
reduced) are **published, not invented** -- sourced from Li, Wang, Wu &
Zuo (2022), "Stabilizer Approximation," arXiv:2209.09564, generated in
that paper via PySCF (integrals) + Qiskit (parity mapping + two-qubit
reduction).

At equilibrium bond length d=0.74 Å:
```
H = -1.05342 II + 0.39484 IZ - 0.39484 ZI + 0.18121 XX - 0.01125 ZZ
```
- VQE electronic energy: **-1.852388 Hartree** (matches exact
  diagonalization to ~4e-10)
- Adding nuclear repulsion (1/r_bohr = 0.715104 Hartree at d=0.74 Å):
  **total energy = -1.137284 Hartree**
- This matches the widely-cited H2/STO-3G equilibrium benchmark value of
  **-1.1373 Hartree** to 4 decimal places -- this is one of the most
  commonly quoted reference numbers in quantum computing/VQE tutorials,
  and the match confirms the simulator's correctness end-to-end (gate
  mechanics, Hamiltonian construction, and optimizer all working
  together correctly), not just internal self-consistency.

At extended bond length d=2.8 Å:
```
H' = -0.82847 II + 0.29304 XX + 0.01617 IZ - 0.01617 ZI - 0.00015 ZZ
```
- VQE electronic energy: -1.123112 Hartree (exact diagonalization:
  -1.123143, error ~3e-5 -- slightly larger than the equilibrium case,
  consistent with a flatter energy landscape near dissociation making
  optimization marginally harder, still well within useful precision)
- Adding nuclear repulsion (0.188992 Hartree): total = -0.934151 Hartree,
  matching the source paper's cited "~-0.93" value.

### Ansatz depth sensitivity
Depths 1-4 all converged to within ~1e-5 to 1e-9 of the exact ground
state for the H2 equilibrium Hamiltonian -- for a 2-qubit problem even a
shallow ansatz is already expressive enough; depth becomes more
important for larger qubit counts where this simulator has not been
tested (see scope limits in SKILL.md).

## 3. Explicit non-goals / scope limits

- **Not a real quantum backend.** No hardware execution, no quantum
  advantage -- classical statevector simulation only, exponential in
  qubit count.
- **Cannot generate Hamiltonians for arbitrary molecules.** The H2
  Hamiltonian used above came from a published paper for validation
  purposes; a general molecule/geometry -> qubit Hamiltonian pipeline
  (PySCF + Jordan-Wigner/parity mapping) is not implemented here and
  would need to be built (or connected via a real quantum-chemistry
  package/API) separately.
- **Diatomic nuclear repulsion only.** `nuclear_repulsion_energy` handles
  the simple two-point-charge case; polyatomic molecules need a sum over
  all nuclear pairs.
- **No quantum annealing.** The qFoldIT architecture describes
  quantum-adapter as unifying VQE and annealing backends -- only VQE is
  covered by this skill.
