<template>
  <div class="detail-panel">

    <button class="close-btn" @click="$emit('close')" aria-label="Close">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"
           stroke-linecap="round" stroke-linejoin="round">
        <line x1="18" y1="6" x2="6" y2="18"/>
        <line x1="6" y1="6" x2="18" y2="18"/>
      </svg>
    </button>

    <Transition name="slide-fade" mode="out-in">
      <div v-if="loading" key="loading" class="loading">
        <div class="spinner"></div>
      </div>

      <div v-else-if="detail" :key="detail.id" class="content">

        <!-- ① Hero: centrado -->
        <div class="hero">
          <img :src="detail.sprite" :alt="detail.name" class="artwork" />
          <p class="dex-num">#{{ String(detail.id).padStart(3, '0') }}</p>
          <h2 class="poke-name">{{ formatName(detail.name) }}</h2>
          <div class="meta-row">
            <span
              v-for="t in detail.types"
              :key="t"
              :class="`type-badge type-${t}`"
            >{{ t.toUpperCase() }}</span>
            <span class="gen-badge">{{ detail.generation }}</span>
          </div>
          <p class="flavor-text">{{ detail.flavorText }}</p>
        </div>

        <!-- ② Abilities + Height in one row -->
        <section class="section">
          <div class="info-row">
            <div class="info-cell">
              <p class="cell-label">ABILITIES</p>
              <div class="abilities">
                <span
                  v-for="ab in detail.abilities"
                  :key="ab.name"
                  :class="['ability-chip', { hidden: ab.isHidden }]"
                  :title="ab.isHidden ? 'Hidden ability' : undefined"
                >
                  {{ formatName(ab.name) }}
                  <svg v-if="ab.isHidden" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                       stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94"/>
                    <path d="M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19"/>
                    <line x1="1" y1="1" x2="23" y2="23"/>
                  </svg>
                </span>
              </div>
            </div>
            <div class="info-cell info-cell-height">
              <p class="cell-label">HEIGHT</p>
              <p class="cell-value">{{ detail.height }}m</p>
            </div>
          </div>
        </section>

        <!-- ③ Strengths & Weaknesses side by side -->
        <section class="section">
          <div class="sw-grid">
            <div class="sw-cell">
              <p class="cell-label">STRENGTHS</p>
              <div class="type-circle-list">
                <template v-for="group in groupedStrengths" :key="group.mult">
                  <span class="mult-label">{{ group.mult }}</span>
                  <span
                    v-for="w in group.types"
                    :key="w"
                    :class="`type-circle type-${w}`"
                    :data-tooltip="`${formatName(w)} — ${group.mult}`"
                  ></span>
                </template>
                <span v-if="!groupedStrengths.length" class="none-label">—</span>
              </div>
            </div>
            <div class="sw-cell">
              <p class="cell-label">WEAKNESSES</p>
              <div class="type-circle-list">
                <template v-for="group in groupedWeaknesses" :key="group.mult">
                  <span class="mult-label">{{ group.mult }}</span>
                  <span
                    v-for="w in group.types"
                    :key="w"
                    :class="`type-circle type-${w}`"
                    :data-tooltip="`${formatName(w)} — ${group.mult}`"
                  ></span>
                </template>
                <span v-if="!groupedWeaknesses.length" class="none-label">—</span>
              </div>
            </div>
          </div>
        </section>

        <!-- ④ Stats -->
        <section class="section">
          <p class="cell-label cell-label-center">STATS</p>
          <div class="stat-circles">
            <div v-for="s in statList" :key="s.key" class="stat-circle-wrap">
              <div :class="`stat-circle stat-${s.key}`" :data-tooltip="s.fullLabel">
                <span class="stat-abbr">{{ s.label }}</span>
              </div>
              <span class="stat-num">{{ detail.stats[s.key] }}</span>
            </div>
            <div class="stat-circle-wrap">
              <div class="stat-circle stat-tot" data-tooltip="Total">
                <span class="stat-abbr">TOT</span>
              </div>
              <span class="stat-num">{{ totalStats }}</span>
            </div>
          </div>
        </section>

        <!-- ⑤ Evolution -->
        <section class="section" v-if="detail.evolutionChain.length > 1">
          <p class="cell-label cell-label-center">EVOLUTION</p>
          <div class="evo-chain">
            <template v-for="(evo, i) in detail.evolutionChain" :key="evo.id">
              <div
                :class="['evo-item', { active: evo.id === detail.id }]"
                @click="$emit('navigate', { id: evo.id, name: evo.name })"
              >
                <img
                  :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${evo.id}.png`"
                  :alt="evo.name"
                />
                <span>{{ formatName(evo.name) }}</span>
              </div>
              <div v-if="i < detail.evolutionChain.length - 1" class="evo-arrow">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M5 12h14M13 6l6 6-6 6"/>
                </svg>
                <span v-if="detail.evolutionChain[i + 1].level" class="evo-level">
                  Lv {{ detail.evolutionChain[i + 1].level }}
                </span>
              </div>
            </template>
          </div>
        </section>

      </div>
    </Transition>

    <!-- Nav bar -->
    <div v-if="detail && !loading" class="nav-bar">
      <button
        class="nav-btn"
        :disabled="!prevPokemon"
        @click="prevPokemon && $emit('navigate', prevPokemon)"
      >
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
             stroke-linecap="round" stroke-linejoin="round">
          <path d="M15 18l-6-6 6-6"/>
        </svg>
        <span v-if="prevPokemon">
          <img :src="prevPokemon.sprite" :alt="prevPokemon.name" class="nav-sprite" />
          {{ formatName(prevPokemon.name) }}
          <em>#{{ String(prevPokemon.id).padStart(3, '0') }}</em>
        </span>
      </button>
      <button
        class="nav-btn nav-btn-right"
        :disabled="!nextPokemon"
        @click="nextPokemon && $emit('navigate', nextPokemon)"
      >
        <span v-if="nextPokemon">
          <em>#{{ String(nextPokemon.id).padStart(3, '0') }}</em>
          {{ formatName(nextPokemon.name) }}
          <img :src="nextPokemon.sprite" :alt="nextPokemon.name" class="nav-sprite" />
        </span>
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
             stroke-linecap="round" stroke-linejoin="round">
          <path d="M9 18l6-6-6-6"/>
        </svg>
      </button>
    </div>

  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue'

const props = defineProps({
  pokemon: Object,
  prevPokemon: Object,
  nextPokemon: Object,
})
defineEmits(['navigate', 'close'])

const detail  = ref(null)
const loading = ref(false)
const cache   = new Map()

const statList = [
  { key: 'hp',  label: 'HP',  fullLabel: 'Hit Points'  },
  { key: 'atk', label: 'ATK', fullLabel: 'Attack'      },
  { key: 'def', label: 'DEF', fullLabel: 'Defense'     },
  { key: 'spa', label: 'SpA', fullLabel: 'Sp. Attack'  },
  { key: 'spd', label: 'SpD', fullLabel: 'Sp. Defense' },
  { key: 'spe', label: 'SPE', fullLabel: 'Speed'       },
]

const totalStats = computed(() => {
  if (!detail.value) return 0
  return Object.values(detail.value.stats).reduce((a, b) => a + b, 0)
})

function groupByMult(arr) {
  const groups = {}
  for (const w of arr) {
    if (!groups[w.mult]) groups[w.mult] = []
    groups[w.mult].push(w.type)
  }
  return Object.entries(groups)
    .sort(([a], [b]) => b.localeCompare(a))
    .map(([mult, types]) => ({ mult, types }))
}

const groupedWeaknesses = computed(() =>
  detail.value ? groupByMult(detail.value.weaknesses) : []
)
const groupedStrengths = computed(() =>
  detail.value ? groupByMult(detail.value.strengths) : []
)

watch(
  () => props.pokemon,
  async (poke) => {
    if (!poke) return
    if (cache.has(poke.id)) { detail.value = cache.get(poke.id); return }

    loading.value = true
    detail.value  = null

    try {
      let pokeData, speciesData
      if (poke.id > 10000) {
        pokeData    = await fetch(`https://pokeapi.co/api/v2/pokemon/${poke.id}`).then(r => r.json())
        speciesData = await fetch(pokeData.species.url).then(r => r.json())
      } else {
        ;[pokeData, speciesData] = await Promise.all([
          fetch(`https://pokeapi.co/api/v2/pokemon/${poke.id}`).then(r => r.json()),
          fetch(`https://pokeapi.co/api/v2/pokemon-species/${poke.id}`).then(r => r.json()),
        ])
      }

      const typeNames = pokeData.types.map(t => t.type.name)
      const [evoData, ...typeDataArr] = await Promise.all([
        fetch(speciesData.evolution_chain.url).then(r => r.json()),
        ...typeNames.map(t => fetch(`https://pokeapi.co/api/v2/type/${t}`).then(r => r.json())),
      ])

      const result = {
        id:         pokeData.id,
        name:       pokeData.name,
        generation: formatGeneration(speciesData.generation.name),
        types:      typeNames,
        sprite:     pokeData.id <= 649
          ? `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/versions/generation-v/black-white/animated/${pokeData.id}.gif`
          : `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokeData.id}.png`,
        flavorText: (speciesData.flavor_text_entries.find(e => e.language.name === 'en')?.flavor_text ?? '')
          .replace(/\f/g, ' '),
        abilities:  pokeData.abilities.map(a => ({ name: a.ability.name, isHidden: a.is_hidden })),
        height:     pokeData.height / 10,
        weaknesses: calculateWeaknesses(typeDataArr),
        strengths:  calculateStrengths(typeDataArr),
        stats: {
          hp:  pokeData.stats[0].base_stat,
          atk: pokeData.stats[1].base_stat,
          def: pokeData.stats[2].base_stat,
          spa: pokeData.stats[3].base_stat,
          spd: pokeData.stats[4].base_stat,
          spe: pokeData.stats[5].base_stat,
        },
        evolutionChain: flattenChain(evoData.chain),
      }

      cache.set(poke.id, result)
      detail.value = result
    } catch (e) {
      console.error('Failed to load detail:', e)
    } finally {
      loading.value = false
    }
  },
  { immediate: true }
)

