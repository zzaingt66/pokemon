# API Contracts: Battle Fixes & Status Moves

**Feature**: 005-battle-fixes-status-moves
**Date**: 2025-12-02

---

## Internal Contracts (TypeScript Interfaces)

### StatStages Interface

```typescript
// src/models/statStages.ts

export interface StatStages {
  atk: number
  def: number
  spAtk: number
  spDef: number
  speed: number
  accuracy: number
  evasion: number
}

export const DEFAULT_STAT_STAGES: StatStages = {
  atk: 0,
  def: 0,
  spAtk: 0,
  spDef: 0,
  speed: 0,
  accuracy: 0,
  evasion: 0
}

/**
 * Stat stage multipliers based on stage value
 * Stage -6 = 2/8, -5 = 2/7, ..., 0 = 2/2, ..., +5 = 7/2, +6 = 8/2
 */
export const STAGE_MULTIPLIERS: Record<number, number> = {
  '-6': 2 / 8,
  '-5': 2 / 7,
  '-4': 2 / 6,
  '-3': 2 / 5,
  '-2': 2 / 4,
  '-1': 2 / 3,
  '0': 1,
  '1': 3 / 2,
  '2': 4 / 2,
  '3': 5 / 2,
  '4': 6 / 2,
  '5': 7 / 2,
  '6': 8 / 2
}
```

### StatusCondition Interface

```typescript
// src/models/statusCondition.ts

export type StatusCondition =
  | 'paralysis'
  | 'sleep'
  | 'poison'
  | 'burn'
  | 'freeze'
  | 'badly-poisoned'

export interface StatusConditionState {
  condition: StatusCondition
  turnsRemaining?: number // For sleep
  poisonCounter?: number  // For badly-poisoned (increases each turn)
}
```

### MoveEffect Interface

```typescript
// src/models/moveEffect.ts

import type { StatusCondition } from './statusCondition'

export type MoveEffectType = 'stat-change' | 'status-condition' | 'healing'
export type EffectTarget = 'self' | 'opponent'
export type AffectedStat = 'atk' | 'def' | 'spAtk' | 'spDef' | 'speed' | 'accuracy' | 'evasion'

export interface MoveEffect {
  type: MoveEffectType
  target: EffectTarget

  // Stat change effects
  stat?: AffectedStat
  stages?: number

  // Status condition effects
  condition?: StatusCondition
  chance?: number // 0-100, default 100
}
```

### Extended Move Interface

```typescript
// src/models/move.ts (updated)

import type { MoveEffect } from './moveEffect'

export interface Move {
  id: string
  name: string
  type: string
  power: number
  accuracy: number
  category: 'physical' | 'special' | 'status'
  pp?: number
  currentPp?: number
  effect?: MoveEffect
}
```

### Extended Pokemon Interface

```typescript
// src/models/pokemon.ts (updated)

import type { StatStages, DEFAULT_STAT_STAGES } from './statStages'
import type { StatusConditionState } from './statusCondition'
import type { Move } from './move'

export interface Pokemon {
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

  // Battle-only state (optional, not persisted)
  statStages?: StatStages
  statusCondition?: StatusConditionState | null
}

// Factory function
export function initBattlePokemon(pokemon: Pokemon): Pokemon {
  return {
    ...pokemon,
    statStages: { ...DEFAULT_STAT_STAGES },
    statusCondition: null
  }
}
```

---

## Service Contracts

### StatStageService

```typescript
// src/services/battle/statStageService.ts

import type { Pokemon } from '@/models/pokemon'
import type { AffectedStat, MoveEffect } from '@/models/moveEffect'

export interface StatStageService {
  /**
   * Apply a stat stage change to a Pokemon
   * @returns Object with success status and battle log message
   */
  applyStatChange(
    pokemon: Pokemon,
    stat: AffectedStat,
    stages: number
  ): { success: boolean; message: string }

  /**
   * Get the effective stat value after applying stage multipliers
   */
  getEffectiveStat(pokemon: Pokemon, stat: AffectedStat): number

  /**
   * Reset all stat stages to 0 (used when switching out)
   */
  resetStages(pokemon: Pokemon): void
}
```

### StatusConditionService

```typescript
// src/services/battle/statusConditionService.ts

import type { Pokemon } from '@/models/pokemon'
import type { StatusCondition } from '@/models/statusCondition'

export interface StatusConditionService {
  /**
   * Apply a status condition to a Pokemon
   * @returns Object with success status and battle log message
   */
  applyStatus(
    pokemon: Pokemon,
    condition: StatusCondition
  ): { success: boolean; message: string }

  /**
   * Check if a Pokemon can act this turn (considering paralysis, sleep, freeze)
   * @returns Object with canAct boolean and optional skip message
   */
  checkCanAct(pokemon: Pokemon): { canAct: boolean; message?: string }

  /**
   * Apply end-of-turn effects (poison damage, burn damage)
   * @returns Damage dealt (if any) and battle log message
   */
  applyEndOfTurnEffects(pokemon: Pokemon): { damage: number; message?: string }

  /**
   * Check if a Pokemon is immune to a status based on its type
   */
  isImmuneTo(pokemon: Pokemon, condition: StatusCondition): boolean

  /**
   * Clear status condition (e.g., when healed)
   */
  clearStatus(pokemon: Pokemon): void
}
```

### TeamPersistenceContract

```typescript
// src/stores/team.ts (loadTeam method contract)

export interface TeamStore {
  /**
   * Load team from localStorage
   * MUST be called before accessing roster in any component
   * @returns true if team was loaded, false if no saved team exists
   */
  loadTeam(): boolean

  /**
   * Save current roster to localStorage
   */
  saveTeam(): void

  /**
   * Current team roster
   */
  roster: Pokemon[]
}
```

---

## Battle Engine Contract Updates

```typescript
// src/domain/battle/engine/battleEngine.ts (interface only)

export interface BattleEngine {
  /**
   * Execute a turn with status move support
   * @param playerMove The move selected by player
   * @param npcMove The move selected by NPC AI
   * @returns Array of battle events/messages
   */
  executeTurn(playerMove: Move, npcMove: Move): BattleEvent[]

  /**
   * Initialize battle with proper stat stages and status reset
   */
  initBattle(playerTeam: Pokemon[], npcTeam: Pokemon[]): void
}

export interface BattleEvent {
  type: 'damage' | 'status-applied' | 'stat-changed' | 'faint' | 'turn-skipped' | 'end-of-turn'
  target: 'player' | 'npc'
  message: string
  value?: number
}
```
