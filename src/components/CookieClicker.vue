<template>
  <div class="cookie-clicker-container">
    <h1>BAYROU Clicker SIMULATOR</h1>
    <img class="cookie-btn" src="../assets/image.png" @click="$emit('click')" alt="Cookie">
    <p>Number of cookies {{ count }}</p>
  </div>
</template>

<script setup>
defineProps({
  count: {
    type: Number,
    required: true
  }
})

defineEmits(['click'])
</script>

<style scoped>
.cookie-clicker-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.cookie-btn {
  width: 300px;
  height: auto;
  cursor: pointer;
  border: none;
  background: transparent;
  transition: transform 0.1s;
}
.cookie-btn:hover {
  transform: scale(1.05);
}
.cookie-btn:active {
  transform: scale(0.95);
}
</style>