function calculateWeaknesses(typeDataArr) {
  const m = {}
  for (const td of typeDataArr) {
    const dr = td.damage_relations
    for (const t of dr.double_damage_from) m[t.name] = (m[t.name] || 1) * 2
    for (const t of dr.half_damage_from)   m[t.name] = (m[t.name] || 1) * 0.5
    for (const t of dr.no_damage_from)     m[t.name] = 0
  }
  return Object.entries(m).filter(([, v]) => v > 1)
    .map(([type, v]) => ({ type, mult: v >= 4 ? '4x' : '2x' }))
}

function calculateStrengths(typeDataArr) {
  const m = {}
  for (const td of typeDataArr) {
    const dr = td.damage_relations
    for (const t of dr.double_damage_to) m[t.name] = (m[t.name] || 1) * 2
    for (const t of dr.half_damage_to)   m[t.name] = (m[t.name] || 1) * 0.5
    for (const t of dr.no_damage_to)     m[t.name] = 0
  }
  return Object.entries(m).filter(([, v]) => v > 1)
    .map(([type, v]) => ({ type, mult: v >= 4 ? '4x' : '2x' }))
}

function flattenChain(node, result = []) {
  const id    = parseInt(node.species.url.split('/').filter(Boolean).pop())
  const level = node.evolution_details[0]?.min_level ?? null
  result.push({ name: node.species.name, id, level })
  for (const next of node.evolves_to) flattenChain(next, result)
  return result
}

