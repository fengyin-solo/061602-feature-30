<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import type { GameState } from '@/types/game'
import { WEATHER_NAMES } from '@/utils/constants'

const props = defineProps<{
  state: GameState
}>()

const weatherClass = computed(() => {
  switch (props.state.currentWeather) {
    case 'sunny': return 'bg-gradient-to-r from-amber-400/20 to-yellow-300/20'
    case 'rainy': return 'bg-gradient-to-r from-blue-500/20 to-gray-500/20'
    case 'snowy': return 'bg-gradient-to-r from-blue-200/30 to-white/20'
    case 'stormy': return 'bg-gradient-to-r from-purple-600/30 to-gray-700/30'
  }
})

const foodPulse = ref(false)
const foodChange = ref(0)
const showFoodChange = ref(false)

watch(
  () => props.state.foodStock,
  (newVal, oldVal) => {
    if (oldVal !== undefined && newVal > oldVal) {
      foodChange.value = newVal - oldVal
      foodPulse.value = true
      showFoodChange.value = true
      setTimeout(() => {
        foodPulse.value = false
      }, 400)
      setTimeout(() => {
        showFoodChange.value = false
      }, 1200)
    }
  }
)
</script>

<template>
  <div
    class="w-full glass rounded-2xl px-4 py-3 flex items-center justify-between gap-4"
    :class="weatherClass"
  >
    <div class="flex items-center gap-4">
      <div class="flex items-center gap-2 bg-black/30 px-3 py-1.5 rounded-xl">
        <span class="text-2xl">{{ WEATHER_NAMES[state.currentWeather].split(' ')[0] }}</span>
        <span class="text-white font-medium text-sm">{{ WEATHER_NAMES[state.currentWeather].split(' ')[1] }}</span>
      </div>

      <div class="flex items-center gap-2 bg-black/30 px-3 py-1.5 rounded-xl">
        <span class="text-lg">📅</span>
        <span class="text-white font-medium">第 {{ state.day }} 天</span>
        <div class="w-16 h-2 bg-black/40 rounded-full overflow-hidden ml-1">
          <div
            class="h-full bg-gradient-to-r from-orange-300 to-amber-400 transition-all"
            :style="{ width: `${state.dayProgress * 100}%` }"
          />
        </div>
      </div>
    </div>

    <div class="flex items-center gap-4">
      <div class="flex items-center gap-2 bg-black/30 px-3 py-1.5 rounded-xl relative overflow-hidden">
        <span class="text-xl" :class="{ 'animate-bounce': foodPulse }">🍒</span>
        <span
          class="text-white font-bold text-lg transition-all duration-200"
          :class="{ 'scale-125 text-yellow-300': foodPulse }"
        >
          {{ state.foodStock }}
        </span>
        <span class="text-white/70 text-xs">食物</span>
        <Transition name="food-change">
          <span
            v-if="showFoodChange && foodChange > 0"
            class="absolute -top-1 right-2 text-sm font-bold text-green-400 animate-food-pop"
          >
            +{{ foodChange }}
          </span>
        </Transition>
      </div>

      <div class="flex items-center gap-2 bg-black/30 px-3 py-1.5 rounded-xl">
        <span class="text-xl">🐦</span>
        <span class="text-white font-bold text-lg">
          {{ state.birds.filter(b => !b.isDead).length }}/{{ state.birds.length }}
        </span>
        <span class="text-white/70 text-xs">存活</span>
      </div>

      <div v-if="state.breedingCount > 0" class="flex items-center gap-2 bg-pink-500/40 px-3 py-1.5 rounded-xl">
        <span class="text-xl">💝</span>
        <span class="text-white font-bold">{{ state.breedingCount }}/{{ state.maxBreedingRounds }}</span>
      </div>
    </div>
  </div>
</template>
