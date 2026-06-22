<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// Data lives in public/ and is fetched at runtime so the root pjp_qris.json
// stays the single source of truth (no build-time copy in src/).
const providers = ref([])
const dateUpdated = ref('-')
const loading = ref(true)
const loadError = ref(false)

const byNns = computed(
  () => new Map(providers.value.map((p) => [String(p.nns), p])),
)

onMounted(async () => {
  try {
    const res = await fetch(`${import.meta.env.BASE_URL}pjp_qris.json`)
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    const rawData = await res.json()
    // First entry holds the metadata (date_updated); the rest are providers.
    dateUpdated.value = rawData.find((x) => x.date_updated)?.date_updated ?? '-'
    providers.value = rawData.filter((x) => x.nns)
  } catch (e) {
    loadError.value = true
  } finally {
    loading.value = false
  }
})

// --- i18n ---------------------------------------------------------------
// Indonesian is the default since the app is used in Indonesia. The choice
// is persisted to localStorage. Provider names and the literal QRIS phrase
// "Dicetak oleh:" are intentionally NOT translated.
const translations = {
  id: {
    section1Heading: 'Di mana menemukan kode NNS Anda',
    section1Intro: 'Pada struk QRIS Anda, cari baris yang bertuliskan:',
    section1Note: 'Angka 8 digit itu adalah kode NNS merchant.',
    heroSubtitle:
      'Masukkan kode NNS untuk langsung menemukan penyelenggara jasa pembayarannya.',
    inputPlaceholder: 'mis. 93600002',
    notFound: 'Tidak ada penyelenggara untuk NNS',
    loadError: 'Gagal memuat data penyelenggara. Silakan muat ulang.',
    popularProviders: 'Penyelenggara populer',
    dataUpdated: 'Data diperbarui:',
    providersIndexed: 'penyelenggara terindeks',
    langLabel: 'Bahasa',
  },
  en: {
    section1Heading: 'Where to find your NNS code',
    section1Intro: 'On your QRIS printout, look for a line that reads:',
    section1Note: "That 8-digit number is the merchant's NNS code.",
    heroSubtitle:
      'Enter an NNS code to instantly find its payment service provider.',
    inputPlaceholder: 'e.g. 93600002',
    notFound: 'No provider found for NNS',
    loadError: 'Could not load provider data. Please refresh.',
    popularProviders: 'Popular providers',
    dataUpdated: 'Data updated:',
    providersIndexed: 'providers indexed',
    langLabel: 'Language',
  },
}

const LANGS = ['id', 'en']
const stored =
  typeof localStorage !== 'undefined' ? localStorage.getItem('lang') : null
const lang = ref(LANGS.includes(stored) ? stored : 'id')

const t = computed(() => translations[lang.value])

watch(
  lang,
  (val) => {
    if (typeof localStorage !== 'undefined') localStorage.setItem('lang', val)
    if (typeof document !== 'undefined') document.documentElement.lang = val
  },
  { immediate: true },
)
// -----------------------------------------------------------------------

// Curated highlight list for section 3, in the order the spec requires.
const HIGHLIGHTS = [
  { label: 'BRI', nns: 93600002 },
  { label: 'Mandiri', nns: 93600008 },
  { label: 'BCA', nns: 93600014 },
  { label: 'BNI', nns: 93600009 },
  { label: 'GoPay', nns: 93600914 },
  { label: 'OVO', nns: 93600912 },
  { label: 'DANA', nns: 93600915 },
  { label: 'ShopeePay', nns: 93600918 },
  { label: 'LinkAja', nns: 93600911 },
]

const query = ref('')

const cleanQuery = computed(() => query.value.replace(/\s+/g, '').trim())

const result = computed(() => {
  if (!cleanQuery.value || loading.value) return null
  return byNns.value.get(cleanQuery.value) ?? 'not-found'
})

function pickHighlight(nns) {
  query.value = String(nns)
  document.getElementById('nns-input')?.focus()
}

function tidy(text) {
  const t = (text ?? '').trim()
  return t.length ? t : '-'
}
</script>

