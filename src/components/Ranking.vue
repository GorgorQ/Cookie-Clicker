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
        <button @click="refreshRanking" class="refresh-btn">🔄</button>
      </div>
      
      <div class="ranking-list">
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
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps(['cookies', 'cps', 'playerName'])

const isOpen = ref(false)

const players = ref([
  { id: 1, name: "CookieMaster", cookies: 15847293, cps: 25000, isCurrentPlayer: false },
  { id: 2, name: "BakeQueen", cookies: 12934587, cps: 19500, isCurrentPlayer: false },
  { id: 3, name: "ChocolateKing", cookies: 8456321, cps: 15200, isCurrentPlayer: false },
  { id: 4, name: "SweetTooth", cookies: 6234789, cps: 12800, isCurrentPlayer: false },
  { id: 5, name: "CrumbCollector", cookies: 4987654, cps: 9600, isCurrentPlayer: false },
  { id: 6, name: "DoughMaster", cookies: 3456789, cps: 7400, isCurrentPlayer: false },
  { id: 7, name: "OvenLord", cookies: 2345678, cps: 5200, isCurrentPlayer: false },
  { id: 8, name: "FlourPower", cookies: 1234567, cps: 3800, isCurrentPlayer: false }
])

const toggleDropdown = () => {
  isOpen.value = !isOpen.value
  if (isOpen.value) {
    updateCurrentPlayer()
  }
}

const refreshRanking = () => {
  // Simulation d'actualisation des données
  console.log("Refreshing ranking...")
  updateCurrentPlayer()
}

const updateCurrentPlayer = () => {
  // Ajouter/mettre à jour le joueur actuel
  const currentPlayerData = {
    id: 999,
    name: props.playerName || "You",
    cookies: props.cookies || 0,
    cps: props.cps || 0,
    isCurrentPlayer: true
  }
  
  // Retirer l'ancien joueur actuel
  const existingIndex = players.value.findIndex(p => p.isCurrentPlayer)
  if (existingIndex !== -1) {
    players.value.splice(existingIndex, 1)
  }
  
  // Ajouter le nouveau et trier
  players.value.push(currentPlayerData)
  players.value.sort((a, b) => b.cookies - a.cookies)
}

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
  background: linear-gradient(135deg, #007bff, #0056b3);
  border: 2px solid #0056b3;
  border-radius: 8px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.3s ease;
  font-size: 16px;
  min-height: 60px;
}

.ranking-btn:hover {
  background: linear-gradient(135deg, #0056b3, #004085);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 123, 255, 0.3);
}

.ranking-btn.active {
  background: linear-gradient(135deg, #0056b3, #007bff);
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
  max-height: 400px;
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.ranking-dropdown::-webkit-scrollbar {
  display: none;
}

.ranking-header {
  display: flex;
  justify-content: space-between;
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

.refresh-btn {
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  padding: 5px;
  border-radius: 4px;
  transition: background 0.2s;
}

.refresh-btn:hover {
  background: rgba(0, 0, 0, 0.1);
}

.ranking-list {
  padding: 10px;
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