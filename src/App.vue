<script setup lang="ts">
import { Ref, ref } from "vue";
import Questions from "./components/Questions.vue";

const playing: Ref<boolean> = ref(false);

function startDailyChallenge() {
  playing.value = true;
  // Populate list with player pairs from daily challenge data
}

// function startRandomGame() {
//   playing.value = true;
// }
</script>

<template>
  <div>
    <h1>HOOPS O/U</h1>
    <h3>Click on the player with the higher per game average for the selected stat. For each question, the range between
      each players' average PPG is narrowed down.
    </h3>
    <h3>Complete today's quiz to see where you rank against the field, or test your knowledge against a random set of
      player pairs.</h3>
    <button v-if="!playing" @click="startDailyChallenge">Daily Challenge</button>
    <Questions v-if="playing"></Questions>
  </div>
</template>

<style scoped></style>
