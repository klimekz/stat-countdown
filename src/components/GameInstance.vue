<script setup lang="ts">
import { onMounted, ref, Ref, computed } from "vue";
import QuestionRow from "./QuestionRow.vue";
import DailyStats from "./DailyStats.vue";
import { getFirestore, collection, getDocs, doc, setDoc } from "firebase/firestore";
import { initializeApp } from "firebase/app";
import { firebaseConfig } from "./FirebaseConfig.ts";

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const displayQuestions: Ref<QData[]> = ref([]);
const emit = defineEmits();
const gameComplete = ref(false);
const loaded: Ref<boolean> = ref(false);
const numCorrect: Ref<number> = ref(0);
const numCorrectStreak: Ref<number> = ref(0);
const numIncorrect: Ref<number> = ref(0);
const timerSeconds = ref(5);
const questions: Ref<QData[]> = ref([]);
let numQuestions = 1;
let result: string[][] = [];
const startTime: Ref<Date> = ref(new Date());
const endTime: Ref<Date> = ref(new Date());
const instanceTime: Ref<number | undefined> = ref(undefined);
type PropType = {
    numCompleted: number,
}
const props = defineProps<PropType>();

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

function addNextQuestion() {
    if (numQuestions == 10) {
        if (numCorrect.value === 10) {
            alert("Congrats, perfect game!")
        }
        endTime.value = new Date();
        instanceTime.value = (endTime.value.getTime() - startTime.value.getTime()) / 1000
        gameComplete.value = true;
        emitCompletedGame();
        if (props.numCompleted < 1) {
            setDoc(doc(db, "daily-challenge", getDate(), "results", new Date().toISOString()), {
                numCorrectScore: numCorrect.value,
                streakCorrectScore: numCorrectStreak.value
            })
        }
    }
    else if (numQuestions < 10) {
        displayQuestions.value.push(questions.value[numQuestions]);
        numQuestions++;
        if (numQuestions == 4) {
            timerSeconds.value = 4;
        }
        if (numQuestions == 7) {
            timerSeconds.value = 3;
        }
        if (numQuestions == 10) {
            timerSeconds.value = 2;
        }
    }
    console.log(result)

}

function getDate() {
    const d: Date = new Date()
    const month: string = d.getMonth() < 9 ? "0" + (d.getMonth() + 1).toString() : (d.getMonth() + 1).toString();
    const date: string = d.getDate() < 10 ? "0" + d.getDate() : d.getDate().toString();
    return d.getFullYear() + "-" + month + "-" + date;
}

let returnStr = ""
let streakStr = ""

function appendResult(a: string) {
    console.log(a, 'emojis')
    returnStr = returnStr + a
}

function emitRestartGame() {
    emit('restart-game', "Payload");
}

function emitCompletedGame() {
    emit('count-completed', "Payload");
}

function incrCorrect() {
    if (numIncorrect.value === 0) {
        numCorrectStreak.value++;
        streakStr = streakStr + "✅"
    }
    numCorrect.value++;
    appendResult('🟩')

}

function incrIncorrect() {
    numIncorrect.value++;
    appendResult('🟥')
}

function qMissed() {
    numIncorrect.value++;
    appendResult('🟧')
}

onMounted(() => {
    startTime.value = new Date();
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
        <h2 v-if="loaded">Daily Challenge</h2>
        <QuestionRow v-if="loaded" @question-missed="qMissed" @question-correct="incrCorrect"
            @question-incorrect="incrIncorrect" @question-completed="addNextQuestion" :time="timerSeconds"
            v-for="q in displayQuestions" :question="q" />
        <h3 class="resultText" v-if="gameComplete">{{ numCorrect }} / {{ numQuestions }}</h3>

        <h3 class="resultText" v-if="gameComplete">You answered {{ numCorrectStreak }} {{ numCorrectStreak === 1 ?
            "question" :
            "questions"
        }}
            correctly before making a mistake. (xth
            percentile)</h3>

        <br />
        <div class="resultCard" v-if="gameComplete">
            <p class="cardElement">My {{ new Date().toLocaleDateString("en-us") }} #StatCountdown 🏀</p>
            <p v-if="props.numCompleted <= 1" class="cardElement">{{ numCorrect }} / {{ numQuestions }} correct in {{
                instanceTime?.toFixed() }} seconds.
            </p>
            <p v-if="props.numCompleted > 1" class="cardElement">{{ numCorrect }} / {{ numQuestions }} correct in {{
                instanceTime?.toFixed() }} seconds (Attempt {{
        numCompleted }}).
            </p>
            <p class="cardElement">{{ returnStr }}<br />{{ streakStr }}</p>
        </div>
        <br /><br />
        <button v-if="gameComplete" @click="emitRestartGame">Restart</button>
        <br />
        <DailyStats v-if="gameComplete" />
    </div>
</template>

<style scoped>
.col {
    display: flex;
    flex-direction: column;
}

.resultText {
    /* text-align: left; */
    /* justify-content: flex-start; */
}

.results {
    display: flex;
    align-items: center;
    padding: 3em;
    justify-content: space-around;
}



.resultCard {
    padding-right: 1.5em;
    padding-left: 1.5em;
    padding-top: 1em;
    padding-bottom: 1em;
    outline: solid 4px black;
    border-radius: 1px;
    display: inline-block;
    background-color: white;
    text-align: left;
    max-width: 60%;
}

.cardElement {
    margin: 0;
    padding: .2em;
}


button {
    margin: 1em;
}
</style>
