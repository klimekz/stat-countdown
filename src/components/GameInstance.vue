<script setup lang="ts">
import { onMounted, ref, Ref } from "vue";
import QuestionRow from "./QuestionRow.vue";
import DailyStats from "./DailyStats.vue";
import { getFirestore, collection, getDocs, doc, setDoc } from "firebase/firestore";
import { initializeApp } from "firebase/app";
import JSConfetti from 'js-confetti';
import { firebaseConfig } from "./FirebaseConfig.ts";
import TimesGraph from "./TimesGraph.vue";
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
const answerTimes: Ref<number[]> = ref([])
const questions: Ref<QData[]> = ref([]);
const startTime: Ref<Date> = ref(new Date());
const endTime: Ref<Date> = ref(new Date());
const instanceTime: Ref<number | undefined> = ref(undefined);
let numQuestions = 1;
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
            const confetti = new JSConfetti()
            const emojis = new JSConfetti()
            confetti.addConfetti({ confettiColors: ['#17408b', '#c9082a', '#ffffff', '#fdb927', '#007A33'], })
            emojis.addConfetti({
                emojis: ['🏀', '🏆'],
                confettiNumber: 23
            });
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
    returnStr = returnStr + a
}


function emitRestartGame() {
    emit('restart-game', null);
}

function emitCompletedGame() {
    emit('count-completed', null);
}

function incrCorrect(d: number) {
    if (numIncorrect.value === 0) {
        numCorrectStreak.value++;
        streakStr = streakStr + "✅"
    }
    answerTimes.value.push(d);
    numCorrect.value++;
    appendResult('🟩')

}

function incrIncorrect(d: number) {
    answerTimes.value.push(d)
    numIncorrect.value++;
    appendResult('🟥')
}

function qMissed(d: number) {
    answerTimes.value.push(d);
    numIncorrect.value++;
    appendResult('🟧')
}

function getCorrectClass() {
    return numCorrect.value === 8 ? "eight" : "resultText";
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
    <div class="col">
        <div class="row title">
            <h2>Daily Challenge:
            </h2>
            <h3 class="date">{{ new Date().toLocaleDateString('en-us') }}</h3>
        </div>
        <h3 v-if="!loaded">Loading. . . </h3>
        <QuestionRow v-if="loaded" @question-missed="(d) => qMissed(d)" @question-correct="(d) =>
            incrCorrect(d)" @question-incorrect="(d) => incrIncorrect(d)" @question-completed="addNextQuestion"
            :time="timerSeconds" v-for="q in displayQuestions" :question="q" />
        <h3>
            <span class="resultText" v-if="gameComplete"><span :class="getCorrectClass()">{{ numCorrect }}</span> / {{
                numQuestions }}</span>
        </h3>
        <h3 class="resultText perfectText" v-if="numCorrect === 10 && numQuestions === 10">Perfect Game!</h3>
        <h3 class="resultText" v-if="gameComplete">You answered {{ numCorrectStreak }} {{
            numCorrectStreak === 1 ?
            "question" :
            "questions"
        }}
            correctly before making a mistake.</h3>
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
            <br />
            <a href="https://www.statcountdown.com">statcountdown.com</a>
        </div>
        <br /><br />
        <TimesGraph v-if="gameComplete" :answers="answerTimes" />
        <button v-if="gameComplete" @click="emitRestartGame">Restart</button>
        <div class="stats">
            <DailyStats v-if="gameComplete" />
        </div>
    </div>
</template>

<style scoped>
.line {
    width: 0%;
    height: 60%;
}

.eight {
    margin-top: 0;
    padding-top: 0;
    max-width: 75%;
    align-self: center;
    color: #552583;
}

.col {
    display: flex;
    flex-direction: column;
    justify-content: center;

}

.results {
    display: flex;
    align-items: center;
    padding: 3em;
    justify-content: space-around;
}

button {
    max-width: 10em;
    align-self: center;
}

.resultCard {
    padding-right: 1.5em;
    padding-left: 1.5em;
    padding-top: 1em;
    padding-bottom: 1em;
    border-radius: 6px;
    display: inline-block;
    background-color: white;
    text-align: left;
    align-self: center;
    max-width: 70%;
    box-shadow: 2px 4px 4px hsl(0deg 0% 0% / 0.33);
}

.resultCard:hover {
    box-shadow: 2.1px 4.2px 4.2px hsl(0deg 0% 0% / 0.3);
    transform: scale(1.01);
}

.cardElement {
    margin: 0;
    padding: .2em;
}

.date {
    margin-top: auto;
    margin-bottom: auto;
}

.resultText {
    margin-top: 0;
    padding-top: 0;
    max-width: 75%;
    align-self: center;
}

.title {
    justify-content: space-evenly;
}

.stats {
    max-width: 85%;
    align-self: center;

}

@media (min-width: 850px) {

    .stats {
        max-width: 100%;
    }
}


button {
    margin: 1em;
    width: auto;
}

h2 {
    padding-bottom: 0em;
}
</style>
