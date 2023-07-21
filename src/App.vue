<script setup lang="ts">
import { Ref, ref } from "vue";
import GameInstance from "./components/GameInstance.vue";

const playing: Ref<boolean> = ref(false);

function startDailyChallenge() {
  playing.value = true;
  // Populate list with player pairs from daily challenge data
}

function startRandomGame() {
  // playing.value = true;
  alert("'Free Play' coming soon! 'Free Play' will allow additional options to filter time ranges against the data. Please create an account to have access to 'Free Play' on drop!")
}
</script>

<template>
  <div>
    <h1>SHOT CLOCK O/U</h1>
    <h3>Click on the player with the higher per game average for the selected stat. For each question, the range between
      each players' average PPG is narrowed down.
    </h3>
    <h3>Complete today's quiz to see where you rank against the field, or test your knowledge against a random set of
      player pairs.</h3>
    <button v-if="!playing" @click="startDailyChallenge">Daily Challenge</button>
    <button v-if="!playing" @click="startRandomGame">Free Play</button>
    <GameInstance v-if="playing"></GameInstance>
  </div>
</template>

<style scoped>
button {
  margin: 2em;
}
</style>
