<script setup lang="ts">
interface Props {
  label?: string
  type?: 'primary' | 'secondary' | 'icon'
  isDisabled?: boolean
  size?: 'sm' | 'md' | 'lg' | 'xl'
  attention?: boolean
}

withDefaults(defineProps<Props>(), {
  label: 'NO LABEL DEFINED!',
  type: 'primary',
  attention: false
})

defineEmits(['click'])
</script>

<template lang="pug">
  div(
    :class="{\
      'scale-60' : size === 'sm',\
      'scale-80' : size === 'md',\
      'scale-110' : size === 'lg',\
      'scale-120' : size === 'xl',\
      'attention-bounce': attention,\
      'opacity-50 grayscale pointer-events-none': isDisabled,\
      'inline-block': type === 'icon',\
      'w-full': type !== 'icon'\
    }"
  )
    //- PRIMARY: sunlight yellow pill with dark navy text
    button(
      v-if="type === 'primary'"
      type="button"
      @click="$emit('click')"
      class="s-btn-primary group relative w-full inline-flex items-center justify-center cursor-pointer select-none touch-manipulation rounded-full px-6 md:px-8 py-3 md:py-4 min-w-[120px] md:min-w-[160px] transition-all duration-150 ease-out hover:scale-[103%] active:scale-[97%]"
    )
      span(class="relative block text-sm md:text-base font-black tracking-wide uppercase")
        slot {{ label }}

    //- SECONDARY: transparent pill, warm orange border + navy text
    button(
      v-else-if="type === 'secondary'"
      type="button"
      @click="$emit('click')"
      class="s-btn-secondary group relative w-full inline-flex items-center justify-center cursor-pointer select-none touch-manipulation bg-transparent rounded-full px-6 md:px-8 py-3 md:py-4 min-w-[120px] md:min-w-[160px] transition-all duration-150 ease-out hover:scale-[103%] active:scale-[97%]"
    )
      span(class="relative block text-sm md:text-base font-semibold tracking-wide uppercase")
        slot {{ label }}

    //- ICON: circular warm orange button with white glyph and warm glow
    button(
      v-else
      type="button"
      @click="$emit('click')"
      :aria-label="label"
      class="s-btn-icon group relative inline-flex items-center justify-center cursor-pointer select-none touch-manipulation rounded-full w-12 h-12 md:w-14 md:h-14 text-white transition-all duration-150 ease-out hover:scale-[105%] active:scale-[95%]"
    )
      span(class="relative flex items-center justify-center w-full h-full text-white")
        slot
</template>

<style scoped lang="sass">
button
  -webkit-tap-highlight-color: transparent

.s-btn-primary
  background-color: #F1C40F
  color: #2C3E50
  box-shadow: 0 6px 18px -4px rgba(44, 62, 80, 0.35), 0 2px 6px rgba(44, 62, 80, 0.2), inset 0 -2px 0 rgba(44, 62, 80, 0.12)

  &:hover
    background-color: #f5cf3a
    box-shadow: 0 10px 24px -4px rgba(44, 62, 80, 0.4), 0 4px 10px rgba(44, 62, 80, 0.25), inset 0 -2px 0 rgba(44, 62, 80, 0.12)

.s-btn-secondary
  color: #2C3E50
  border: 1.5px solid #E67E22
  backdrop-filter: blur(4px)
  box-shadow: 0 0 0 0 rgba(230, 126, 34, 0)

  &:hover
    background-color: rgba(230, 126, 34, 0.08)
    box-shadow: 0 0 18px -2px rgba(230, 126, 34, 0.45)

.s-btn-icon
  background-color: #E67E22
  border: 2px solid rgba(255, 255, 255, 0.85)
  box-shadow: 0 0 18px -2px rgba(230, 126, 34, 0.7), 0 0 32px -8px rgba(241, 196, 15, 0.45), inset 0 1px 0 rgba(255, 255, 255, 0.18)

  &:hover
    background-color: #f08a30
    box-shadow: 0 0 24px -2px rgba(230, 126, 34, 0.85), 0 0 44px -6px rgba(241, 196, 15, 0.55), inset 0 1px 0 rgba(255, 255, 255, 0.22)
</style>
