<template>
  <div class="app-container">
    <header class="app-header">
      <div class="header-content">
        <a href="/" class="app-title">Stat Countdown</a>
        <a href="https:github.com/klimekz/stat-countdown" class="app-title">
          GitHub
        </a>
      </div>
    </header>
    <div class="main-content pageTop">
      <div>
        <h2>Stat Countdown</h2>

        <button v-if="!playing" @click="startDailyChallenge">Daily Challenge</button>
        <button v-if="!playing && !showStats" @click="toggleStats">Show Stats</button>
        <button v-if="!playing && showStats" @click="toggleStats">Hide Stats</button>
        <p v-if="!playing && !showStats">Start a game or view today's stats.</p>
        <GameInstance class="gameBorder" v-if="playing" @count-completed="handleCompletedGame"
          @restart-game="titleClickHandler" :numCompleted="completedGames"></GameInstance>
        <div class="stats">
          <DailyStats v-if="!playing && showStats" />
        </div>
      </div>
    </div>
    <div class="footerContainer">
      <div class="footerRow">
        <p><a href="https://www.statcountdown.com">STAT COUNTDOWN</a> S'23</p>
        <p><a href="https://www.twitter.com/statcountdown">@statcountdown</a></p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
const menuOpen = ref(false);
import GameInstance from './components/GameInstance.vue'
import DailyStats from './components/DailyStats.vue'


const playing = ref(false);
const showStats = ref(false);
const completedGames = ref(0);

function startDailyChallenge() {
  playing.value = true;
}

function titleClickHandler() {
  if (playing.value) {
    playing.value = !playing.value;
    showStats.value = false;
  }
}

function handleCompletedGame() {
  completedGames.value = completedGames.value + 1;
}



function toggleStats() {
  showStats.value = !showStats.value;
}

const toggleMenu = () => {
  menuOpen.value = !menuOpen.value;
};
</script>

<style scoped>
.app-container {
  width: 100%;
  padding-top: 3rem;
}

.footerContainer {
  margin-top: 3em;
  margin-bottom: 3em;
}

.footerRow {
  display: flex;
  justify-content: space-evenly;
}

.header-content {
  max-width: 1280px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.8rem 1rem;
}

.app-header {
  box-sizing: border-box;
  position: fixed;
  /* This pins the header to the top */
  top: 0;
  left: 0;
  right: 0;
  background-color: #eee;
  width: 100%;
  z-index: 1000;
}

.app-title {
  font-size: 1.2rem;
}

button {
  margin: 1em;
  width: 10em;
}

.main-content {
  display: flex;
  flex-direction: column;
  justify-content: start;
}
</style>