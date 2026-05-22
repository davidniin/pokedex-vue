<template>
  <div class="type-select" ref="root">
    <button
      class="ts-btn"
      :class="{ 'ts-btn--active': modelValue }"
      @click="open = !open"
    >
      <span v-if="modelValue" :class="`ts-dot type-${modelValue}`"></span>
      <span class="ts-label">{{ modelValue ? capitalize(modelValue) : placeholder }}</span>
      <svg class="ts-chevron" :class="{ 'ts-chevron--open': open }"
           viewBox="0 0 10 6" fill="none">
        <path d="M1 1l4 4 4-4" stroke="currentColor" stroke-width="1.5"
              stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </button>

    <Transition name="ts-drop">
      <div v-if="open" class="ts-panel">
        <button class="ts-item ts-item--clear" @click="select('')">
          All {{ placeholder }}
        </button>
        <button
          v-for="t in types"
          :key="t"
          :class="['ts-item', { 'ts-item--active': modelValue === t }]"
          @click="select(t)"
        >
          <span :class="`ts-dot type-${t}`"></span>
          {{ capitalize(t) }}
        </button>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

defineProps({
  modelValue: String,
  placeholder: { type: String, default: 'Type' },
})
const emit = defineEmits(['update:modelValue'])

const open = ref(false)
const root = ref(null)

const types = [
  'normal','fire','water','grass','electric','ice','fighting','poison',
  'ground','flying','psychic','bug','rock','ghost','dragon','dark','steel','fairy',
]

function select(t) { emit('update:modelValue', t); open.value = false }
function capitalize(s) { return s.charAt(0).toUpperCase() + s.slice(1) }

function onOutside(e) {
  if (root.value && !root.value.contains(e.target)) open.value = false
}
onMounted(()  => document.addEventListener('mousedown', onOutside))
onUnmounted(() => document.removeEventListener('mousedown', onOutside))
</script>

<style scoped>
.type-select {
  position: relative;
}

/* ── Trigger button ── */
.ts-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px 8px 14px;
  border: 1.5px solid var(--border);
  border-radius: 30px;
  background: #fff;
  font-size: 13px;
  font-weight: 500;
  color: var(--text);
  cursor: pointer;
  white-space: nowrap;
  transition: border-color 0.15s, background 0.15s;
  outline: none;
}

.ts-btn:hover       { border-color: #c8c8e0; }
.ts-btn--active     { border-color: var(--accent); background: #fff8f9; color: var(--accent); font-weight: 600; }

.ts-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  flex-shrink: 0;
}

.ts-label { line-height: 1; }

.ts-chevron {
  width: 10px;
  height: 6px;
  color: var(--muted);
  flex-shrink: 0;
  transition: transform 0.2s;
}
.ts-chevron--open { transform: rotate(180deg); }
.ts-btn--active .ts-chevron { color: var(--accent); }

/* ── Dropdown panel ── */
.ts-panel {
  position: absolute;
  top: calc(100% + 6px);
  left: 0;
  z-index: 100;
  background: #fff;
  border: 1.5px solid var(--border);
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.1);
  padding: 6px;
  min-width: 150px;
  max-height: 260px;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: var(--border) transparent;
}

/* ── Items ── */
.ts-item {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
  padding: 7px 10px;
  border: none;
  background: none;
  border-radius: 7px;
  font-size: 12px;
  font-weight: 500;
  color: var(--text);
  cursor: pointer;
  text-align: left;
  transition: background 0.12s;
}
.ts-item:hover          { background: var(--bg); }
.ts-item--active        { background: #fde8ec; color: var(--accent); font-weight: 700; }
.ts-item--clear         { color: var(--muted); font-size: 11px; margin-bottom: 2px; }
.ts-item--clear:hover   { background: var(--bg); color: var(--text); }

/* ── Transition ── */
.ts-drop-enter-active { transition: opacity 0.15s ease, transform 0.15s ease; }
.ts-drop-leave-active { transition: opacity 0.1s ease,  transform 0.1s ease;  }
.ts-drop-enter-from   { opacity: 0; transform: translateY(-6px); }
.ts-drop-leave-to     { opacity: 0; transform: translateY(-4px); }
</style>
