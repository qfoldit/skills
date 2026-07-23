# [qFoldIT](https://qfoldit.github.io/claude/) Agent Skills Plugin

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-standard-green)](https://agentskills.io)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

![](https://qfoldit.github.io/claude/claude_science.png)

**Mission (qFoldIT's own product positioning):** qFoldIT manages **multiplayer gamification of science** across these Skills -- the scientific-educational ones below (VQE, protein folding, bio-mining, corrosion, plant growth, L-systems, plastics) supply the science; `game-designer` + `digital-twin-builder` structure it into a UAG; the engine adapters (including `skills/uefn-fortnite-world-builder`, the multiplayer connector) put it in front of players. This framing describes qFoldIT's own layer, not the upstream tools/papers any individual skill wraps or cites -- none of those describe themselves as gamification tools.

# Included Agent Skills

qFoldIT Skills are modular AI capabilities. Each Skill contains: domain knowledge, workflows, validation rules, examples, and MCP integration points where relevant. Folder names below match exactly what's in `skills/` -- use them to install/reference a specific skill.

Repository: https://github.com/qfoldit/skills/

---

# Scientific AI Skills

## `skills/qfold` -- Quantum Molecular Folding
2D HP-lattice protein folding (Dill, 1985) with a QAOA-style circuit on Amazon Braket LocalSimulator, plus optional delegation to a full qFold MCP server when available.

## `skills/bionemo-agent-toolkit` -- Structure Prediction
Protein/complex structure prediction via AlphaFold2 (NVIDIA BioNeMo NIM), AlphaFold3 (local Docker via `alphafold3_mcp`), and OpenFold3 (open re-implementation of AlphaFold3, NIM), plus DiffDock docking, MolMIM molecule generation, Evo 2 genomic sequence generation, and Boltz-2 combined structure+affinity prediction. **A compatible client, not an official NVIDIA or Google DeepMind integration.** For ESMFold/RFdiffusion/ProteinMPNN, see this org's separate `protein-design-mcp` server, which runs those locally/via Docker with a real test suite rather than as NIM clients.

## `skills/quantum` -- VQE Simulator
Classical statevector VQE simulator (numpy-only), validated against exact diagonalization and a real published H2/STO-3G benchmark (arXiv:2209.09564, matches -1.1373 Hartree to 4 decimal places).

---

# Bioindustrial & Environmental Skills

## `skills/mining` -- Bioprocesses in Mining
Bio-oxidation, biosorption (gold-cyanide/copper/REE/lithium), and cyanide biodegradation kinetics for metal recovery.

## `skills/prospecting` -- Microbial Ore Prospecting
Indicator-taxa statistics on microbial abundance data for biogeochemical mineral exploration.

## `skills/meor` -- Microbial Enhanced Oil Recovery
Bacterial growth, biosurfactant production, IFT reduction, and incremental oil recovery modeling.

## `skills/oilgas` -- CO2 Corrosion in Pipelines
Pipeline corrosion rate and remaining-wall-life estimation (de Waard-Milliams correlation).

---

# Biological Simulation Skills

## `skills/l-systems` -- L-System Fractals & Plants
Procedural plant/fractal generation (Koch, dragon, Sierpinski, branching plants) rendered as SVG.

## `skills/plant-growth` -- Plant Nutrition & Light Model
Growth rate, morphology, and secondary-metabolite indices from NPK nutrition and light spectrum -- qualitative/illustrative, not species-calibrated. Can feed `skills/l-systems` to visualize the resulting shape.

---

# Materials & Industrial Skills

## `skills/plastic-characterization` -- Plastic Pyrolysis Estimator
Pyrolysis oil/gas/char yield ranges and chlorine-contamination risk for mixed plastic feedstock.

---

# VR / Interactive Molecular Dynamics & Gamification

## `skills/nanover` -- NanoVer VR Bridge
Bridges qFoldIT structures (from `qfold`/`bionemo-agent-toolkit`) into **NanoVer**, a real, independently published and maintained open-source framework (IRL2, Univ. of Santiago de Compostela; JOSS 2025, DOI 10.21105/joss.08118) for collaborative interactive molecular dynamics in VR. qFoldIT's role is a thin adapter only -- **NanoVer is not a qFoldIT invention; citing the source publications is required** whenever this skill's output is used. Requires a local conda install of `nanover-server`.

## `skills/molecular-tetris` -- Molecular Quantum Tetris
A playable, self-contained HTML5 puzzle game (falling element-labeled blocks, flood-fill match-4 clearing) for presentations, demos, and trade-show engagement -- the actual, working piece of qFoldIT's original "creative jump through gamification" thesis. **Gamification only, explicitly not a chemistry simulation** -- element matching is a game mechanic, not a model of real bonding rules; the skill itself says so before/when presenting the game. Ships with 12 passing headless logic tests (`evals/logic_test.js`: group-clear, flood-fill on L-shapes, gravity, chain reactions, wall collisions).

---

# Digital Twin & Engine Skills

These share one engine-neutral format -- the **Universal Assembly Graph (UAG)**, schema in `skills/game-designer/references/uag_schema.md`. `game-designer` produces a UAG; `digital-twin-builder` adds simulation-grade physical semantics; each engine skill below exports the (enriched) UAG to one target platform. Every engine skill validates against `skills/game-designer/scripts/uag_validate.py` before exporting -- and every skill states plainly whether its target integration is official or community-maintained. (UAG here is a concrete 3D scene-graph format, narrower than and distinct from the broader scientific-activity graph/execution model -- SKG (Scientific Knowledge Graph) + SEM (Scientific Execution Model) -- defined in the ecosystem's [Scientific World Schema](https://github.com/qfoldit/scientific-world-schema); a UAG document is what an SKG `DigitalTwin` node's content looks like when it targets a game engine.)

| Skill folder | Target | Status |
|---|---|---|
| `skills/game-designer` | produces the UAG (engine-neutral) | qFoldIT-authored |
| `skills/digital-twin-builder` | enriches UAG with physical/simulation semantics | qFoldIT-authored |
| `skills/unreal-world-builder` | Unreal Engine | ✅ Official ("Unreal MCP", plugin id `ModelContextProtocol`, shipped Experimental in UE 5.8) -- but Epic's own documented client targets are Claude Code, Cursor, VS Code, Gemini, and Codex; **Claude Desktop is not among them**. For Desktop-based workflows, see the note below. |
| `skills/unity-experience-builder` | Unity | ✅ Official primary target: Unity MCP Server (Unity Technologies, Unity AI Beta, `com.unity.ai.assistant` package, Unity 6.0+) -- this skill authors workflows against that server's own tool set (`Unity_ManageGameObject`, `Unity_ManageScene`, etc.) by default. ⚠️ Falls back to community [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp) ("MCP for Unity", not affiliated with Unity Technologies; MIT-licensed) only for Unity <6.0 or without the AI Assistant package -- cite Wu, S. & Barnett, J.P. (CoPlay), "MCP-Unity: Protocol-Driven Framework for Interactive 3D Authoring," *SIGGRAPH Asia 2025 Technical Communications*, DOI [10.1145/3757376.3771417](https://doi.org/10.1145/3757376.3771417), if that fallback is used in research. |
| `skills/unigine-simulation-engineer` | UNIGINE 2 / UNIGINE 2 Sim | ✅ Official pattern ([unigine-engine/ai-docs](https://github.com/unigine-engine/ai-docs)), docs-grounded to avoid hallucinated API calls. Pairs with the official [MCPBridge Plugin](https://store.unigine.com/en/add-on/1f1237c6-234c-6f20-8dee-b35a5bf2dc28/description) (free, UNIGINE Add-On Store, v2.21) for hands-on Editor control -- see the note below on how it's configured. |
| `skills/openusd-architect` | OpenUSD / NVIDIA Omniverse | ✅ Official ([NVIDIA-Omniverse/kit-usd-agents](https://github.com/NVIDIA-Omniverse/kit-usd-agents)) -- **note this is a coding/API-assistance MCP** (Kit extension search, USD class/method docs, code examples), not a live scene-editing bridge like the rows above; there is no verified general-purpose live scene-editing MCP for Omniverse Kit as of this check. |
| `skills/apple-spatial-designer` | RealityKit / visionOS / USDZ | ✅ Official Xcode<->Claude Agent SDK integration; RealityKit specifics from public docs + community skills, not one single authoritative source |
| `skills/threejs-web-designer` | Three.js / React Three Fiber / WebXR | Community ecosystem; no single official skill exists for this stack |
| `skills/uefn-fortnite-world-builder` | UEFN (Unreal Editor for Fortnite) / Fortnite islands | ⚠️ Community only -- Epic has no published UEFN MCP today (roadmap: merges into Unreal Engine 6, targeting late 2027). Primary: [KirChuvakov/uefn-mcp-server](https://github.com/KirChuvakov/uefn-mcp-server) (MIT, 28 tools). Alternative: [quangdang46/uefn-verse-mcp](https://github.com/quangdang46/uefn-verse-mcp) (AGPL-3.0 + attribution term, 354 actions). **The multiplayer-gamification connector**: unlike single-player `skills/molecular-tetris`, a published Fortnite island is inherently multiplayer -- but this skill only authors the editor scene, it doesn't wire Verse/Device gameplay or publish the island. |
| `skills/scene-export` | universal files: OBJ, glTF 2.0, standalone Three.js HTML, Godot 4.x GDScript | qFoldIT-authored -- fallback/interchange layer for when the live MCP adapters above aren't installed/available, or a portable file is needed instead of a live editor session |

**Note on Unity:** falls back to `CoplayDev/unity-mcp` only when the official server isn't available. Confirmed via each project's own docs on 2026-07-12. The two are not interchangeable -- different tool sets -- so a workflow authored against one won't run unmodified against the other. **If you have both installed for fallback purposes, watch the config key names**: the official server's own docs use `unity-mcp` (kebab-case), while CoplayDev's auto-configurator defaults to `unityMCP` (camelCase) -- similar enough to invite a copy-paste mistake that silently overwrites the wrong entry in a shared `claude_desktop_config.json`.

**Note on Unreal:** unlike Unity, there is no single "official + community, both Desktop-capable" pair here -- Epic's own official plugin simply doesn't document Claude Desktop as a target. For Claude Code, run `ModelContextProtocol.GenerateClientConfig ClaudeCode` in the Unreal Editor console (after enabling the Unreal MCP and AllToolsets plugins) and see Epic's own companion repo, [EpicGames/unreal-engine-skills-for-claude-code-plugin](https://github.com/EpicGames/unreal-engine-skills-for-claude-code-plugin). For Claude Desktop, this org's `Protein-Design-MCP` repo's `claude_desktop_config.json` configures the independent, EXPERIMENTAL community project [chongdashu/unreal-mcp](https://github.com/chongdashu/unreal-mcp) (not affiliated with Epic Games) instead.

**Note on UNIGINE:** MCPBridge is not configured via `claude_desktop_config.json` at all -- installing it (drag the `.upackage` into the Asset Browser) auto-generates a project-root `.mcp.json`, which is **Claude Code's** project-scoped MCP convention, not Claude Desktop's. Use Claude Code from the UNIGINE project directory to pick it up automatically. Pair with [UNIGINE AI Docs](https://github.com/unigine-engine/ai-docs) for combined engine-knowledge and hands-on control.

**Note on UEFN/Fortnite:** unlike the three engines above, there is no official-vs-community *pair* here -- only community options exist, since Epic has not published an MCP for UEFN itself (only for standalone Unreal Engine 5.8+, a different product). `claude_desktop_config.json`'s `uefn` entry configures KirChuvakov/uefn-mcp-server (MIT) by default. This skill is the one most directly tied to qFoldIT's stated multiplayer-gamification mission -- see `skills/uefn-fortnite-world-builder/SKILL.md`'s own scope-limit note before describing it as "publishing a multiplayer science game": it authors the editor scene only.

---

## Links

- [qFoldIT](https://qfoldit.github.io/claude/)
- [Agent Skills specification](https://agentskills.io)
- [Claude Code documentation](https://docs.claude.com/en/docs/claude-code)

---

*Built with curiosity by the qFoldIT community.*
