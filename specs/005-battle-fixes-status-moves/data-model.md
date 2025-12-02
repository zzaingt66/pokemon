# Data Model: Battle Fixes & Status Moves

**Feature**: 005-battle-fixes-status-moves  
**Date**: 2025-12-02

---

## Entities

### 1. Pokemon (Extended)

**Current State**:
```typescript
interface Pokemon {
  id: string
  name: string
  types: string[]
  level: number
  stats: {
    hp: number
    atk: number
    def: number
    spAtk: number
    spDef: number
    speed: number
  }
  currentHp: number
  moves: Move[]
}
```

**Extended State** (for status effects):
```typescript
interface Pokemon {
  // ... existing fields ...
  
  // NEW: Stat stage modifiers (-6 to +6)
  statStages?: {
    atk: number
    def: number
    spAtk: number
    spDef: number
    speed: number
    accuracy: number
    evasion: number
  }
  
  // NEW: Active status condition
  statusCondition?: StatusCondition | null
}

type StatusCondition = 'paralysis' | 'sleep' | 'poison' | 'burn' | 'freeze' | 'badly-poisoned'
```

### 2. Move (Extended)

**Current State**:
```typescript
interface Move {
  id: string
  name: string
  type: string
  power: number
  accuracy: number
  category: 'physical' | 'special'
  pp?: number
}
```

**Extended State**:
```typescript
interface Move {
  // ... existing fields ...
  
  // NEW: Effect for status moves
  effect?: MoveEffect
}

interface MoveEffect {
  type: 'stat-change' | 'status-condition' | 'healing'
  target: 'self' | 'opponent'
  
  // For stat changes
  stat?: 'atk' | 'def' | 'spAtk' | 'spDef' | 'speed' | 'accuracy' | 'evasion'
  stages?: number // -6 to +6
  
  // For status conditions
  condition?: StatusCondition
  chance?: number // 0-100 probability
}
```

### 3. BattleState (Extended)

**Current State** (in `battle.ts` store):
```typescript
interface BattleState {
  turn: number
  phase: 'select' | 'execute' | 'ended'
  player: Pokemon
  npc: Pokemon
  playerTeam: Pokemon[]
  npcTeam: Pokemon[]
  currentPlayerIndex: number
  currentNpcIndex: number
  winner: 'player' | 'npc' | null
  log: string[]
}
```

**No changes required** - stat stages and conditions are stored in Pokemon entities.

---

## Relationships

```
BattleState
├── player: Pokemon (has statStages, statusCondition)
├── npc: Pokemon (has statStages, statusCondition)
├── playerTeam: Pokemon[]
└── npcTeam: Pokemon[]

Pokemon
├── moves: Move[]
│   └── effect?: MoveEffect
├── statStages?: StatStages
└── statusCondition?: StatusCondition
```

---

## Validation Rules

### Pokemon Stat Stages
- Each stage value MUST be between -6 and +6
- Default value is 0 for all stats
- Stages reset to 0 when Pokemon is switched out

### Status Conditions
- Only ONE status condition can be active at a time
- Burn halves physical attack damage
- Paralysis reduces speed by 50% and has 25% chance to skip turn
- Sleep prevents action for 1-3 turns
- Freeze prevents action until thawed (20% chance per turn)
- Poison deals 1/8 max HP per turn end
- Badly-poisoned deals increasing damage each turn

### Move Effects
- `stages` must be between -6 and +6
- `chance` must be between 0 and 100 (default 100)
- If `type` is 'stat-change', `stat` and `stages` are required
- If `type` is 'status-condition', `condition` is required

---

## State Transitions

### Stat Stage Application
```
1. Move selected with stat-change effect
2. Accuracy check passes
3. Calculate new stage = current + effect.stages
4. Clamp result to [-6, +6]
5. Update pokemon.statStages[stat]
6. Log message: "{Pokemon}'s {stat} {fell/rose}!"
```

### Status Condition Application
```
1. Move selected with status-condition effect
2. Accuracy check passes
3. Check if target already has condition → fail if yes
4. Check type immunity (Electric immune to Paralysis, etc.)
5. Apply condition: pokemon.statusCondition = effect.condition
6. Log message: "{Pokemon} was paralyzed/burned/etc!"
```

### Battle Turn with Status
```
1. Turn start
2. Check for status preventing action (Sleep, Freeze, Paralysis skip)
3. If can act → execute move
4. Apply move effect (damage OR status)
5. Turn end
6. Apply end-of-turn effects (Poison damage, Burn damage)
7. Check for faint
```
