<script setup lang="ts">
import { Ref, ref } from "vue";
import GameInstance from "./components/GameInstance.vue";
import DailyStats from "./components/DailyStats.vue";

const playing: Ref<boolean> = ref(false);
const showStats: Ref<boolean> = ref(false);

function startDailyChallenge() {
  playing.value = true;
}

function startRandomGame() {
  alert("'Free Play' coming soon! 'Free Play' will allow additional options to filter time ranges against the data. Please create an account to have access to 'Free Play' on drop !")
}

function titleClickHandler() {
  if (playing.value) {
    playing.value = !playing.value;
  }
}

function toggleStats() {
  showStats.value = !showStats.value;
}
</script>

<template>
  <div>
    <h1 @click="titleClickHandler">STAT COUNTDOWN</h1>
    <div class="buttons">
      <button v-if="!playing" @click="startDailyChallenge">Daily Challenge</button>
      <button v-if="!playing" @click="startRandomGame">Free Play</button>
      <button v-if="!playing && !showStats" @click="toggleStats">Show Stats</button>
      <button v-if="!playing && showStats" @click="toggleStats">Hide Stats</button>
    </div>
    <h3>Click on the player with the higher per game average for the displayed stat. For each question, the range between
      each players' average PPG is narrowed down.
    </h3>
    <h3>Complete today's quiz to see where you rank against the field, or test your knowledge against a random set of
      player pairs.</h3>
    <GameInstance class="gameBorder" v-if="playing"></GameInstance>
    <DailyStats v-if="!playing && showStats" />
  </div>
</template>

<style scoped>
button {
  margin: 2em;
}

.gameBorder {
  margin-bottom: 2em;
}
</style>
