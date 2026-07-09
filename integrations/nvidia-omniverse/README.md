# qFoldIT + NVIDIA Omniverse Integration

## Purpose

This integration connects qFoldIT molecular digital twins with NVIDIA Omniverse pipelines.

qFoldIT remains the scientific data layer:
- molecular digital twin JSON
- residue/ligand metadata
- energy and interaction data
- engine-neutral scene plans

NVIDIA Omniverse is the USD-based visualization and simulation layer.

## Supported bridge

Official NVIDIA project:
- NVIDIA-Omniverse/kit-usd-agents

Role:
- USD scene generation workflows
- Kit/USD agent automation patterns
- Omniverse UI and extension workflows

## Architecture

```
Claude / MCP Client
        |
        +-- qFoldIT MCP
        |      |
        |      +-- Digital Twin
        |      +-- Scene Plan
        |
        +-- NVIDIA Omniverse Kit/USD Agents
               |
               +-- USD Stage
               +-- Molecular visualization
               +-- Simulation assets
```

## Scene mapping

qFoldIT node -> USD Prim

- molecule atom/residue -> USD geometry prim
- interaction edge -> USD relationship
- energy value -> material/metadata attribute
- folding state -> animation parameter

## Important

This module provides integration documentation and schemas.
The Omniverse MCP/agent implementation is maintained by NVIDIA and should
be installed separately according to NVIDIA documentation.
