<template>
  <div class="filter-bar">
    <!-- Row 1: search -->
    <div class="row row-search">
      <div class="search-wrap">
        <input
          :value="search"
          @input="$emit('update:search', $event.target.value)"
          type="text"
          placeholder="Search your Pokémon!"
          class="search-input"
        />
        <button class="search-btn" aria-label="Search">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"
               stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="10"/>
            <line x1="2" y1="12" x2="22" y2="12"/>
            <circle cx="12" cy="12" r="3" fill="currentColor" stroke="none"/>
          </svg>
        </button>
      </div>
    </div>

    <!-- Row 2: sort + type filters -->
    <div class="row row-filters">
      <select
        :value="sortOrder"
        @change="$emit('update:sortOrder', $event.target.value)"
        class="sort-select"
      >
        <option value="num-asc">N.º  1 → 1025</option>
        <option value="num-desc">N.º  1025 → 1</option>
        <option value="name-asc">Name A → Z</option>
        <option value="name-desc">Name Z → A</option>
      </select>

      <TypeSelect
        :modelValue="filterType"
        placeholder="Type"
        @update:modelValue="$emit('update:filterType', $event)"
      />

      <TypeSelect
        :modelValue="filterStrength"
        placeholder="Strengths"
        @update:modelValue="$emit('update:filterStrength', $event)"
      />

      <TypeSelect
        :modelValue="filterWeakness"
        placeholder="Weaknesses"
        @update:modelValue="$emit('update:filterWeakness', $event)"
      />

      <button class="reset-btn" @click="$emit('reset')" title="Reset filters">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
             stroke-linecap="round" stroke-linejoin="round">
          <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"/>
          <path d="M3 3v5h5"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import TypeSelect from './TypeSelect.vue'

defineProps({
  search:         String,
  sortOrder:      String,
  filterType:     String,
  filterStrength: String,
  filterWeakness: String,
})
defineEmits([
  'update:search',
  'update:sortOrder',
  'update:filterType',
  'update:filterStrength',
  'update:filterWeakness',
  'reset',
])
</script>

<style scoped>
.filter-bar {
  background: var(--card);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  padding: 16px 20px;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.row {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

/* ── Search ── */
.search-wrap {
  flex: 1;
  min-width: 220px;
  position: relative;
  display: flex;
  align-items: center;
}

.search-input {
  width: 100%;
  padding: 12px 52px 12px 18px;
  border: 1.5px solid var(--border);
  border-radius: 30px;
  font-size: 14px;
  color: var(--text);
  background: #fff;
  outline: none;
  transition: border-color 0.2s;
}
.search-input:focus       { border-color: var(--accent); }
.search-input::placeholder { color: var(--muted); }

.search-btn {
  position: absolute;
  right: 8px;
  width: 36px;
  height: 36px;
  border: none;
  border-radius: 50%;
  background: var(--accent);
  color: #fff;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}
.search-btn svg { width: 18px; height: 18px; }

/* ── Sort select ── */
.sort-select {
  padding: 8px 28px 8px 14px;
  border: 1.5px solid var(--border);
  border-radius: 30px;
  font-size: 13px;
  font-weight: 500;
  color: var(--text);
  background: #fff;
  cursor: pointer;
  outline: none;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 10 6' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M1 1l4 4 4-4' stroke='%239a9ab0' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 10px center;
  background-size: 10px;
  transition: border-color 0.15s;
}
.sort-select:focus { border-color: var(--accent); }

/* ── Reset ── */
.reset-btn {
  width: 36px;
  height: 36px;
  border: 1.5px solid var(--border);
  border-radius: 10px;
  background: #fff;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: auto;
  transition: background 0.2s;
  flex-shrink: 0;
}
.reset-btn svg   { width: 16px; height: 16px; color: var(--muted); }
.reset-btn:hover { background: var(--border); }
</style>
