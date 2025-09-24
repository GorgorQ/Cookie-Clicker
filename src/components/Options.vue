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
        <button @click="exportSave" class="control-btn export-btn">
          📤 Export Save
        </button>
        
        <button @click="triggerImport" class="control-btn import-btn">
          📥 Import Save
        </button>
        
        <button @click="clearSave" class="control-btn clear-btn">
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
      
      <!-- Toujours afficher les infos de sauvegarde si on a des données -->
      <div v-if="saveInfo" class="save-info">
        <p><strong>Current Save:</strong></p>
        <p>Player: {{ displayName || (isConnected ? username : 'Guest') }}</p>
        <p>Cookies: {{ formatNumber(saveInfo.cookies) }}</p>
        <p>Total Clicks: {{ formatNumber(saveInfo.totalClicks) }}</p>
        <p>CPS: {{ formatNumber(saveInfo.cps) }}</p>
        <p>Achievements: {{ (saveInfo.achievements || []).filter(a => a.unlocked).length }}</p>
        <p>Last Saved: {{ formatDate(saveInfo.lastSaved) }}</p>
      </div>
      
      <!-- Message pour les invités -->
      <div v-if="!isConnected" class="guest-info">
        <p>🌟 You are currently playing as a guest.</p>
        <p>Your progress can still be exported/imported but won't be saved permanently.</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps(['isConnected', 'username', 'displayName', 'gameData'])
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

// Exporter la sauvegarde (disponible pour tous)
const exportSave = () => {
  console.log('Export save clicked')
  console.log('Props gameData:', props.gameData)
  
  try {
    // Obtenir les données depuis localStorage directement
    const localData = localStorage.getItem('cookieClickerSave')
    let saveData
    
    if (localData) {
      console.log('Using localStorage data')
      saveData = JSON.parse(localData)
    } else if (props.gameData) {
      console.log('Using props gameData')
      saveData = props.gameData
    } else {
      console.log('No data available')
      alert('No save data to export! Play the game first.')
      return
    }
    
    // Créer l'objet d'export
    const exportData = {
      version: "1.0",
      player: props.displayName || (props.isConnected ? props.username : 'Guest'),
      cookies: saveData.cookies || 0,
      totalClicks: saveData.totalClicks || 0,
      cps: saveData.cps || 0,
      upgrades: saveData.upgrades || [],
      achievements: saveData.achievements || [],
      exportDate: new Date().toISOString()
    }
    
    console.log('Export data prepared:', exportData)
    
    // Encoder en base64 de manière sécurisée pour gérer les caractères Unicode
    const jsonString = JSON.stringify(exportData)
    const exportString = btoa(unescape(encodeURIComponent(jsonString)))
    
    // Créer et télécharger le fichier
    const displayNameForFile = props.displayName || (props.isConnected ? props.username : 'guest')
    const filename = `${displayNameForFile.toLowerCase().replace(/\s+/g, '_')}_save.txt`
    
    // Méthode simple de téléchargement
    const element = document.createElement('a')
    element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(exportString))
    element.setAttribute('download', filename)
    element.style.display = 'none'
    
    document.body.appendChild(element)
    element.click()
    document.body.removeChild(element)
    
    console.log('Export completed')
    alert('Save exported successfully!')
    
  } catch (error) {
    console.error('Export error:', error)
    alert(`Export failed: ${error.message}`)
  }
}

// Déclencher l'import (disponible pour tous)
const triggerImport = () => {
  fileInput.value?.click()
}

