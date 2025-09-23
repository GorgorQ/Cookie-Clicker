<template>
  <div class="connection-container">
    <!-- Utilisateur connecté -->
    <div v-if="isConnected" class="user-info">
      <span class="username">{{ username }}</span>
    </div>

    <!-- Formulaire de connexion/inscription -->
    <div v-if="!isConnected && showForm" class="auth-form">
      <div class="form-tabs">
        <button 
          @click="activeTab = 'signin'" 
          :class="{ active: activeTab === 'signin' }"
          class="tab-btn"
        >
          Sign In
        </button>
        <button 
          @click="activeTab = 'signup'" 
          :class="{ active: activeTab === 'signup' }"
          class="tab-btn"
        >
          Sign Up
        </button>
      </div>

      <!-- Sign In Form -->
      <div v-if="activeTab === 'signin'" class="form-content">
        <h3>Sign In</h3>
        <div class="form-group">
          <input 
            v-model="signinData.username" 
            type="text" 
            placeholder="Username"
            @keyup.enter="signin"
          />
        </div>
        <div class="form-group">
          <input 
            v-model="signinData.password" 
            type="password" 
            placeholder="Password"
            @keyup.enter="signin"
          />
        </div>
        <button @click="signin" class="submit-btn" :disabled="!canSignin">
          🔐 Sign In
        </button>
      </div>

      <!-- Sign Up Form -->
      <div v-if="activeTab === 'signup'" class="form-content">
        <h3>Sign Up</h3>
        <div class="form-group">
          <input 
            v-model="signupData.username" 
            type="text" 
            placeholder="Username"
            @keyup.enter="signup"
          />
        </div>
        <div class="form-group">
          <input 
            v-model="signupData.password" 
            type="password" 
            placeholder="Password"
            @keyup.enter="signup"
          />
        </div>
        <button @click="signup" class="submit-btn" :disabled="!canSignup">
          ✨ Sign Up
        </button>
      </div>

      <button @click="showForm = false" class="cancel-btn">
        ❌ Cancel
      </button>

      <div v-if="errorMessage" class="error-message">
        {{ errorMessage }}
      </div>
    </div>

    <!-- Bouton de connexion/déconnexion -->
    <button 
      v-if="!showForm"
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
import { ref, computed, onMounted } from 'vue'

const isConnected = ref(false)
const username = ref('')
const showForm = ref(false)
const activeTab = ref('signin')
const errorMessage = ref('')
const users = ref([])

const signinData = ref({
  username: '',
  password: ''
})

const signupData = ref({
  username: '',
  password: ''
})

// Props pour recevoir les données du jeu depuis App.vue
const props = defineProps(['cookies', 'upgrades', 'achievements', 'totalClicks', 'cps'])
const emit = defineEmits(['load-user-data'])

// Charger les données au démarrage
onMounted(async () => {
  await loadUsersFromJSON()
  loadCurrentUser()
})

const canSignin = computed(() => {
  return signinData.value.username.trim() && signinData.value.password.trim()
})

const canSignup = computed(() => {
  return signupData.value.username.trim() && signupData.value.password.trim()
})

// Charger les utilisateurs depuis le fichier JSON
const loadUsersFromJSON = async () => {
  try {
    const response = await fetch('/users.json')
    const data = await response.json()
    users.value = data.users || []
    console.log('Users loaded from JSON:', users.value.length, 'users')
  } catch (error) {
    console.error('Error loading users from JSON:', error)
    users.value = []
  }
}

// Sauvegarder les utilisateurs (sans téléchargement automatique)
const saveUsersToStorage = () => {
  // Sauvegarder dans localStorage comme backup
  localStorage.setItem('cookieClicker_users_backup', JSON.stringify(users.value))
  
  console.log('Users data updated and saved to localStorage backup')
}

// Fonction manuelle pour télécharger le JSON (optionnelle)
const downloadUpdatedJSON = () => {
  const blob = new Blob([JSON.stringify({ users: users.value }, null, 2)], {
    type: 'application/json'
  })
  
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'users_updated.json'
  a.style.display = 'none'
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
  
  console.log('Updated users.json downloaded')
}

const saveUser = (username, password, gameData = null) => {
  const userData = {
    username,
    password,
    createdAt: new Date().toISOString(),
    gameData: gameData || {
      cookies: 0,
      totalClicks: 0,
      cps: 0,
      upgrades: [],
      achievements: [],
      lastSaved: new Date().toISOString()
    }
  }
  users.value.push(userData)
  saveUsersToStorage()
}

const findUser = (username, password) => {
  return users.value.find(user => user.username === username && user.password === password)
}

const userExists = (username) => {
  return users.value.some(user => user.username === username)
}

const saveCurrentUser = (username) => {
  localStorage.setItem('cookieClicker_currentUser', username)
}

const loadCurrentUser = () => {
  const savedUser = localStorage.getItem('cookieClicker_currentUser')
  if (savedUser) {
    if (userExists(savedUser)) {
      isConnected.value = true
      username.value = savedUser
      setTimeout(() => {
        loadUserGameData()
      }, 100)
    } else {
      removeCurrentUser()
    }
  }
}

const removeCurrentUser = () => {
  localStorage.removeItem('cookieClicker_currentUser')
}