function formatGeneration(gen) {
  return `Gen ${gen.split('-')[1].toUpperCase()}`
}

function formatName(s) {
  if (!s) return ''
  return s.split('-').map(w => w.charAt(0).toUpperCase() + w.slice(1)).join(' ')
}
</script>

<style scoped>
/* ── Panel shell ── */
.detail-panel {
  background: var(--card);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  padding: 14px 14px 0;
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  scrollbar-width: none;
  -ms-overflow-style: none;
  position: relative;
}
.detail-panel::-webkit-scrollbar { display: none; }

/* ── Close ── */
.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 26px;
  height: 26px;
  border: none;
  background: var(--bg);
  border-radius: 7px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--muted);
  z-index: 10;
  transition: background 0.15s, color 0.15s;
}
.close-btn:hover { background: var(--border); color: var(--text); }
.close-btn svg   { width: 13px; height: 13px; }

/* ── Transition ── */
.slide-fade-enter-active { transition: opacity 0.2s ease, transform 0.2s ease; }
.slide-fade-leave-active { transition: opacity 0.1s ease, transform 0.1s ease; }
.slide-fade-enter-from   { opacity: 0; transform: translateY(10px); }
.slide-fade-leave-to     { opacity: 0; transform: translateY(-6px); }

/* ── Loading ── */
.loading {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 200px;
}
.spinner {
  width: 26px; height: 26px;
  border: 3px solid var(--border);
  border-top-color: var(--accent);
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* ── Content wrapper ── */
.content {
  display: flex;
  flex-direction: column;
  gap: 0;
  flex: 1;
}

/* ── Section divider ── */
.section {
  border-top: 1px solid var(--border);
  padding: 10px 0;
}

/* ── Shared label ── */
.cell-label {
  font-size: 8px;
  font-weight: 700;
  letter-spacing: 1.2px;
  color: var(--muted);
  margin-bottom: 5px;
  text-transform: uppercase;
}
.cell-label-center { text-align: center; }

/* ── ① Hero ── */
.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 4px 0 12px;
  text-align: center;
}

.artwork {
  width: 150px;
  height: 150px;
  object-fit: contain;
  image-rendering: pixelated;
}

.dex-num {
  font-size: 11px;
  color: var(--muted);
  font-weight: 500;
}

.poke-name {
  font-size: 20px;
  font-weight: 800;
  letter-spacing: -0.3px;
}

.meta-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
  flex-wrap: wrap;
}

.type-badge {
  font-size: 10px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 20px;
  letter-spacing: 0.4px;
}

