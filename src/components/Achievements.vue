<template>
  <div class="achievements-container">
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
import { ref, computed } from 'vue'

const props = defineProps(['cookies', 'totalClicks', 'upgrades'])

const isOpen = ref(false)

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
    name: "Millionnaire",
    description: "Avoir 1,000,000 cookies",
    icon: "💰",
    unlocked: false,
    condition: () => props.cookies >= 1000000
  },
  {
    id: 4,
    name: "Grand-mère Adoptée",
    description: "Acheter 1 Grand-mère",
    icon: "👵",
    unlocked: false,
    condition: () => props.upgrades?.find(u => u.name === "Grandma")?.owned >= 1
  },
  {
    id: 5,
    name: "Industriel",
    description: "Acheter 10 améliorations",
    icon: "🏭",
    unlocked: false,
    condition: () => props.upgrades?.reduce((total, u) => total + u.owned, 0) >= 10
  }
])

const unlockedCount = computed(() => {
  // Vérifier les conditions et débloquer les succès
  achievements.value.forEach(achievement => {
    if (!achievement.unlocked && achievement.condition()) {
      achievement.unlocked = true
    }
  })
  
  return achievements.value.filter(a => a.unlocked).length
})

const toggleDropdown = () => {
  isOpen.value = !isOpen.value
}
</script>

<style scoped>
.achievements-container {
  position: relative;
  margin-bottom: 20px;
}

.achievements-btn {
  width: 100%;
  padding: 12px 16px;
  background: linear-gradient(135deg, #ffd700, #ffb347);
  border: 2px solid #daa520;
  border-radius: 8px;
  color: #333;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.3s ease;
}

.achievements-btn:hover {
  background: linear-gradient(135deg, #ffed4e, #ffc974);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 215, 0, 0.3);
}

.achievements-btn.active {
  background: linear-gradient(135deg, #ffb347, #ffd700);
}

.arrow {
  transition: transform 0.3s ease;
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