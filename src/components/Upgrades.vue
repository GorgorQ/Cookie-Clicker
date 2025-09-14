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
        @buy="$emit('buy-upgrade', upgrade.id)"
      />
    </div>
  </div>
</template>

<script setup>
import UpgradeItem from './UpgradeItem.vue'

defineProps({
  cookies: {
    type: Number,
    default: 0
  },
  upgrades: {
    type: Array,
    default: () => []
  }
})

defineEmits(['buy-upgrade'])
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
    id: 3,
    name: "Farm",
    description: "Cultive des cookies",
    price: 1100,
    owned: 0
  }
])


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
