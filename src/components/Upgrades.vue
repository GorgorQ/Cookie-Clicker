<template>
  <div class="upgrades-container">
    <h2>Améliorations</h2>
    <div class="upgrades-list">
      <UpgradeItem
        v-for="upgrade in upgrades"
        :key="upgrade.id"
        :name="upgrade.name"
        :description="upgrade.description"
        :price="upgrade.price"
        :owned="upgrade.owned"
        :canAfford="cookies >= upgrade.price"
        @buy="buyUpgrade(upgrade.id)"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue"
import UpgradeItem from './UpgradeItem.vue'

const props = defineProps(['cookies'])
const emit = defineEmits(['buy-upgrade', 'cps-update', 'upgrades-update'])

const upgrades = ref([
  {
    id: 1,
    name: "Cursor",
    description: "Clicks automatically",
    basePrice: 15,
    price: 15,
    owned: 0,
    cps: 1
  },
  {
    id: 2,
    name: "Grandma",
    description: "A nice grandma to bake cookies",
    basePrice: 100,
    price: 100,
    owned: 0,
    cps: 3
  },
  {
    id: 3,
    name: "Farm",
    description: "Grows cookie plants",
    basePrice: 1100,
    price: 1100,
    owned: 0,
    cps: 8
  },
  {
    id: 4,
    name: "Mine",
    description: "Mines out cookie dough",
    basePrice: 12000,
    price: 12000,
    owned: 0,
    cps: 47
  },
  {
    id: 5,
    name: "Factory",
    description: "Mass-produces cookies",
    basePrice: 130000,
    price: 130000,
    owned: 0,
    cps: 260
  },
  {
    id: 6,
    name: "Bank",
    description: "Generates cookies from interest",
    basePrice: 1400000,
    price: 1400000,
    owned: 0,
    cps: 1400
  },
  {
    id: 7,
    name: "Temple",
    description: "Full of precious, ancient chocolate",
    basePrice: 20000000,
    price: 20000000,
    owned: 0,
    cps: 7800
  },
  {
    id: 8,
    name: "Wizard Tower",
    description: "Summons cookies with magic spells",
    basePrice: 330000000,
    price: 330000000,
    owned: 0,
    cps: 44000
  },
  {
    id: 9,
    name: "Shipment",
    description: "Brings in fresh cookies from the cookie planet",
    basePrice: 5100000000,
    price: 5100000000,
    owned: 0,
    cps: 260000
  },
  {
    id: 10,
    name: "Alchemy Lab",
    description: "Turns gold into cookies!",
    basePrice: 75000000000,
    price: 75000000000,
    owned: 0,
    cps: 1600000
  }
])

const cookiesPerSecond = computed(() => {
  return upgrades.value.reduce((total, upgrade) => {
    return total + (upgrade.owned * upgrade.cps)
  }, 0)
})

// Émet le CPS chaque fois qu'il change
watch(cookiesPerSecond, (newCps) => {
  emit('cps-update', newCps)
}, { immediate: true })

// Émet les upgrades à chaque changement
watch(upgrades, (newUpgrades) => {
  emit('upgrades-update', newUpgrades)
}, { deep: true, immediate: true })

const buyUpgrade = (upgradeId) => {
  const upgrade = upgrades.value.find(u => u.id === upgradeId)
  if (upgrade && props.cookies >= upgrade.price) {
    emit('buy-upgrade', upgrade.price)
    upgrade.owned++
    upgrade.price = Math.floor(upgrade.basePrice * Math.pow(1.15, upgrade.owned))
  }
}
</script>

<style scoped>
.upgrades-container {
  width: 100%;
  padding: 20px;
}

.upgrades-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
</style>
