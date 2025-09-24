<template>
  <div class="container">
    <section class="left">
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
      <Ranking :cookies="count" :cps="cookiesPerSecond" :playerName="'You'" />
    </section>
    <section class="center">
      <div class="cookies-stats">
        <p>Number of cookies {{ count }}</p>
        <p>Cookies per second: {{ cookiesPerSecond }}</p>
      </div>
      <img class="cookie-btn" src="./assets/image.png" @click="handleCookieClick" alt="Cookie">
    </section>
    <section class="right">
      <Upgrades :cookies="count" :savedUpgrades="currentUpgrades" @buy-upgrade="buyUpgrade" @cps-update="updateCps" @upgrades-update="updateUpgrades" />
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


const count = ref(0)
const cookiesPerSecond = ref(0)
const totalClicks = ref(0)
const currentUpgrades = ref([])
const connectionRef = ref(null)

// Ajouter une ref pour les achievements
const currentAchievements = ref([])

// Variable pour déclencher le reset des achievements
const resetTrigger = ref(0)

// Fonction pour sauvegarder dans le localStorage
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

// Fonction pour charger depuis le localStorage
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

const handleCookieClick = () => {
  count.value++
  totalClicks.value++
}

const buyUpgrade = (cost) => {
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
  
  console.log('User data loaded successfully')
}

// Fonction pour obtenir les upgrades par défaut
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

// Fonction pour remettre le jeu à zéro lors de la déconnexion
const resetGameToDefault = () => {
  count.value = 0
  totalClicks.value = 0
  cookiesPerSecond.value = 0  
  currentUpgrades.value = getDefaultUpgrades() // Utiliser les upgrades par défaut
  currentAchievements.value = []
  
  // Déclencher le reset des achievements
  resetTrigger.value++
  
  // Effacer aussi le localStorage pour éviter les conflits
  localStorage.removeItem('cookieClickerSave')
  
  console.log('Game reset to default state after disconnection')
}

// Nouvelle fonction pour gérer l'import de sauvegarde
const handleLoadSaveData = (saveData) => {
  count.value = saveData.cookies || 0
  totalClicks.value = saveData.totalClicks || 0
  cookiesPerSecond.value = saveData.cps || 0
  currentUpgrades.value = saveData.upgrades || []
  currentAchievements.value = saveData.achievements || []
  
  // Sauvegarder dans le localStorage
  saveToLocalStorage()
  
  // Sauvegarder les nouvelles données pour l'utilisateur connecté
  if (connectionRef.value?.isConnected) {
    connectionRef.value.saveUserGameData()
  }
  
  console.log('Save data loaded from Options:', saveData)
}

// Fonction pour effacer la sauvegarde
const handleClearSaveData = (emptyData) => {
  // Utiliser la même logique que resetGameToDefault pour un reset complet
  count.value = 0
  totalClicks.value = 0
  cookiesPerSecond.value = 0  
  currentUpgrades.value = getDefaultUpgrades() // Utiliser les upgrades par défaut
  currentAchievements.value = []
  
  // Déclencher le reset des achievements
  resetTrigger.value++
  
  // Effacer aussi le localStorage
  localStorage.removeItem('cookieClickerSave')
  
  // Sauvegarder les données vides
  if (connectionRef.value?.isConnected) {
    connectionRef.value.saveUserGameData()
  }
  
  console.log('Save data cleared completely')
}

// Computed pour les données de jeu actuelles
const currentGameData = computed(() => ({
  cookies: count.value,
  totalClicks: totalClicks.value,
  cps: cookiesPerSecond.value,
  upgrades: currentUpgrades.value,
  achievements: currentAchievements.value,
  lastSaved: new Date().toISOString()
}))

let interval = null
const saveTimeout = ref(null)
const dbSaveTimeout = ref(null)
const isMounted = ref(false)

// Sauvegarder automatiquement quand les cookies changent
watch([count, totalClicks, cookiesPerSecond, currentUpgrades, currentAchievements], () => {
  // Ne sauvegarder que si le composant est monté
  if (!isMounted.value) return
  
  // Toujours sauvegarder dans le localStorage
  clearTimeout(saveTimeout.value)
  saveTimeout.value = setTimeout(() => {
    saveToLocalStorage()
  }, 500) // Sauvegarder 0.5 seconde après le dernier changement
  
  // Aussi sauvegarder dans la base de données si connecté
  if (connectionRef.value?.isConnected) {
    clearTimeout(dbSaveTimeout.value)
    dbSaveTimeout.value = setTimeout(() => {
      connectionRef.value.saveUserGameData()
    }, 1000) // Sauvegarder 1 seconde après le dernier changement
  }
}, { deep: true })

onMounted(() => {
  // Marquer le composant comme monté
  isMounted.value = true
  
  // Charger les données depuis le localStorage seulement si pas d'utilisateur connecté
  setTimeout(() => {
    // Si aucun utilisateur n'est connecté, charger le localStorage
    if (!connectionRef.value?.isConnected) {
      loadFromLocalStorage()
    }
    // Sinon, les données seront chargées par Connection.vue via loadUserGameData
  }, 300) // Délai suffisant pour laisser Connection.vue gérer la connexion
  
  window.addEventListener('beforeunload', () => {
    // Sauvegarder avant de quitter la page
    saveToLocalStorage()
    if (connectionRef.value?.isConnected) {
      connectionRef.value.saveUserGameData()
    }
  })
  
  interval = setInterval(() => {
    count.value += cookiesPerSecond.value
    
    // Sauvegarder toutes les 10 secondes dans le localStorage
    if (Date.now() % 10000 < 1000) {
      saveToLocalStorage()
    }
    
    // Sauvegarder toutes les 10 secondes si connecté
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
}
.cookies-stats {
  text-align: center;
}
.cookie-btn {
  width: 300px;
  height: auto;
  border-radius: 50%;
  box-shadow: 0 4px 16px rgba(0,0,0,0.2);
  cursor: pointer;
  border: 3px solid #d2b48c;
  background: #fff;
  transition: transform 0.1s, box-shadow 0.1s, border-color 0.1s;
}
.cookie-btn:hover {
  transform: scale(1.08);
  box-shadow: 0 8px 32px rgba(0,0,0,0.3);
  border-color: #e2c48c;
}
.cookie-btn:active {
  transform: scale(0.95);
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
  border-color: #b8860b;
  filter: brightness(0.95);
}

.right {
  overflow-y: auto;
  overflow-x: hidden;
  max-height: 100vh;
}
.right::-webkit-scrollbar {
  display: none; 
}

.left {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  min-width: 0;
  padding: 20px;

  padding-top: 120px; /* Décalage général vers le bas */
}

</style>