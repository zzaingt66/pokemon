# Tasks: Battle Fixes & Status Moves

**Input**: Design documents from `/specs/005-battle-fixes-status-moves/`  
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: Not explicitly requested - implementation tasks only

**Organization**: Tasks grouped by user story to enable independent implementation and testing

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: No new infrastructure needed - project already configured

- [X] T001 Verify project compiles with `npm run type-check`
- [X] T002 Verify existing tests pass with `npm run test`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core models and types that multiple user stories depend on

**⚠️ CRITICAL**: These models must exist before status move tasks can begin

- [X] T003 [P] Create StatStages interface and constants in src/models/statStages.ts
- [X] T004 [P] Create StatusCondition type in src/models/statusCondition.ts
- [X] T005 [P] Create MoveEffect interface in src/models/moveEffect.ts
- [X] T006 Update Move interface to include optional effect field in src/domain/battle/engine/entities.ts

**Checkpoint**: Foundation ready - user story implementation can begin

---

## Phase 3: User Story 1 - Team Persistence on Refresh (Priority: P1) 🎯 MVP

**Goal**: Fix the critical bug where team is lost on page refresh despite localStorage persistence

**Independent Test**: 
1. Build team in Team Builder
2. Start battle
3. Refresh page (F5)
4. Team should persist with all Pokemon and correct data

### Implementation for User Story 1

- [X] T007 [US1] Add teamStore.loadTeam() call at start of onMounted in src/components/BattleScreen.vue
- [X] T008 [US1] Add empty team validation with user-friendly error in src/components/BattleScreen.vue
- [X] T009 [US1] Verify localStorage key 'pokemon-team-v1' is correctly read in src/stores/team.ts

**Checkpoint**: User Story 1 complete - team persists on page refresh

---

## Phase 4: User Story 2 - Status Move Effects (Priority: P2)

**Goal**: Status moves apply their actual effects (stat changes, status conditions)

**Independent Test**: 
1. Use Growl move in battle
2. Opponent's Attack stat should decrease
3. Battle log shows "Enemy's Attack fell!"

### Implementation for User Story 2

- [X] T010 [P] [US2] Create statStageService with applyStatChange and getEffectiveStat in src/services/battle/statStageService.ts
- [X] T011 [P] [US2] Create statusConditionService with applyStatus and checkCanAct in src/services/battle/statusConditionService.ts
- [X] T012 [US2] Update damage calculation to apply stat stage multipliers in src/domain/battle/calc/damage.ts
- [ ] T013 [US2] Add status effect handling in battle turn execution in src/domain/battle/engine/battleEngine.ts
- [ ] T014 [US2] Add move effect data to existing moves in src/data/moves.ts (Growl, Leer, etc.)
- [ ] T015 [US2] Display stat change messages in battle log in src/components/BattleScreen.vue

**Checkpoint**: User Story 2 complete - status moves work correctly

---

## Phase 5: User Story 3 - Modular Battle UI (Priority: P3)

**Goal**: Confirm BattleScreen is properly modularized with clear component interfaces

**Independent Test**: 
1. Navigate to battle
2. All UI components render correctly
3. Fight/Bag/Pokemon/Run buttons work
4. No console errors

### Implementation for User Story 3

- [X] T016 [US3] Verify BattleField.vue has correct props interface in src/components/battle/BattleField.vue
- [X] T017 [US3] Verify BattleActionMenu.vue has correct events interface in src/components/battle/BattleActionMenu.vue
- [X] T018 [US3] Verify BagScreen.vue integrates correctly in src/components/battle/BagScreen.vue
- [X] T019 [US3] Document component API in comments for BattleScreen orchestrator in src/components/BattleScreen.vue

**Checkpoint**: User Story 3 complete - modular UI verified

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final validation and cleanup

- [X] T020 [P] Run npm run type-check - ensure no TypeScript errors
- [X] T021 [P] Run npm run test - ensure all tests pass (pre-existing issues noted)
- [ ] T022 Manual test: Complete battle flow with page refresh
- [ ] T023 Manual test: Use status move and verify effect appears in log
- [ ] T024 Run quickstart.md validation checklist

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1: Setup
    ↓
Phase 2: Foundational (models for status effects)
    ↓
┌─────────────────────────────────────────────────────┐
│ After Foundational, user stories can run in parallel│
│ or sequentially by priority (P1 → P2 → P3)          │
└─────────────────────────────────────────────────────┘
    ↓         ↓         ↓
Phase 3    Phase 4    Phase 5
 (US1)      (US2)      (US3)
    ↓         ↓         ↓
    └─────────┴─────────┘
              ↓
        Phase 6: Polish
```

### User Story Dependencies

| User Story | Phase | Can Start After | Dependencies on Other Stories |
|------------|-------|-----------------|-------------------------------|
| US1 (P1) - Team Persistence | 3 | Foundational | **None** - fully independent |
| US2 (P2) - Status Effects | 4 | Foundational | **None** - uses Foundational models |
| US3 (P3) - Modular UI | 5 | Foundational | **None** - verification only |

### Within Each User Story

- US1: T007 → T008 → T009 (sequential - same file)
- US2: T010, T011 [P] → T012, T013 → T014 → T015
- US3: T016, T017, T018, T019 all [P] - different files

### Parallel Opportunities

```bash
# Phase 2 - Run in parallel (different files):
T003: Create StatStages in src/models/statStages.ts
T004: Create StatusCondition in src/models/statusCondition.ts
T005: Create MoveEffect in src/models/moveEffect.ts

# Phase 4 (US2) - Run in parallel (different files):
T010: Create statStageService in src/services/battle/statStageService.ts
T011: Create statusConditionService in src/services/battle/statusConditionService.ts

# Phase 5 (US3) - Run in parallel (verification tasks):
T016: Verify BattleField.vue
T017: Verify BattleActionMenu.vue
T018: Verify BagScreen.vue

# Phase 6 - Run in parallel:
T020: npm run type-check
T021: npm run test
```

---

## Implementation Strategy

### MVP First (User Story 1 Only) - RECOMMENDED

1. ✅ Complete Phase 1: Setup (T001-T002)
2. ⏭️ Skip Phase 2 (not needed for US1)
3. ✅ Complete Phase 3: User Story 1 (T007-T009)
4. **STOP and VALIDATE**: Test page refresh - team should persist
5. ✅ Deploy/demo immediately - critical bug fixed!

**Time Estimate**: ~15-30 minutes for MVP (US1 only)

### Full Feature Delivery

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (models)
3. Complete Phase 3: US1 (team persistence) → Validate
4. Complete Phase 4: US2 (status effects) → Validate
5. Complete Phase 5: US3 (modular UI verification) → Validate
6. Complete Phase 6: Polish

**Time Estimate**: ~2-4 hours for full feature

### Task Summary

| Phase | Task Count | Parallel Tasks | User Story |
|-------|------------|----------------|------------|
| 1: Setup | 2 | 0 | - |
| 2: Foundational | 4 | 3 | - |
| 3: US1 | 3 | 0 | Team Persistence |
| 4: US2 | 6 | 2 | Status Effects |
| 5: US3 | 4 | 4 | Modular UI |
| 6: Polish | 5 | 2 | - |
| **Total** | **24** | **11** | |

---

## Notes

- **US1 is the critical fix** - can be deployed independently as MVP
- US2 (Status Effects) is the main feature work requiring Foundational models
- US3 is verification only - modularization already complete from previous work
- Commit after each task or logical group
- Run `npm run type-check` after any TypeScript changes
