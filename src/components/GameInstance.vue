<script setup lang="ts">
import { onMounted, ref, Ref } from "vue";
import QuestionRow from "./QuestionRow.vue";
import DailyStats from "./DailyStats.vue";
import { getFirestore, collection, getDocs, doc, setDoc } from "firebase/firestore";
import { initializeApp } from "firebase/app";
const loaded: Ref<boolean> = ref(false);

const firebaseConfig = {
    apiKey: "AIzaSyBg4jyjrEXh02M3iFpbsXGx9hsta-3Mfd0",
    authDomain: "vue-nba-guess.firebaseapp.com",
    projectId: "vue-nba-guess",
    storageBucket: "vue-nba-guess.appspot.com",
    messagingSenderId: "608907352543",
    appId: "1:608907352543:web:36f7fc4d78d70bcf7acc6a"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

async function getQs() {
    const querySnapshot = await getDocs(collection(db, "daily-challenge", getDate(), "questions"));
    querySnapshot.forEach((doc) => {
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
    const month: string = d.getMonth() < 9 ? "0" + (d.getMonth() + 1).toString() : (d.getMonth() + 1).toString();
    const date: string = d.getDate() < 10 ? "0" + d.getDate() : d.getDate().toString();
    return d.getFullYear() + "-" + month + "-" + date;
}

let numQuestions = 1;
const questions: Ref<QData[]> = ref([]);
const displayQuestions: Ref<QData[]> = ref([]);
const numCorrect: Ref<number> = ref(0);
const numIncorrect: Ref<number> = ref(0);
const numCorrectStreak: Ref<number> = ref(0);


function addNextQuestion() {
    if (numQuestions == 10) {
        gameComplete.value = true;
        setDoc(doc(db, "daily-challenge", getDate(), "results", new Date().toISOString()), {
            numCorrectScore: numCorrect.value,
            streakCorrectScore: numCorrectStreak.value
        })
    }
    else if (numQuestions < 10) {
        displayQuestions.value.push(questions.value[numQuestions]);
        numQuestions++;
    }

}

function incrCorrect() {
    if (numIncorrect.value === 0)
        numCorrectStreak.value++;
    numCorrect.value++;
}

function incrIncorrect() {
    numIncorrect.value++;
}

const gameComplete = ref(false);


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
        <h3 v-if="!loaded">Loading. . . </h3>
        <QuestionRow v-if="loaded" @question-correct="incrCorrect" @question-incorrect="incrIncorrect"
            @question-completed="addNextQuestion" :time="5" v-for="q in displayQuestions" :question="q" />
        <h3 v-if="gameComplete">{{ numCorrect }} / {{ numQuestions }}</h3>
        <h3 v-if="gameComplete">You answered {{ numCorrectStreak }} questions correctly before making a mistake. (xth
            percentile)</h3>
        <DailyStats v-if="gameComplete" />
    </div>
</template>

<style scoped>
button {
    margin: 1em;
}
</style>
