<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import MoveSelector from './MoveSelector.vue'
import MyTeam from './MyTeam.vue'
import LogPanel from './LogPanel.vue'
import StatusPanel from './StatusPanel.vue'

// Interfaces
interface Move {
  id: number
  name: string
  type: string
  power: number
  pp: number
  maxPp: number
  accuracy?: number
}

interface Pokemon {
  id: number
  name: string
  level: number
  currentHp: number
  maxHp: number
  currentExp?: number
  maxExp?: number
  sprite: string
  backSprite?: string
  gender?: 'male' | 'female'
  moves: Move[]
  status?: string
}

// Estado de la batalla
const currentView = ref<'main' | 'fight' | 'bag' | 'pokemon'>('main')
const battleLog = ref<string[]>(['¡La batalla ha comenzado!'])

// Pokémon del jugador
const playerPokemon = ref<Pokemon>({
  id: 6,
  name: 'CHARIZARD',
  level: 36,
  currentHp: 120,
  maxHp: 150,
  currentExp: 2500,
  maxExp: 3500,
  sprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/6.png',
  backSprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/back/6.png',
  gender: 'male',
  moves: [
    { id: 1, name: 'Flamethrower', type: 'fire', power: 90, pp: 15, maxPp: 15, accuracy: 100 },
    { id: 2, name: 'Air Slash', type: 'flying', power: 75, pp: 15, maxPp: 15, accuracy: 95 },
    { id: 3, name: 'Dragon Claw', type: 'dragon', power: 80, pp: 15, maxPp: 15, accuracy: 100 },
    { id: 4, name: 'Fire Spin', type: 'fire', power: 35, pp: 15, maxPp: 15, accuracy: 85 }
  ]
})

// Pokémon enemigo
const enemyPokemon = ref<Pokemon>({
  id: 130,
  name: 'GYARADOS',
  level: 35,
  currentHp: 140,
  maxHp: 140,
  sprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/130.png',
  gender: 'male',
  moves: []
})

// Equipo del jugador (6 Pokémon)
const playerTeam = ref<Pokemon[]>([
  playerPokemon.value,
  {
    id: 25,
    name: 'PIKACHU',
    level: 32,
    currentHp: 70,
    maxHp: 70,
    sprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/25.png',
    backSprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/back/25.png',
    gender: 'male',
    moves: []
  },
  {
    id: 3,
    name: 'VENUSAUR',
    level: 35,
    currentHp: 135,
    maxHp: 135,
    sprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/3.png',
    backSprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/back/3.png',
    gender: 'female',
    moves: []
  },
  {
    id: 9,
    name: 'BLASTOISE',
    level: 36,
    currentHp: 142,
    maxHp: 142,
    sprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/9.png',
    backSprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/back/9.png',
    gender: 'male',
    moves: []
  },
  {
    id: 131,
    name: 'LAPRAS',
    level: 34,
    currentHp: 125,
    maxHp: 125,
    sprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/131.png',
    backSprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/back/131.png',
    gender: 'female',
    moves: []
  },
  {
    id: 143,
    name: 'SNORLAX',
    level: 38,
    currentHp: 0,
    maxHp: 180,
    sprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/143.png',
    backSprite: 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-iii/firered-leafgreen/back/143.png',
    gender: 'male',
    status: 'fainted',
    moves: []
  }
])

// Computed
const playerHpPercent = computed(() =>
  (playerPokemon.value.currentHp / playerPokemon.value.maxHp) * 100
)

const enemyHpPercent = computed(() =>
  (enemyPokemon.value.currentHp / enemyPokemon.value.maxHp) * 100
)

const playerExpPercent = computed(() => {
  if (!playerPokemon.value.currentExp || !playerPokemon.value.maxExp) return 0
  return (playerPokemon.value.currentExp / playerPokemon.value.maxExp) * 100
})

const getHpColor = (percent: number) => {
  if (percent > 50) return '#10b981'
  if (percent > 25) return '#fbbf24'
  return '#ef4444'
}

// Handlers
const handleFight = () => {
  currentView.value = 'fight'
  addLog(`¿Qué movimiento usará ${playerPokemon.value.name}?`)
}

const handleBag = () => {
  currentView.value = 'bag'
  addLog('Selecciona un objeto de la mochila.')
}

const handlePokemon = () => {
  currentView.value = 'pokemon'
  addLog('¿Qué Pokémon enviarás?')
}

