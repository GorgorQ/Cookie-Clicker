<template>
  <div class="upgrades-container">
    <div class="multipliers-section">
      <h3>Bonus</h3>
      <div class="multipliers-compact">
        <div 
          v-for="bonus in multiplierBonuses" 
          :key="bonus.id"
          class="bonus-compact"
          :class="{ disabled: !canAffordBonus(bonus), owned: bonus.owned }"
          @click="buyBonus(bonus.id)"
          @mouseenter="showTooltip(bonus)"
          @mouseleave="hideTooltip"
        >
          <span class="bonus-emoji">{{ bonus.icon }}</span>
          <div v-if="bonus.owned" class="bonus-check">✅</div>
        </div>
      </div>
      
      <div 
        v-if="activeTooltip"
        class="centralized-tooltip"
      >
        <div class="tooltip-header">
          <strong>{{ activeTooltip.name }}</strong>
          <span class="tooltip-multiplier">+{{ activeTooltip.multiplier }}%</span>
        </div>
        <div class="tooltip-description">{{ activeTooltip.description }}</div>
        <div class="tooltip-price">
          <img src="../assets/image.png" alt="Cookie" class="tooltip-cookie-icon">
          {{ formatPrice(activeTooltip.price) }}
        </div>
        <div v-if="!activeTooltip.requirement()" class="tooltip-requirement">
          Condition non remplie
        </div>
      </div>
    </div>

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

const props = defineProps(['cookies', 'savedUpgrades'])
const emit = defineEmits(['buy-upgrade', 'cps-update', 'upgrades-update', 'buy-bonus'])

const multiplierBonuses = ref([
  {
    id: 'bonus1',
    name: 'Lucky Finger',
    description: 'Améliore l\'efficacité des clics',
    icon: '🍀',
    price: 1000,
    multiplier: 1,
    owned: false,
    requirement: () => props.cookies >= 500
  },
  {
    id: 'bonus2', 
    name: 'Golden Touch',
    description: 'Augmente la production globale',
    icon: '✨',
    price: 25000,
    multiplier: 2,
    owned: false,
    requirement: () => getTotalUpgrades() >= 5
  },
  {
    id: 'bonus3',
    name: 'Cookie Blessing',
    description: 'Bénédiction divine des cookies',
    icon: '🙏',
    price: 500000,
    multiplier: 5,
    owned: false,
    requirement: () => getTotalUpgrades() >= 15
  },
  {
    id: 'bonus4',
    name: 'Master Baker',
    description: 'Expertise ultime en pâtisserie',
    icon: '👨‍🍳',
    price: 10000000,
    multiplier: 10,
    owned: false,
    requirement: () => getTotalUpgrades() >= 50
  },
  {
    id: 'bonus5',
    name: 'Cookie Dimension',
    description: 'Accès à la dimension infinie des cookies',
    icon: '🌌',
    price: 1000000000,
    multiplier: 25,
    owned: false,
    requirement: () => getTotalUpgrades() >= 100
  }
])

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

// Fonctions utilitaires
const getTotalUpgrades = () => {
  return upgrades.value.reduce((total, upgrade) => total + upgrade.owned, 0)
}

const getTotalMultiplier = () => {
  return multiplierBonuses.value.reduce((total, bonus) => {
    return total + (bonus.owned ? bonus.multiplier : 0)
  }, 0)
}

const canAffordBonus = (bonus) => {
  return props.cookies >= bonus.price && bonus.requirement() && !bonus.owned
}

const formatPrice = (price) => {
  if (price >= 1e9) return (price / 1e9).toFixed(1) + 'B'
  if (price >= 1e6) return (price / 1e6).toFixed(1) + 'M'
  if (price >= 1e3) return (price / 1e3).toFixed(1) + 'K'
  return price.toString()
}

const cookiesPerSecond = computed(() => {
  const baseCps = upgrades.value.reduce((total, upgrade) => {
    return total + (upgrade.owned * upgrade.cps)
  }, 0)
  
  // Appliquer les bonus multiplicateurs
  const multiplierBonus = getTotalMultiplier()
  return baseCps * (1 + multiplierBonus / 100)
})