.gen-badge {
  font-size: 10px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 20px;
  background: var(--border);
  color: var(--muted);
  letter-spacing: 0.4px;
}

.flavor-text {
  font-size: 11px;
  line-height: 1.6;
  color: var(--muted);
  max-width: 280px;
}

/* ── ② Abilities + Height ── */
.info-row {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 8px;
  align-items: start;
}

.info-cell { }
.info-cell-height { text-align: center; min-width: 52px; }

.abilities {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.ability-chip {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 10px;
  border-radius: 20px;
  border: 1.5px solid var(--border);
  background: #fff;
  font-size: 10px;
  font-weight: 600;
  color: var(--text);
  white-space: nowrap;
}
.ability-chip.hidden { background: var(--bg); color: var(--muted); }
.ability-chip svg    { width: 10px; height: 10px; color: var(--muted); flex-shrink: 0; }

.cell-value {
  font-size: 15px;
  font-weight: 800;
  color: var(--text);
}

/* ── ③ Strengths & Weaknesses ── */
.sw-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

.sw-cell {
  background: var(--bg);
  border-radius: 8px;
  padding: 8px;
}

.type-circle-list {
  display: flex;
  align-items: center;
  gap: 3px;
  flex-wrap: wrap;
}

.mult-label {
  font-size: 8px;
  font-weight: 700;
  color: var(--muted);
  background: var(--border);
  padding: 1px 4px;
  border-radius: 3px;
}

.none-label {
  font-size: 11px;
  color: var(--muted);
}

/* Type circle */
.type-circle {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  display: inline-block;
  position: relative;
  cursor: default;
  flex-shrink: 0;
}

/* ── Tooltip ── */
[data-tooltip] { position: relative; }
[data-tooltip]::after {
  content: attr(data-tooltip);
  position: absolute;
  bottom: calc(100% + 5px);
  left: 50%;
  transform: translateX(-50%);
  background: #1a1a2e;
  color: #fff;
  padding: 3px 7px;
  border-radius: 5px;
  font-size: 9px;
  font-weight: 600;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.15s;
  z-index: 30;
}
[data-tooltip]:hover::after { opacity: 1; }

/* ── ④ Stats ── */
.stat-circles {
  display: flex;
  gap: 4px;
  justify-content: center;
  flex-wrap: wrap;
  margin-top: 2px;
}

.stat-circle-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
}

.stat-circle {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: default;
}

.stat-abbr {
  font-size: 7px;
  font-weight: 800;
  color: #fff;
  letter-spacing: 0.1px;
  pointer-events: none;
}

.stat-num {
  font-size: 10px;
  font-weight: 700;
  color: var(--text);
}

.stat-hp  { background: #ff4757; }
.stat-atk { background: #ff6b35; }
.stat-def { background: #a3cb38; }
.stat-spa { background: #3c6be3; }
.stat-spd { background: #1dd1a1; }
.stat-spe { background: #9b59b6; }
.stat-tot { background: #2c3e50; }

/* ── ⑤ Evolution ── */
.evo-chain {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  flex-wrap: wrap;
}

.evo-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  cursor: pointer;
  padding: 4px 6px;
  border-radius: 8px;
  transition: background 0.2s;
  font-size: 9px;
  font-weight: 600;
}
.evo-item:hover  { background: var(--bg); }
.evo-item.active { background: #fde8ec; color: var(--accent); }
.evo-item img    { width: 40px; height: 40px; image-rendering: pixelated; }

.evo-arrow {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1px;
  color: var(--muted);
}
.evo-arrow svg { width: 14px; height: 14px; }

.evo-level {
  font-size: 8px;
  font-weight: 600;
  color: var(--muted);
}

/* ── Nav bar ── */
.nav-bar {
  display: flex;
  justify-content: space-between;
  border-top: 1px solid var(--border);
  padding: 8px 0;
  margin-top: auto;
  position: sticky;
  bottom: 0;
  background: var(--card);
}

.nav-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  background: none;
  border: none;
  cursor: pointer;
  color: var(--text);
  font-size: 10px;
  font-weight: 600;
  padding: 4px 5px;
  border-radius: 6px;
  transition: background 0.2s;
  max-width: 48%;
}
.nav-btn:hover:not(:disabled) { background: var(--bg); }
.nav-btn:disabled { color: var(--muted); cursor: default; }
.nav-btn svg { width: 14px; height: 14px; flex-shrink: 0; }

.nav-btn span {
  display: flex;
  align-items: center;
  gap: 3px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.nav-btn em {
  font-style: normal;
  color: var(--muted);
  font-size: 9px;
}

.nav-btn-right { margin-left: auto; flex-direction: row-reverse; }

.nav-sprite {
  width: 22px;
  height: 22px;
  image-rendering: pixelated;
}
</style>
