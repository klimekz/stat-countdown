<script setup lang="ts">
import { ref, Ref } from "vue";
import QuestionRow from "./QuestionRow.vue";
const x: Ref<number> = ref(1);

function incr() {
    x.value++;
}
</script>

<template>
    <div>
        <QuestionRow :time="5" v-for="i in x" />
        <button @click="incr"></button>
    </div>
</template>

<style scoped></style>