// Importer la sauvegarde (disponible pour tous)
const importSave = (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      let saveString = e.target.result
      let saveData
      
      try {
        // Essayer d'abord avec la nouvelle méthode de décodage (gère Unicode)
        saveData = JSON.parse(decodeURIComponent(escape(atob(saveString))))
      } catch {
        try {
          // Fallback vers l'ancienne méthode
          saveData = JSON.parse(atob(saveString))
        } catch {
          // Fallback vers JSON direct (non encodé)
          saveData = JSON.parse(saveString)
        }
      }
      
      if (typeof saveData.cookies !== 'number' || saveData.cookies < 0) {
        throw new Error('Invalid save format: cookies')
      }
      
      const confirmImport = confirm(
        `Import save from ${saveData.player || 'Unknown'}?\n` +
        `Cookies: ${formatNumber(saveData.cookies)}\n` +
        `CPS: ${formatNumber(saveData.cps || 0)}\n` +
        `Export Date: ${saveData.exportDate ? new Date(saveData.exportDate).toLocaleString() : 'Unknown'}\n\n` +
        `This will overwrite your current save!`
      )
      
      if (!confirmImport) return
      
      const importedData = {
        cookies: saveData.cookies || 0,
        totalClicks: saveData.totalClicks || 0,
        cps: saveData.cps || 0,
        upgrades: saveData.upgrades || [],
        achievements: saveData.achievements || [],
        lastSaved: new Date().toISOString()
      }
      
      emit('load-save-data', importedData)
      
      console.log('Save imported for', props.isConnected ? props.username : 'Guest')
      alert('Save imported successfully!')
    } catch (error) {
      console.error('Error importing save:', error)
      alert('Invalid save file format!')
    }
  }
  
  reader.readAsText(file)
  event.target.value = ''
}

// Effacer la sauvegarde (disponible pour tous)
const clearSave = () => {
  const confirmClear = confirm(
    'Are you sure you want to clear your save?\n' +
    'This action cannot be undone!\n\n' +
    'Current progress:\n' +
    `Cookies: ${formatNumber(props.gameData?.cookies || 0)}\n` +
    `CPS: ${formatNumber(props.gameData?.cps || 0)}`
  )
  
  if (!confirmClear) return
  
  // Les données vides - les upgrades seront gérés par App.vue
  const emptyData = {
    cookies: 0,
    totalClicks: 0,
    cps: 0,
    upgrades: [], // App.vue gérera la réinitialisation complète
    achievements: [],
    lastSaved: new Date().toISOString()
  }
  
  emit('clear-save-data', emptyData)
  
  console.log('Save cleared for', props.isConnected ? props.username : 'Guest')
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
  width: 80%;
}

.save-manager-btn {
  width: 100%;
  padding: 16px 20px;
  background: 
    linear-gradient(135deg, rgba(23, 162, 184, 0.8), rgba(19, 132, 150, 0.9)),
    rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(15px);
  -webkit-backdrop-filter: blur(15px);
  border: 1px solid rgba(23, 162, 184, 0.3);
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
    0 4px 16px rgba(23, 162, 184, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  position: relative;
  overflow: hidden;
}

.save-manager-btn::before {
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

.save-manager-btn:hover::before {
  transform: translateX(100%);
}

.save-manager-btn:hover {
  background: 
    linear-gradient(135deg, rgba(23, 162, 184, 0.9), rgba(19, 132, 150, 1)),
    rgba(255, 255, 255, 0.15);
  border-color: rgba(23, 162, 184, 0.4);
  transform: translateY(-2px);
  box-shadow: 
    0 6px 20px rgba(23, 162, 184, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
}

.save-manager-btn.active {
  background: 
    linear-gradient(135deg, rgba(19, 132, 150, 0.9), rgba(23, 162, 184, 0.8)),
    rgba(255, 255, 255, 0.15);
  border-color: rgba(23, 162, 184, 0.4);
  box-shadow: 
    0 4px 16px rgba(23, 162, 184, 0.25),
    inset 0 1px 0 rgba(255, 255, 255, 0.25);
}

.save-manager-btn:focus {
  outline: none;
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

.guest-info {
  padding: 15px;
  text-align: center;
  color: #856404;
  background: #fff3cd;
  border-top: 1px solid #ffeaa7;
  font-style: italic;
  font-size: 13px;
}
</style>