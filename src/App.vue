<template>
  <div class="container">
    <section class="left">
      <DisplayName
        ref="displayNameRef"
        :isConnected="connectionRef?.isConnected || false"
        :username="connectionRef?.username || ''"
        @display-name-changed="refreshDisplayName"
      />
      <Connection 
        ref="connectionRef"
        :cookies="count" 
        :upgrades="currentUpgrades" 
        :achievements="currentAchievements"
        :totalClicks="totalClicks" 
        :cps="cookiesPerSecond"
        @load-user-data="loadUserData"
        @user-disconnected="resetGameToDefault"
      />
      <Options 
        :isConnected="connectionRef?.isConnected || false"
        :username="connectionRef?.username || ''"
        :displayName="currentDisplayName"
        :gameData="currentGameData"
        @load-save-data="handleLoadSaveData"
        @clear-save-data="handleClearSaveData"
      />
      <Achievements 
        :cookies="count" 
        :totalClicks="totalClicks" 
        :upgrades="currentUpgrades"
        :resetTrigger="resetTrigger"
        @achievements-update="currentAchievements = $event"
      />
      <Ranking 
        :cookies="count" 
        :cps="cookiesPerSecond" 
        :playerName="'You'" 
        :isConnected="connectionRef?.isConnected || false"
        :username="currentDisplayName"
      />
    </section>
    <section class="center">
      <div class="cookies-stats">
        <p>Number of cookies {{ formatNumber(count) }}</p>
        <p>Cookies per second: {{ formatNumber(cookiesPerSecond) }}</p>      </div>
      <div class="cookie-container">
        <img class="cookie-btn" src="./assets/image.png" @click="handleCookieClick" alt="Cookie">
      
        <div 
          v-for="floatingNumber in floatingNumbers" 
          :key="floatingNumber.id"
          class="floating-number"
          :style="{ left: floatingNumber.x + 'px', top: floatingNumber.y + 'px' }"
        >
          +{{ floatingNumber.value }}
        </div>
      </div>
    </section>
    <section class="right">
      <Upgrades :cookies="count" :savedUpgrades="currentUpgrades" @buy-upgrade="buyUpgrade" @buy-bonus="buyBonus" @cps-update="updateCps" @upgrades-update="updateUpgrades" />
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed, watch } from "vue"
import Upgrades from "./components/Upgrades.vue"
import Achievements from "./components/Achievements.vue"
import Options from "./components/Options.vue"
import Connection from "./components/Connection.vue"
import Ranking from "./components/Ranking.vue"
import DisplayName from "./components/DisplayName.vue"


const count = ref(0)
const cookiesPerSecond = ref(0)
const totalClicks = ref(0)
const currentUpgrades = ref([])
const connectionRef = ref(null)
const displayNameRef = ref(null)

const currentAchievements = ref([])
const resetTrigger = ref(0)
const displayNameTrigger = ref(0)
const floatingNumbers = ref([])

