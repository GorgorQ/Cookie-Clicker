<template>
  <div class="container">
    <section class="left">
      <!-- Contenu à définir -->
    </section>
    <section class="center">
      <div class="cookies-stats">
        <p>Number of cookies {{ count }}</p>
        <p>Cookies per second: {{ cookiesPerSecond }}</p>
      </div>
      <img class="cookie-btn" src="./assets/image.png" @click="count++" alt="Cookie">
    </section>
    <section class="right">
      <Upgrades :cookies="count" :upgrades="upgrades" @buy-upgrade="buyUpgrade" />
    </section>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue"
import Upgrades from "./components/Upgrades.vue"

const count = ref(0)

const upgrades = ref([
  {
    id: 1,
    name: "Cursor",
    description: "Ajoute 1 clic par seconde",
    basePrice: 15,
    price: 15,
    owned: 0,
    cps: 1
  },
  {
    id: 2,
    name: "Grandma",
    description: "Une grand-mère gentille pour faire des cookies",
    basePrice: 100,
    price: 100,
    owned: 0,
    cps: 10
  },
  {
    id: 3,
    name: "Farm",
    description: "Cultive des cookies",
    basePrice: 1100,
    price: 1100,
    owned: 0,
    cps: 15
  }
])

const cookiesPerSecond = computed(() => {
  return upgrades.value.reduce((total, upgrade) => {
    return total + (upgrade.owned * upgrade.cps)
  }, 0)
})

const buyUpgrade = (upgradeId) => {
  const upgrade = upgrades.value.find(u => u.id === upgradeId)
  if (upgrade && count.value >= upgrade.price) {
    count.value -= upgrade.price
    upgrade.owned++
    upgrade.price = Math.floor(upgrade.basePrice * Math.pow(1.15, upgrade.owned))
  }
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


</style>