const signin = () => {
  if (!canSignin.value) return
  
  if (signinData.value.username.length < 3) {
    errorMessage.value = 'Username must be at least 3 characters'
    return
  }
  
  if (signinData.value.password.length < 4) {
    errorMessage.value = 'Password must be at least 4 characters'
    return
  }
  
  const user = findUser(signinData.value.username, signinData.value.password)
  if (!user) {
    errorMessage.value = 'Invalid username or password'
    return
  }
  
  isConnected.value = true
  username.value = signinData.value.username
  saveCurrentUser(signinData.value.username)
  showForm.value = false
  errorMessage.value = ''
  
  resetForms()
  
  setTimeout(() => {
    loadUserGameData()
  }, 100)
  
  console.log('Signed in as', username.value)
}

const signup = () => {
  if (!canSignup.value) return
  
  if (signupData.value.username.length < 3) {
    errorMessage.value = 'Username must be at least 3 characters'
    return
  }
  
  if (signupData.value.password.length < 4) {
    errorMessage.value = 'Password must be at least 4 characters'
    return
  }
  
  if (userExists(signupData.value.username)) {
    errorMessage.value = 'Username already exists'
    return
  }
  
  saveUser(signupData.value.username, signupData.value.password)
  
  isConnected.value = true
  username.value = signupData.value.username
  saveCurrentUser(signupData.value.username)
  showForm.value = false
  errorMessage.value = ''
  
  resetForms()
  console.log('Signed up as', username.value)
}

const resetForms = () => {
  signinData.value = { username: '', password: '' }
  signupData.value = { username: '', password: '' }
}

const toggleConnection = () => {
  if (isConnected.value) {
    saveUserGameData()
    
    isConnected.value = false
    username.value = ''
    removeCurrentUser()
    console.log('Disconnected')
  } else {
    showForm.value = true
    activeTab.value = 'signin'
    errorMessage.value = ''
  }
}

const saveUserGameData = () => {
  if (!isConnected.value) return
  
  const userIndex = users.value.findIndex(user => user.username === username.value)
  
  if (userIndex !== -1) {
    users.value[userIndex].gameData = {
      cookies: props.cookies || 0,
      totalClicks: props.totalClicks || 0,
      cps: props.cps || 0,
      upgrades: props.upgrades || [],
      achievements: props.achievements || [],
      lastSaved: new Date().toISOString()
    }
    saveUsersToStorage()
    console.log('Game data saved for', username.value)
  }
}

const loadUserGameData = () => {
  if (!isConnected.value) return
  
  const user = users.value.find(user => user.username === username.value)
  
  if (user && user.gameData) {
    emit('load-user-data', user.gameData)
    console.log('Game data loaded for', username.value, user.gameData)
  }
}

defineExpose({
  saveUserGameData,
  isConnected,
  downloadUpdatedJSON  // Optionnel: exposer la fonction pour usage manuel
})
</script>

<style scoped>
.connection-container {
  width: 80%;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  position: relative;
}

.auth-form {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 400px;
  max-width: 90vw;
  background: #ffffff;
  border: 1px solid #e1e5e9;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  z-index: 9999;
}

.auth-form::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.5);
  z-index: -1;
}

.form-tabs {
  display: flex;
  margin-bottom: 20px;
  border-bottom: 1px solid #e1e5e9;
}

.tab-btn {
  flex: 1;
  padding: 12px;
  background: none;
  border: none;
  color: #656d76;
  font-weight: 500;
  cursor: pointer;
  border-bottom: 2px solid transparent;
  transition: all 0.2s ease;
}

.tab-btn.active {
  color: #0969da;
  border-bottom-color: #0969da;
}

.tab-btn:hover {
  color: #0969da;
}

.form-content h3 {
  margin: 0 0 16px 0;
  color: #24292f;
  font-size: 18px;
  font-weight: 600;
  text-align: center;
}

.form-group {
  margin-bottom: 16px;
}

.form-group input {
  width: 100%;
  padding: 12px;
  border: 1px solid #e1e5e9;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.2s ease;
  box-sizing: border-box;
}

.form-group input:focus {
  outline: none;
  border-color: #0969da;
  box-shadow: 0 0 0 2px rgba(9, 105, 218, 0.1);
}

.submit-btn {
  width: 100%;
  padding: 12px;
  background: #0969da;
  border: 1px solid #0969da;
  border-radius: 6px;
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  margin-bottom: 12px;
}

.submit-btn:hover:not(:disabled) {
  background: #0860ca;
  border-color: #0860ca;
}

.submit-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.cancel-btn {
  width: 100%;
  padding: 10px;
  background: #f6f8fa;
  border: 1px solid #e1e5e9;
  border-radius: 6px;
  color: #24292f;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.cancel-btn:hover {
  background: #eef2f5;
  border-color: #d0d7de;
}

.error-message {
  margin-top: 12px;
  padding: 8px 12px;
  background: #fff1f0;
  border: 1px solid #ffccc7;
  border-radius: 6px;
  color: #cf1322;
  font-size: 13px;
  text-align: center;
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
  width: 100%;
  text-align: center;
}

.username::before {
  content: '👤 ';
}
</style>