// Calcul des cookies par clic
const cookiesPerClick = computed(() => {
  let baseClick = 1
  let clickBonus = 0
  let totalUpgrades = 0
  
  // Bonus basé sur chaque type d'upgrade 
  currentUpgrades.value.forEach(upgrade => {
    if (upgrade.owned > 0) {
      totalUpgrades += upgrade.owned
      
      // Pourcentages différents selon le type d'upgrade
      let bonusPercentage = 0
      switch(upgrade.name) {
        case "Cursor":
          bonusPercentage = 0.15 
          break
        case "Grandma":
          bonusPercentage = 0.12 
          break
        case "Farm":
          bonusPercentage = 0.08 
          break
        case "Mine":
          bonusPercentage = 0.06 
          break
        case "Factory":
          bonusPercentage = 0.05
          break
        case "Bank":
          bonusPercentage = 0.04
          break
        case "Temple":
          bonusPercentage = 0.03
          break
        case "Wizard Tower":
          bonusPercentage = 0.025
          break
        default:
          bonusPercentage = 0.02 
      }
      
      clickBonus += upgrade.owned * upgrade.cps * bonusPercentage
    }
  })
  
  // Bonus global basé sur le nombre total d'upgrades 
  const globalBonus = baseClick * totalUpgrades * 0.01
  
  // Bonus multiplicateur des achievements de clic 
  let clickMultiplier = 1
  if (totalClicks.value >= 500) clickMultiplier += 0.25 // +25% après 500 clics
  if (totalClicks.value >= 1000) clickMultiplier += 0.5 // +75% total après 1000 clics
  if (totalClicks.value >= 5000) clickMultiplier += 0.75 // +150% total après 5000 clics
  if (totalClicks.value >= 10000) clickMultiplier += 1 // +250% total après 10000 clics
  
  return Math.max(1, (baseClick + clickBonus + globalBonus) * clickMultiplier)
})
const formatNumber = (num) => {
  if (num >= 1000000000000) return (num / 1000000000000).toFixed(1) + 'T'
  if (num >= 1000000000) return (num / 1000000000).toFixed(1) + 'B'
  if (num >= 1000000) return (num / 1000000).toFixed(1) + 'M'
  if (num >= 1000) return (num / 1000).toFixed(1) + 'K'
  return Math.floor(num).toString()
}

const formatNumberWithDecimals = (num) => {
  if (num >= 1000000000000) return (num / 1000000000000).toFixed(2) + 'T'
  if (num >= 1000000000) return (num / 1000000000).toFixed(2) + 'B'
  if (num >= 1000000) return (num / 1000000).toFixed(2) + 'M'
  if (num >= 1000) return (num / 1000).toFixed(2) + 'K'
  return num.toFixed(2)
}

const updatePageTitle = () => {
  const cookieCount = formatNumber(count.value)
  const cps = cookiesPerSecond.value > 0 ? ` | ${formatNumber(cookiesPerSecond.value)}/s` : ''
  document.title = `${cookieCount}${cps}`
}

const saveToLocalStorage = () => {
  const gameData = {
    cookies: count.value,
    totalClicks: totalClicks.value,
    cps: cookiesPerSecond.value,
    upgrades: currentUpgrades.value,
    achievements: currentAchievements.value,
    lastSaved: new Date().toISOString()
  }
  localStorage.setItem('cookieClickerSave', JSON.stringify(gameData))
}

const loadFromLocalStorage = () => {
  const savedData = localStorage.getItem('cookieClickerSave')
  if (savedData) {
    try {
      const gameData = JSON.parse(savedData)
      count.value = gameData.cookies || 0
      totalClicks.value = gameData.totalClicks || 0
      cookiesPerSecond.value = gameData.cps || 0
      currentUpgrades.value = gameData.upgrades || []
      currentAchievements.value = gameData.achievements || []
      console.log('Game data loaded from localStorage:', gameData)
    } catch (error) {
      console.error('Error loading from localStorage:', error)
    }
  }
}

const handleCookieClick = (event) => {
  const cookiesGained = cookiesPerClick.value
  count.value += cookiesGained
  totalClicks.value++
  createFloatingNumber(event || window.event, cookiesGained)
}

const createFloatingNumber = (event, cookiesGained = 1) => {
  const floatingNumber = {
    id: Date.now() + Math.random(),
    value: formatNumberWithDecimals(cookiesGained), 
    x: event.clientX,
    y: event.clientY
  }
  
  floatingNumbers.value.push(floatingNumber)
  
  setTimeout(() => {
    const index = floatingNumbers.value.findIndex(fn => fn.id === floatingNumber.id)
    if (index > -1) {
      floatingNumbers.value.splice(index, 1)
    }
  }, 2000)
}

const buyUpgrade = (cost) => {
  count.value -= cost
}

const buyBonus = (cost) => {
  count.value -= cost
}

const updateCps = (newCps) => {
  cookiesPerSecond.value = newCps
}

