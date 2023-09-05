<script setup lang="ts">
import { Ref, ref } from "vue";
import GameInstance from "./components/GameInstance.vue";
import DailyStats from "./components/DailyStats.vue";

const playing: Ref<boolean> = ref(false);
const showStats: Ref<boolean> = ref(false);
const completedGames: Ref<number> = ref(0);

function startDailyChallenge() {
  playing.value = true;
}

// function startRandomGame() {
//   alert("'Free Play' coming soon! 'Free Play' will allow additional options to filter time ranges against the data. Please create an account to have access to 'Free Play' on drop !")
// }

function titleClickHandler() {
  if (playing.value) {
    playing.value = !playing.value;
    showStats.value = true;
  }
}

function handleCompletedGame() {
  completedGames.value = completedGames.value + 1;
}



function toggleStats() {
  showStats.value = !showStats.value;
}

</script>

<template>
  <div class="page">
    <h1 @click="titleClickHandler">STAT COUNTDOWN</h1>
    <div class="headerContainer" v-if="!playing">
      <h4 class="textContent">STAT COUNTDOWN is a sports trivia game against the clock.</h4>
      <h4 class="textContent">Click on the player with the higher average per game for the displayed statistic
        (PTS/REB/AST). The range
        between the players' average for the chosen statistic per game will be narrowed down for each question.
      </h4>
      <h4 class="textContent">Complete today's quiz to see how you rank against the field.</h4>
    </div>
    <div class="buttons">
      <button v-if="!playing" @click="startDailyChallenge">Daily Challenge</button>
      <!-- <button v-if="!playing" @click="startRandomGame">Free Play</button> -->
      <button v-if="!playing && !showStats" @click="toggleStats">Show Stats</button>
      <button v-if="!playing && showStats" @click="toggleStats">Hide Stats</button>

    </div>
    <GameInstance class="gameBorder" v-if="playing" @count-completed="handleCompletedGame"
      @restart-game="titleClickHandler" :numCompleted="completedGames"></GameInstance>
    <DailyStats v-if="!playing && showStats" />
  </div>
  <div class="footerContainer">
    <div>
      <p><a href="https://www.statcountdown.com">STAT COUNTDOWN</a> S'23</p>
    </div>
  </div>
</template>

<style scoped>
button {
  margin: 2em;
}

.headerContainer {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 75%;
  align-self: center;
}

h4 {
  max-width: 65%;

}

.footerContainer {
  margin-top: 2.66em;
  margin-bottom: 2.66em;
}

.gameBorder {
  margin-bottom: 2em;
}

.page {
  display: flex;
  flex-direction: column;
}

.textContent {
  max-width: 55%;
  margin-bottom: .33em;
}

h1,
p {
  margin-bottom: .116em;

}
</style>
