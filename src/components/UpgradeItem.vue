<template>
  <div class="upgrade-item" :class="{ disabled: !canAfford }" @click="buyUpgrade">
    <div class="upgrade-icon">
      {{ getIcon(name) }}
    </div>
    <div class="upgrade-info">
      <h3>{{ name }}</h3>
      <p>{{ description }}</p>
      <div class="upgrade-stats">
        <span class="price">💰 {{ formatPrice(price) }}</span>      </div>
    </div>
    <div class="upgrade-count" v-if="owned > 0">
      <span class="count-number">{{ owned }}</span>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  name: String,
  description: String,
  price: Number,
  owned: Number,
  canAfford: Boolean,
  cps: Number
})

const emit = defineEmits(['buy'])

const buyUpgrade = () => {
  if (props.canAfford) {
    emit('buy')
  }
}

const getIcon = (name) => {
  const icons = {
    'Cursor': '👆',
    'Grandma': '👵',
    'Farm': '🚜',
    'Mine': '⛏️',
    'Factory': '🏭',
    'Bank': '🏦',
    'Temple': '🏛️',
    'Wizard Tower': '🧙‍♂️',
    'Shipment': '🚀',
    'Alchemy Lab': '⚗️'
  }
  return icons[name] || '🍪'
}

const formatPrice = (price) => {
  if (price >= 1e9) return (price / 1e9).toFixed(1) + 'B'
  if (price >= 1e6) return (price / 1e6).toFixed(1) + 'M'
  if (price >= 1e3) return (price / 1e3).toFixed(1) + 'K'
  return price.toString()
}

const formatCps = () => {
  const totalCps = props.cps * props.owned
  if (totalCps >= 1e6) return (totalCps / 1e6).toFixed(1) + 'M'
  if (totalCps >= 1e3) return (totalCps / 1e3).toFixed(1) + 'K'
  return totalCps.toString()
}
</script>

<style scoped>
.upgrade-item {
  display: flex;
  align-items: center;
  padding: 16px 20px;
  margin-bottom: 12px;
  background: #ffffff;
  border: 1px solid #e1e5e9;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
  position: relative;
}

.upgrade-item:hover:not(.disabled) {
  background: #f8f9fa;
  border-color: #d0d7de;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
}

.upgrade-item:active:not(.disabled) {
  transform: translateY(0);
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
}

.upgrade-item.disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background: #f6f8fa;
  border-color: #e1e5e9;
}

.upgrade-icon {
  font-size: 28px;
  margin-right: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  background: #f6f8fa;
  border-radius: 8px;
  border: 1px solid #e1e5e9;
  transition: all 0.2s ease;
}

.upgrade-item:hover:not(.disabled) .upgrade-icon {
  background: #eef2f5;
  border-color: #d0d7de;
}

.upgrade-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.upgrade-info h3 {
  margin: 0;
  color: #24292f;
  font-size: 16px;
  font-weight: 600;
}

.upgrade-info p {
  margin: 0;
  font-size: 13px;
  color: #656d76;
  line-height: 1.4;
}

.upgrade-stats {
  display: flex;
  gap: 12px;
  align-items: center;
  margin-top: 4px;
}

.price {
  font-weight: 500;
  color: #0969da;
  font-size: 14px;
  padding: 2px 6px;
  background: #f6f8fa;
  border-radius: 4px;
  border: 1px solid #e1e5e9;
}

.cps {
  font-weight: 500;
  color: #1a7f37;
  font-size: 12px;
  padding: 2px 6px;
  background: #f6f8fa;
  border-radius: 4px;
  border: 1px solid #e1e5e9;
}

.upgrade-count {
  background: #24292f;
  color: white;
  border-radius: 6px;
  min-width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 14px;
  padding: 0 8px;
  border: 1px solid #32383f;
}

.count-number {
  line-height: 1;
}
</style>
