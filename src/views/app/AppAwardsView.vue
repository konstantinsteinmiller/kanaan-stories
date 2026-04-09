<script setup lang="ts">
import { useI18n } from 'vue-i18n'
import { useRouter } from 'vue-router'
import SBottomNav from '@/components/atoms/SBottomNav.vue'

const { t } = useI18n()
const router = useRouter()

const navItems = [
  { id: 'home', label: '', icon: 'home' as const },
  { id: 'awards', label: '', icon: 'awards' as const },
  { id: 'profile', label: '', icon: 'profile' as const }
]

function onNav(id: string) {
  if (id === 'home') router.push({ name: 'app-main' })
  if (id === 'awards') router.push({ name: 'app-awards' })
  if (id === 'profile') router.push({ name: 'app-profile' })
}

const placeholders = [
  { id: 'first-listen', icon: '🎧', title: 'First Listen', desc: 'Finish your first audio story.' },
  { id: 'first-read', icon: '📖', title: 'First Reader', desc: 'Read a whole story by yourself.' },
  { id: 'fruit-fan', icon: '🍎', title: 'Fruit Fan', desc: 'Collect every Fruit Agents book.' },
  { id: 'collector', icon: '⭐', title: 'Collector', desc: 'Save 5 stories to your watch list.' }
]
</script>

<template lang="pug">
  div(class="app-page min-h-screen w-full")
    header(class="px-5 pb-2 pt-[max(env(safe-area-inset-top),1.25rem)]")
      div(class="max-w-2xl mx-auto")
        p(class="text-xs md:text-sm font-bold uppercase tracking-wider text-white/75") {{ t('keepGoing') }}
        h1(class="text-2xl md:text-3xl font-black text-white") {{ t('awards') }}

    section(class="px-5 mt-6")
      div(class="max-w-2xl mx-auto grid grid-cols-2 gap-3 md:gap-4")
        div(
          v-for="award in placeholders"
          :key="award.id"
          class="award-tile relative rounded-2xl bg-white/95 p-4 text-center shadow-md"
        )
          div(class="award-medal mx-auto w-14 h-14 rounded-full flex items-center justify-center text-2xl") {{ award.icon }}
          h3(class="mt-2 text-sm md:text-base font-black text-[#2C3E50]") {{ award.title }}
          p(class="mt-1 text-[11px] md:text-xs text-[#7F8C8D]") {{ award.desc }}

    div(class="h-32")
    SBottomNav(model-value="awards" :items="navItems" accent="orange" @update:model-value="onNav")
</template>

<style scoped lang="sass">
.app-page
  background: radial-gradient(1200px 600px at 50% -10%, rgba(241, 196, 15, 0.35), transparent 60%), linear-gradient(180deg, #3498DB 0%, #1F6FA8 100%)

.award-medal
  background: linear-gradient(135deg, #F1C40F 0%, #E67E22 100%)
  box-shadow: 0 0 20px -2px rgba(241, 196, 15, 0.55)
</style>

<i18n>
en:
  awards: "Awards"
  keepGoing: "Keep going"
de:
  awards: "Auszeichnungen"
  keepGoing: "Weiter so"
</i18n>
