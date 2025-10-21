<template>
  <div id="game-app">
    <div class="tabs">
      <button :class="{active: currentLevel === 1}" @click="currentLevel = 1">第一关</button>
      <button :class="{active: currentLevel === 2}" @click="currentLevel = 2">第二关</button>
      <button :class="{active: currentLevel === 3}" @click="currentLevel = 3">第三关</button>
    </div>
    <div class="stage">
      <LevelOne v-if="currentLevel === 1" @next-level="goToNextLevel" />
      <LevelTwo v-else-if="currentLevel === 2" />
      <LevelThree v-else />
    </div>
  </div>
  
</template>

<script>
import LevelOne from './components/LevelOne.vue'
import LevelTwo from './components/LevelTwo.vue'
import LevelThree from './components/LevelThree.vue'
export default {
  name: 'App',
  components: {
    LevelOne,
    LevelTwo,
    LevelThree
  },
  data() {
    return {
      currentLevel: 1
    }
  },
  methods: {
    goToNextLevel() {
      if (this.currentLevel < 3) {
        this.currentLevel++
      }
    }
  }
}
</script>

<style>
#game-app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 24px auto;
  max-width: 900px;
  padding: 0 12px;
}
.tabs { display: flex; gap: 8px; justify-content: center; margin-bottom: 16px; }
.tabs button { padding: 8px 14px; border-radius: 8px; border: 1px solid #ccc; background: #fff; cursor: pointer; }
.tabs button.active { background: #42b983; color: #fff; border-color: #42b983; }
.stage { min-height: 480px; }
</style>
