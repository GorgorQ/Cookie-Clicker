<template>
  <div class="connection-container">
    <div v-if="isConnected" class="user-info">
      <span class="username">{{ username }}</span>
    </div>
    <button 
      class="connection-btn" 
      :class="{ connected: isConnected }"
      @click="toggleConnection"
    >
      <span class="icon">{{ isConnected ? '🔓' : '🔒' }}</span>
      {{ isConnected ? 'Disconnect' : 'Connect' }}
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const isConnected = ref(false)
const username = ref('Player')

const toggleConnection = () => {
  if (isConnected.value) {
    // Déconnexion
    isConnected.value = false
    console.log('Disconnected')
  } else {
    // Connexion (simulation)
    isConnected.value = true
    username.value = 'Player' + Math.floor(Math.random() * 1000)
    console.log('Connected as', username.value)
  }
}
</script>

<style scoped>
.connection-container {
  width: 80%;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.connection-btn {
  width: 100%;
  padding: 16px 20px;
  background: linear-gradient(135deg, #dc3545, #c82333);
  border: 2px solid #c82333;
  border-radius: 8px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  transition: all 0.3s ease;
  font-size: 16px;
  min-height: 60px;
}

.connection-btn.connected {
  background: linear-gradient(135deg, #28a745, #218838);
  border-color: #218838;
}

.connection-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(220, 53, 69, 0.3);
}

.connection-btn.connected:hover {
  box-shadow: 0 4px 12px rgba(40, 167, 69, 0.3);
}

.connection-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
}

.icon {
  font-size: 18px;
}

.user-info {
  background: rgba(40, 167, 69, 0.1);
  border: 1px solid #28a745;
  border-radius: 6px;
  padding: 8px 12px;
  color: #28a745;
  font-size: 14px;
  font-weight: bold;
}

.username {
  display: flex;
  align-items: center;
  gap: 5px;
}

.username::before {
  content: '👤';
}
</style>