watch(cookiesPerSecond, (newCps) => {
  emit('cps-update', newCps)
}, { immediate: true })


watch(() => props.savedUpgrades, (savedUpgrades) => {
  if (savedUpgrades && savedUpgrades.length > 0) {

    upgrades.value.forEach(upgrade => {
      const savedUpgrade = savedUpgrades.find(s => s.id === upgrade.id)
      if (savedUpgrade) {
        upgrade.owned = savedUpgrade.owned || 0
        upgrade.price = savedUpgrade.price || upgrade.basePrice
      }
    })
  }
}, { immediate: true })


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

const buyBonus = (bonusId) => {
  const bonus = multiplierBonuses.value.find(b => b.id === bonusId)
  if (bonus && canAffordBonus(bonus)) {
    emit('buy-bonus', bonus.price)
    bonus.owned = true
    console.log(`Bonus acheté: ${bonus.name} (+${bonus.multiplier}% production)`)
  }
}

const activeTooltip = ref(null)

const showTooltip = (bonus) => {
  activeTooltip.value = bonus
}

const hideTooltip = () => {
  activeTooltip.value = null
}
</script>

<style scoped>
.upgrades-container {
  width: 100%;
  padding: 20px;
}

.multipliers-section {
  margin-bottom: 20px;
  padding: 12px 16px;
  background: linear-gradient(135deg, #fff3cd, #ffeaa7);
  border: 2px solid #ffc107;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(255, 193, 7, 0.2);
  position: relative;
}

.multipliers-section h3 {
  margin: 0 0 8px 0;
  color: #856404;
  font-size: 14px;
  text-align: center;
  font-weight: 600;
}

.multipliers-compact {
  display: flex;
  gap: 8px;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
}

.bonus-compact {
  position: relative;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.9);
  border: 2px solid #f0c040;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 18px;
  z-index: 1;
}

.bonus-compact:hover {
  transform: scale(1.1);
  border-color: #ffc107;
  box-shadow: 0 4px 12px rgba(255, 193, 7, 0.4);
}

.bonus-compact.disabled {
  opacity: 0.3;
  cursor: not-allowed;
  filter: grayscale(100%);
}

.bonus-compact.disabled:hover {
  transform: none;
  box-shadow: none;
}

.bonus-compact.owned {
  background: rgba(40, 167, 69, 0.2);
  border-color: #28a745;
}

.bonus-compact.owned .bonus-emoji {
  filter: brightness(1.2);
}

.bonus-emoji {
  font-size: 20px;
  transition: all 0.2s ease;
}

.bonus-check {
  position: absolute;
  top: -4px;
  right: -4px;
  font-size: 12px;
  background: white;
  border-radius: 50%;
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #28a745;
}

/* Tooltip centralisé en overlay */
.centralized-tooltip {
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-top: 8px;
  background: rgba(0, 0, 0, 0.95);
  color: white;
  padding: 28px;
  border-radius: 16px;
  font-size: 16px;
  text-align: left;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.2);
  z-index: 1000;
  min-width: 350px;
  animation: tooltipSlideIn 0.2s ease-out;
}

@keyframes tooltipSlideIn {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

.tooltip-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  padding-bottom: 4px;
}

.tooltip-multiplier {
  color: #28a745;
  font-weight: bold;
}

.tooltip-description {
  margin-bottom: 6px;
  font-style: italic;
  color: rgba(255, 255, 255, 0.8);
  
}

.tooltip-price {
  color: #ffc107;
  font-weight: bold;
  display: flex;
  align-items: center;
  gap: 4px;
}

.tooltip-cookie-icon {
  width: 25px;
  height: 25px;
  object-fit: contain;
}

.tooltip-requirement {
  color: #dc3545;
  font-size: 11px;
  font-style: italic;
  margin-top: 4px;
}

.upgrades-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
</style>
