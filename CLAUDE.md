# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PipeSpline is a Blueprint-only Unreal Engine 5.8 tool for the Fab marketplace. It creates pipes along a spline with consistent UV tiling (texture density is maintained regardless of segment length).

## Architecture

- **BP_SplinePipe** (`Content/Bluperints/`) -- Main Blueprint actor. Contains spline component, Spline Mesh Components for pipe segments, and HISM component for connection meshes.
- **Spline Mesh segments** use `SM_PipeSpline` mesh with `M_PipeSpline_Tiled` (Material Instance).
- **Connection meshes** use `SM_PipeConnection` via HISM with `M_PipeSpline` (base material). Connections are placed at each spline point (doubled at middle points to represent joint between two segments).

## Material System

- `M_PipeSpline` -- Base material shared by both pipe and connection meshes.
- `M_PipeSpline_Tiled` -- Material Instance of `M_PipeSpline`. Overrides a **Static Switch Parameter** that enables UV tiling calculation in the shader, maintaining texture density across varying spline segment lengths.
- Both materials compile to separate shader permutations (Static Switch), so there is zero runtime branching cost.
- Textures use packed ORM format (Occlusion/Roughness/Metallic in one texture).

## Key Design Decisions

- **HISM for connections**: All connection meshes are instances in a single Hierarchical Instanced Static Mesh component, minimizing draw calls.
- **Spline Mesh Components** are inherently non-instancable (UE limitation) -- each segment is a separate draw call.
- **Static Switch** (not dynamic bool) controls UV tiling path -- each Material Instance compiles only the shader code it needs.

## Content Structure

```
Content/
  Bluperints/       -- BP_SplinePipe (note: folder has typo "Bluperints")
  Materials/        -- M_PipeSpline, M_PipeSpline_Tiled
  Meshes/           -- SM_PipeSpline, SM_PipeConnection
  Textures/         -- BaseColor, Normal, ORM textures
```

## Important Notes

- This is a Blueprint-only project (no C++ source, no .Build.cs). All logic is in Blueprint assets (.uasset), which are binary and cannot be read/diffed as text.
- The project targets Fab marketplace distribution, so Fab guidelines apply (LODs, documentation, proper collision, etc.).
- All assets are binary `.uasset` files -- Claude Code cannot inspect or modify Blueprint graphs, material graphs, or mesh assets directly.