const updateUpgrades = (upgrades) => {
  currentUpgrades.value = upgrades
}

const loadUserData = (gameData) => {
  console.log('Loading user data:', gameData)
  
  count.value = gameData.cookies || 0
  totalClicks.value = gameData.totalClicks || 0
  cookiesPerSecond.value = gameData.cps || 0
  currentUpgrades.value = gameData.upgrades || []
  currentAchievements.value = gameData.achievements || []
  
  setTimeout(() => updatePageTitle(), 100)
  
  console.log('User data loaded successfully')
}

const getDefaultUpgrades = () => [
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
]

const resetGameToDefault = () => {
  count.value = 0
  totalClicks.value = 0
  cookiesPerSecond.value = 0  
  currentUpgrades.value = getDefaultUpgrades()
  currentAchievements.value = []
  resetTrigger.value++
  updatePageTitle()
  localStorage.removeItem('cookieClickerSave')
  
  console.log('Game reset to default state after disconnection')
}

const handleLoadSaveData = (saveData) => {
  count.value = saveData.cookies || 0
  totalClicks.value = saveData.totalClicks || 0
  cookiesPerSecond.value = saveData.cps || 0
  currentUpgrades.value = saveData.upgrades || []
  currentAchievements.value = saveData.achievements || []
  
  saveToLocalStorage()
  if (connectionRef.value?.isConnected) {
    connectionRef.value.saveUserGameData()
  }
  
  console.log('Save data loaded from Options:', saveData)
}

const handleClearSaveData = (emptyData) => {
  count.value = 0
  totalClicks.value = 0
  cookiesPerSecond.value = 0  
  currentUpgrades.value = getDefaultUpgrades()
  currentAchievements.value = []
  
  resetTrigger.value++
  
  updatePageTitle()
  
  localStorage.removeItem('cookieClickerSave')
  if (connectionRef.value?.isConnected) {
    connectionRef.value.saveUserGameData()
  }
  
  console.log('Save data cleared completely')
}

const refreshDisplayName = () => {
  displayNameTrigger.value++
}

const currentGameData = computed(() => ({
  cookies: count.value,
  totalClicks: totalClicks.value,
  cps: cookiesPerSecond.value,
  upgrades: currentUpgrades.value,
  achievements: currentAchievements.value,
  lastSaved: new Date().toISOString()
}))

const currentDisplayName = computed(() => {
  displayNameTrigger.value
  const saved = localStorage.getItem('cookieClicker_displayName')
  if (saved && saved.trim()) {
    console.log('currentDisplayName: using saved name:', saved.trim())
    return saved.trim()
  }
  const fallbackName = (connectionRef.value?.isConnected && connectionRef.value?.username) ? connectionRef.value.username : 'Guest'
  console.log('currentDisplayName: using fallback:', fallbackName)
  return fallbackName
})

let interval = null
const saveTimeout = ref(null)
const dbSaveTimeout = ref(null)
const isMounted = ref(false)

watch([count, cookiesPerSecond], () => {
  if (isMounted.value) {
    updatePageTitle()
  }
})

watch([count, totalClicks, cookiesPerSecond, currentUpgrades, currentAchievements], () => {
  if (!isMounted.value) return
  clearTimeout(saveTimeout.value)
  saveTimeout.value = setTimeout(() => {
    saveToLocalStorage()
  }, 500)
  if (connectionRef.value?.isConnected) {
    clearTimeout(dbSaveTimeout.value)
    dbSaveTimeout.value = setTimeout(() => {
      connectionRef.value.saveUserGameData()
    }, 1000)
  }
}, { deep: true })