<template>
  <div class="relative min-h-screen overflow-hidden bg-[#07030f] text-violet-50">
    <!-- Aurora background blobs -->
    <div class="pointer-events-none absolute inset-0 overflow-hidden">
      <div
        class="aurora absolute -left-32 -top-40 h-[34rem] w-[34rem] rounded-full bg-violet-600/30 blur-[120px]"
      />
      <div
        class="aurora-delayed absolute -right-32 top-20 h-[30rem] w-[30rem] rounded-full bg-fuchsia-600/25 blur-[120px]"
      />
      <div
        class="aurora absolute bottom-0 left-1/3 h-[26rem] w-[26rem] rounded-full bg-purple-700/20 blur-[120px]"
      />
    </div>

    <!-- Language toggle -->
    <div class="absolute right-4 top-4 z-10">
      <div
        class="flex items-center gap-1 rounded-full border border-white/10 bg-white/5 p-1 backdrop-blur-xl"
        role="group"
        :aria-label="t.langLabel"
      >
        <button
          v-for="code in LANGS"
          :key="code"
          type="button"
          class="rounded-full px-3 py-1 text-xs font-semibold uppercase tracking-wider transition"
          :class="
            lang === code
              ? 'bg-gradient-to-r from-violet-500 to-fuchsia-500 text-white shadow-[0_0_16px_-4px_rgba(217,70,239,0.8)]'
              : 'text-violet-200/60 hover:text-violet-100'
          "
          :aria-pressed="lang === code"
          @click="lang = code"
        >
          {{ code }}
        </button>
      </div>
    </div>

    <main
      class="relative mx-auto flex max-w-3xl flex-col gap-12 px-5 py-14 sm:py-20"
    >
      <!-- SECTION 1: how to find the NNS code -->
      <section
        class="rounded-3xl border border-white/10 bg-white/5 p-6 backdrop-blur-xl"
      >
        <h2
          class="mb-3 flex items-center gap-2 text-xs font-semibold uppercase tracking-[0.2em] text-violet-300"
        >
          <span
            class="inline-block h-2 w-2 rounded-full bg-fuchsia-400 shadow-[0_0_10px_2px_rgba(217,70,239,0.8)]"
          />
          {{ t.section1Heading }}
        </h2>
        <p class="leading-relaxed text-violet-100/80">
          {{ t.section1Intro }}
        </p>
        <div
          class="mt-3 rounded-xl border border-violet-400/20 bg-black/40 px-4 py-3 font-mono text-sm text-violet-200"
        >
          Dicetak oleh:
          <span class="font-semibold text-fuchsia-300">12345678</span>
        </div>
        <p class="mt-3 text-sm text-violet-200/60">
          {{ t.section1Note }}
        </p>
      </section>

      <!-- SECTION 2: HERO title + input + result -->
      <section class="py-4 text-center">
        <h1
          class="bg-gradient-to-r from-violet-300 via-fuchsia-300 to-violet-200 bg-clip-text text-5xl font-black tracking-tight text-transparent drop-shadow-[0_0_25px_rgba(168,85,247,0.35)] sm:text-6xl"
        >
          PJP QRIS Check
        </h1>
        <p class="mx-auto mt-3 max-w-md text-violet-200/70">
          {{ t.heroSubtitle }}
        </p>

        <div
          class="search-glow mx-auto mt-8 max-w-xl rounded-2xl ring-1 ring-violet-500/40 transition"
        >
          <input
            id="nns-input"
            v-model="query"
            type="text"
            inputmode="numeric"
            :placeholder="t.inputPlaceholder"
            class="w-full rounded-2xl border-0 bg-violet-950/50 px-6 py-5 text-center text-2xl font-semibold tracking-[0.25em] text-violet-50 placeholder:font-normal placeholder:tracking-normal placeholder:text-violet-400/40 outline-none backdrop-blur-xl"
          />
        </div>

        <p v-if="loadError" class="mt-4 text-sm text-rose-300">
          {{ t.loadError }}
        </p>

        <!-- result -->
        <transition
          enter-active-class="transition duration-300 ease-out"
          enter-from-class="opacity-0 translate-y-2"
          enter-to-class="opacity-100 translate-y-0"
        >
          <div v-if="result" class="mx-auto mt-6 max-w-xl text-left">
            <div
              v-if="result === 'not-found'"
              class="rounded-2xl border border-rose-500/30 bg-rose-950/30 px-5 py-4 text-center text-rose-200 backdrop-blur-xl"
            >
              {{ t.notFound }}
              <span class="font-mono font-semibold">{{ cleanQuery }}</span
              >.
            </div>

            <div
              v-else
              class="rounded-2xl border border-fuchsia-400/40 bg-gradient-to-br from-violet-900/70 to-fuchsia-900/40 p-6 backdrop-blur-xl shadow-[0_0_40px_-8px_rgba(217,70,239,0.5)]"
            >
              <div
                class="font-mono text-xs font-semibold uppercase tracking-[0.2em] text-fuchsia-300"
              >
                NNS {{ result.nns }}
              </div>
              <div class="mt-2 text-2xl font-bold text-white">
                {{ tidy(result.nama_penyelenggara) }}
              </div>
              <div class="mt-1 text-violet-200/80">
                {{ tidy(result.nama_produk) }}
              </div>
            </div>
          </div>
        </transition>
      </section>

      <!-- SECTION 3: highlighted providers + footer -->
      <section
        class="rounded-3xl border border-white/10 bg-white/5 p-6 backdrop-blur-xl"
      >
        <h2
          class="mb-4 text-xs font-semibold uppercase tracking-[0.2em] text-violet-300"
        >
          {{ t.popularProviders }}
        </h2>
        <ul class="grid gap-2.5 sm:grid-cols-3">
          <li v-for="h in HIGHLIGHTS" :key="h.nns">
            <button
              type="button"
              class="group flex w-full flex-col items-start gap-1 rounded-xl border border-white/10 bg-black/20 px-4 py-3 text-left transition duration-200 hover:-translate-y-0.5 hover:border-fuchsia-400/60 hover:bg-violet-800/40 hover:shadow-[0_0_24px_-6px_rgba(217,70,239,0.6)]"
              @click="pickHighlight(h.nns)"
            >
              <span class="font-semibold text-violet-50 group-hover:text-white">
                {{ h.label }}
              </span>
              <span class="font-mono text-sm text-fuchsia-300/90">
                {{ h.nns }}
              </span>
            </button>
          </li>
        </ul>

        <footer
          class="mt-6 border-t border-white/10 pt-4 text-center text-xs text-violet-300/50"
        >
          <p>{{ t.dataUpdated }} {{ dateUpdated }}</p>
          <p class="mt-1">
            PJP QRIS Check &middot; {{ providers.length }} {{ t.providersIndexed }}
          </p>
        </footer>
      </section>
    </main>
  </div>
</template>
