<script setup lang="ts">
import { computed, ref } from 'vue'
import type { BerryType } from '@/types/game'
import { BERRY_COLORS, BERRY_EMOJI, BERRY_VALUES } from '@/utils/constants'

const props = defineProps<{
  type: BerryType
  value: number
  x: number
  y: number
  combo?: number
}>()

const emit = defineEmits<{
  (e: 'collect'): void
}>()

const isClicked = ref(false)

const bgStyle = computed(() => ({
  left: `${props.x}%`,
  top: `${props.y}%`,
  '--berry-color': BERRY_COLORS[props.type],
}))

const glowColor = computed(() => {
  if (props.type === 'golden') return 'shadow-yellow-400/60 animate-pulse-glow'
  if (props.type === 'blue') return 'shadow-blue-400/50'
  return 'shadow-red-400/50'
})

const valueBgClass = computed(() => {
  if (props.type === 'golden') return 'bg-yellow-400 text-yellow-900'
  if (props.type === 'blue') return 'bg-blue-400 text-white'
  return 'bg-red-400 text-white'
})

const handleClick = () => {
  if (isClicked.value) return
  isClicked.value = true
  emit('collect')
}
</script>

<template>
  <button
    class="absolute -translate-x-1/2 -translate-y-1/2 w-12 h-12 rounded-full flex items-center justify-center
           text-2xl cursor-pointer animate-pop-in hover:scale-125 active:scale-95 transition-all duration-150
           berry-shadow bg-white/90 border-2 border-white/50 z-10 shadow-lg"
    :class="[glowColor, { 'animate-berry-collect': isClicked }]"
    :style="bgStyle"
    @click.stop="handleClick"
  >
    <span class="drop-shadow select-none">{{ BERRY_EMOJI[type] }}</span>
    <span
      class="absolute -bottom-4 text-[10px] font-bold whitespace-nowrap px-1.5 py-0.5 rounded-full"
      :class="valueBgClass"
    >
      +{{ BERRY_VALUES[type] }}
    </span>
    <div
      v-if="combo && combo >= 2"
      class="absolute -top-2 -right-2 w-6 h-6 bg-orange-500 text-white text-xs font-bold
             rounded-full flex items-center justify-center animate-bounce-in z-20 shadow-lg"
    >
      {{ combo }}x
    </div>
    <div
      v-if="isClicked"
      class="absolute inset-0 rounded-full bg-white/50 animate-ripple pointer-events-none"
    />
  </button>
</template>