const handleRun = () => {
  const canRun = Math.random() > 0.5
  if (canRun) {
    addLog('¡Has huido con éxito de la batalla!')
    setTimeout(() => {
      alert('¡Escapaste de la batalla!')
    }, 800)
  } else {
    addLog('¡No puedes escapar!')
    setTimeout(() => enemyAttack(), 1000)
  }
}

// CORREGIDO: Ahora SÍ hace daño
const handleMoveSelected = (moveIndex: number) => {
  console.log('=== ATAQUE INICIADO ===')
  console.log('Índice:', moveIndex)
  console.log('Moves disponibles:', playerPokemon.value.moves)

  const move = playerPokemon.value.moves[moveIndex]

  if (!move) {
    console.error('❌ Movimiento no encontrado en índice:', moveIndex)
    addLog('Error: Movimiento no válido')
    return
  }

  console.log('✅ Movimiento encontrado:', move.name)

  if (move.pp <= 0) {
    addLog('¡No quedan PP para este movimiento!')
    return
  }

  // Reducir PP
  move.pp--
  console.log('PP restantes:', move.pp)

  // CALCULAR DAÑO
  const baseDamage = move.power
  const randomFactor = 0.85 + Math.random() * 0.3
  const damage = Math.max(1, Math.floor(baseDamage * randomFactor))

  console.log('💥 Daño calculado:', damage)
  console.log('HP enemigo ANTES:', enemyPokemon.value.currentHp)

  // APLICAR DAÑO AL ENEMIGO
  enemyPokemon.value.currentHp = Math.max(0, enemyPokemon.value.currentHp - damage)

  console.log('HP enemigo DESPUÉS:', enemyPokemon.value.currentHp)

  addLog(`¡${playerPokemon.value.name} usó ${move.name}!`)
  addLog(`¡Causó ${damage} puntos de daño!`)

  if (enemyPokemon.value.currentHp === 0) {
    addLog(`¡${enemyPokemon.value.name} enemigo se debilitó!`)
    addLog('¡Ganaste la batalla!')

    if (playerPokemon.value.currentExp !== undefined && playerPokemon.value.maxExp !== undefined) {
      const expGained = 250
      playerPokemon.value.currentExp = Math.min(
        playerPokemon.value.maxExp,
        playerPokemon.value.currentExp + expGained
      )
      addLog(`¡${playerPokemon.value.name} ganó ${expGained} puntos de EXP!`)
    }
  } else {
    setTimeout(() => enemyAttack(), 1500)
  }

  currentView.value = 'main'
}

// CORREGIDO: Cambio de Pokémon
const handleSwitchPokemon = (pokemonIndex: number) => {
  console.log('=== CAMBIO DE POKÉMON ===')
  console.log('Índice seleccionado:', pokemonIndex)

  const newPokemon = playerTeam.value[pokemonIndex]

  if (!newPokemon) {
    console.error('❌ Pokémon no encontrado')
    return
  }

  console.log('Pokémon seleccionado:', newPokemon.name)

  if (newPokemon.currentHp === 0) {
    addLog(`¡${newPokemon.name} está debilitado!`)
    return
  }

  if (newPokemon.id === playerPokemon.value.id) {
    addLog(`¡${newPokemon.name} ya está en combate!`)
    return
  }

  const oldPokemonName = playerPokemon.value.name

  // Asignar el nuevo Pokémon
  playerPokemon.value = newPokemon

  addLog(`¡Vuelve, ${oldPokemonName}!`)
  addLog(`¡Adelante, ${newPokemon.name}!`)

  currentView.value = 'main'
  setTimeout(() => enemyAttack(), 1500)
}

const enemyAttack = () => {
  if (enemyPokemon.value.currentHp === 0) return

  const damage = Math.floor(Math.random() * 20) + 15
  playerPokemon.value.currentHp = Math.max(0, playerPokemon.value.currentHp - damage)

  addLog(`¡${enemyPokemon.value.name} enemigo atacó!`)
  addLog(`¡Causó ${damage} puntos de daño!`)

  if (playerPokemon.value.currentHp === 0) {
    addLog(`¡${playerPokemon.value.name} se debilitó!`)

    const hasOtherPokemon = playerTeam.value.some(p => p.currentHp > 0 && p.id !== playerPokemon.value.id)
    if (hasOtherPokemon) {
      currentView.value = 'pokemon'
      addLog('¡Elige otro Pokémon!')
    } else {
      addLog('¡Perdiste la batalla!')
    }
  }
}