onMounted(() => {
  isMounted.value = true
  updatePageTitle()
  setTimeout(() => {
    if (!connectionRef.value?.isConnected) {
      loadFromLocalStorage()
    }
    
    updatePageTitle()
  }, 300)
  
  window.addEventListener('beforeunload', () => {
    saveToLocalStorage()
    if (connectionRef.value?.isConnected) {
      connectionRef.value.saveUserGameData()
    }
  })
  
  interval = setInterval(() => {
    count.value += cookiesPerSecond.value
    
    if (Date.now() % 10000 < 1000) {
      saveToLocalStorage()
    }
    if (Date.now() % 10000 < 1000 && connectionRef.value?.isConnected) {
      connectionRef.value.saveUserGameData()
    }
  }, 1000)
})

onUnmounted(() => {
  if (interval) clearInterval(interval)
  if (saveTimeout.value) clearTimeout(saveTimeout.value)
  if (dbSaveTimeout.value) clearTimeout(dbSaveTimeout.value)
})
</script>

<style>
@import './style.css';

html, body, #app {
  height: 100%;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.container {
  display: flex;
  min-height: 100vh;
  height: 100vh;
  width: 100vw;
  min-width: 100vw;
  overflow: hidden;
}
.center, .right {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  min-width: 0;
  padding: 20px;
}
.center {
  gap: 50px;
  padding-top: 3%;
}

.cookie-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}
.cookies-stats {
  text-align: center;
}

.cookies-per-click {
  color: #28a745;
  font-weight: bold;
  font-size: 16px;
}
.cookie-btn {
  width: 350px;
  height: auto;
  cursor: pointer;
  border: none;
  background: transparent;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  position: relative;
  

  filter: drop-shadow(0 0 15px rgba(255, 215, 0, 0.4))
          drop-shadow(0 0 30px rgba(255, 215, 0, 0.3))
          drop-shadow(0 0 45px rgba(255, 193, 77, 0.2))
          drop-shadow(0 0 60px rgba(255, 165, 0, 0.1));
  animation: cookieGlow 3s ease-in-out infinite alternate;
}



.cookie-btn:hover {
  transform: scale(1.05);
  filter: drop-shadow(0 0 20px rgba(255, 215, 0, 0.5))
          drop-shadow(0 0 40px rgba(255, 215, 0, 0.4))
          drop-shadow(0 0 60px rgba(255, 193, 77, 0.3))
          drop-shadow(0 0 80px rgba(255, 165, 0, 0.2));
}

.cookie-btn:active {
  transform: scale(0.95);
  filter: drop-shadow(0 0 10px rgba(255, 215, 0, 0.6))
          drop-shadow(0 0 20px rgba(255, 215, 0, 0.4))
          drop-shadow(0 0 30px rgba(255, 193, 77, 0.3));
}

@keyframes cookieGlow {
  0% {
    filter: drop-shadow(0 0 15px rgba(255, 215, 0, 0.4))
            drop-shadow(0 0 30px rgba(255, 215, 0, 0.3))
            drop-shadow(0 0 45px rgba(255, 193, 77, 0.2))
            drop-shadow(0 0 60px rgba(255, 165, 0, 0.1));
  }
  100% {
    filter: drop-shadow(0 0 25px rgba(255, 215, 0, 0.5))
            drop-shadow(0 0 50px rgba(255, 215, 0, 0.4))
            drop-shadow(0 0 75px rgba(255, 193, 77, 0.3))
            drop-shadow(0 0 100px rgba(255, 165, 0, 0.2));
  }
}

@keyframes rotateLight {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.right {
  overflow-y: auto;
  overflow-x: hidden;
  max-height: 100vh;
}
.right::-webkit-scrollbar {
  display: none; 
}

.floating-number {
  position: fixed;
  color: #ffc107;
  font-weight: bold;
  font-size: 24px;
  pointer-events: none;
  z-index: 1000;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  animation: floatUp 2s ease-out forwards;
}

@keyframes floatUp {
  0% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
  50% {
    opacity: 1;
    transform: translateY(-50px) scale(1.2);
  }
  100% {
    opacity: 0;
    transform: translateY(-100px) scale(0.8);
  }
}

.left {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  min-width: 0;
  padding: 20px;
  padding-top: 120px;
}

</style>