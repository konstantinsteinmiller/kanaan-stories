<script setup lang="ts">
interface CategoryItem {
  id: string | number
  label: string
  icon?: string
}

interface Props {
  items: CategoryItem[]
  modelValue?: string | number | null
  variant?: 'chips' | 'tabs'
}

withDefaults(defineProps<Props>(), {
  modelValue: null,
  variant: 'chips'
})

const emit = defineEmits(['update:modelValue', 'select'])

function onSelect(item: CategoryItem) {
  emit('update:modelValue', item.id)
  emit('select', item)
}
</script>

<template lang="pug">
  //- CHIPS variant
  div(
    v-if="variant === 'chips'"
    class="s-chips flex flex-wrap gap-2 md:gap-3"
  )
    button(
      v-for="item in items"
      :key="item.id"
      type="button"
      @click="onSelect(item)"
      :class="[\
        's-chip group relative inline-flex items-center gap-2 cursor-pointer select-none touch-manipulation rounded-xl px-3 py-2 md:px-4 md:py-2 transition-all duration-150 ease-out hover:scale-[103%] active:scale-[97%]',\
        modelValue === item.id ? 'is-active' : 'is-inactive'\
      ]"
    )
      span(
        v-if="item.icon"
        class="s-chip-icon text-base md:text-lg leading-none"
      ) {{ item.icon }}
      span(class="s-chip-label text-xs md:text-sm font-bold whitespace-nowrap") {{ item.label }}

  //- TABS variant
  div(
    v-else
    class="s-tabs relative flex items-center gap-5 md:gap-7 border-b border-white/30"
  )
    button(
      v-for="item in items"
      :key="item.id"
      type="button"
      @click="onSelect(item)"
      :class="[\
        's-tab relative cursor-pointer select-none touch-manipulation pb-3 pt-2 transition-colors duration-150 ease-out',\
        modelValue === item.id ? 'is-active' : 'is-inactive'\
      ]"
    )
      span(class="text-sm md:text-base font-bold tracking-wide") {{ item.label }}
      //- underline indicator for active tab
      span(
        v-if="modelValue === item.id"
        class="s-tab-underline absolute left-0 right-0 -bottom-px h-[2px] rounded-full"
      )
</template>

<style scoped lang="sass">
button
  -webkit-tap-highlight-color: transparent
  background: transparent
  border: none

// ---------- Chips ----------
.s-chip
  background-color: #FFFFFF
  border: 1px solid #ECF0F1
  color: #2C3E50
  box-shadow: 0 2px 8px -2px rgba(44, 62, 80, 0.18)

  &.is-inactive:hover
    border-color: #F1C40F
    background-color: #FFFDF2

  &.is-active
    border: 1px solid transparent
    color: #2C3E50
    background-image: linear-gradient(#FFFFFF, #FFFFFF), linear-gradient(135deg, #F1C40F 0%, #E67E22 100%)
    background-origin: border-box
    background-clip: padding-box, border-box
    box-shadow: 0 0 18px -2px rgba(230, 126, 34, 0.55), 0 0 32px -8px rgba(241, 196, 15, 0.45)

// ---------- Tabs ----------
.s-tab
  background: transparent
  border: none

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
