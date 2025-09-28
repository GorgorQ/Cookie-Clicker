<template>
  <div class="connection-container">
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
           Sign Up
        </button>
      </div>

      <button @click="showForm = false" class="cancel-btn">
         Cancel
      </button>

      <div v-if="errorMessage" class="error-message">
        {{ errorMessage }}
      </div>
    </div>

    <!-- Bouton de connexion/déconnexion (toujours visible) -->
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


const props = defineProps(['cookies', 'upgrades', 'achievements', 'totalClicks', 'cps'])
const emit = defineEmits(['load-user-data', 'user-disconnected'])


onMounted(async () => {
  await loadUsersFromJSON()
  loadCurrentUser()
  

  const guestData = localStorage.getItem('cookieClicker_guestData')
  if (guestData && !connectionRef.value?.isConnected) {
    try {
      const data = JSON.parse(guestData)
      count.value = data.cookies || 0
      totalClicks.value = data.totalClicks || 0
      cookiesPerSecond.value = data.cps || 0
      currentUpgrades.value = data.upgrades || []
      currentAchievements.value = data.achievements || []
      console.log('Guest data loaded from localStorage')
    } catch (error) {
      console.error('Error loading guest data:', error)
    }
  }
})

const canSignin = computed(() => {
  return signinData.value.username.trim() && signinData.value.password.trim()
})

const canSignup = computed(() => {
  return signupData.value.username.trim() && signupData.value.password.trim()
})


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


const saveUsersToStorage = () => {

  localStorage.setItem('cookieClicker_users_backup', JSON.stringify(users.value))
  
  console.log('Users data updated and saved to localStorage backup')
}


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
    loadUserGameData(true)
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
    

    setTimeout(() => {
      isConnected.value = false
      username.value = ''
      removeCurrentUser()
      

      emit('user-disconnected')
      
      console.log('Disconnected and data saved')
    }, 100)
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

const loadUserGameData = (isFirstConnection = false) => {
  if (!isConnected.value) return
  

  if (isFirstConnection) {
    const user = users.value.find(user => user.username === username.value)
    
    if (user && user.gameData) {
      emit('load-user-data', user.gameData)
      console.log('Game data loaded from JSON for first connection:', username.value, user.gameData)
      return
    }
  }
  

  const localStorageData = localStorage.getItem('cookieClickerSave')
  
  if (localStorageData) {
    try {
      const gameData = JSON.parse(localStorageData)

      emit('load-user-data', gameData)
      console.log('Game data loaded from localStorage for connected user:', username.value, gameData)
      return
    } catch (error) {
      console.error('Error loading from localStorage for connected user:', error)
    }
  }
  

  const user = users.value.find(user => user.username === username.value)
  
  if (user && user.gameData) {
    emit('load-user-data', user.gameData)
    console.log('Game data loaded from JSON for', username.value, user.gameData)
  }
}

defineExpose({
  saveUserGameData,
  isConnected,
  username, 
  downloadUpdatedJSON  
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
  width: 420px;
  max-width: 90vw;
  background: rgba(255, 255, 255, 0.25);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  padding: 30px;
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.1),
    0 4px 16px rgba(0, 0, 0, 0.05),
    inset 0 1px 0 rgba(255, 255, 255, 0.4);
  z-index: 9999;
  overflow: hidden;
  animation: formSlideIn 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

/* Effet liquid glass animé en arrière-plan */
.auth-form::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: conic-gradient(
    from 0deg,
    rgba(255, 255, 255, 0.1) 0deg,
    rgba(255, 255, 255, 0.2) 60deg,
    rgba(255, 255, 255, 0.1) 120deg,
    rgba(255, 255, 255, 0.3) 180deg,
    rgba(255, 255, 255, 0.1) 240deg,
    rgba(255, 255, 255, 0.2) 300deg,
    rgba(255, 255, 255, 0.1) 360deg
  );
  animation: liquidRotate 20s linear infinite;
  z-index: -1;
}

/* Overlay pour adoucir l'effet */
.auth-form::after {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(2px);
  z-index: -2;
}

@keyframes formSlideIn {
  0% {
    opacity: 0;
    transform: translate(-50%, -60%) scale(0.9);
  }
  100% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
}

@keyframes liquidRotate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.form-tabs {
  display: flex;
  margin-bottom: 25px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 4px;
  backdrop-filter: blur(10px);
}

