<template>
  <div class="ranking-container">
    <button 
      @click="toggleDropdown" 
      class="ranking-btn"
      :class="{ active: isOpen }"
    >
      🏆 Ranking
      <span class="arrow" :class="{ rotated: isOpen }">▼</span>
    </button>
    
    <div v-if="isOpen" class="ranking-dropdown">
      <div class="ranking-header">
        <h3>Top Players</h3>
      </div>
      
      <div class="ranking-list">
        <div v-if="loading" class="loading">
          🔄 Loading players...
        </div>
        <div v-else-if="players.length === 0" class="no-players">
          No players found
        </div>
        <div v-else>
          <div 
            v-for="(player, index) in players" 
            :key="player.id"
            class="player-item"
            :class="{ 'current-player': player.isCurrentPlayer }"
          >
            <div class="rank">
              <span class="rank-number">{{ index + 1 }}</span>
              <span class="medal">{{ getMedal(index) }}</span>
            </div>
            <div class="player-info">
              <span class="player-name">{{ player.name }}</span>
              <span class="player-score">{{ formatNumber(player.cookies) }} cookies</span>
            </div>
            <div class="player-cps">{{ formatNumber(player.cps) }} CPS</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

const props = defineProps(['cookies', 'cps', 'playerName', 'isConnected', 'username'])

const isOpen = ref(false)
const players = ref([])
const loading = ref(false)


// Charger les données des joueurs depuis le JSON
const loadPlayersData = async () => {
  loading.value = true
  try {
    const response = await fetch('/users.json')
    const data = await response.json()
    
    // Convertir les données des utilisateurs en format ranking
    const rankingData = data.users
      .filter(user => user.gameData && user.gameData.cookies > 0) // Filtrer les joueurs avec des cookies
      .map((user, index) => ({
        id: index + 1,
        name: user.username,
        cookies: user.gameData.cookies || 0,
        cps: user.gameData.cps || 0,
        isCurrentPlayer: false
      }))
    
    players.value = rankingData
    console.log('Players data loaded:', players.value)
    
  } catch (error) {
    console.error('Error loading players data:', error)
    // Données de fallback en cas d'erreur
    players.value = [
      { id: 1, name: "CookieMaster", cookies: 15847293, cps: 25000, isCurrentPlayer: false },
      { id: 2, name: "BakeQueen", cookies: 12934587, cps: 19500, isCurrentPlayer: false },
      { id: 3, name: "ChocolateKing", cookies: 8456321, cps: 15200, isCurrentPlayer: false }
    ]
  } finally {
    loading.value = false
  }
}

const toggleDropdown = async () => {
  isOpen.value = !isOpen.value
  if (isOpen.value) {
    await loadPlayersData()
    updateCurrentPlayer()
  }
}



const updateCurrentPlayer = () => {
  // Déterminer le nom du joueur actuel - utiliser toujours props.username (qui contient le display name)
  let currentPlayerName = props.username || "Guest"
  
  console.log('updateCurrentPlayer called with:', {
    username: props.username,
    isConnected: props.isConnected,
    cookies: props.cookies,
    cps: props.cps,
    currentPlayerName
  })
  
  // Retirer l'ancien joueur actuel et le joueur connecté s'il existe déjà dans la liste
  players.value = players.value.filter(p => !p.isCurrentPlayer && p.name !== currentPlayerName)
  
  // Ajouter le joueur actuel avec ses stats en temps réel
  const currentPlayerData = {
    id: 999,
    name: currentPlayerName,
    cookies: props.cookies || 0,
    cps: props.cps || 0,
    isCurrentPlayer: true
  }
  
  players.value.push(currentPlayerData)
  
  console.log('Current player added to ranking:', currentPlayerData)
  console.log('All players:', players.value)
  
  // Trier par nombre de cookies (ordre décroissant)
  players.value.sort((a, b) => b.cookies - a.cookies)
}

// Charger les données au montage du composant
onMounted(() => {
  loadPlayersData().then(() => {
    updateCurrentPlayer() // S'assurer que le joueur actuel est ajouté dès le début
  })
})

const getMedal = (index) => {
  if (index === 0) return "🥇"
  if (index === 1) return "🥈"
  if (index === 2) return "🥉"
  return ""
}

const formatNumber = (num) => {
  if (num >= 1e9) return (num / 1e9).toFixed(1) + "B"
  if (num >= 1e6) return (num / 1e6).toFixed(1) + "M"
  if (num >= 1e3) return (num / 1e3).toFixed(1) + "K"
  return num.toString()
}



