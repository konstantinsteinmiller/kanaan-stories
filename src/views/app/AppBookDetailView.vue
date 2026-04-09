<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, ref, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute, useRouter } from 'vue-router'
import SBookPlayer from '@/components/atoms/SBookPlayer.vue'
import SBottomNav from '@/components/atoms/SBottomNav.vue'
import useModels from '@/use/useModels'

const { t, locale } = useI18n()
const route = useRoute()
const router = useRouter()
const { getBook, getSeriesOfBook, isInWatchList, toggleWatchList, setLastRead } = useModels()

const bookId = computed(() => String(route.params.bookId))
const book = computed(() => getBook(bookId.value))
const series = computed(() => (book.value ? getSeriesOfBook(book.value.id) : undefined))

const inWatchList = computed(() => (book.value ? isInWatchList(book.value.id) : false))

// ----- Audio playback (real <audio> element) -----
const audioEl = ref<HTMLAudioElement | null>(null)
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)

const audioSrc = computed(() => {
  if (!book.value) return ''
  const lang = locale.value as 'en' | 'de'
  return book.value.audio[lang] || book.value.audio.en
})

function ensureAudio() {
  if (!audioEl.value) return
  audioEl.value.addEventListener('timeupdate', onTimeUpdate)
  audioEl.value.addEventListener('loadedmetadata', onLoadedMeta)
  audioEl.value.addEventListener('ended', onEnded)
}

function onTimeUpdate() {
  if (audioEl.value) currentTime.value = audioEl.value.currentTime
}

function onLoadedMeta() {
  if (audioEl.value) duration.value = audioEl.value.duration || 0
}

function onEnded() {
  isPlaying.value = false
  currentTime.value = 0
}

function togglePlay() {
  if (!audioEl.value) return
  if (isPlaying.value) {
    audioEl.value.pause()
    isPlaying.value = false
  } else {
    audioEl.value.play().then(() => {
      isPlaying.value = true
    }).catch(() => {
      // playback may fail (placeholder URLs) — still flip the UI so it feels alive
      isPlaying.value = !isPlaying.value
    })
  }
}

function rewind() {
  if (!audioEl.value) return
  audioEl.value.currentTime = Math.max(0, (audioEl.value.currentTime || 0) - 10)
  currentTime.value = audioEl.value.currentTime
}

function forward() {
  if (!audioEl.value) return
  audioEl.value.currentTime = Math.min(duration.value || 0, (audioEl.value.currentTime || 0) + 10)
  currentTime.value = audioEl.value.currentTime
}

function seek(value: number) {
  if (!audioEl.value) return
  audioEl.value.currentTime = value
  currentTime.value = value
}

// Mark last read on mount
onMounted(() => {
  if (book.value) setLastRead(book.value.id)
  ensureAudio()
})
onUnmounted(() => {
  if (audioEl.value) {
    audioEl.value.pause()
    audioEl.value.removeEventListener('timeupdate', onTimeUpdate)
    audioEl.value.removeEventListener('loadedmetadata', onLoadedMeta)
    audioEl.value.removeEventListener('ended', onEnded)
  }
})

watch(audioSrc, () => {
  isPlaying.value = false
  currentTime.value = 0
  duration.value = 0
})

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

function goBack() {
  if (series.value) router.push({ name: 'app-series', params: { seriesId: series.value.id } })
  else router.push({ name: 'app-main' })
}

function formatDate(iso: string) {
  try {
    return new Date(iso).toLocaleDateString()
  } catch {
    return iso
  }
}

function onToggleWatchList() {
  if (book.value) toggleWatchList(book.value.id)
}

const selectedPageId = ref(1)
const PAGE_SKIP = 5

const totalPages = computed(() => book.value?.content?.length || 0)
const isFirstPage = computed(() => selectedPageId.value <= 1)
const isLastPage = computed(() => selectedPageId.value >= totalPages.value)