.tab-btn {
  flex: 1;
  padding: 12px 16px;
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.8);
  font-weight: 600;
  cursor: pointer;
  border-radius: 8px;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  position: relative;
}

.tab-btn.active {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.tab-btn:hover {
  background: rgba(255, 255, 255, 0.15);
  color: white;
  transform: translateY(-1px);
}

.form-content h3 {
  margin: 0 0 20px 0;
  color: white;
  font-size: 20px;
  font-weight: 700;
  text-align: center;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 20px;
}

.form-group input {
  width: 100%;
  padding: 16px 20px;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  font-size: 16px;
  color: white;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  box-sizing: border-box;
}

.form-group input::placeholder {
  color: rgba(255, 255, 255, 0.6);
}

.form-group input:focus {
  outline: none;
  border-color: rgba(255, 255, 255, 0.4);
  background: rgba(255, 255, 255, 0.2);
  box-shadow: 
    0 0 0 2px rgba(255, 255, 255, 0.1),
    0 4px 12px rgba(0, 0, 0, 0.1);
  transform: translateY(-2px);
}

.submit-btn {
  width: 100%;
  padding: 16px 24px;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.3), rgba(255, 255, 255, 0.1));
  backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 12px;
  color: white;
  font-weight: 700;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  margin-bottom: 16px;
  box-shadow: 
    0 4px 16px rgba(0, 0, 0, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
}

.submit-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.4), rgba(255, 255, 255, 0.2));
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-2px);
  box-shadow: 
    0 6px 20px rgba(0, 0, 0, 0.15),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
}

.submit-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  transform: none;
}

.cancel-btn {
  width: 100%;
  padding: 12px 20px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.8);
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.cancel-btn:hover {
  background: rgba(255, 255, 255, 0.15);
  border-color: rgba(255, 255, 255, 0.3);
  color: white;
  transform: translateY(-1px);
}

.submit-btn:focus,
.cancel-btn:focus,
.tab-btn:focus {
  outline: none;
}

.error-message {
  margin-top: 16px;
  padding: 12px 16px;
  background: rgba(255, 99, 132, 0.2);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 99, 132, 0.3);
  border-radius: 10px;
  color: #fff;
  font-size: 14px;
  font-weight: 600;
  text-align: center;
  animation: errorShake 0.5s cubic-bezier(0.36, 0.07, 0.19, 0.97);
}

@keyframes errorShake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

.connection-btn {
  width: 100%;
  padding: 16px 20px;
  background: 
    linear-gradient(135deg, rgba(220, 53, 69, 0.8), rgba(200, 35, 51, 0.9)),
    rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);
  border: 1px solid rgba(220, 53, 69, 0.3);
  border-radius: 12px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  gap: 8px;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  font-size: 16px;
  min-height: 60px;
  box-shadow: 
    0 4px 16px rgba(220, 53, 69, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  position: relative;
  overflow: hidden;
}

.connection-btn::before {
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

.connection-btn:hover::before {
  transform: translateX(100%);
}

.connection-btn.connected {
  background: 
    linear-gradient(135deg, rgba(40, 167, 69, 0.8), rgba(33, 136, 56, 0.9)),
    rgba(255, 255, 255, 0.1);
  border-color: rgba(40, 167, 69, 0.3);
  box-shadow: 
    0 4px 16px rgba(40, 167, 69, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
}

.connection-btn:hover {
  transform: translateY(-2px);
  background: 
    linear-gradient(135deg, rgba(220, 53, 69, 0.9), rgba(200, 35, 51, 1)),
    rgba(255, 255, 255, 0.15);
  border-color: rgba(220, 53, 69, 0.4);
  box-shadow: 
    0 6px 20px rgba(220, 53, 69, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
}

.connection-btn.connected:hover {
  background: 
    linear-gradient(135deg, rgba(40, 167, 69, 0.9), rgba(33, 136, 56, 1)),
    rgba(255, 255, 255, 0.15);
  border-color: rgba(40, 167, 69, 0.4);
  box-shadow: 
    0 6px 20px rgba(40, 167, 69, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
}

.connection-btn:active {
  transform: translateY(0);
  box-shadow: 
    0 2px 8px rgba(0, 0, 0, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
}

.connection-btn:focus {
  outline: none;
}

.icon {
  font-size: 18px;
}


</style>