<template>
  <h1>BAYROU Clicker SIMULATOR</h1>
  <div class="BAYROU">
    <img class="cookie-btn" src="./assets/image.png" @click="count++" alt="Cookie">
  </div>
  <div>
    <p>Number of cookies {{ count }}</p>
  </div>
</template>

<script setup>
  import { ref } from "vue"
  const count = ref(0)
</script>

<style>


.BAYROU {
  width: 300px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 auto;
}

.cookie-btn {
  width: 300px;
  height: 300px;
  border-radius: 50%;
  box-shadow: 0 4px 16px rgba(0,0,0,0.2);
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  border: 3px solid #d2b48c;
  background: #fff;
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