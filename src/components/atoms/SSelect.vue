<script setup lang="ts">
import { computed } from 'vue'

interface Option {
  value: string | number
  label: string
  subLabel?: string
  icon?: string
  avatar?: string
}

interface Props {
  modelValue: string | number
  options: Option[]
  label?: string
  multiple?: boolean
  accent?: 'orange' | 'yellow'
}

const props = withDefaults(defineProps<Props>(), {
  label: '',
  multiple: false,
  accent: 'orange'
})

const emit = defineEmits(['update:modelValue', 'change'])

const accentClass = computed(() => (props.accent === 'yellow' ? 'accent-yellow' : 'accent-orange'))

function isSelected(option: Option) {
  return props.modelValue === option.value
}

function selectOption(option: Option) {
  emit('update:modelValue', option.value)
  emit('change', option)
}
</script>

<template lang="pug">
  div(:class="['s-select w-full', accentClass]")
    //- Optional label
    div(
      v-if="label"
      class="s-select-label mb-2 ml-1 text-xs md:text-sm font-bold uppercase tracking-wider"
    ) {{ label }}

    //- Tile list
    div(class="flex flex-col gap-2 md:gap-3")
      button(
        v-for="option in options"
        :key="option.value"
        type="button"
        @click="selectOption(option)"
        :class="[\
          's-select-tile group relative flex items-center w-full gap-3 md:gap-4 cursor-pointer select-none touch-manipulation rounded-2xl px-3 py-3 md:px-4 md:py-3 text-left transition-all duration-150 ease-out hover:scale-[101%] active:scale-[99%]',\
          isSelected(option) ? 'is-active' : 'is-inactive'\
        ]"
      )
        //- Left: avatar / icon
        div(class="s-select-avatar relative shrink-0 inline-flex items-center justify-center w-10 h-10 md:w-11 md:h-11 rounded-full overflow-hidden")
          img(
            v-if="option.avatar"
            :src="option.avatar"
            :alt="option.label"
            class="absolute inset-0 w-full h-full object-cover"
          )
          span(
            v-else-if="option.icon"
            class="text-lg md:text-xl leading-none"
          ) {{ option.icon }}
          span(
            v-else
            class="text-sm font-bold uppercase"
          ) {{ option.label.charAt(0) }}

        //- Center: text block
        div(class="flex-1 min-w-0 flex flex-col")
          span(class="s-select-title text-sm md:text-base font-bold truncate") {{ option.label }}
          span(
            v-if="option.subLabel"
            class="s-select-sub mt-0.5 text-[11px] md:text-xs font-medium truncate"
          ) {{ option.subLabel }}

        //- Right: radio / checkbox indicator
        div(
          :class="[\
            's-select-indicator shrink-0 relative inline-flex items-center justify-center w-6 h-6 md:w-7 md:h-7 border-2 transition-all duration-150',\
            multiple ? 'rounded-md' : 'rounded-full',\
            isSelected(option) ? 'is-checked' : ''\
          ]"
        )
          span(
            v-if="!multiple && isSelected(option)"
            class="block w-2.5 h-2.5 md:w-3 md:h-3 rounded-full bg-white"
          )
          svg(
            v-else-if="multiple && isSelected(option)"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="3.5"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="w-4 h-4 text-white"
          )
            polyline(points="20 6 9 17 4 12")
</template>

<style scoped lang="sass">
button
  -webkit-tap-highlight-color: transparent
  background: transparent
  border: none

.s-select-label
  color: rgba(255, 255, 255, 0.85)

.s-select-tile
  background-color: #FFFFFF
  border: 2px solid #ECF0F1
  color: #2C3E50
  box-shadow: 0 4px 14px -6px rgba(44, 62, 80, 0.35)

  &.is-inactive:hover
    border-color: #F1C40F
    background-color: #FFFDF2

.s-select-avatar
  background-color: #ECF0F1
  border: 1px solid #ECF0F1
  color: #34495E

.s-select-title
  color: #2C3E50

.s-select-sub
  color: #7F8C8D

.s-select-indicator
  border-color: #BDC3C7

// ---------- Orange accent ----------
.accent-orange
  .s-select-tile.is-active
    border-color: #E67E22
    background-color: #FFF6EC
    box-shadow: 0 0 0 1px rgba(230, 126, 34, 0.35), 0 0 22px -2px rgba(230, 126, 34, 0.55), 0 0 44px -10px rgba(241, 196, 15, 0.45)

  .s-select-indicator.is-checked
    border-color: #E67E22
    background-color: #E67E22
    box-shadow: 0 0 12px rgba(230, 126, 34, 0.7)

// ---------- Yellow accent ----------
.accent-yellow
  .s-select-tile.is-active
    border-color: #F1C40F
    background-color: #FFFCE6
    box-shadow: 0 0 0 1px rgba(241, 196, 15, 0.45), 0 0 22px -2px rgba(241, 196, 15, 0.6), 0 0 44px -10px rgba(241, 196, 15, 0.45)

  .s-select-indicator.is-checked
    border-color: #F1C40F
    background-color: #F1C40F
    box-shadow: 0 0 12px rgba(241, 196, 15, 0.75)
</style>
