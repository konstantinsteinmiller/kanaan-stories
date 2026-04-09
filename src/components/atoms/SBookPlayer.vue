<script setup lang="ts">
import { computed, ref } from 'vue'

interface Props {
  isPlaying?: boolean
  currentTime?: number
  duration?: number
  skipSeconds?: number
}

const props = withDefaults(defineProps<Props>(), {
  isPlaying: false,
  currentTime: 0,
  duration: 0,
  skipSeconds: 10
})

const emit = defineEmits<{
  (e: 'toggle'): void
  (e: 'rewind'): void
  (e: 'forward'): void
  (e: 'seek', value: number): void
}>()

const trackRef = ref<HTMLDivElement | null>(null)
const dragging = ref(false)

const progress = computed(() => {
  if (!props.duration) return 0
  return Math.min(100, Math.max(0, (props.currentTime / props.duration) * 100))
})

function formatTime(seconds: number) {
  if (!Number.isFinite(seconds) || seconds < 0) seconds = 0
  const m = Math.floor(seconds / 60)
  const s = Math.floor(seconds % 60)
  return `${m}:${s.toString().padStart(2, '0')}`
}

function ratioFromEvent(clientX: number) {
  const el = trackRef.value
  if (!el) return 0
  const rect = el.getBoundingClientRect()
  return Math.min(1, Math.max(0, (clientX - rect.left) / rect.width))
}

function seekToEvent(clientX: number) {
  if (!props.duration) return
  emit('seek', ratioFromEvent(clientX) * props.duration)
}

function onPointerDown(e: PointerEvent) {
  dragging.value = true
  ;(e.target as HTMLElement).setPointerCapture?.(e.pointerId)
  seekToEvent(e.clientX)
}

function onPointerMove(e: PointerEvent) {
  if (!dragging.value) return
  seekToEvent(e.clientX)
}

function onPointerUp(e: PointerEvent) {
  if (!dragging.value) return
  dragging.value = false
  ;(e.target as HTMLElement).releasePointerCapture?.(e.pointerId)
}
</script>

<template lang="pug">
  div(class="s-book-player relative w-full max-w-md mx-auto rounded-3xl px-5 py-6 md:px-7 md:py-7 select-none")
    //- Slider
    div(class="s-bp-slider-wrap")
      div(
        ref="trackRef"
        @pointerdown="onPointerDown"
        @pointermove="onPointerMove"
        @pointerup="onPointerUp"
        @pointercancel="onPointerUp"
        class="s-bp-track relative w-full h-[6px] rounded-full cursor-pointer touch-none"
      )
        //- filled
        div(
          class="s-bp-fill absolute inset-y-0 left-0 rounded-full"
          :style="{ width: `${progress}%` }"
        )
        //- handle
        div(
          class="s-bp-handle absolute top-1/2 -translate-y-1/2 -translate-x-1/2 w-5 h-5 rounded-full"
          :style="{ left: `${progress}%` }"
        )

      //- time labels
      div(class="mt-3 flex items-center justify-between text-[11px] md:text-xs font-bold tracking-wider tabular-nums s-bp-time")
        span {{ formatTime(currentTime) }}
        span {{ formatTime(duration) }}

    //- Controls row
    div(class="mt-6 md:mt-7 flex items-center justify-center gap-8 md:gap-10")
      //- Skip backward (half size of play)
      button(
        type="button"
        @click="emit('rewind')"
        :aria-label="`Skip back ${skipSeconds} seconds`"
        class="s-bp-skip relative inline-flex items-center justify-center w-10 h-10 md:w-12 md:h-12 rounded-full cursor-pointer transition-all duration-150 ease-out hover:scale-[105%] active:scale-[95%]"
      )
        svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5 md:w-6 md:h-6 -mt-2")
          path(d="M11 5 4 12l7 7")
          path(d="M4 12h12a5 5 0 0 1 0 10h-2")
        span(class="absolute inset-0 flex items-center justify-center pointer-events-none text-[8px] md:text-[9px] font-black translate-y-[1px] -mb-4") {{ skipSeconds }}

      //- Play / Pause
      button(
        type="button"
        @click="emit('toggle')"
        :aria-label="isPlaying ? 'Pause' : 'Play'"
        class="s-bp-play relative inline-flex items-center justify-center w-20 h-20 md:w-24 md:h-24 rounded-full cursor-pointer transition-all duration-150 ease-out hover:scale-[104%] active:scale-[96%]"
      )
        svg(
          v-if="isPlaying"
          viewBox="0 0 24 24"
          fill="currentColor"
          class="w-9 h-9 md:w-10 md:h-10"
        )
          rect(x="6" y="5" width="4" height="14" rx="1")
          rect(x="14" y="5" width="4" height="14" rx="1")
        svg(
          v-else
          viewBox="0 0 24 24"
          fill="currentColor"
          class="w-9 h-9 md:w-10 md:h-10 translate-x-[2px]"
        )
          path(d="M8 5v14l11-7z")

      //- Skip forward (half size of play)
      button(
        type="button"
        @click="emit('forward')"
        :aria-label="`Skip forward ${skipSeconds} seconds`"
        class="s-bp-skip relative inline-flex items-center justify-center w-10 h-10 md:w-12 md:h-12 rounded-full cursor-pointer transition-all duration-150 ease-out hover:scale-[105%] active:scale-[95%]"
      )
        svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5 md:w-6 md:h-6 -mt-2")
          path(d="m13 5 7 7-7 7")
          path(d="M20 12H8a5 5 0 0 0 0 10h2")
        span(class="absolute inset-0 flex items-center justify-center pointer-events-none text-[8px] md:text-[9px] font-black translate-y-[1px] -mb-4") {{ skipSeconds }}
</template>

<style scoped lang="sass">
button
  -webkit-tap-highlight-color: transparent
  background: transparent
  border: none
  padding: 0

.s-book-player
  background: linear-gradient(160deg, #3498DB 0%, #1F6FA8 100%)
  border: 1px solid rgba(255, 255, 255, 0.18)
  box-shadow: 0 0 0 1px rgba(255, 255, 255, 0.08), 0 24px 60px -20px rgba(44, 62, 80, 0.55)

.s-bp-track
  background-color: rgba(255, 255, 255, 0.25)

.s-bp-fill
  background: linear-gradient(90deg, #F1C40F 0%, #E67E22 100%)
  box-shadow: 0 0 8px rgba(241, 196, 15, 0.6)

.s-bp-handle
  background-color: #FFFFFF
  box-shadow: 0 0 0 4px rgba(255, 255, 255, 0.18), 0 0 14px 2px rgba(241, 196, 15, 0.7), 0 0 28px 4px rgba(230, 126, 34, 0.5)

.s-bp-time
  color: rgba(255, 255, 255, 0.75)

.s-bp-skip
  background-color: rgba(255, 255, 255, 0.12)
  color: #FFFFFF
  border: 1.5px solid rgba(255, 255, 255, 0.6)

  &:hover
    background-color: rgba(255, 255, 255, 0.2)
    border-color: #F1C40F

.s-bp-play
  background-color: #FFFFFF
  color: #2C3E50
  border: 3px solid #F1C40F
  box-shadow: 0 0 0 1px rgba(241, 196, 15, 0.18), 0 0 24px -2px rgba(241, 196, 15, 0.55), 0 12px 32px -8px rgba(44, 62, 80, 0.5)

  &:hover
    box-shadow: 0 0 0 1px rgba(241, 196, 15, 0.3), 0 0 36px -2px rgba(241, 196, 15, 0.7), 0 16px 40px -8px rgba(44, 62, 80, 0.55)
</style>
