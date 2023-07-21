<script setup lang="ts">
import { onMounted, ref, Ref } from "vue";
import QuestionRow from "./QuestionRow.vue";

type QData = {
    playerAId: string,
    playerAName: string,
    playerAStat: number,
    playerBId: string,
    playerBName: string,
    playerBStat: number,
    season: string,
    answer: string,
    statCategory: string
}


const questionData: QData = {
    playerAId: "1628418",
    playerAName: "Thomas Bryant",
    playerAStat: 9.8,
    playerBId: "201565",
    playerBName: "Derrick Rose",
    playerBStat: 5.6,
    season: "22-23",
    answer: "Thomas Bryant",
    statCategory: "PTS"
}

const questionDataB: QData = {
    playerAId: "1628369",
    playerAName: "Jayson Tatum",
    playerAStat: 4.6,
    playerBId: "1630162",
    playerBName: "Anthony Edwards",
    playerBStat: 4.4,
    season: "22-23",
    answer: "Jayson Tatum",
    statCategory: "AST"
}

let numQuestions = 1;
const questions: Ref<QData[]> = ref([]);
const displayQuestions: Ref<QData[]> = ref([]);
const numCorrect: Ref<number> = ref(0);

function addNextQuestion() {
    if (numQuestions < 10) {
        displayQuestions.value.push(questions.value[numQuestions]);
        numQuestions++;
    }
}

function incrCorrect() {
    numCorrect.value++;
}

onMounted(() => {
    for (let i = 0; i < 5; i++) {
        questions.value.push(questionData)
        questions.value.push(questionDataB)
    }
    displayQuestions.value.push(questions.value[0]);
    // questions.value = [questions.value[0]];

})

</script>

<template>
    <div>
        <QuestionRow @question-correct="incrCorrect" @question-completed="addNextQuestion" :time="3"
            v-for="q in displayQuestions" :question="q" />
        <h3>{{ numCorrect }} / {{ numQuestions }}</h3>
    </div>
</template>

<style scoped>
button {
    margin: 1em;
}
</style>
