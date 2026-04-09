<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute, useRouter } from 'vue-router'
import SCard from '@/components/atoms/SCard.vue'
import SBottomNav from '@/components/atoms/SBottomNav.vue'
import useModels from '@/use/useModels'

const { t } = useI18n()
const route = useRoute()
const router = useRouter()
const { getSeries, getBooksOfSeries } = useModels()

const seriesId = computed(() => String(route.params.seriesId))
const series = computed(() => getSeries(seriesId.value))
const books = computed(() => getBooksOfSeries(seriesId.value))

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

function openBook(bookId: string) {
  router.push({ name: 'app-book', params: { bookId } })
}

function goBack() {
  router.push({ name: 'app-main' })
}

function formatDate(iso: string) {
  try {
    return new Date(iso).getFullYear().toString()
  } catch {
    return ''
  }
}
</script>

<template lang="pug">
  div(class="app-page min-h-screen w-full")
    //- Header
    header(class="px-5 pb-2 pt-[max(env(safe-area-inset-top),1.25rem)]")
      div(class="max-w-2xl mx-auto flex items-center gap-3")
        button(
          type="button"
          @click="goBack"
          class="back-btn shrink-0 w-10 h-10 rounded-full bg-white/95 text-[#2C3E50] flex items-center justify-center shadow-md hover:scale-[105%] active:scale-[95%] transition-transform"
        )
          svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5")
            path(d="m15 6-6 6 6 6")
        div(class="min-w-0")
          p(class="text-[10px] md:text-xs font-bold uppercase tracking-wider text-white/75") {{ t('bookSeries') }}
          h1(class="text-2xl md:text-3xl font-black text-white truncate") {{ series?.name || '—' }}

    //- Description
    section(
      v-if="series"
      class="px-5 mt-3"
    )
      div(class="max-w-2xl mx-auto")
        p(class="text-sm md:text-base text-white/85 leading-relaxed") {{ series.description }}

    //- Book list
    section(class="px-5 mt-6")
      div(class="max-w-2xl mx-auto grid grid-cols-2 md:grid-cols-3 gap-3 md:gap-4")
        SCard(
          v-for="(book, idx) in books"
          :key="book.id"
          :title="t(book.title)"
          :subtext="`#${idx + 1} · ${formatDate(book.releaseDate)}`"
          class="w-full"
          @click="openBook(book.id)"
        )
          template(#image)
            div(
              class="absolute inset-0"
              :style="{ background: book.cover || 'linear-gradient(135deg,#3498DB,#9B59B6)' }"
            )
            img(
              v-if="book.previewImage || book.coverImage"
              :src="book.previewImage || book.coverImage"
              :alt="t(book.title)"
              class="absolute inset-0 w-full h-full object-cover"
              loading="lazy"
            )

    div(class="h-32")
    SBottomNav(model-value="home" :items="navItems" accent="orange" @update:model-value="onNav")
</template>

<style scoped lang="sass">
.app-page
  background: radial-gradient(1200px 600px at 50% -10%, rgba(241, 196, 15, 0.35), transparent 60%), linear-gradient(180deg, #3498DB 0%, #1F6FA8 100%)
</style>

<i18n>
en:
  bookSeries: "Book series"
de:
  bookSeries: "Buchreihe"
</i18n>