const handleBack = () => {
  currentView.value = 'main'
  addLog(`¿Qué hará ${playerPokemon.value.name}?`)
}

const addLog = (message: string) => {
  battleLog.value.push(message)
  if (battleLog.value.length > 50) {
    battleLog.value.shift()
  }
}

onMounted(() => {
  addLog(`¡Un ${enemyPokemon.value.name} salvaje apareció!`)
  addLog(`¡Adelante, ${playerPokemon.value.name}!`)
})
</script>

<template>
  <div class="battle-container">
    <div class="battle-screen">
      <!-- Campo de batalla -->
      <div class="battlefield">
        <!-- Información del Pokémon enemigo -->
        <div class="enemy-info-panel">
          <div class="info-box enemy-box">
            <div class="name-level-row">
              <span class="pokemon-name">{{ enemyPokemon.name }}</span>
              <span v-if="enemyPokemon.gender" class="gender-icon" :class="enemyPokemon.gender">
                {{ enemyPokemon.gender === 'male' ? '♂' : '♀' }}
              </span>
            </div>
            <div class="level-row">:L{{ enemyPokemon.level }}</div>
            <div class="hp-display">
              <span class="hp-text">HP</span>
              <div class="hp-bar-outer">
                <div
                  class="hp-bar-inner"
                  :style="{
                    width: enemyHpPercent + '%',
                    backgroundColor: getHpColor(enemyHpPercent)
                  }"
                />
              </div>
            </div>
          </div>
        </div>

        <!-- Sprite enemigo -->
        <div class="enemy-sprite-area">
          <div class="sprite-platform enemy-platform"></div>
          <img
            :src="enemyPokemon.sprite"
            :alt="enemyPokemon.name"
            class="pokemon-sprite enemy-sprite"
          >
        </div>

        <!-- Sprite jugador -->
        <div class="player-sprite-area">
          <div class="sprite-platform player-platform"></div>
          <img
            :src="playerPokemon.backSprite || playerPokemon.sprite"
            :alt="playerPokemon.name"
            class="pokemon-sprite player-sprite"
          >
        </div>

        <!-- Información del Pokémon jugador -->
        <div class="player-info-panel">
          <div class="info-box player-box">
            <div class="name-level-row">
              <span class="pokemon-name">{{ playerPokemon.name }}</span>
              <span v-if="playerPokemon.gender" class="gender-icon" :class="playerPokemon.gender">
                {{ playerPokemon.gender === 'male' ? '♂' : '♀' }}
              </span>
            </div>
            <div class="level-row">:L{{ playerPokemon.level }}</div>
            <div class="hp-display">
              <span class="hp-text">HP</span>
              <div class="hp-bar-outer">
                <div
                  class="hp-bar-inner"
                  :style="{
                    width: playerHpPercent + '%',
                    backgroundColor: getHpColor(playerHpPercent)
                  }"
                />
              </div>
            </div>
            <div class="hp-numbers">{{ playerPokemon.currentHp }} / {{ playerPokemon.maxHp }}</div>
            <div class="exp-display">
              <div class="exp-bar-outer">
                <div class="exp-bar-inner" :style="{ width: playerExpPercent + '%' }" />
              </div>
            </div>
          </div>
        </div>

        <!-- StatusPanel (componente enlazado) -->
        <StatusPanel
          v-if="playerPokemon && enemyPokemon"
          :player-pokemon="playerPokemon"
          :enemy-pokemon="enemyPokemon"
          class="status-overlay"
        />
      </div>

      <!-- Panel de control - AGRANDADO -->
      <div class="control-area">
        <!-- Vista principal -->
        <template v-if="currentView === 'main'">
          <!-- LogPanel enlazado CON SCROLL -->
          <LogPanel
            :messages="battleLog"
            :max-messages="10"
            :is-battle-style="true"
          />

          <div class="action-panel">
            <button class="action-btn" @click="handleFight">
              <span class="action-text">FIGHT</span>
            </button>
            <button class="action-btn" @click="handleBag">
              <span class="action-text">BAG</span>
            </button>
            <button class="action-btn" @click="handlePokemon">
              <span class="action-text">POKéMON</span>
            </button>
            <button class="action-btn" @click="handleRun">
              <span class="action-text">RUN</span>
            </button>
          </div>
        </template>

        <!-- MoveSelector enlazado -->
        <MoveSelector
          v-else-if="currentView === 'fight'"
          :moves="playerPokemon.moves"
          :is-battle-style="true"
          @select-move="handleMoveSelected"
          @back="handleBack"
        />

        <!-- Vista de mochila -->
        <div v-else-if="currentView === 'bag'" class="bag-panel">
          <div class="bag-header">MOCHILA</div>
          <div class="bag-items">
            <div class="bag-item">
              <img
                src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/items/potion.png"
                alt="Potion"
                class="item-sprite"
              />
              <span class="item-label">Poción</span>
              <span class="item-qty">x3</span>
            </div>
            <div class="bag-item">
              <img
                src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/items/poke-ball.png"
                alt="Pokeball"
                class="item-sprite"
              />
              <span class="item-label">Poké Ball</span>
              <span class="item-qty">x5</span>
            </div>
            <div class="bag-item disabled">
              <img
                src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/items/super-potion.png"
                alt="Super Potion"
                class="item-sprite"
              />
              <span class="item-label">Superpoción</span>
              <span class="item-qty">x0</span>
            </div>
            <div class="bag-item disabled">
              <img
                src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/items/revive.png"
                alt="Revive"
                class="item-sprite"
              />
              <span class="item-label">Revivir</span>
              <span class="item-qty">x0</span>
            </div>
          </div>
          <button class="back-button" @click="handleBack">
            <span class="back-arrow">←</span> ATRÁS
          </button>
        </div>

        <!-- MyTeam enlazado -->
        <MyTeam
          v-else-if="currentView === 'pokemon'"
          :team="playerTeam"
          :current-pokemon-id="playerPokemon.id"
          @switch-pokemon="handleSwitchPokemon"
          @back="handleBack"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.battle-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, oklch(var(--color-background)) 0%, oklch(var(--color-muted)) 100%);
  padding: 20px;
}

