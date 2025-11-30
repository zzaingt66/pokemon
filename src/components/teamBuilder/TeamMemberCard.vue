<!--
  TeamMemberCard Component
  Feature: 003-pokemon-team-builder
  Task: T032

  Displays a team member with Pokemon sprite, name, types, level, and moves
-->
<script setup lang="ts">
import type { TeamMember } from '@/models/teamBuilder'

interface Props {
  member: TeamMember
  isLead?: boolean
}

interface Emits {
  (e: 'remove', position: number): void
}

const props = withDefaults(defineProps<Props>(), {
  isLead: false,
})

const emit = defineEmits<Emits>()

/**
 * Get type color class for styling
 */
function getTypeColor(type: string): string {
  const typeColors: Record<string, string> = {
    Normal: 'bg-gray-400',
    Fire: 'bg-red-500',
    Water: 'bg-blue-500',
    Electric: 'bg-yellow-400',
    Grass: 'bg-green-500',
    Ice: 'bg-cyan-400',
    Fighting: 'bg-orange-700',
    Poison: 'bg-purple-500',
    Ground: 'bg-yellow-700',
    Flying: 'bg-indigo-400',
    Psychic: 'bg-pink-500',
    Bug: 'bg-lime-500',
    Rock: 'bg-yellow-800',
    Ghost: 'bg-purple-700',
    Dragon: 'bg-indigo-700',
    Dark: 'bg-gray-700',
    Steel: 'bg-gray-500',
    Fairy: 'bg-pink-400',
  }
  return typeColors[type] ?? 'bg-gray-400'
}

/**
 * Get category label
 */
function getCategoryLabel(category: string): string {
  const labels: Record<string, string> = {
    Physical: 'Phys',
    Special: 'Spec',
    Status: 'Stat',
  }
  return labels[category] ?? category
}

/**
 * Handle remove button click
 */
function handleRemove(): void {
  emit('remove', props.member.position)
}
</script>

<template>
  <div class="team-member-card bg-white rounded-lg shadow-md p-4 relative">
    <!-- Lead Badge -->
    <div v-if="isLead" class="lead-badge absolute top-2 right-2 bg-yellow-500 text-white px-2 py-1 rounded text-xs font-bold">
      ⭐ LEAD
    </div>

    <!-- Pokemon Header -->
    <div class="flex items-center gap-4 mb-3">
      <!-- Sprite -->
      <img
        :src="member.pokemon.sprite"
        :alt="member.pokemon.name"
        class="w-20 h-20 object-contain"
        loading="lazy"
      />

      <!-- Info -->
      <div class="flex-1">
        <h4 class="text-xl font-bold text-gray-900">{{ member.pokemon.name }}</h4>
        <div class="flex gap-1 mt-1">
          <span
            v-for="type in member.pokemon.types"
            :key="type"
            class="type-badge px-2 py-0.5 rounded text-white text-xs font-bold"
            :class="getTypeColor(type)"
          >
            {{ type }}
          </span>
        </div>
        <div class="text-sm text-gray-600 mt-1">Level {{ member.level }}</div>
      </div>
    </div>

    <!-- HP Bar -->
    <div class="hp-bar mb-3">
      <div class="flex justify-between text-xs text-gray-600 mb-1">
        <span>HP</span>
        <span>{{ member.currentHp }} / {{ member.maxHp }}</span>
      </div>
      <div class="h-2 bg-gray-200 rounded-full overflow-hidden">
        <div
          class="h-full bg-green-500 transition-all"
          :style="{ width: `${(member.currentHp / member.maxHp) * 100}%` }"
        ></div>
      </div>
    </div>

    <!-- Moves List -->
    <div class="moves-list mb-3">
      <div class="text-sm font-semibold text-gray-700 mb-1">Moves ({{ member.selectedMoves.length }})</div>
      <div class="space-y-1">
        <div
          v-for="move in member.selectedMoves"
          :key="move.id"
          class="move-item flex items-center justify-between text-xs bg-gray-50 rounded px-2 py-1"
        >
          <div class="flex items-center gap-2">
            <span class="font-semibold">{{ move.name }}</span>
            <span
              class="type-badge px-1.5 py-0.5 rounded text-white text-xs font-bold"
              :class="getTypeColor(move.type)"
            >
              {{ move.type }}
            </span>
          </div>
          <div class="flex items-center gap-2 text-gray-600">
            <span class="category-label text-xs font-medium">{{ getCategoryLabel(move.category) }}</span>
            <span v-if="move.power">{{ move.power }}</span>
            <span v-else>—</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Stats Preview -->
    <div class="stats-preview grid grid-cols-3 gap-2 text-xs mb-3">
      <div class="stat text-center">
        <div class="text-gray-600">ATK</div>
        <div class="font-bold">{{ member.pokemon.stats.attack }}</div>
      </div>
      <div class="stat text-center">
        <div class="text-gray-600">DEF</div>
        <div class="font-bold">{{ member.pokemon.stats.defense }}</div>
      </div>
      <div class="stat text-center">
        <div class="text-gray-600">SPD</div>
        <div class="font-bold">{{ member.pokemon.stats.speed }}</div>
      </div>
    </div>

    <!-- Remove Button -->
    <button
      class="remove-button w-full px-3 py-2 bg-red-500 text-white rounded font-semibold hover:bg-red-600 transition-colors"
      :aria-label="`Remove ${member.pokemon.name} from team`"
      @click="handleRemove"
    >
      Remove from Team
    </button>
  </div>
</template>

<style scoped>
.team-member-card {
  transition: transform 0.2s, box-shadow 0.2s;
}

.team-member-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.lead-badge {
  z-index: 10;
}
</style>
