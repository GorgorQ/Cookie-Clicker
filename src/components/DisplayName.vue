<template>
  <div class="display-name-container">
    <input 
      v-model="displayName" 
      type="text" 
      placeholder="Bakery Name"
      class="display-name-input"
      maxlength="20"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const props = defineProps(['isConnected', 'username'])
const emit = defineEmits(['display-name-changed'])

const displayName = ref('')


const currentDisplayName = computed(() => {

  if (displayName.value && displayName.value.trim()) {
    return displayName.value.trim()
  }

  return props.isConnected && props.username ? props.username : 'Guest'
})


const loadDisplayName = () => {
  const saved = localStorage.getItem('cookieClicker_displayName')
  if (saved) {
    displayName.value = saved
  }
}


const saveDisplayName = () => {
  const trimmedName = displayName.value.trim()
  
  if (trimmedName && trimmedName.length >= 2) {
    localStorage.setItem('cookieClicker_displayName', trimmedName)
    console.log('Display name saved:', trimmedName)
  } else if (trimmedName === '') {

    localStorage.removeItem('cookieClicker_displayName')
    console.log('Display name cleared')
  }
  

  emit('display-name-changed')
}




watch(displayName, (newValue) => {
  saveDisplayName()
}, { immediate: false })

onMounted(() => {
  loadDisplayName()
})


defineExpose({
  currentDisplayName
})
</script>

<style scoped>
.display-name-container {
  width: 80%;
  margin-bottom: 15px;
}

.display-name-input {
  width: 100%;
  padding: 16px 20px;
  border: 2px solid #6f42c1;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  color: #212529;
  text-align: center;
  background: white;
  transition: all 0.3s ease;
  box-sizing: border-box;
}

.display-name-input:focus {
  outline: none;
  border-color: #5a32a3;
  box-shadow: 0 0 0 3px rgba(111, 66, 193, 0.2);
  transform: translateY(-1px);
}

.display-name-input::placeholder {
  color: #6c757d;
  font-style: italic;
}

.display-name-input:hover {
  border-color: #5a32a3;
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(111, 66, 193, 0.1);
}
</style>