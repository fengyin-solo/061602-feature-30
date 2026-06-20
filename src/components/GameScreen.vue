<script setup lang="ts">
import { watch, onMounted, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useGameState } from '@/composables/useGameState'
import StatusBar from './StatusBar.vue'
import NestScene from './NestScene.vue'
import WeatherOverlay from './WeatherOverlay.vue'
import BirdCard from './BirdCard.vue'
import EventModal from './EventModal.vue'
import { WEATHER_COLORS, COMBO_MULTIPLIERS } from '@/utils/constants'

const router = useRouter()
const {
  state, allAdults, aliveCount,
  collectBerry, feedBird, calmBird, buryBird,
  releaseBirds, keepAndBreed, returnToStart, tryLoadGame,
} = useGameState()

const comboText = computed(() => {
  const count = state.combo.count
  if (count >= 7) return '🔥 超级连击!'
  if (count >= 5) return '⚡ 完美连击!'
  if (count >= 3) return '✨ 连击中!'
  return ''
})

const comboMultiplierText = computed(() => {
  const mult = state.combo.multiplier
  if (mult > 1) return `×${mult.toFixed(1)} 倍率`
  return ''
})

const showCombo = computed(() => {
  return state.combo.count >= 2 && Date.now() - state.combo.lastCollectAt < 1200
})

onMounted(() => {
  if (state.phase === 'start') {
    tryLoadGame()
    if (state.phase === 'start') {
      router.push('/')
    }
  }
})

watch(
  () => state.phase,
  (phase) => {
    if (phase === 'ended') {
      setTimeout(() => router.push('/end'), 1000)
    }
  }
)

const handleSelectBird = (id: string) => {
  state.selectedBirdId = state.selectedBirdId === id ? undefined : id
}

const handleCollect = (id: string) => {
  const val = collectBerry(id)
  if (val > 0) {
  }
}
</script>

<template>
  <div
    class="w-full h-full bg-gradient-to-br from-forest-dark via-forest to-forest-light relative overflow-hidden"
  >
    <div
      :class="['absolute inset-0 transition-colors duration-1000 bg-gradient-to-b', WEATHER_COLORS[state.currentWeather]]"
    />
    <WeatherOverlay :weather="state.currentWeather" />

    <div class="relative z-20 w-full h-full flex flex-col p-4 gap-4 max-w-[1400px] mx-auto">
      <StatusBar :state="state" />

      <div class="flex-1 min-h-0 grid grid-cols-1 lg:grid-cols-12 gap-4">
        <div class="lg:col-span-3 order-2 lg:order-1 min-h-0 flex flex-col gap-3 overflow-y-auto scrollbar-hide">
          <div class="font-display text-lg text-amber-300 flex items-center gap-2 px-1">
            <span>🐦</span> 小鸟档案
          </div>
          <div class="flex flex-col gap-3 pb-4">
            <BirdCard
              v-for="bird in state.birds"
              :key="bird.id"
              :bird="bird"
              :selected="state.selectedBirdId === bird.id"
              :food-stock="state.foodStock"
              @select="handleSelectBird(bird.id)"
              @feed="((amt: number) => feedBird(bird.id, amt))"
              @calm="calmBird(bird.id)"
              @bury="buryBird(bird.id)"
            />
          </div>
        </div>

        <div class="lg:col-span-6 order-1 lg:order-2 relative rounded-3xl overflow-hidden bg-black/20 border border-white/10 backdrop-blur-sm min-h-[400px]">
          <NestScene
            :birds="state.birds"
            :berries="state.berries"
            :selected-bird-id="state.selectedBirdId"
            :combo="state.combo"
            :collect-feedbacks="state.collectFeedbacks"
            @select-bird="handleSelectBird"
            @collect-berry="handleCollect"
          />

          <Transition name="combo">
            <div
              v-if="showCombo"
              class="absolute top-3 left-1/2 -translate-x-1/2 z-30"
            >
              <div class="glass rounded-2xl px-6 py-3 bg-gradient-to-r from-orange-500/80 to-red-500/80 backdrop-blur-md border border-orange-300/50 shadow-xl">
                <div class="text-center">
                  <div class="text-white font-bold text-xl animate-pulse">
                    {{ comboText }}
                  </div>
                  <div v-if="comboMultiplierText" class="text-yellow-200 text-sm font-semibold">
                    {{ comboMultiplierText }}
                  </div>
                </div>
              </div>
            </div>
          </Transition>

          <div class="absolute bottom-3 left-1/2 -translate-x-1/2 glass rounded-xl px-4 py-2 text-xs text-white/80 flex items-center gap-2">
            <span class="animate-sparkle">✨</span>
            快速连续点击获得连击加成！
            <span class="animate-sparkle" style="animation-delay: 0.5s">✨</span>
          </div>
        </div>

        <div class="lg:col-span-3 order-3 min-h-0 flex flex-col rounded-2xl bg-black/20 border border-white/10">
          <EventModal
            :state="state"
            :all-adults="allAdults"
            @release="releaseBirds"
            @breed="keepAndBreed"
          />
        </div>
      </div>

      <div class="flex justify-center gap-3">
        <button
          class="px-4 py-2 glass rounded-xl text-white/80 text-sm hover:bg-white/20 transition-all flex items-center gap-1.5"
          @click="returnToStart(); router.push('/')"
        >
          <span>🏠</span> 返回主页
        </button>
        <div class="glass rounded-xl px-4 py-2 text-white/80 text-sm flex items-center gap-2">
          <span>💚</span> 存活 {{ aliveCount }} 只
          <span class="mx-1 text-white/30">|</span>
          孵化 {{ state.totalHatched }}
          <span class="mx-1 text-white/30">|</span>
          离世 {{ state.totalDied }}
        </div>
      </div>
    </div>
  </div>
</template>
