<script setup lang="ts">
interface Props {
  title?: string
  subtext?: string
  image?: string
  imageAlt?: string
}

withDefaults(defineProps<Props>(), {
  title: '',
  subtext: '',
  image: '',
  imageAlt: ''
})

defineEmits(['click'])
</script>

<template lang="pug">
  div(
    @click="$emit('click')"
    class="s-card group relative inline-block w-full max-w-[260px] cursor-pointer select-none overflow-hidden transition-all duration-200 ease-out hover:scale-[102%] active:scale-[98%]"
  )
    //- Character / illustration area
    div(class="s-card-image relative w-full aspect-[4/5] overflow-hidden")
      slot(name="image")
        img(
          v-if="image"
          :src="image"
          :alt="imageAlt || title"
          class="absolute inset-0 w-full h-full object-cover"
        )
      //- soft top vignette to blend with the info strip
      span(class="pointer-events-none absolute inset-x-0 bottom-0 h-1/3 bg-gradient-to-t from-[#2C3E50]/35 to-transparent")

    //- Info strip
    div(class="s-card-info relative px-4 py-3 md:px-5 md:py-4")
      slot(name="info")
        h3(class="s-card-title font-extrabold text-base md:text-lg leading-tight tracking-wide truncate")
          | {{ title }}
        p(
          v-if="subtext"
          class="s-card-subtext mt-1 text-[11px] md:text-xs font-bold uppercase tracking-wider"
        )
          | {{ subtext }}
</template>

<style scoped lang="sass">
.s-card
  border-radius: 24px
  background-color: #FFFFFF
  border: 1px solid #ECF0F1
  box-shadow: 0 0 0 1px rgba(52, 152, 219, 0.10), 0 0 24px -4px rgba(241, 196, 15, 0.25), 0 12px 32px -8px rgba(44, 62, 80, 0.25)
  -webkit-tap-highlight-color: transparent
  transition: box-shadow 200ms ease-out, transform 200ms ease-out

  &:hover
    box-shadow: 0 0 0 1px rgba(230, 126, 34, 0.25), 0 0 36px -4px rgba(241, 196, 15, 0.45), 0 16px 40px -8px rgba(44, 62, 80, 0.35)

.s-card-image
  border-top-left-radius: 24px
  border-top-right-radius: 24px
  background-color: #ECF0F1

.s-card-info
  border-bottom-left-radius: 24px
  border-bottom-right-radius: 24px
  background-color: #FFFFFF

.s-card-title
  color: #2C3E50

.s-card-subtext
  color: #E67E22
</style>
