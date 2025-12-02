# Feature Specification: Battle Fixes & Status Moves

**Feature ID**: 005-battle-fixes-status-moves  
**Branch**: `005-battle-fixes-status-moves`  
**Date**: 2025-12-02  
**Author**: Team  
**Status**: In Progress

---

## 1. Overview

### Problem Statement

The current battle system has several bugs and missing features that impact gameplay:

1. **Team Persistence Bug**: When refreshing the page, the player's Pokemon team is lost from the battle view, even though the data persists correctly in localStorage. The team store needs to be rehydrated before battle initialization.

2. **Status Moves Not Implemented**: The battle engine currently only supports damaging moves. Status moves (e.g., Growl, Leer, Thunder Wave) are converted to 'special' category with 0 power, but their actual effects (stat changes, status conditions) are not applied.

3. **BattleScreen Complexity**: The `BattleScreen.vue` component has grown too large and handles too many responsibilities, making it difficult to maintain and extend.

### Goals

- Fix the team persistence issue on page refresh
- Implement status move effects in the battle engine
- Modularize BattleScreen into smaller, focused components
- Maintain type safety and test coverage

### Non-Goals

- Adding new Pokemon or moves to the database
- Implementing abilities or held items
- Multiplayer battle synchronization

---

## 2. User Stories

### US-001: Team Persistence on Refresh
**As a** player  
**I want** my Pokemon team to persist when I refresh the battle page  
**So that** I don't lose my progress or have to recreate my team

**Acceptance Criteria**:
- [ ] Team is loaded from localStorage before battle initialization
- [ ] Player's Pokemon appears with correct sprite on page refresh
- [ ] Team data (HP, moves, stats) is preserved correctly
- [ ] No console errors during team rehydration

### US-002: Status Move Effects
**As a** player  
**I want** status moves to apply their actual effects  
**So that** battles have strategic depth beyond just damage

**Acceptance Criteria**:
- [ ] Stat-modifying moves (Growl, Leer, etc.) change target stats
- [ ] Status condition moves (Thunder Wave, Sleep Powder) apply conditions
- [ ] Move effects are logged in the battle log
- [ ] Status effects persist until battle end or removal

### US-003: Modular Battle UI
**As a** developer  
**I want** BattleScreen to be modular  
**So that** I can easily modify individual battle components

**Acceptance Criteria**:
- [ ] BattleField component handles sprite rendering
- [ ] BattleActionMenu handles user controls
- [ ] BagScreen is a separate component
- [ ] Each component has clear props/events interface

---

## 3. Technical Requirements

### TR-001: Team Rehydration
- Load team from `teamStore.loadTeam()` in `onMounted` before battle starts
- Ensure `battleStore.startBattle()` is called AFTER team is loaded
- Handle edge cases: empty team, corrupted localStorage data

### TR-002: Status Move Engine
- Extend `Move` interface with optional `effect` field
- Create `StatusEffectHandler` service for applying effects
- Support stat stages (-6 to +6 for each stat)
- Support status conditions: paralysis, sleep, poison, burn, freeze

### TR-003: Component Architecture
- Extract `BattleField.vue` for visual battlefield
- Extract `BattleActionMenu.vue` for control panel
- Keep `BattleScreen.vue` as orchestrator only
- Move styles to respective components (scoped)

---

## 4. Functional Requirements

### FR-001: Team Loading Sequence
```
1. BattleScreen.onMounted()
2. → teamStore.loadTeam() from localStorage
3. → Validate team has Pokemon with moves
4. → battleStore.startBattle(playerTeam, npcTeam)
5. → Render battle with loaded team
```

### FR-002: Status Effect Application
```
1. Player selects status move
2. Battle engine checks accuracy
3. If hit: Apply effect via StatusEffectHandler
4. Log effect to battle log
5. Update affected Pokemon's state
```

---

## 5. Dependencies

- **Depends On**:
  - Feature 003: Pokemon Team Builder (team storage)
  - Feature 002: PokeAPI Type Integration (type system)
  
- **Depended By**: 
  - Future: Multiplayer battles
  - Future: Advanced battle mechanics

---

## 6. Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Status effects introduce new bugs | High | Comprehensive unit tests for each effect |
| Team rehydration race conditions | Medium | Use async/await with proper sequencing |
| Component refactor breaks existing features | High | Integration tests for battle flow |

---

## 7. Definition of Done

- [ ] All acceptance criteria met
- [ ] `npm run type-check` passes
- [ ] `npm run test` passes (existing + new tests)
- [ ] No console errors in battle flow
- [ ] Manual testing: refresh page, team persists
- [ ] Code reviewed and approved
- [ ] Documentation updated if needed
