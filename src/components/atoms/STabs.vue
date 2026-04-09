<script setup lang="ts">
export interface TabOption {
  label: string
  value: string | number
  icon?: string
}

interface Props {
  modelValue: string | number
  options: TabOption[]
}

const props = defineProps<Props>()
const emit = defineEmits(['update:modelValue'])

const selectTab = (value: string | number) => {
  emit('update:modelValue', value)
}
</script>

<template lang="pug">
  div(class="s-tabs relative flex items-center gap-5 md:gap-7 border-b border-white/30")
    button(
      v-for="tab in options"
      :key="tab.value"
      type="button"
      @click="selectTab(tab.value)"
      :class="[\
        's-tab relative cursor-pointer select-none touch-manipulation pb-3 pt-2 transition-colors duration-150 ease-out',\
        modelValue === tab.value ? 'is-active' : 'is-inactive'\
      ]"
    )
      span(class="inline-flex items-center gap-2")
        img(
          v-if="tab.icon"
          :src="tab.icon"
          :alt="tab.label"
          class="w-5 h-5 object-contain"
        )
        span(class="text-sm md:text-base font-bold tracking-wide") {{ tab.label }}
      span(
        v-if="modelValue === tab.value"
        class="s-tab-underline absolute left-0 right-0 -bottom-px h-[2px] rounded-full"
      )
</template>

<style lang="sass" scoped>
button
  -webkit-tap-highlight-color: transparent
  background: transparent
  border: none

.s-tab
  &.is-inactive
    color: rgba(255, 255, 255, 0.7)

    &:hover
      color: #FFFFFF

  &.is-active
    color: #F1C40F

.s-tab-underline
  background-color: #F1C40F
  box-shadow: 0 0 10px rgba(241, 196, 15, 0.7)
</style>
