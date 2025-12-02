# Research: Battle Fixes & Status Moves

**Feature**: 005-battle-fixes-status-moves  
**Date**: 2025-12-02  
**Status**: Complete

---

## Research Tasks

### RT-001: Team Persistence Bug Analysis

**Task**: Investigate why team is lost on page refresh despite localStorage persistence

**Findings**:

1. **Root Cause Identified**: The `BattleScreen.vue` component's `onMounted` hook does NOT call `teamStore.loadTeam()` before accessing `teamStore.roster`. 

2. **Data Flow Analysis**:
   ```
   Current (Broken):
   1. Page refresh
   2. BattleScreen.onMounted() executes
   3. getPlayerTeam computed accesses teamStore.roster (EMPTY - not loaded yet)
   4. Team validation fails → "No Team Builder roster available"
   5. Battle starts with empty/placeholder team
   ```

3. **localStorage Verification**: Data IS present in localStorage under `pkmn-mmo-team` key. The `teamStore.loadTeam()` function correctly reads and parses this data, but it's never called before battle initialization.

4. **Fix Required**: Add `teamStore.loadTeam()` call at the START of `onMounted` in `BattleScreen.vue`, BEFORE accessing `getPlayerTeam.value`.

**Decision**: Implement explicit team loading in BattleScreen.onMounted  
**Rationale**: Ensures team hydration happens before any computed properties access roster  
**Alternatives Rejected**: 
- Auto-loading in store constructor (violates Pinia best practices)
- Using localStorage directly (duplicates logic)

---

### RT-002: Status Move Categories in Battle Engine

**Task**: Understand how status moves should be handled

**Findings**:

1. **Current Behavior**: Status moves are converted to `category: 'special'` with `power: 0`. This causes:
   - Damage calculation returns 0 (correct for non-damaging moves)
   - No actual effect is applied (stat changes, conditions)
   - Move appears to "do nothing" in battle

2. **Pokemon Status Effects Categories**:
   - **Stat Modifiers**: Growl (-1 Atk), Leer (-1 Def), Swords Dance (+2 Atk)
   - **Status Conditions**: Paralysis, Sleep, Poison, Burn, Freeze
   - **Field Effects**: Reflect, Light Screen (not in scope)
   - **Healing**: Recover, Synthesis (not in scope initially)

3. **Implementation Approach**:
   ```typescript
   interface StatusEffect {
     type: 'stat-change' | 'status-condition'
     target: 'self' | 'opponent'
     stat?: 'atk' | 'def' | 'spAtk' | 'spDef' | 'speed'
     stages?: number // -6 to +6
     condition?: 'paralysis' | 'sleep' | 'poison' | 'burn' | 'freeze'
   }
   ```

4. **Stat Stage Multipliers** (from official formula):
   | Stage | Multiplier |
   |-------|------------|
   | -6    | 2/8 (0.25) |
   | -5    | 2/7 (0.29) |
   | -4    | 2/6 (0.33) |
   | -3    | 2/5 (0.40) |
   | -2    | 2/4 (0.50) |
   | -1    | 2/3 (0.67) |
   | 0     | 2/2 (1.00) |
   | +1    | 3/2 (1.50) |
   | +2    | 4/2 (2.00) |
   | +3    | 5/2 (2.50) |
   | +4    | 6/2 (3.00) |
   | +5    | 7/2 (3.50) |
   | +6    | 8/2 (4.00) |

**Decision**: Extend Pokemon interface with `statStages` object, create StatusEffectHandler service  
**Rationale**: Clean separation of concerns, testable logic  
**Alternatives Rejected**: 
- Inline effect handling in damage calc (violates SRP)
- Separate effect entity (over-engineering for current scope)

---

### RT-003: BattleScreen Modularization Status

**Task**: Review current component structure after refactoring

**Findings**:

1. **Already Completed**:
   - ✅ `BattleField.vue` - Handles sprite rendering, HP bars, battlefield visuals
   - ✅ `BattleActionMenu.vue` - Handles Fight/Bag/Pokemon/Run controls + MoveSelector
   - ✅ `BagScreen.vue` - Handles item display and usage

2. **Remaining in BattleScreen.vue** (~350 lines):
   - Battle initialization logic (onMounted)
   - Pokemon conversion functions
   - Event handlers for all child components
   - Battle flow watchers (faint detection, trainer switch)
   - Audio integration

3. **Further Modularization Opportunities** (future):
   - Extract `useBattleFlow` composable for turn sequencing
   - Extract `useBattleAudio` composable for sound effects
   - Extract conversion functions to a service

**Decision**: Current modularization is sufficient for this feature  
**Rationale**: BattleScreen is now ~350 lines (was ~1000+), manageable as orchestrator

---

## Summary

| Research Task | Decision | Confidence |
|---------------|----------|------------|
| RT-001 Team Persistence | Add loadTeam() call in onMounted | High |
| RT-002 Status Moves | Create StatusEffectHandler with stat stages | High |
| RT-003 Modularization | Current state sufficient | High |

All NEEDS CLARIFICATION items resolved.
