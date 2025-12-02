# Implementation Plan: Battle Fixes & Status Moves

**Branch**: `005-battle-fixes-status-moves` | **Date**: 2025-12-02 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/005-battle-fixes-status-moves/spec.md`

## Summary

This feature fixes a **critical bug** where the player's Pokemon team is lost on page refresh despite being saved in localStorage, and establishes the foundation for status move effects. The root cause is that `BattleScreen.vue` accesses `teamStore.roster` before calling `teamStore.loadTeam()` in its `onMounted()` hook. The fix is a single line addition at the start of the initialization sequence.

## Technical Context

**Language/Version**: TypeScript 5.x with Vue 3.5+ Composition API  
**Primary Dependencies**: Vue 3, Pinia (state management), Vite  
**Storage**: localStorage (key: `pkmn-mmo-team`)  
**Testing**: Vitest with Vue Test Utils (`npm run type-check`, `npm run test`)  
**Target Platform**: Modern browsers (Chrome, Firefox, Safari)  
**Project Type**: Single-page web application (Vue SPA)  
**Performance Goals**: 60 FPS battle animations, <100ms state transitions  
**Constraints**: Offline-capable (localStorage persistence), no backend  
**Scale/Scope**: Single-player Pokemon battle game, 6-Pokemon team limit

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| Simplicity First | ✅ PASS | Single-line fix for persistence bug |
| Component Modularity | ✅ PASS | Leverages existing modular architecture |
| Type Safety | ✅ PASS | All changes TypeScript-strict compliant |
| Store Pattern | ✅ PASS | Uses existing Pinia stores correctly |
| No Over-Engineering | ✅ PASS | Minimal changes, no new abstractions |

## Project Structure

### Documentation (this feature)

```text
specs/005-battle-fixes-status-moves/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Research findings (root cause analysis)
├── data-model.md        # Entity definitions for status effects
├── quickstart.md        # Development guide
├── contracts/           # TypeScript interfaces
│   └── battle-contracts.ts
└── tasks.md             # Implementation tasks (Phase 2)
```

### Source Code (repository root)

```text
src/
├── components/
│   ├── BattleScreen.vue       # FIX: Add loadTeam() call
│   └── battle/
│       ├── BattleField.vue    # Visual battlefield (existing)
│       ├── BattleActionMenu.vue # Control panel (existing)
│       └── BagScreen.vue      # Items display (existing)
├── models/
│   ├── statStages.ts          # NEW: Stat stage types (Phase 2)
│   ├── statusCondition.ts     # NEW: Status condition types (Phase 2)
│   └── moveEffect.ts          # NEW: Move effect types (Phase 2)
├── services/
│   └── battle/
│       ├── statStageService.ts      # NEW: Stat modification (Phase 2)
│       └── statusConditionService.ts # NEW: Status logic (Phase 2)
├── stores/
│   ├── team.ts                # Existing: loadTeam() method
│   └── battle.ts              # Existing: Battle state
└── domain/
    └── battle/
        └── calc/
            └── damage.ts      # UPDATE: Apply stat stages (Phase 2)

tests/
├── integration/
│   └── components/
│       └── BattleUI.spec.ts   # Existing test file
└── unit/
    └── domain/                # Existing unit tests
```

**Structure Decision**: Single Vue SPA structure. No architectural changes needed - fix is surgical and status effects follow existing service/model patterns.

## Complexity Tracking

> No violations - this is a minimal fix with optional enhancements

| Aspect | Assessment |
|--------|------------|
| Files Changed | 1 (BattleScreen.vue) for critical fix |
| Lines Changed | ~3 lines (import + loadTeam call) |
| New Dependencies | None |
| Test Impact | Manual verification sufficient |