// Watcher pour mettre à jour le ranking quand les cookies du joueur changent
watch(() => props.cookies, () => {
  if (isOpen.value) {
    updateCurrentPlayer()
  }
})

// Watcher pour mettre à jour quand le statut de connexion change
watch(() => props.isConnected, () => {
  if (isOpen.value) {
    updateCurrentPlayer()
  }
})

// Watcher pour mettre à jour quand le username/display name change
watch(() => props.username, () => {
  if (isOpen.value) {
    updateCurrentPlayer()
  }
})


</script>

<style scoped>
.ranking-container {
  position: relative;
  margin-bottom: 20px;
  width: 80%;
}

.ranking-btn {
  width: 100%;
  padding: 16px 20px;
  background: 
    linear-gradient(135deg, rgba(0, 123, 255, 0.8), rgba(0, 86, 179, 0.9)),
    rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);
  border: 1px solid rgba(0, 123, 255, 0.3);
  border-radius: 12px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  font-size: 16px;
  min-height: 60px;
  box-shadow: 
    0 4px 16px rgba(0, 123, 255, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  position: relative;
  overflow: hidden;
}

.ranking-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg, 
    transparent 30%, 
    rgba(255, 255, 255, 0.1) 50%, 
    transparent 70%);
  transform: translateX(-100%);
  transition: transform 0.6s ease;
}

.ranking-btn:hover::before {
  transform: translateX(100%);
}

.ranking-btn:hover {
  background: 
    linear-gradient(135deg, rgba(0, 123, 255, 0.9), rgba(0, 86, 179, 1)),
    rgba(255, 255, 255, 0.15);
  border-color: rgba(0, 123, 255, 0.4);
  transform: translateY(-2px);
  box-shadow: 
    0 6px 20px rgba(0, 123, 255, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
}

.ranking-btn.active {
  background: 
    linear-gradient(135deg, rgba(0, 86, 179, 0.9), rgba(0, 123, 255, 0.8)),
    rgba(255, 255, 255, 0.15);
  border-color: rgba(0, 123, 255, 0.4);
  box-shadow: 
    0 4px 16px rgba(0, 123, 255, 0.25),
    inset 0 1px 0 rgba(255, 255, 255, 0.25);
}

.ranking-btn:focus {
  outline: none;
}

.arrow {
  transition: transform 0.3s ease;
  font-size: 18px;
}

.arrow.rotated {
  transform: rotate(180deg);
}

.ranking-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: white;
  border: 2px solid #0056b3;
  border-top: none;
  border-radius: 0 0 8px 8px;
  max-height: 40vh; /* Réduire un peu pour éviter de sortir de l'écran */
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

/* Cacher la barre de défilement */
.ranking-dropdown::-webkit-scrollbar {
  display: none;
}

.ranking-dropdown {
  -ms-overflow-style: none;  /* IE et Edge */
  scrollbar-width: none;  /* Firefox */
}

.ranking-header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 15px;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  border-bottom: 1px solid #dee2e6;
}



.ranking-header h3 {
  margin: 0;
  color: #333;
  font-size: 18px;
}



.ranking-list {
  padding: 10px;
  padding-bottom: 40px; /* Encore plus d'espace en bas pour voir le dernier élément */
}

.player-item {
  display: flex;
  align-items: center;
  padding: 12px;
  border-radius: 6px;
  margin-bottom: 8px;
  background: #f8f9fa;
  transition: all 0.2s ease;
}

.player-item:hover {
  background: #e9ecef;
}

.player-item.current-player {
  background: linear-gradient(135deg, #fff3cd, #ffeaa7);
  border: 2px solid #ffc107;
  font-weight: bold;
  position: relative;
}

.player-item.current-player::after {
  content: "YOU";
  position: absolute;
  top: 5px;
  right: 10px;
  background: #ffc107;
  color: #495057;
  padding: 2px 8px;
  border-radius: 10px;
  font-size: 0.7em;
  font-weight: bold;
}

.loading, .no-players {
  text-align: center;
  padding: 20px;
  color: #6c757d;
  font-style: italic;
}

.rank {
  display: flex;
  align-items: center;
  margin-right: 15px;
  min-width: 50px;
}

.rank-number {
  font-weight: bold;
  color: #333;
  margin-right: 5px;
}

.medal {
  font-size: 16px;
}

.player-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.player-name {
  font-weight: bold;
  color: #333;
  margin-bottom: 2px;
}

.player-score {
  font-size: 12px;
  color: #666;
}

.player-cps {
  font-size: 12px;
  color: #007bff;
  font-weight: bold;
  text-align: right;
  min-width: 80px;
}
</style>