/* AGRANDADO: Mayor tamaño */
.battle-screen {
  width: 720px;
  height: 480px;
  display: flex;
  flex-direction: column;
  background: oklch(var(--color-card));
  border: 4px solid oklch(var(--color-border));
  border-radius: 8px;
  overflow: hidden;
  box-shadow:
    0 8px 24px rgba(0, 0, 0, 0.15),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  font-family: 'Press Start 2P', 'Courier New', monospace;
}

.battlefield {
  flex: 1;
  position: relative;
  background: linear-gradient(
    to bottom,
    #87CEEB 0%,
    #87CEEB 40%,
    #90EE90 40%,
    #90EE90 55%,
    #8B7355 55%,
    #8B7355 100%
  );
  overflow: hidden;
}

.sprite-platform {
  position: absolute;
  border-radius: 50%;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.enemy-platform {
  width: 80px;
  height: 20px;
  background: radial-gradient(ellipse, rgba(0, 0, 0, 0.2), transparent);
  top: 42%;
  right: 22%;
}

.player-platform {
  width: 90px;
  height: 25px;
  background: radial-gradient(ellipse, rgba(0, 0, 0, 0.25), transparent);
  bottom: 22%;
  left: 12%;
}

.pokemon-sprite {
  position: absolute;
  image-rendering: pixelated;
  animation: float 3s ease-in-out infinite;
}

.enemy-sprite {
  width: 100px;
  height: 100px;
  top: 18%;
  right: 18%;
  animation-delay: 0s;
}

.player-sprite {
  width: 120px;
  height: 120px;
  bottom: 16%;
  left: 8%;
  animation-delay: 1.5s;
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-8px); }
}

.info-box {
  background: rgba(255, 255, 255, 0.95);
  border: 2px solid #000;
  border-radius: 8px;
  padding: 8px 12px;
  box-shadow: 2px 2px 0 rgba(0, 0, 0, 0.3);
}

.enemy-info-panel {
  position: absolute;
  top: 8%;
  left: 6%;
}

.enemy-box {
  min-width: 180px;
}

.player-info-panel {
  position: absolute;
  bottom: 8%;
  right: 6%;
}

.player-box {
  min-width: 200px;
}

.name-level-row {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-bottom: 3px;
}

.pokemon-name {
  font-size: 11px;
  font-weight: bold;
  color: #2d2d2d;
  letter-spacing: 0.5px;
}

