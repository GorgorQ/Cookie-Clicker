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
  border-radius: 50%;
  box-shadow: 0 4px 16px rgba(0,0,0,0.2);
  cursor: pointer;
  border: 3px solid #d2b48c;
  background: #fff;
  transition: transform 0.1s, box-shadow 0.1s, border-color 0.1s;
}
.cookie-btn:hover {
  transform: scale(1.08);
  box-shadow: 0 8px 32px rgba(0,0,0,0.3);
  border-color: #e2c48c;
}
.cookie-btn:active {
  transform: scale(0.95);
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
  border-color: #b8860b;
  filter: brightness(0.95);
}
</style>
