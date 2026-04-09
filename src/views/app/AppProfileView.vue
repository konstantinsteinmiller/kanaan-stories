<script setup lang="ts">
import { computed, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRouter } from 'vue-router'
import SBottomNav from '@/components/atoms/SBottomNav.vue'
import SSelect from '@/components/atoms/SSelect.vue'
import useModels from '@/use/useModels'
import useUser from '@/use/useUser'

const { t } = useI18n()
const { locale } = useI18n({ useScope: 'global' })
const router = useRouter()
const { watchListBooks, removeFromWatchList } = useModels()
const { userLanguage, setSettingValue } = useUser()

// Keep i18n locale in sync with userLanguage
if (locale.value !== userLanguage.value) locale.value = userLanguage.value
watch(userLanguage, value => {
  locale.value = value
})

const languageOptions = computed(() => [
  { value: 'en', label: t('english'), subLabel: 'English', icon: '🇬🇧' },
  { value: 'de', label: t('german'), subLabel: 'Deutsch', icon: '🇩🇪' }
])

function onLanguageChange(value: string | number) {
  setSettingValue('language', String(value))
}

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
</script>

<template lang="pug">
  div(class="app-page min-h-screen w-full")
    header(class="px-5 pb-2 pt-[max(env(safe-area-inset-top),1.25rem)]")
      div(class="max-w-2xl mx-auto flex items-center gap-4")
        div(class="profile-avatar shrink-0 w-16 h-16 rounded-full flex items-center justify-center text-2xl text-white shadow-lg") 👤
        div(class="min-w-0")
          p(class="text-[10px] md:text-xs font-bold uppercase tracking-wider text-white/75") {{ t('profile') }}
          h1(class="text-xl md:text-2xl font-black text-white truncate") {{ t('littleReader') }}

    //- Language
    section(class="px-5 mt-6")
      div(class="max-w-2xl mx-auto")
        h2(class="mb-2 text-[10px] md:text-xs font-black uppercase tracking-wider text-white/85") {{ t('language') }}
        SSelect(
          :model-value="userLanguage"
          :options="languageOptions"
          accent="orange"
          @update:model-value="onLanguageChange"
        )

    //- Watch list
    section(class="px-5 mt-6")
      div(class="max-w-2xl mx-auto")
        div(class="flex items-center justify-between mb-2")
          h2(class="text-[10px] md:text-xs font-black uppercase tracking-wider text-white/85") {{ t('watchList') }}
          span(class="text-xs font-bold text-white/70") {{ watchListBooks.length }}

        div(
          v-if="watchListBooks.length === 0"
          class="rounded-2xl bg-white/90 p-5 text-center text-sm text-[#7F8C8D]"
        ) {{ t('emptyWatchList') }}

        div(v-else class="flex flex-col gap-3")
          div(
            v-for="book in watchListBooks"
            :key="book.id"
            class="watch-row flex items-center gap-3 rounded-2xl bg-white/95 px-3 py-3 shadow-md"
          )
            button(
              type="button"
              @click="openBook(book.id)"
              class="flex items-center gap-3 flex-1 min-w-0 text-left hover:scale-[101%] active:scale-[99%] transition-transform"
            )
              div(
                class="relative shrink-0 w-14 h-16 rounded-xl overflow-hidden"
                :style="{ background: book.cover || 'linear-gradient(135deg,#E67E22,#F1C40F)' }"
              )
                img(
                  v-if="book.previewImage || book.coverImage"
                  :src="book.previewImage || book.coverImage"
                  :alt="t(book.title)"
                  class="absolute inset-0 w-full h-full object-cover"
                  loading="lazy"
                )
              div(class="flex-1 min-w-0")
                p(class="text-base font-black text-[#2C3E50] truncate") {{ t(book.title) }}
                p(class="text-xs text-[#7F8C8D] truncate") {{ book.author }}
            button(
              type="button"
              @click="removeFromWatchList(book.id)"
              :aria-label="t('remove')"
              class="shrink-0 w-9 h-9 rounded-full bg-[#FFF6EC] text-[#E67E22] flex items-center justify-center hover:bg-[#E67E22] hover:text-white transition-colors"
            )
              svg(viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5")
                path(d="M3 6h18")
                path(d="M8 6V4a1 1 0 0 1 1-1h6a1 1 0 0 1 1 1v2")
                path(d="M19 6v14a1 1 0 0 1-1 1H6a1 1 0 0 1-1-1V6")

    div(class="h-32")
    SBottomNav(model-value="profile" :items="navItems" accent="orange" @update:model-value="onNav")
</template>

<style scoped lang="sass">
.app-page
  background: radial-gradient(1200px 600px at 50% -10%, rgba(241, 196, 15, 0.35), transparent 60%), linear-gradient(180deg, #3498DB 0%, #1F6FA8 100%)

.profile-avatar
  background: linear-gradient(135deg, #F1C40F 0%, #E67E22 100%)
</style>

<i18n>
en:
  profile: "Profile"
  littleReader: "Little Reader"
  watchList: "Watch list"
  emptyWatchList: "Your watch list is empty. Tap the bookmark on a story to save it here."
  remove: "Remove"
  language: "Language"
  english: "English"
  german: "German"
de:
  profile: "Profil"
  littleReader: "Kleiner Leser"
  watchList: "Merkliste"
  emptyWatchList: "Deine Merkliste ist leer. Tippe auf das Lesezeichen einer Geschichte, um sie hier zu speichern."
  remove: "Entfernen"
  language: "Sprache"
  english: "Englisch"
  german: "Deutsch"
</i18n>