const currentPage = computed(() => {
  if (!book.value) return { title: '', text: '' }
  const page = book.value.content?.[selectedPageId.value - 1]
  if (!page) return { title: '', text: '' }
  return {
    title: t(page.title) || '',
    text: t(page.text) || ''
  }
})

const pageTitleEl = ref<HTMLElement | null>(null)

function scrollToPageTitle() {
  nextTick(() => {
    const el = pageTitleEl.value
    if (!el) return
    const rect = el.getBoundingClientRect()
    const top = rect.top + window.scrollY - 16
    window.scrollTo({ top, behavior: 'smooth' })
  })
}

function goToPage(page: number, scroll = false) {
  if (!totalPages.value) return
  const next = Math.min(totalPages.value, Math.max(1, page))
  if (next === selectedPageId.value) return
  selectedPageId.value = next
  if (scroll) scrollToPageTitle()
}

const goToFirst = () => goToPage(1, true)
const goToLast = () => goToPage(totalPages.value, true)
const goPrev = () => goToPage(selectedPageId.value - 1, true)
const goNext = () => goToPage(selectedPageId.value + 1, true)
const skipBack = () => goToPage(selectedPageId.value - PAGE_SKIP, true)
const skipForward = () => goToPage(selectedPageId.value + PAGE_SKIP, true)

// Reset to page 1 whenever the book changes
watch(bookId, () => {
  selectedPageId.value = 1
})

// Keyboard navigation: ←/→ for prev/next, Shift+←/→ to skip 5, Home/End for first/last
function onKeydown(e: KeyboardEvent) {
  const tag = (e.target as HTMLElement | null)?.tagName
  if (tag === 'INPUT' || tag === 'TEXTAREA') return
  switch (e.key) {
    case 'ArrowLeft':
      e.shiftKey ? skipBack() : goPrev()
      break
    case 'ArrowRight':
      e.shiftKey ? skipForward() : goNext()
      break
    case 'Home':
      goToFirst()
      break
    case 'End':
      goToLast()
      break
    default:
      return
  }
  e.preventDefault()
}

onMounted(() => window.addEventListener('keydown', onKeydown))
onUnmounted(() => window.removeEventListener('keydown', onKeydown))
</script>

