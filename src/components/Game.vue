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
const questions: Ref<QData[]> = ref([questionData]);

function addNextQuestion() {
    if (numQuestions < 7) {
        questions.value.push(questionDataB);
        numQuestions++;

    }
}

onMounted(() => {
    questions.value = [questionData];
})

</script>

<template>
    <div>
        <QuestionRow @question-completed="addNextQuestion" :time="6" v-for="i in questions" :question="i" />
        <button @click="addNextQuestion">Add question (testing)</button>
    </div>
</template>

<style scoped></style>
