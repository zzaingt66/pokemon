# Quickstart Guide: Battle Fixes & Status Moves

**Feature**: 005-battle-fixes-status-moves  
**Date**: 2025-12-02

---

## Overview

This feature fixes the critical team persistence bug and lays groundwork for status move support. The modular battle UI from previous work (BattleField, BattleActionMenu) is already in place.

---

## Critical Fix: Team Persistence on Refresh

### Problem
When refreshing the page during a battle, the player's team is lost even though it's saved in localStorage.

### Root Cause
`BattleScreen.vue`'s `onMounted()` hook accesses `teamStore.roster` BEFORE calling `teamStore.loadTeam()`.

### Solution

**File**: `src/components/BattleScreen.vue`

```typescript
// BEFORE (broken)
onMounted(() => {
  battleStore.startBattle(getPlayerTeam.value, getEnemyTeam.value)
  // ...
})

// AFTER (fixed)
onMounted(() => {
  // CRITICAL: Load team from localStorage FIRST
  teamStore.loadTeam()
  
  battleStore.startBattle(getPlayerTeam.value, getEnemyTeam.value)
  // ...
})
```

### Verification Steps
1. Build team in Team Builder
2. Start a battle
3. Refresh the page (F5)
4. Team should persist with all members

---

## Development Setup

```bash
# Install dependencies (if not done)
npm install

# Run development server
npm run dev

# Type checking
npm run type-check

# Run tests
npm run test
```

---

## Key Files to Modify

### Phase 1: Team Persistence Fix (Priority)

| File | Change |
|------|--------|
| `src/components/BattleScreen.vue` | Add `teamStore.loadTeam()` call in `onMounted()` |
| `src/stores/team.ts` | Verify `loadTeam()` properly hydrates roster |

### Phase 2: Status Effects Foundation (Optional)

| File | Purpose |
|------|---------|
| `src/models/statStages.ts` | NEW - Stat stage types and constants |
| `src/models/statusCondition.ts` | NEW - Status condition types |
| `src/models/moveEffect.ts` | NEW - Move effect types |
| `src/services/battle/statStageService.ts` | NEW - Stat modification logic |
| `src/services/battle/statusConditionService.ts` | NEW - Status condition logic |
| `src/domain/battle/calc/damage.ts` | UPDATE - Apply stat stage multipliers |
| `src/data/moves.ts` | UPDATE - Add effect property to status moves |

---

## Testing Checklist

### Team Persistence
- [ ] Team persists on page refresh during battle
- [ ] Team persists on page refresh in Team Builder
- [ ] Empty team handled gracefully (redirect or message)
- [ ] localStorage key `pkmn-mmo-team` contains valid data

### Type Checking
```bash
npm run type-check
# Expected: No errors
```

---

## Architecture Notes

### Current Battle Component Structure
```
BattleScreen.vue (orchestrator)
├── BattleField.vue (visuals: sprites, HP bars, platforms)
├── BattleActionMenu.vue (controls: Fight/Bag/Pokemon/Run)
├── BagScreen.vue (items display)
├── MoveSelector.vue (move selection)
└── PokemonTeamSwitcher.vue (switch Pokemon)
```

### Store Flow
```
BattleScreen.onMounted()
  ↓
teamStore.loadTeam()      ← ADD THIS
  ↓
teamStore.roster          ← Now properly populated
  ↓
battleStore.startBattle(playerTeam, npcTeam)
```

---

## Common Issues

### "Team is empty after refresh"
- **Cause**: `loadTeam()` not called before accessing roster
- **Fix**: Add `teamStore.loadTeam()` at start of `onMounted()`

### TypeScript errors with union types in templates
- **Cause**: Inline type casting in Vue templates
- **Fix**: Use computed properties instead of inline casts

### Sprites not loading
- **Cause**: Pokemon name format mismatch
- **Fix**: Use `useSpriteLoader` composable with proper name normalization
