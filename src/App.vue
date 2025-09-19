<template>
  <div class="container">
    <section class="left">
      <Connection />
      <Options />
      <Ranking :cookies="count" :cps="cookiesPerSecond" :playerName="'You'" />
      <Achievements :cookies="count" :totalClicks="totalClicks" :upgrades="currentUpgrades" />
    </section>
    <section class="center">
      <div class="cookies-stats">
        <p>Number of cookies {{ count }}</p>
        <p>Cookies per second: {{ cookiesPerSecond }}</p>
      </div>
      <img class="cookie-btn" src="./assets/image.png" @click="handleCookieClick" alt="Cookie">
    </section>
    <section class="right">
      <Upgrades :cookies="count" @buy-upgrade="buyUpgrade" @cps-update="updateCps" @upgrades-update="updateUpgrades" />
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue"
import Upgrades from "./components/Upgrades.vue"
import Achievements from "./components/Achievements.vue"
import Options from "./components/Options.vue"
import Connection from "./components/Connection.vue"
import Ranking from "./components/Ranking.vue"

const count = ref(0)
const cookiesPerSecond = ref(0)
const totalClicks = ref(0)
const currentUpgrades = ref([])

const handleCookieClick = () => {
  count.value++
  totalClicks.value++
}

const buyUpgrade = (cost) => {
  count.value -= cost
}

const updateCps = (newCps) => {
  cookiesPerSecond.value = newCps
}

const updateUpgrades = (upgrades) => {
  currentUpgrades.value = upgrades
}

let interval = null

onMounted(() => {
  interval = setInterval(() => {
    count.value += cookiesPerSecond.value
  }, 1000)
})

onUnmounted(() => {
  if (interval) {
    clearInterval(interval)
  }
})
</script>

<style>
@import './style.css';

html, body, #app {
  height: 100%;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.container {
  display: flex;
  min-height: 100vh;
  height: 100vh;
  width: 100vw;
  min-width: 100vw;
  overflow: hidden;
}
.left, .center, .right {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  min-width: 0;
  padding: 20px;
}
.center {
  gap: 50px;
}
.cookies-stats {
  text-align: center;
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

.right {
  overflow-y: auto;
  overflow-x: hidden;
  max-height: 100vh;
}
.right::-webkit-scrollbar {
  display: none; 
}

</style>