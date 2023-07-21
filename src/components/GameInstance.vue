<script setup lang="ts">
import { onMounted, ref, Ref } from "vue";
import QuestionRow from "./QuestionRow.vue";
import { getFirestore, collection, getDocs } from "firebase/firestore";
import { initializeApp } from "firebase/app";
const loaded = ref(false)
const firebaseConfig = {
    apiKey: "AIzaSyBg4jyjrEXh02M3iFpbsXGx9hsta-3Mfd0",
    authDomain: "vue-nba-guess.firebaseapp.com",
    projectId: "vue-nba-guess",
    storageBucket: "vue-nba-guess.appspot.com",
    messagingSenderId: "608907352543",
    appId: "1:608907352543:web:36f7fc4d78d70bcf7acc6a"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app)

async function getQs() {
    const querySnapshot = await getDocs(collection(db, "daily-challenge", "2023-07-21", "questions"));
    querySnapshot.forEach((doc) => {
        console.log(doc.id, " => ", doc.data());
        const q: QData = {
            playerAId: doc.data().PlayerAId,
            playerAName: doc.data().PlayerAName,
            playerAStat: doc.data().PlayerAStatperGame,
            playerATeam: doc.data().PlayerATeam,
            playerBId: doc.data().PlayerBId,
            playerBName: doc.data().PlayerBName,
            playerBStat: doc.data().PlayerBStatperGame,
            playerBTeam: doc.data().PlayerBTeam,
            season: doc.data().SeasonStart,
            answer: doc.data().Answer,
            statCategory: doc.data().StatCategory,
            interval: doc.data().Interval
        }
        questions.value.push(q)
    });
}


type QData = {
    playerAId: string,
    playerAName: string,
    playerAStat: number,
    playerATeam: string,
    playerBId: string,
    playerBName: string,
    playerBStat: number,
    playerBTeam: string,
    season: string,
    answer: string,
    statCategory: string,
    interval: number
}

function getDate() {
    const d: Date = new Date()
    const month: string = d.getMonth() < 10 ? "0" + d.getMonth() : d.getMonth().toString();
    const date: string = d.getDate() < 10 ? "0" + d.getDate() : d.getDate().toString();
    return d.getFullYear() + "-" + month + "-" + date;
}

console.log(getDate())



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
    getQs().then(() => {
        questions.value.sort((a, b) => {
            return b.interval - a.interval;
        })
    }).then(() => {
        displayQuestions.value.push(questions.value[0]);
        loaded.value = true;
    });
})
</script>

<template>
    <div>
        <QuestionRow v-if="loaded" @question-correct="incrCorrect" @question-completed="addNextQuestion" :time="3"
            v-for="q in displayQuestions" :question="q" />
        <h3>{{ numCorrect }} / {{ numQuestions }}</h3>
    </div>
</template>

<style scoped>
button {
    margin: 1em;
}
</style>
