# qFoldIT Agent Skills

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-standard-green)](https://agentskills.io)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

**Eight scientific & engineering agent skills for Claude Code (and other Agent Skills platforms).**  
Designed for researchers and engineers working in bio‑mining, corrosion, environmental microbiology, oil recovery, quantum simulation, procedural modelling, plant science, and plastic waste valorisation.

---

## About

qFoldIT is an open research initiative developing computational tools for molecular design, biochemistry, and materials science. This repository hosts our entire collection of agent skills – each one is a self‑contained module that can be run inside Claude Code using the open **Agent Skills** standard (also compatible with OpenAI Codex, Gemini CLI, GitHub Copilot, Cursor, and 20+ other platforms).

All skills are deterministic, transparent, and ship with their own `SKILL.md` manifest, scripts, and reference documentation. They are intended for **exploration, prototyping, and research**, not as turn‑key production tools without local calibration.

---

## Requirements

- **Claude Code** (or any other client that supports Agent Skills)  
- macOS, Linux, or Windows (via WSL / Git Bash)  
- Git  

No API keys or cloud services required – everything runs locally.

---

## Installation

Because this is a **monorepo**, the easiest way to use a skill is to clone the whole repository and install the desired skill from a local path:

```bash
# 1. Clone the repository
git clone https://github.com/qfoldit/skills.git
cd skills

# 2. Inside Claude Code, install the skill by pointing to its folder
/plugin install ./qfoldit-mining
```

Replace `qfoldit-mining` with the folder name of any skill.

**Alternative (if you create separate repositories later):**  
You can add a remote marketplace and install with one line:
```bash
/plugins marketplace add qfoldit https://github.com/qfoldit/<skill-repo>
/plugin install <skill-name>
```

---

## Available Skills

| Skill | Description | Folder |
|-------|-------------|--------|
| **qfoldit-mining** | Bio‑oxidation, biosorption & cyanide biodegradation in gold mining | [qfoldit-mining/](./qfoldit-mining/) |
| **qfoldit-oilgas** | CO₂ corrosion rate prediction for carbon steel pipelines | [qfoldit-oilgas/](./qfoldit-oilgas/) |
| **qfoldit-prospecting** | Microbial community analysis for hidden ore deposit detection | [qfoldit-prospecting/](./qfoldit-prospecting/) |
| **qfoldit-meor** | Microbial enhanced oil recovery (MEOR) from inoculation to incremental oil | [qfoldit-meor/](./qfoldit-meor/) |
| **qfoldit-quantum** | VQE simulator for ground‑state energy of small Hamiltonians (2–4 qubits) | [qfoldit-quantum/](./qfoldit-quantum/) |
| **qfoldit-l-systems** | L‑system fractal & plant generator, renders to SVG | [qfoldit-l-systems/](./qfoldit-l-systems/) |
| **qfoldit-plant-growth-model** | Qualitative plant growth influenced by NPK, light intensity & spectrum | [qfoldit-plant-growth-model/](./qfoldit-plant-growth-model/) |
| **qfoldit-plastic-characterization** | Pyrolysis yield estimator for mixed plastic waste, chlorine risk & fouling score | [qfoldit-plastic-characterization/](./qfoldit-plastic-characterization/) |

---

## Detailed Skill Descriptions

<details>
<summary><strong>qfoldit-mining</strong> – Bioprocesses in Mining</summary>

**Purpose:** Three bio‑technology modules for gold mining:  
- Bio‑oxidation of refractory sulphide ores (pyrite/arsenopyrite)  
- Biosorption of metals from cyanide solutions (Au, Cu, REE, Li)  
- Biological cyanide degradation in tailings  

**Scientific basis:** Shrinking core model + Arrhenius kinetics + cardinal temperature/pH models; Langmuir/Freundlich isotherms + pseudo‑second order kinetics; Aiba substrate‑inhibition kinetics.  

**Key capabilities:** Time‑dependent conversion, equilibrium uptake, cyanide depletion curves; `fit_*` functions for parameter calibration from experimental data.  

**Calibration status:** All parameters are literature‑order‑of‑magnitude defaults. Fitting functions tested on synthetic data (R²>0.998 for isotherms).  

**Limitations:** No raw data handling; no metal competition in biosorption; cyanide model ignores pH/temperature; absolute times highly sensitive to initial biomass.  

**Example:** *“Estimate conversion of gold ore at 35°C, pH 1.8, 20 µm particles, continuous reactor.”*
</details>

<details>
<summary><strong>qfoldit-oilgas</strong> – CO₂ Corrosion in Pipelines</summary>

**Purpose:** Predict internal sweet‑corrosion rate of carbon steel and remaining wall life.  

**Scientific basis:** de Waard–Milliams correlation (1991/1993) with optional FeCO₃ scale protection, flow factor, inhibitor efficiency, and heuristic pitting risk.  

**Key capabilities:** Corrosion rate (mm/year), lifetime estimation, out‑of‑range warnings, calibration offset fitting.  

**Calibration status:** Base correlation is published; scale/flow/pitting factors are simplified. Offset fitting validated on synthetic data.  

**Limitations:** Only CO₂ corrosion (no H₂S, MIC, O₂); pH correction missing; flow factor simplified; pitting risk is qualitative.  

**Example:** *“Estimate corrosion at 60°C, pCO₂=2 bar, flow 2 m/s, salinity 25 ppt, inhibitor 85%.”*
</details>

<details>
<summary><strong>qfoldit-prospecting</strong> – Microbial Ore Prospecting</summary>

**Purpose:** Analyse 16S or metagenomic community tables to find biogeochemical indicators of buried ore.  

**Scientific basis:** Ronholm et al. (2023) on kimberlites. Statistical tools: diversity indices, Mann‑Whitney U, response ratios, anomaly scores, permutation test, hierarchical clustering.  

**Key capabilities:** Indicator taxon detection, significance testing, anomaly scoring for new samples.  

**Calibration status:** Code verified on synthetic data; **not validated on real qFoldIT samples**. Generalisation to other metals is extrapolation.  

**Limitations:** Requires OTU/ASV table (not raw FASTQ); no multiple‑testing correction built‑in; anomaly score simplified.  

**Example:** *“Here is an OTU table of 20 samples (10 over ore, 10 background), find indicator species and test significance.”*
</details>

<details>
<summary><strong>qfoldit-meor</strong> – Microbial Enhanced Oil Recovery</summary>

**Purpose:** Simulate batch MEOR process: bacterial growth → biosurfactant production → IFT reduction → residual oil mobilisation via capillary number.  

**Scientific basis:** Monod growth, Luedeking–Piret product kinetics, Hill‑type IFT curve, capillary number (Chatzis–Morrow) desaturation curve.  

**Key capabilities:** Full pipeline from inoculation to incremental oil prediction; correct baseline (no spurious oil on day 0); IFT Hill parameter fitting.  

**Calibration status:** All kinetic parameters are literature order‑of‑magnitude defaults, not field‑calibrated. IFT fitting works; growth/CDC fitting not yet implemented.  

**Limitations:** Batch model only (no reservoir flooding physics); ignores surfactant adsorption, degradation, heterogeneity; incremental oil as fraction of residual oil after waterflood.  

**Example:** *“Model MEOR for flow velocity 1e‑6 m/s, oil viscosity 10 mPa·s, 90 days.”*
</details>

<details>
<summary><strong>qfoldit-quantum</strong> – VQE Simulator (Classical)</summary>

**Purpose:** Compute ground‑state energy of small molecular/spin Hamiltonians (2–4 qubits) using a classical variational quantum eigensolver.  

**Scientific basis:** VQE algorithm (Peruzzo 2014), hardware‑efficient ansatz (Kandala 2017), COBYLA optimizer with restarts.  

**Key capabilities:** Pauli string Hamiltonian builder, exact diagonalisation for reference, VQE runner; H₂ molecule benchmark (STO‑3G, energy –1.137284 Ha at equilibrium vs. literature –1.1373).  

**Calibration status:** Not applicable (deterministic simulator).  

**Limitations:** Classical simulation (exponential scaling); only pre‑computed H₂ or user‑supplied Pauli dictionaries; no quantum annealing backend.  

**Example:** *“Run VQE on H₂ at equilibrium bond length.”*
</details>

<details>
<summary><strong>qfoldit-l-systems</strong> – L‑System Fractals & Plants</summary>

**Purpose:** Generate 2D L‑systems (fractals, branching structures) and render them to SVG.  

**Scientific basis:** Lindenmayer grammars (1968), turtle graphics.  

**Key capabilities:** Built‑in presets (Koch curve, dragon, Sierpinski, plant, bush); custom axiom/rules JSON; `--string-only` debug mode.  

**Calibration status:** Not required – deterministic mathematics.  

**Limitations:** 2D only; no stochastic/parametric L‑systems; fixed line thickness/colour; exponential string growth above ~7 iterations.  

**Example:** *“Generate a plant L‑system” or “Create an SVG for axiom F, rule F→F+F–F–F+F, angle 90°, 4 iterations.”*
</details>

<details>
<summary><strong>qfoldit-plant-growth-model</strong> – Plant Nutrition & Light Model</summary>

**Purpose:** Qualitative assessment of how NPK and light (intensity, spectrum) affect plant growth, morphology, and secondary metabolites. Can emit parameters for `qfoldit-l-systems` visualisation.  

**Scientific basis:** Liebig’s law, N/P/K deficiency symptoms, daily light integral (DLI), R:FR shade‑avoidance, blue light compactness, UV stress → anthocyanins/flavonoids.  

**Key capabilities:** Growth index (0–100), limiting factor report, morphology parameters, stress metabolite index, `--emit-lsystem-params` flag.  

**Calibration status:** Entirely heuristic, not calibrated on real agronomic data. “Optimal” levels are generic.  

**Limitations:** Ignores micronutrients, pH, temperature, humidity, CO₂; morphology‑to‑L‑system mapping is approximate.  

**Example:** *“Evaluate growth for N=150, P=50, K=60 ppm, PPFD=150, photoperiod 12 h, red 30%, blue 10%, far‑red 25%” – then add `--emit-lsystem-params` to visualise the shape.*
</details>

<details>
<summary><strong>qfoldit-plastic-characterization</strong> – Plastic Pyrolysis Estimator</summary>

**Purpose:** Estimate yields (oil/gas/char) of mixed plastic waste pyrolysis, chlorine contamination risk, and fouling tendency.  

**Scientific basis:** Literature ranges for pure polymers (PE, PP, PS, PET, PVC, others) at 450–550°C.  

**Key capabilities:** Accepts weight‑% composition; returns min–max yield ranges; chlorine in oil (from PVC); heuristic fouling score 0–100; process recommendations.  

**Calibration status:** Uses literature average ranges, not calibrated to a specific reactor. Internal `POLYMER_RANGES` dictionary editable for custom equipment.  

**Limitations:** No synergistic effects; chlorine estimate crude; independent of heating rate, reactor design, catalysts; fouling score heuristic. Always report as ranges, not exact numbers.  

**Example:** *“Estimate pyrolysis of 40% PE, 20% PP, 10% PS, 15% PET, 5% PVC, 10% other.”*
</details>

---

## Calibration & Limitations – Read Before Using

- **All models are provided with default parameters taken from published literature.**  
  They are *order‑of‑magnitude* guides, not tuned to your specific ore, oil field, soil, plant species, or pyrolysis unit.
- **Fitting functions** (where available) allow you to calibrate against experimental data, but they have been tested only on synthetic datasets.  
- **Never base engineering or financial decisions solely on these simulations** – always validate with laboratory measurements and qualified domain expertise.
- **Be transparent** when presenting results: clearly state that outputs are illustrative and list the assumptions made.

---

## Repository Structure

```
skills/
├── README.md                           # You are here
├── qfoldit-mining/
│   ├── SKILL.md
│   ├── README.md
│   ├── scripts/
│   └── references/
├── qfoldit-oilgas/
│   └── ...
├── ... (6 more skill folders)
└── LICENSE
```

Each skill folder is a complete Agent Skill, ready to be installed individually or published as a standalone repository.

---

## Contributing

Contributions are welcome!  
- Found a bug? Open an issue.  
- Want to add a calibration dataset or extend a model? Fork the repo and submit a pull request.  
- All skills follow the open Agent Skills specification – contributions should maintain compatibility.

For major changes, please discuss with the maintainers first (contact via GitHub or the qFoldIT website).

---

## License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.

---

## Links

- [qFoldIT website](https://qfoldit.github.io/claude/)
- [Agent Skills specification](https://agentskills.io)
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)

---

*Built with curiosity by the qFoldIT community.*
