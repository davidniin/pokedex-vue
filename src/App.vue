<template>
  <!-- Initial loading overlay -->
  <Transition name="overlay-fade">
    <div v-if="initialLoading" class="initial-loading">
      <div class="spinner"></div>
      <p>Loading Pokédex…</p>
    </div>
  </Transition>

  <div class="page" :class="{ 'page--hidden': initialLoading }">
    <div class="bg-circle bg-circle-1"></div>
    <div class="bg-circle bg-circle-2"></div>

    <div class="container">
      <div class="layout">

        <!-- Left panel -->
        <div class="left-panel">
          <FilterBar
            v-model:search="search"
            v-model:sortOrder="sortOrder"
            v-model:filterType="filterType"
            v-model:filterStrength="filterStrength"
            v-model:filterWeakness="filterWeakness"
            @reset="resetFilters"
          />

          <div v-if="error" class="error-msg">{{ error }}</div>

          <div v-else class="grid">
            <PokemonCard
              v-for="p in displayedList"
              :key="p.id"
              :pokemon="p"
              :selected="selected?.id === p.id"
              @select="selectPokemon"
            />
          </div>

          <!-- Sentinel: no API calls here, just reveals more from the loaded array -->
          <div ref="sentinel" class="sentinel">
            <p v-if="allDisplayed && pokemon.length > 0" class="sentinel-end">
              All {{ pokemon.length }} Pokémon loaded
            </p>
          </div>
        </div>

        <!-- Right panel -->
        <div class="right-panel" :class="{ 'mobile-open': selected }">
          <div v-if="selected" class="mobile-backdrop" @click="selected = null"></div>
          <PokemonDetail
            v-if="selected"
            :pokemon="selected"
            :prevPokemon="prevPokemon"
            :nextPokemon="nextPokemon"
            @navigate="selectPokemon"
            @close="selected = null"
          />
          <div v-else class="empty-detail">
            <img
              src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/25.png"
              alt="Pikachu"
              class="empty-img"
            />
            <p>Select a Pokémon</p>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import FilterBar from './components/FilterBar.vue'
import PokemonCard from './components/PokemonCard.vue'
import PokemonDetail from './components/PokemonDetail.vue'

const TOTAL = 1025   // Gen 1–9, same as original
const STEP  = 40     // Items revealed per scroll tick

const pokemon       = ref([])   // Full list — loaded once on mount
const visibleCount  = ref(STEP) // How many cards are currently visible
const initialLoading = ref(true)
const error         = ref(null)
const selected      = ref(null)
const sentinel      = ref(null)

const search          = ref('')
const sortOrder       = ref('num-asc')
const filterType      = ref('')
const filterStrength  = ref('')
const filterWeakness  = ref('')

// ── Derived lists ──────────────────────────────────────────────
const filteredList = computed(() => {
  const q = search.value.trim().toLowerCase()
  let list = pokemon.value.filter(p => {
    // Match against name with hyphens replaced (e.g. "mr-mime" → "mr mime")
    const matchSearch = !q
      || p.name.replaceAll('-', ' ').includes(q)
      || String(p.id).includes(q)
    const matchType     = !filterType.value     || p.types.includes(filterType.value)
    const matchStrength = !filterStrength.value || p.strengths.includes(filterStrength.value)
    const matchWeakness = !filterWeakness.value || p.weakTo.includes(filterWeakness.value)
    return matchSearch && matchType && matchStrength && matchWeakness
  })
  return list.sort((a, b) => {
    switch (sortOrder.value) {
      case 'num-desc':  return b.id - a.id
      case 'name-asc':  return a.name.localeCompare(b.name)
      case 'name-desc': return b.name.localeCompare(a.name)
      default:          return a.id - b.id
    }
  })
})

// Only the slice that is actually rendered
const displayedList = computed(() =>
  filteredList.value.slice(0, visibleCount.value)
)

const allDisplayed = computed(() =>
  visibleCount.value >= filteredList.value.length
)

// Prev / next for the detail panel
const selectedIndex = computed(() =>
  filteredList.value.findIndex(p => p.id === selected.value?.id)
)

const prevPokemon = computed(() => {
  const i = selectedIndex.value
  return i > 0 ? filteredList.value[i - 1] : null
})

const nextPokemon = computed(() => {
  const i = selectedIndex.value
  return i >= 0 && i < filteredList.value.length - 1
    ? filteredList.value[i + 1]
    : null
})

// ── Reset visible window when filters change ───────────────────
watch([search, filterType, filterStrength, filterWeakness, sortOrder], () => {
  visibleCount.value = STEP
})

