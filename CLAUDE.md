# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**score-notation-app** — a lightweight macOS desktop app for music notation. Musicians enter pitch names into a grid to quickly record musical ideas. Completely silent (no audio playback). Built with Electron + TypeScript + HTML5 Canvas.

## Tech Stack

| Layer | Technology |
|---|---|
| Shell | Electron (macOS) |
| Rendering | HTML5 Canvas / OffscreenCanvas |
| Logic | TypeScript |
| State | Immutable Command Pattern (Undo/Redo) |
| Persistence | JSON files (local filesystem) |
| Tests | Vitest + fast-check (property-based testing) |

## Commands

> No source code exists yet — this repo is in the specification phase. Once implementation begins, expected commands:

```bash
npm install          # Install dependencies
npm run dev          # Development (Electron)
npm run build        # Build distributable
npm run test         # Run unit + property tests (Vitest)
npm run test:bench   # Performance benchmarks (local only, skip in CI)
npm run lint         # TypeScript/linting
```

Run a single test file:
```bash
npx vitest run src/path/to/file.test.ts
```

## Architecture

The app is split into Electron **Main Process** and **Renderer Process**:

```
Main Process:
  FileIOService      — save/load/backup JSON to local filesystem
  AutoBackupService  — every 5 minutes, keep max 36 backup generations
  IPC Bridge         — communicates with Renderer

Renderer Process:
  UI Layer:
    CanvasRenderer   — draws the full grid via virtual scroll (viewport only)
    InputHandler     — keyboard/mouse events
    IMEEngine        — converts romaji keystrokes → PitchName (no OS IME)
  Application Layer:
    AppController    — orchestrates
    CommandBus       — dispatches commands, manages Undo/Redo stacks
  Domain Layer:
    ProjectStore     — immutable state container, notifies Renderer on change
    GridModel        — core data model (tracks, cells, bars)
    UndoStack        — command history
    Clipboard        — copy/paste data
```

### State Change Flow

All mutations go through the Command Pattern:
1. `InputHandler` → `IMEEngine` → `CommandBus.dispatch(command)`
2. `CommandBus` pushes to `UndoStack`, which calls `command.execute(state)`
3. `ProjectStore` publishes the new state → `CanvasRenderer` calls `requestAnimationFrame`

Every command implements `execute(state): ProjectState` and `undo(state): ProjectState`.

## Domain Model

### PitchName (15 types)

12 chromatic pitches: `ド ノ レ ネ ミ ハ バ ソ ゾ ラ ジ シ`  
3 special pitches (no octave): `x ー ッ`

### Romaji → PitchName mapping (IMEEngine)

| PitchName | Romaji |
|---|---|
| ド | do |
| ノ | no |
| レ | re |
| ネ | ne |
| ミ | mi |
| ハ | ha |
| バ | ba |
| ソ | so |
| ゾ | zo |
| ラ | ra |
| ジ | zi / ji |
| シ | si / shi |
| x | x |
| ー | - (hyphen) |
| ッ | tsu / tu / ltsu / ltu |

Invalid sequences: reset buffer, do not advance cell.

### NoteValue widths (unit = 32nd note = 1)

`whole=32, half=16, quarter=8, eighth=4(default), sixteenth=2, thirty_second=1`  
Triplets: `half_triplet=32/3, quarter_triplet=16/3, eighth_triplet=8/3, sixteenth_triplet=4/3`

### VerticalOffset

Pitch height → pixel Y offset. Range: A1–C6 (52 pitches).  
`SEMITONE_HEIGHT_PX = 4` — pixels per semitone.  
`VerticalOffset = (semitone of C6 - current semitone) * SEMITONE_HEIGHT_PX`  
Special pitches: `x` = fixed position; `ー` and `ッ` inherit the VerticalOffset of the last preceding non-special pitch (or fixed if none exists).

## Testing Strategy

Use **both** unit tests and property-based tests — they're complementary.

- **Unit tests**: specific examples, edge cases, error conditions, UI interactions, initial state
- **Property-based tests** (fast-check): universally true properties, min 100 iterations each

Each property test must reference the design doc property number in a comment:
```
// Feature: score-notation-app, Property {N}: {property name}
```

### The 16 Correctness Properties (each needs one property-based test)

| # | Property | Requirement |
|---|---|---|
| 1 | NoteValue width proportionality | 1.4 |
| 2 | All tracks wrap at same SystemBreak | 1.6, 1.10 |
| 3 | LocalSystemBreak applies only to its system | 1.9 |
| 4 | Cell time-axis position invariance after any operation | 1.11 |
| 5 | IMEEngine romaji→PitchName completeness | 2.2, 2.5 |
| 6 | PitchName display is always katakana (x half-width) | 2.4 |
| 7 | VerticalOffset monotonicity (higher pitch = smaller offset) | 3.1–3.4 |
| 8 | ー/ッ inherit previous non-special pitch's VerticalOffset | 3.6, 3.7 |
| 9 | OctaveShift clamps to A1–C6 range | 4.1, 4.2, 4.5 |
| 10 | NoteValue change preserves total note-unit count | 5.5–5.9 |
| 11 | New cells use CurrentNoteValue | 5.3 |
| 12 | Serialize→deserialize round-trip produces equivalent Project | 9.1–9.4, 9.6 |
| 13 | Invalid JSON returns error, leaves grid unchanged | 9.5 |
| 14 | Backup count never exceeds 36 | 9.8 |
| 15 | Undo all then Redo all returns to original state | 10.1, 10.2, 10.9 |
| 16 | Paste only executes when track type order matches exactly | 11.8–11.10 |

## Performance Requirement

**All operations must complete within 16ms** (60fps). This applies even with 100,000+ cells.  
Canvas performance benchmarks (Vitest bench) run locally only — skip in CI.

## Key Invariants

- Cells never shift on the time axis due to any operation (Property 4)
- All tracks wrap at the same SystemBreak position (Property 2)
- NoteValue change absorbs/splits cells but preserves total duration (Property 10)
- Paste is a no-op when clipboard track types don't match target (Property 16)
- Backup failures are silent — never interrupt user operations
- Schema version is stored in serialized JSON (`version: 1` currently); migration attempted on mismatch

## Specs Location

Full requirements and design are in `.kiro/specs/score-notation-app/`:
- `requirements.md` — acceptance criteria for all 11 feature areas
- `design.md` — TypeScript interfaces, data models, architecture diagrams, all 16 correctness properties
