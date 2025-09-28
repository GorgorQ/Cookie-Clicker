<template>
  <div class="achievements-container">
    <!-- Notifications -->
    <div class="notifications-container">
      <div 
        v-for="notification in notifications" 
        :key="notification.id"
        class="notification"
        :class="{ 'fade-in': notification.show }"
      >
        {{ notification.icon }} {{ notification.message }}
      </div>
    </div>

    <button 
      @click="toggleDropdown" 
      class="achievements-btn"
      :class="{ active: isOpen }"
    >
      🏆 Succès ({{ unlockedCount }}/{{ achievements.length }})
      <span class="arrow" :class="{ rotated: isOpen }">▼</span>
    </button>
    
    <div v-if="isOpen" class="achievements-dropdown">
      <div class="achievements-list">
        <div 
          v-for="achievement in achievements" 
          :key="achievement.id"
          class="achievement-item"
          :class="{ unlocked: achievement.unlocked }"
        >
          <div class="achievement-icon">{{ achievement.icon }}</div>
          <div class="achievement-info">
            <h4>{{ achievement.name }}</h4>
            <p>{{ achievement.description }}</p>
            <span v-if="achievement.unlocked" class="unlocked-text">✅ Débloqué</span>
            <span v-else class="locked-text">🔒 Verrouillé</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, nextTick, onMounted } from 'vue'

const props = defineProps(['cookies', 'totalClicks', 'upgrades', 'resetTrigger'])

const emit = defineEmits(['achievements-update'])

const isOpen = ref(false)
const notifications = ref([])

const achievements = ref([
  {
    id: 1,
    name: "Premier Cookie",
    description: "Cliquer 1 fois",
    icon: "🍪",
    unlocked: false,
    condition: () => props.totalClicks >= 1
  },
  {
    id: 2,
    name: "Cliqueur Novice",
    description: "Cliquer 100 fois",
    icon: "👆",
    unlocked: false,
    condition: () => props.totalClicks >= 100
  },
  {
    id: 3,
    name: "Grand-mère Adoptée",
    description: "Acheter 1 Grand-mère",
    icon: "👵",
    unlocked: false,
    condition: () => props.upgrades?.find(u => u.name === "Grandma")?.owned >= 1
  },
  {
    id: 4,
    name: "Industriel",
    description: "Acheter 10 améliorations",
    icon: "🏭",
    unlocked: false,
    condition: () => props.upgrades?.reduce((total, u) => total + u.owned, 0) >= 10
  },
  {
    id: 5,
    name: "Millionnaire",
    description: "Avoir 1,000,000 cookies",
    icon: "💰",
    unlocked: false,
    condition: () => props.cookies >= 1000000
  }
])


const showNotification = (message, icon = '🎉') => {
  const id = Date.now()
  const notification = {
    id,
    message,
    icon,
    show: false
  }
  
  notifications.value.push(notification)
  

  nextTick(() => {
    notification.show = true
  })
  

  setTimeout(() => {
    notification.show = false
    setTimeout(() => {
      const index = notifications.value.findIndex(n => n.id === id)
      if (index > -1) {
        notifications.value.splice(index, 1)
      }
    }, 500)
  }, 3000)
}


const checkAchievements = () => {
  let hasNewAchievement = false
  
  achievements.value.forEach(achievement => {
    if (!achievement.unlocked && achievement.condition()) {
      achievement.unlocked = true
      hasNewAchievement = true
      showNotification(`Succès débloqué: ${achievement.name}!`, achievement.icon)
    }
  })
  

  if (hasNewAchievement) {
    emit('achievements-update', achievements.value)
  }
}


const resetAchievements = () => {
  achievements.value.forEach(achievement => {
    achievement.unlocked = false
  })
  emit('achievements-update', achievements.value)
  console.log('Achievements reset to default state')
}


watch(() => props.resetTrigger, (newValue) => {
  if (newValue > 0) {
    resetAchievements()
  }
})


