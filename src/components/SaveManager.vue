<template>
  <div class="save-manager-container">
    <button 
      @click="toggleDropdown" 
      class="save-manager-btn"
      :class="{ active: isOpen }"
    >
      💾 Save Manager
      <span class="arrow" :class="{ rotated: isOpen }">▼</span>
    </button>
    
    <div v-if="isOpen" class="save-manager-dropdown">
      <div class="save-manager-header">
        <h3>Save Manager</h3>
      </div>
      
      <div class="save-controls">
        <button @click="exportSave" class="control-btn export-btn" :disabled="!isConnected">
          📤 Export Save
        </button>
        
        <button @click="triggerImport" class="control-btn import-btn" :disabled="!isConnected">
          📥 Import Save
        </button>
        
        <button @click="clearSave" class="control-btn clear-btn" :disabled="!isConnected">
          🗑️ Clear Save
        </button>
        
        <input 
          ref="fileInput"
          type="file" 
          accept=".txt,.json"
          @change="importSave"
          style="display: none"
        />
      </div>
      
      <div v-if="saveInfo" class="save-info">
        <p><strong>Current Save:</strong></p>
        <p>Cookies: {{ formatNumber(saveInfo.cookies) }}</p>
        <p>Total Clicks: {{ formatNumber(saveInfo.totalClicks) }}</p>
        <p>CPS: {{ formatNumber(saveInfo.cps) }}</p>
        <p>Last Saved: {{ formatDate(saveInfo.lastSaved) }}</p>
      </div>
      
      <div v-if="!isConnected" class="not-connected">
        <p>⚠️ You must be connected to manage saves</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps(['isConnected', 'username', 'gameData'])
const emit = defineEmits(['load-save-data', 'clear-save-data'])

const isOpen = ref(false)
const fileInput = ref(null)

const saveInfo = computed(() => {
  return props.gameData || null
})

const toggleDropdown = () => {
  isOpen.value = !isOpen.value
}

// Fermer le dropdown si l'utilisateur se déconnecte
watch(() => props.isConnected, (connected) => {
  if (!connected) {
    isOpen.value = false
  }
})

// Exporter la sauvegarde
const exportSave = () => {
  if (!props.isConnected || !props.gameData) {
    alert('No save data to export!')
    return
  }
  
  // Créer un objet de sauvegarde
  const saveData = {
    version: "1.0",
    player: props.username,
    cookies: props.gameData.cookies || 0,
    totalClicks: props.gameData.totalClicks || 0,
    cps: props.gameData.cps || 0,
    upgrades: props.gameData.upgrades || [],
    achievements: props.gameData.achievements || [],
    exportDate: new Date().toISOString()
  }
  
  // Encoder en base64
  const saveString = btoa(JSON.stringify(saveData))
  
  // Créer et télécharger le fichier
  const blob = new Blob([saveString], { type: 'text/plain' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `${props.username}_cookie_save.txt`
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
  
  console.log('Save exported for', props.username)
  alert('Save exported successfully!')
}

// Déclencher l'import
const triggerImport = () => {
  if (!props.isConnected) {
    alert('You must be connected to import a save!')
    return
  }
  fileInput.value?.click()
}

// Importer la sauvegarde
const importSave = (event) => {
  const file = event.target.files[0]
  if (!file || !props.isConnected) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      let saveString = e.target.result
      
      // Décoder depuis base64
      let saveData
      try {
        saveData = JSON.parse(atob(saveString))
      } catch {
        // Si ce n'est pas en base64, essayer directement
        saveData = JSON.parse(saveString)
      }
      
      // Valider les données
      if (typeof saveData.cookies !== 'number' || saveData.cookies < 0) {
        throw new Error('Invalid save format: cookies')
      }
      
      // Confirmer l'import
      const confirmImport = confirm(
        `Import save from ${saveData.player || 'Unknown'}?\n` +
        `Cookies: ${formatNumber(saveData.cookies)}\n` +
        `CPS: ${formatNumber(saveData.cps || 0)}\n` +
        `Export Date: ${saveData.exportDate ? new Date(saveData.exportDate).toLocaleString() : 'Unknown'}\n\n` +
        `This will overwrite your current save!`
      )
      
      if (!confirmImport) return
      
      // Émettre les nouvelles données vers le parent
      const importedData = {
        cookies: saveData.cookies || 0,
        totalClicks: saveData.totalClicks || 0,
        cps: saveData.cps || 0,
        upgrades: saveData.upgrades || [],
        achievements: saveData.achievements || [],
        lastSaved: new Date().toISOString()
      }
      
      emit('load-save-data', importedData)
      
      console.log('Save imported for', props.username)
      alert('Save imported successfully!')
    } catch (error) {
      console.error('Error importing save:', error)
      alert('Invalid save file format!')
    }
  }
  
  reader.readAsText(file)
  event.target.value = '' // Reset input
}

