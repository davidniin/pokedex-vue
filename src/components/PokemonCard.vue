<template>
  <div
    :class="['card', { selected }]"
    @click="$emit('select', pokemon)"
  >
    <img :src="pokemon.sprite" :alt="pokemon.name" loading="lazy" />
    <p class="number">N°{{ String(pokemon.id).padStart(3, '0') }}</p>
    <h3 class="name">{{ formatName(pokemon.name) }}</h3>
    <div class="types">
      <span
        v-for="type in pokemon.types"
        :key="type"
        :class="`type-badge type-${type}`"
      >{{ type.toUpperCase() }}</span>
    </div>
  </div>
</template>

<script setup>
defineProps({ pokemon: Object, selected: Boolean })
defineEmits(['select'])

// "mr-mime" → "Mr Mime", same logic as original's dressUpPayloadValue
function formatName(s) {
  return s.split('-').map(w => w.charAt(0).toUpperCase() + w.slice(1)).join(' ')
}
</script>

<style scoped>
.card {
  background: var(--card);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  padding: 20px 16px 16px;
  cursor: pointer;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  transition: box-shadow 0.2s, transform 0.2s;
  border: 2px solid transparent;
}

.card:hover {
  box-shadow: var(--shadow-hover);
  transform: translateY(-3px);
}

.card.selected {
  border-color: var(--accent);
}

.card img {
  width: 88px;
  height: 88px;
  image-rendering: pixelated;
  margin-bottom: 4px;
}

.number {
  font-size: 12px;
  color: var(--muted);
  font-weight: 500;
}

.name {
  font-size: 15px;
  font-weight: 700;
  color: var(--text);
}

.types {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  justify-content: center;
  margin-top: 2px;
}

.type-badge {
  font-size: 10px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 20px;
  letter-spacing: 0.5px;
}
</style>
