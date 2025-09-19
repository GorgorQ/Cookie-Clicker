<template>
  <div class="options-container">
    <button class="options-btn" @click="handleClick">
      ⚙️ Options
    </button>
  </div>
</template>

<script setup>
const handleClick = () => {
  console.log("Options clicked - no functionality yet")
}
</script>

<style scoped>
.options-container {
  width: 80%;
  margin-bottom: 20px;
}

.options-btn {
  width: 100%;
  padding: 16px 20px;
  background: linear-gradient(135deg, #6c757d, #495057);
  border: 2px solid #495057;
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

.options-btn:hover {
  background: linear-gradient(135deg, #5a6268, #343a40);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(108, 117, 125, 0.3);
}

.options-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 6px rgba(108, 117, 125, 0.2);
}
</style>