// Effacer la sauvegarde
const clearSave = () => {
  if (!props.isConnected) {
    alert('You must be connected to clear your save!')
    return
  }
  
  const confirmClear = confirm(
    'Are you sure you want to clear your save?\n' +
    'This action cannot be undone!\n\n' +
    'Current progress:\n' +
    `Cookies: ${formatNumber(props.gameData?.cookies || 0)}\n` +
    `CPS: ${formatNumber(props.gameData?.cps || 0)}`
  )
  
  if (!confirmClear) return
  
  // Émettre les données vides
  const emptyData = {
    cookies: 0,
    totalClicks: 0,
    cps: 0,
    upgrades: [],
    achievements: [],
    lastSaved: new Date().toISOString()
  }
  
  emit('clear-save-data', emptyData)
  
  console.log('Save cleared for', props.username)
  alert('Save cleared successfully!')
}

// Utilitaires
const formatNumber = (num) => {
  if (num >= 1e9) return (num / 1e9).toFixed(1) + 'B'
  if (num >= 1e6) return (num / 1e6).toFixed(1) + 'M'
  if (num >= 1e3) return (num / 1e3).toFixed(1) + 'K'
  return num.toString()
}

const formatDate = (dateString) => {
  if (!dateString) return 'Never'
  return new Date(dateString).toLocaleString()
}
</script>

<style scoped>
.save-manager-container {
  position: relative;
  margin-bottom: 20px;
  width: 100%;
}

.save-manager-btn {
  width: 100%;
  padding: 16px 20px;
  background: linear-gradient(135deg, #17a2b8, #138496);
  border: 2px solid #138496;
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

.save-manager-btn:hover {
  background: linear-gradient(135deg, #138496, #117a8b);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(23, 162, 184, 0.3);
}

.save-manager-btn.active {
  background: linear-gradient(135deg, #138496, #17a2b8);
}

.arrow {
  transition: transform 0.3s ease;
  font-size: 18px;
}

.arrow.rotated {
  transform: rotate(180deg);
}

.save-manager-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: white;
  border: 2px solid #138496;
  border-top: none;
  border-radius: 0 0 8px 8px;
  max-height: 400px;
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.save-manager-dropdown::-webkit-scrollbar {
  display: none;
}

.save-manager-header {
  padding: 15px;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  border-bottom: 1px solid #dee2e6;
}

.save-manager-header h3 {
  margin: 0;
  color: #333;
  font-size: 18px;
  text-align: center;
}

.save-controls {
  padding: 15px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.control-btn {
  width: 100%;
  padding: 10px 15px;
  border-radius: 6px;
  border: none;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 14px;
}

.export-btn {
  background: #28a745;
  color: white;
}

.export-btn:hover:not(:disabled) {
  background: #218838;
  transform: translateY(-1px);
}

.import-btn {
  background: #007bff;
  color: white;
}

.import-btn:hover:not(:disabled) {
  background: #0056b3;
  transform: translateY(-1px);
}

.clear-btn {
  background: #dc3545;
  color: white;
}

.clear-btn:hover:not(:disabled) {
  background: #c82333;
  transform: translateY(-1px);
}

.control-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.save-info {
  padding: 15px;
  background: #f8f9fa;
  border-top: 1px solid #dee2e6;
  font-size: 13px;
}

.save-info p {
  margin: 4px 0;
  color: #495057;
}

.save-info strong {
  color: #212529;
}

.not-connected {
  padding: 15px;
  text-align: center;
  color: #6c757d;
  font-style: italic;
}
</style>