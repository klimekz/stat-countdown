<script setup lang="ts">
import { ref, Ref } from "vue";
import Questions from "./components/Questions.vue";
const x: Ref<number> = ref(1);

function incr() {
  x.value++;
}
</script>

<template>
  <div>
    <h1>NBA Stat Game</h1>
    <h3>Click on the player with the higher per game average for the selected stat. For each question, the range between
      each players' average PPG is narrowed down.
    </h3>
    <Questions></Questions>
  </div>
</template>

<style scoped></style>
