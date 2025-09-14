<template>
  <div class="upgrade-item" :class="{ disabled: !canAfford }" @click="buyUpgrade">
    <div class="upgrade-info">
      <h3>{{ name }}</h3>
      <p>{{ description }}</p>
      <span class="price">{{ price }} cookies</span>
    </div>
    <div class="upgrade-count" v-if="owned > 0">{{ owned }}</div>
  </div>
</template>

<script setup>
const props = defineProps({
  name: String,
  description: String,
  price: Number,
  owned: Number,
  canAfford: Boolean
})

const emit = defineEmits(['buy'])

const buyUpgrade = () => {
  if (props.canAfford) {
    emit('buy')
  }
}
</script>

<style scoped>
.upgrade-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  border: 2px solid #d2b48c;
  border-radius: 8px;
  background: #fff;
  cursor: pointer;
  transition: all 0.2s;
}

.upgrade-item:hover:not(.disabled) {
  background: #f5f5dc;
  border-color: #b8860b;
  transform: translateY(-2px);
}

.upgrade-item.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.upgrade-info h3 {
  margin: 0 0 5px 0;
  color: #8b4513;
}

.upgrade-info p {
  margin: 0 0 5px 0;
  font-size: 0.9em;
  color: #666;
}

.price {
  font-weight: bold;
  color: #b8860b;
}

.upgrade-count {
  background: #d2b48c;
  color: white;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}
</style>