onMounted(() => {
  emit('achievements-update', achievements.value)
})


watch(() => props.upgrades, checkAchievements, { deep: true })
watch(() => props.cookies, checkAchievements)
watch(() => props.totalClicks, checkAchievements)

const unlockedCount = computed(() => {
  checkAchievements()
  return achievements.value.filter(a => a.unlocked).length
})

const toggleDropdown = () => {
  isOpen.value = !isOpen.value
}
</script>

<style scoped>
.notifications-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 9999;
  pointer-events: none;
}

.notification {
  background: linear-gradient(135deg, #28a745, #20c997);
  color: white;
  padding: 12px 20px;
  margin-bottom: 10px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(40, 167, 69, 0.3);
  font-weight: bold;
  transform: translateX(100%);
  opacity: 0;
  transition: all 0.5s ease;
  max-width: 300px;
  word-wrap: break-word;
}

.notification.fade-in {
  transform: translateX(0);
  opacity: 1;
}

.achievements-container {
  position: relative;
  margin-bottom: 20px;
  width: 80%;
}

.achievements-btn {
  width: 100%;
  padding: 16px 20px;
  background: 
    linear-gradient(135deg, rgba(255, 215, 0, 0.8), rgba(255, 179, 71, 0.9)),
    rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 215, 0, 0.3);
  border-radius: 12px;
  color: #333;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  font-size: 16px;
  min-height: 60px;
  box-shadow: 
    0 4px 16px rgba(255, 215, 0, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  position: relative;
  overflow: hidden;
}

.achievements-btn::before {
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

.achievements-btn:hover::before {
  transform: translateX(100%);
}

.achievements-btn:hover {
  background: 
    linear-gradient(135deg, rgba(255, 237, 78, 0.9), rgba(255, 201, 116, 1)),
    rgba(255, 255, 255, 0.15);
  border-color: rgba(255, 215, 0, 0.4);
  transform: translateY(-2px);
  box-shadow: 
    0 6px 20px rgba(255, 215, 0, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
}

.achievements-btn.active {
  background: 
    linear-gradient(135deg, rgba(255, 179, 71, 0.9), rgba(255, 215, 0, 0.8)),
    rgba(255, 255, 255, 0.15);
  border-color: rgba(255, 215, 0, 0.4);
  box-shadow: 
    0 4px 16px rgba(255, 215, 0, 0.25),
    inset 0 1px 0 rgba(255, 255, 255, 0.25);
}

.achievements-btn:focus {
  outline: none;
}

.arrow {
  transition: transform 0.3s ease;
  font-size: 18px;
}

.arrow.rotated {
  transform: rotate(180deg);
}

.achievements-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: white;
  border: 2px solid #daa520;
  border-top: none;
  border-radius: 0 0 8px 8px;
  max-height: 300px;
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}
.achievements-dropdown::-webkit-scrollbar {
  display: none; 
}

.achievements-list {
  padding: 10px;
}

.achievement-item {
  display: flex;
  align-items: center;
  padding: 10px;
  border-radius: 6px;
  margin-bottom: 8px;
  background: #f5f5f5;
  opacity: 0.6;
  transition: all 0.3s ease;
}

.achievement-item.unlocked {
  background: linear-gradient(135deg, #e8f5e8, #d4edda);
  opacity: 1;
  border: 1px solid #28a745;
}

.achievement-icon {
  font-size: 24px;
  margin-right: 12px;
  width: 40px;
  text-align: center;
}

.achievement-info h4 {
  margin: 0 0 4px 0;
  color: #333;
  font-size: 14px;
}

.achievement-info p {
  margin: 0 0 4px 0;
  color: #666;
  font-size: 12px;
}

.unlocked-text {
  color: #28a745;
  font-size: 11px;
  font-weight: bold;
}

.locked-text {
  color: #6c757d;
  font-size: 11px;
}
</style>