<template lang="pug">
  div(class="app-page min-h-screen w-full")
    //- Header
    header(class="px-5 pb-2 pt-[max(env(safe-area-inset-top),1.25rem)]")
      div(class="max-w-2xl mx-auto flex items-center justify-between gap-3")
        button(
          type="button"
          @click="goBack"
          class="back-btn shrink-0 w-10 h-10 rounded-full bg-white/95 text-[#2C3E50] flex items-center justify-center shadow-md hover:scale-[105%] active:scale-[95%] transition-transform"
        )
          svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5")
            path(d="m15 6-6 6 6 6")

        //- Watch list toggle (bookmark icon button)
        button(
          v-if="book"
          type="button"
          @click="onToggleWatchList"
          :aria-label="inWatchList ? t('removeFromWatchList') : t('addToWatchList')"
          :aria-pressed="inWatchList"
          :class="[\
            'shrink-0 w-10 h-10 rounded-full flex items-center justify-center shadow-md hover:scale-[105%] active:scale-[95%] transition-all',\
            inWatchList ? 'bg-[#E67E22] text-white' : 'bg-white/95 text-[#2C3E50]'\
          ]"
        )
          svg(v-if="inWatchList" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5")
            path(d="M6 3h12a1 1 0 0 1 1 1v17l-7-4-7 4V4a1 1 0 0 1 1-1Z")
          svg(v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5")
            path(d="M6 3h12a1 1 0 0 1 1 1v17l-7-4-7 4V4a1 1 0 0 1 1-1Z")

    template(v-if="book")
      //- Cover card
      section(class="px-5 mt-3")
        div(class="max-w-2xl mx-auto")
          div(
            class="book-cover relative w-full aspect-[16/10] rounded-3xl overflow-hidden shadow-2xl"
            :style="{ background: book.cover || 'linear-gradient(135deg,#3498DB,#9B59B6)' }"
          )
            img(
              v-if="book.coverImage || book.previewImage"
              :src="book.coverImage || book.previewImage"
              :alt="t(book.title)"
              class="absolute inset-0 w-full h-full object-cover"
              loading="lazy"
            )
            div(class="absolute inset-0 bg-gradient-to-t from-[#1F6FA8]/85 via-[#1F6FA8]/30 to-transparent")
            div(class="absolute inset-x-0 bottom-0 p-5")
              p(class="text-[10px] md:text-xs font-bold uppercase tracking-wider text-white/85") {{ series?.name || '' }}
              h1(class="text-2xl md:text-3xl font-black text-white leading-tight") {{ t(book.title) }}
              p(class="mt-1 text-sm md:text-base font-bold text-white/85") {{ book.author }} · {{ formatDate(book.releaseDate) }}

      //- Audio player
      section(class="px-5 mt-5")
        div(class="max-w-2xl mx-auto")
          SBookPlayer(
            :is-playing="isPlaying"
            :current-time="currentTime"
            :duration="duration || 180"
            :skip-seconds="10"
            @toggle="togglePlay"
            @rewind="rewind"
            @forward="forward"
            @seek="seek"
          )
          audio(
            ref="audioEl"
            :src="audioSrc"
            preload="metadata"
            class="hidden"
          )

      //- Description + body text
      section(class="px-5 mt-6")
        div(class="max-w-2xl mx-auto bg-white/95 rounded-3xl p-5 md:p-6 shadow-lg")
          h2(class="text-[10px] md:text-xs font-black uppercase tracking-wider text-[#E67E22]") {{ t('about') }}
          p(class="mt-1 text-sm md:text-base text-[#2C3E50] leading-relaxed") {{ book.description }}

          //- Page header + indicator
          div(class="mt-5 flex items-center justify-between gap-3")
            h2(
              ref="pageTitleEl"
              class="text-[10px] md:text-xs font-black uppercase tracking-wider text-[#E67E22] truncate scroll-mt-4"
            ) {{ currentPage.title }}
            span(
              v-if="totalPages"
              class="shrink-0 rounded-full bg-[#FFF6EC] px-3 py-1 text-[10px] md:text-xs font-black tracking-wider text-[#E67E22] tabular-nums"
            ) {{ selectedPageId }} / {{ totalPages }}

          //- Page text
          div(class="mt-2 text-base md:text-lg text-[#2C3E50] leading-relaxed whitespace-pre-line min-h-[6rem]") {{ currentPage.text }}

          //- Pagination controls
          nav(
            v-if="totalPages > 1"
            class="mt-5 pt-4 border-t border-[#ECF0F1] flex items-center justify-center gap-1.5 md:gap-2"
            :aria-label="t('paginationLabel')"
          )
            //- Jump to first
            button(
              type="button"
              @click="goToFirst"
              :disabled="isFirstPage"
              :aria-label="t('firstPage')"
              class="page-btn"
            )
              svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" class="w-4 h-4 md:w-5 md:h-5")
                path(d="M19 19 12 12l7-7")
                path(d="M11 19 4 12l7-7")

            //- Skip back 5
            button(
              type="button"
              @click="skipBack"
              :disabled="isFirstPage"
              :aria-label="t('skipBackPages', { n: 5 })"
              class="page-btn relative"
            )
              svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" class="w-4 h-4 md:w-5 md:h-5")
                path(d="m15 19-7-7 7-7")
              span(class="absolute bottom-[0.125rem] left-[50%] -translate-x-[50%] text-[8px] md:text-[9px] font-black leading-none") 5

            //- Previous
            button(
              type="button"
              @click="goPrev"
              :disabled="isFirstPage"
              :aria-label="t('prevPage')"
              class="page-btn page-btn--accent"
            )
              svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5 md:w-6 md:h-6")
                path(d="m15 18-6-6 6-6")

            //- Page indicator (compact, between accents)
            div(class="px-2 md:px-3 text-xs md:text-sm font-black text-[#2C3E50] tabular-nums select-none")
              | {{ selectedPageId }}
              span(class="text-[#7F8C8D] font-bold") &nbsp;/ {{ totalPages }}

            //- Next
            button(
              type="button"
              @click="goNext"
              :disabled="isLastPage"
              :aria-label="t('nextPage')"
              class="page-btn page-btn--accent"
            )
              svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5 md:w-6 md:h-6")
                path(d="m9 18 6-6-6-6")

            //- Skip forward 5
            button(
              type="button"
              @click="skipForward"
              :disabled="isLastPage"
              :aria-label="t('skipForwardPages', { n: 5 })"
              class="page-btn relative"
            )
              svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" class="w-4 h-4 md:w-5 md:h-5")
                path(d="m9 5 7 7-7 7")
              span(class="absolute bottom-[0.125rem] left-[50%] -translate-x-[50%] text-[8px] md:text-[9px] font-black leading-none") 5

            //- Jump to last
            button(
              type="button"
              @click="goToLast"
              :disabled="isLastPage"
              :aria-label="t('lastPage')"
              class="page-btn"
            )
              svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" class="w-4 h-4 md:w-5 md:h-5")
                path(d="m5 5 7 7-7 7")
                path(d="m13 5 7 7-7 7")

    div(v-else class="px-5 mt-10 text-center text-white/85") {{ t('notFound') }}

    div(class="h-32")
    SBottomNav(model-value="home" :items="navItems" accent="orange" @update:model-value="onNav")
</template>

<style scoped lang="sass">
.app-page
  background: radial-gradient(1200px 600px at 50% -10%, rgba(241, 196, 15, 0.35), transparent 60%), linear-gradient(180deg, #3498DB 0%, #1F6FA8 100%)

.page-btn
  -webkit-tap-highlight-color: transparent
  display: inline-flex
  align-items: center
  justify-content: center
  width: 36px
  height: 36px
  border-radius: 9999px
  background-color: #FFFFFF
  color: #2C3E50
  border: 1.5px solid #ECF0F1
  cursor: pointer
  transition: all 150ms ease-out

  &:hover:not(:disabled)
    border-color: #E67E22
    background-color: #FFF6EC
    color: #E67E22
    transform: scale(1.06)

  &:active:not(:disabled)
    transform: scale(0.94)

  &:disabled
    opacity: 0.35
    cursor: not-allowed

  @media (min-width: 768px)
    width: 40px
    height: 40px

.page-btn--accent
  background-color: #E67E22
  color: #FFFFFF
  border-color: #E67E22
  box-shadow: 0 4px 14px -6px rgba(230, 126, 34, 0.7)

  &:hover:not(:disabled)
    background-color: #f08a30
    color: #FFFFFF
    border-color: #f08a30
</style>

<i18n>
en:
  about: "About this story"
  readAlong: "Read along"
  addToWatchList: "Add to watch list"
  removeFromWatchList: "Remove from watch list"
  notFound: "Story not found."
  paginationLabel: "Story pages"
  firstPage: "First page"
  lastPage: "Last page"
  prevPage: "Previous page"
  nextPage: "Next page"
  skipBackPages: "Skip back {n} pages"
  skipForwardPages: "Skip forward {n} pages"
de:
  about: "Über diese Geschichte"
  readAlong: "Mitlesen"
  addToWatchList: "Zur Merkliste hinzufügen"
  removeFromWatchList: "Von Merkliste entfernen"
  notFound: "Geschichte nicht gefunden."
  paginationLabel: "Seiten der Geschichte"
  firstPage: "Erste Seite"
  lastPage: "Letzte Seite"
  prevPage: "Vorherige Seite"
  nextPage: "Nächste Seite"
  skipBackPages: "{n} Seiten zurück"
  skipForwardPages: "{n} Seiten vor"
</i18n>