.gender-icon {
  font-size: 13px;
  font-weight: bold;
}

.gender-icon.male { color: #3b82f6; }
.gender-icon.female { color: #ec4899; }

.level-row {
  font-size: 9px;
  color: #666;
  margin-bottom: 5px;
}

.hp-display {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-bottom: 3px;
}

.hp-text {
  font-size: 8px;
  color: #f59e0b;
  font-weight: bold;
}

.hp-bar-outer {
  flex: 1;
  height: 7px;
  background: #2d2d2d;
  border-radius: 3px;
  border: 1px solid #000;
  overflow: hidden;
}

.hp-bar-inner {
  height: 100%;
  transition: width 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  border-radius: 2px;
}

.hp-numbers {
  font-size: 10px;
  color: #2d2d2d;
  text-align: right;
  margin-top: 2px;
}

.exp-display {
  margin-top: 4px;
}

.exp-bar-outer {
  width: 100%;
  height: 5px;
  background: #2d2d2d;
  border-radius: 2px;
  border: 1px solid #000;
  overflow: hidden;
}

.exp-bar-inner {
  height: 100%;
  background: linear-gradient(to right, #3b82f6, #60a5fa);
  transition: width 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.status-overlay {
  opacity: 0;
  pointer-events: none;
}

/* AGRANDADO: Más altura para el panel */
.control-area {
  height: 150px;
  background: oklch(var(--color-muted));
  border-top: 3px solid oklch(var(--color-border));
  display: grid;
  grid-template-columns: 1.4fr 1fr;
  gap: 8px;
  padding: 8px;
}

.action-panel {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

.action-btn {
  background: oklch(var(--color-card));
  border: 3px solid oklch(var(--color-border));
  border-radius: 6px;
  padding: 16px;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.1s ease;
  box-shadow: 3px 3px 0 rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
}

.action-btn:hover {
  background: oklch(var(--color-accent));
  transform: translate(1px, 1px);
  box-shadow: 2px 2px 0 rgba(0, 0, 0, 0.2);
}

.action-btn:active {
  transform: translate(3px, 3px);
  box-shadow: 0 0 0 rgba(0, 0, 0, 0.2);
}

.action-text {
  font-size: 10px;
  font-weight: bold;
  color: oklch(var(--color-foreground));
  letter-spacing: 0.5px;
}

.bag-panel {
  grid-column: 1 / -1;
  background: oklch(var(--color-card));
  border: 3px solid oklch(var(--color-border));
  border-radius: 6px;
  padding: 10px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.bag-header {
  font-size: 11px;
  font-weight: bold;
  color: oklch(var(--color-foreground));
  text-align: center;
  padding-bottom: 6px;
  border-bottom: 2px solid oklch(var(--color-border));
}

.bag-items {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 6px;
  flex: 1;
}

.bag-item {
  background: oklch(var(--color-muted));
  border: 2px solid oklch(var(--color-border));
  border-radius: 4px;
  padding: 6px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
  cursor: pointer;
  transition: all 0.1s ease;
}

.bag-item:hover:not(.disabled) {
  background: oklch(var(--color-accent));
  transform: scale(1.05);
}

.bag-item.disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.item-sprite {
  width: 28px;
  height: 28px;
  image-rendering: pixelated;
}

.item-label {
  font-size: 7px;
  text-align: center;
  color: oklch(var(--color-foreground));
  line-height: 1.2;
}

.item-qty {
  font-size: 6px;
  color: oklch(var(--color-muted-foreground));
  font-weight: bold;
}

.back-button {
  background: oklch(var(--color-card));
  border: 2px solid oklch(var(--color-border));
  border-radius: 4px;
  padding: 8px 12px;
  font-size: 9px;
  font-weight: bold;
  color: oklch(var(--color-foreground));
  cursor: pointer;
  align-self: flex-start;
  font-family: inherit;
  display: flex;
  align-items: center;
  gap: 5px;
  transition: all 0.1s ease;
}

.back-button:hover {
  background: oklch(var(--color-accent));
}

.back-arrow {
  font-size: 11px;
}

@media (max-width: 800px) {
  .battle-screen {
    width: 95vw;
height: auto;
aspect-ratio: 3 / 2;
}
}
@media (min-width: 1200px) {
.battle-screen {
width: 960px;
height: 640px;
}
.control-area {
height: 200px;
}
}

</style>

//Corrige el código:3
//Gracias