// ── Data loading: same strategy as the original ────────────────
// 1 call for all names  +  18 parallel calls for types  =  19 total API calls.
// Sprites come directly from GitHub — no individual pokemon calls needed.
async function loadAllPokemon() {
  try {
    // Step 1 — all names in one request
    const listData = await fetch(
      `https://pokeapi.co/api/v2/pokemon?limit=${TOTAL}`
    ).then(r => r.json())

    // Build the pokemon objects and an id→object map for fast type assignment
    const idMap = new Map()
    const all = listData.results.map((p, i) => {
      const entry = {
        id:        i + 1,
        name:      p.name,
        types:     [],
        strengths: [],
        weakTo:    [],
        sprite: (i + 1) <= 649
          ? `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/animated/${i + 1}.gif`
          : `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${i + 1}.png`,
      }
      idMap.set(i + 1, entry)
      return entry
    })

    // Step 2 — all 18 type endpoints in parallel, assign types
    const typeData = await Promise.all(
      Array.from({ length: 18 }, (_, i) =>
        fetch(`https://pokeapi.co/api/v2/type/${i + 1}`).then(r => r.json())
      )
    )

    for (const type of typeData) {
      for (const entry of type.pokemon) {
        const id = parseInt(entry.pokemon.url.split('/').filter(Boolean).pop())
        idMap.get(id)?.types.push(type.name)
      }
    }

    // Build damage lookups from already-fetched type data (no extra API calls)
    const dmgTo   = {}
    const dmgFrom = {}
    for (const type of typeData) {
      dmgTo[type.name]   = type.damage_relations.double_damage_to.map(t => t.name)
      dmgFrom[type.name] = type.damage_relations.double_damage_from.map(t => t.name)
    }

    for (const entry of all) {
      const strong = new Set()
      const weak   = new Set()
      for (const typeName of entry.types) {
        for (const t of dmgTo[typeName]   ?? []) strong.add(t)
        for (const t of dmgFrom[typeName] ?? []) weak.add(t)
      }
      entry.strengths = [...strong]
      entry.weakTo    = [...weak]
    }

    pokemon.value = all
    selected.value = all[0]

  } catch (e) {
    error.value = 'Failed to load Pokédex. Please try again.'
    console.error(e)
  } finally {
    initialLoading.value = false
  }
}

// ── Infinite scroll — just increments visibleCount, no fetch ──
let observer = null

onMounted(async () => {
  await loadAllPokemon()

  observer = new IntersectionObserver(
    ([entry]) => {
      if (entry.isIntersecting && !allDisplayed.value) {
        visibleCount.value = Math.min(
          visibleCount.value + STEP,
          filteredList.value.length
        )
      }
    },
    { rootMargin: '300px' }
  )

  if (sentinel.value) observer.observe(sentinel.value)
})

onUnmounted(() => observer?.disconnect())

// ── Helpers ───────────────────────────────────────────────────
function selectPokemon(poke) {
  selected.value = pokemon.value.find(p => p.id === poke.id) ?? poke
}

function resetFilters() {
  search.value         = ''
  filterType.value     = ''
  filterStrength.value = ''
  filterWeakness.value = ''
  sortOrder.value      = 'num-asc'
}
</script>

<style scoped>
/* ── Initial loading overlay ── */
.initial-loading {
  position: fixed;
  inset: 0;
  background: var(--bg);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  z-index: 999;
  font-size: 14px;
  color: var(--muted);
}

.spinner {
  width: 36px;
  height: 36px;
  border: 3px solid var(--border);
  border-top-color: var(--accent);
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

.overlay-fade-leave-active { transition: opacity 0.4s ease; }
.overlay-fade-leave-to     { opacity: 0; }

.page--hidden { visibility: hidden; }

/* ── Layout ── */
.page {
  min-height: 100vh;
  position: relative;
  overflow: hidden;
}

.bg-circle {
  position: fixed;
  border-radius: 50%;
  pointer-events: none;
  z-index: 0;
}

.bg-circle-1 {
  width: 320px;
  height: 320px;
  background: rgba(200, 200, 220, 0.25);
  top: -80px;
  left: -80px;
}

.bg-circle-2 {
  width: 200px;
  height: 200px;
  background: rgba(200, 200, 220, 0.15);
  top: 60px;
  left: 60px;
}

.container {
  max-width: 1240px;
  margin: 0 auto;
  padding: 24px 20px;
  position: relative;
  z-index: 1;
}

.layout {
  display: flex;
  gap: 20px;
  align-items: flex-start;
}

/* ── Left panel ── */
.left-panel {
  flex: 1;
  min-width: 0;
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 14px;
}

/* ── Sentinel ── */
.sentinel {
  min-height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px 0;
}

.sentinel-end {
  font-size: 12px;
  color: var(--muted);
}

/* ── Right panel ── */
.right-panel {
  width: 360px;
  flex-shrink: 0;
  position: sticky;
  top: 24px;
  height: calc(100vh - 48px);
}

.mobile-backdrop { display: none; }

.empty-detail {
  background: var(--card);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  color: var(--muted);
  font-size: 14px;
}

.empty-img {
  width: 120px;
  height: 120px;
  opacity: 0.4;
}

.error-msg {
  text-align: center;
  color: var(--accent);
  padding: 40px;
}

@media (max-width: 900px) {
  .right-panel {
    display: none;
    position: fixed;
    inset: 0;
    z-index: 200;
    width: 100%;
    height: 100%;
    align-items: flex-end;
  }

  .right-panel.mobile-open {
    display: flex;
  }

  .mobile-backdrop {
    display: block;
    position: absolute;
    inset: 0;
    background: rgba(0, 0, 0, 0.45);
    backdrop-filter: blur(2px);
  }

  .right-panel > .detail-panel-wrapper,
  .right-panel > :not(.mobile-backdrop) {
    position: relative;
    z-index: 1;
    width: 100%;
    max-height: 88vh;
    border-radius: var(--radius) var(--radius) 0 0;
  }
}
</style>
