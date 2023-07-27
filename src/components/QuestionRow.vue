<script setup lang="ts">
import { Ref, onMounted, ref } from "vue";
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
type PropType = {
    time: number,
    question: QData
}
const emit = defineEmits();
const clicked: Ref<boolean> = ref(false);
const guess: Ref<string> = ref("");
const props = defineProps<PropType>();
const seconds: Ref<number> = ref(props.time);
var timerId = 0;

const emitQuestionCompleted = () => {
    emit('question-completed', "Payload")
}

const emitQuestionCorrect = () => {
    emit('question-correct', "Payload");
}

const emitQuestionIncorrect = () => {
    emit('question-incorrect', "Payload");
}

const getPlayerPath = (playerId: string): string => {
    return "https://ak-static.cms.nba.com/wp-content/uploads/headshots/nba/latest/260x190/" +
        playerId + ".png";
}

const onCardClick = (name: string) => {
    if (!clicked.value && seconds.value > 0) {
        guess.value = name;
        clearInterval(timerId);
        clicked.value = !clicked.value;
        if (guess.value === props.question.answer)
            emitQuestionCorrect();
        else
            emitQuestionIncorrect();
        emitQuestionCompleted();
    }
}

const getCardStyle = (name: string, answer: string) => {
    if (seconds.value === 0)
        return "info clicked expired"
    if (!clicked.value) {
        return "info";
    } else {
        if (guess.value === answer) {
            return name == guess.value ? "info clickedCorrect" : "info clicked";
        }
        else {
            return name == guess.value ? "info clickedIncorrect" : "info clicked";
        }

    }
}

const updateTime = () => {
    seconds.value--;
    if (seconds.value === 0) {
        seconds.value = 0;
        clearInterval(timerId);
        emitQuestionCompleted();
    }
}


const runTimer = () => {
    timerId = setInterval(updateTime, 1000);
}

const getTimerStyle = (time: number) => {
    return time > 2 ? "rowText timer" : time == 2 ? "rowText timer timerClose" : time == 1 ? "rowText timer timerCloser" : "rowText timer timerExpired"
}

onMounted(() => {
    runTimer();
})
</script>

<template>
    <div class="row qRow">
        <div class="qCol">
            <h2 class="rowText">{{ props.question.statCategory }}</h2>
            <h3 class="yearText">{{ props.question.season }}</h3>
        </div>
        <div class="playerCard" @click="onCardClick(props.question.playerAName)">
            <img class="playerHeadshot" :src="getPlayerPath(props.question.playerAId)" />
            <div :class="getCardStyle(props.question.playerAName, props.question.answer)">
                <div class="nameDiv">
                    <h4 class="disableSelect nameText">{{ props.question.playerAName.split(" ")[0] }} </h4>
                    <h4 class="disableSelect nameText">{{ props.question.playerAName.split(" ")[1] }} </h4>
                </div>
                <div class="row stat">
                    <p v-show="clicked || seconds === 0" class="disableSelect">{{ props.question.playerAStat.toFixed(1) }}
                    </p>
                </div>
            </div>
        </div>
        <p class="rowText">or</p>
        <div class="playerCard" @click="onCardClick(props.question.playerBName)">
            <img class="playerHeadshot" :src="getPlayerPath(props.question.playerBId)" />
            <div :class="getCardStyle(props.question.playerBName, props.question.answer)">
                <div class="nameDiv">
                    <h4 class="disableSelect nameText">{{ props.question.playerBName.split(" ")[0] }} </h4>
                    <h4 class="disableSelect nameText">{{ props.question.playerBName.split(" ")[1] }} </h4>
                </div>
                <div class="row stat">
                    <p v-show="clicked || seconds == 0" class="disableSelect">{{ props.question.playerBStat.toFixed(1) }}
                    </p>
                </div>
            </div>
        </div>
        <div :class="getTimerStyle(seconds)">
            <h4 class="timerText disableSelect" @click="runTimer">0{{ seconds }}</h4>
        </div>
    </div>
</template>

<style scoped>
.nameDiv {}

.info {
    border-top: solid 6px black;
    padding-top: .5em;
    padding-bottom: .5em;

}

.row {
    display: flex;
    flex-direction: row;
    justify-content: center;
}

.stat {
    justify-content: right;
}


.qRow {
    margin-top: 3em;
    margin-bottom: 3em;
}

.nameText {
    margin: 0;
    padding-right: .5em;
    padding-left: .5em;
}

.qCol {
    display: flex;
    flex-direction: column;
    margin-right: 2em;
    padding-right: 1em;
}

.rowText {
    margin-top: auto;
    margin-bottom: auto;
}

.playerCard {
    outline: solid 9px black;
    margin-left: 1.66em;
    margin-right: 1.66em;
    background-color: white;
    min-width: 23%;
    border-radius: 1px;
}

.clicked {
    background-color: rgba(54, 69, 79, 0.364);
}

.expired {
    background-color: rgba(128, 0, 0, 0.3);
}

.clickedCorrect {
    background-color: rgba(0, 128, 0, 0.3);
}

.clickedIncorrect {
    background-color: rgba(128, 0, 0, 0.3);
}

.playerCard:hover {
    transform: scale(1.02);
}

.clicked:hover,
.clickedIncorrect:hover,
.clickedCorrect:hover {
    transform: scale(1.0);
}

p {
    margin: 0;
    padding: 0;

}

.disableSelect {
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    user-select: none;
    padding: 0;
    padding-right: .5em;
}

.playerHeadshot {
    max-width: 100%;
    max-height: 100%;
}

.yearText {
    margin-bottom: auto;
}

.timer {
    outline: solid 1px black;
    padding: .2em;
    min-width: 2.5em;
    min-height: 2.5em;
    max-width: 2.5em;
    max-height: 2.5em;
    background-color: black;
    color: white;
    margin-left: 1em;
    display: flex;
    flex-direction: column;
    justify-content: center;

}

.timerExpired {
    /* outline: solid 3px rgba(128, 0, 0, 0.5); */
    background-color: rgba(128, 0, 0, 0.3);
    outline: solid 3px black;
    color: black;
}

.timerClose {
    color: white;
    outline: solid 3px rgba(255, 150, 0, .7);
}


.timerCloser {
    outline: solid 3px rgba(128, 0, 0, 0.5);
    color: white;
}

p {
    padding-right: .4em;
    padding-top: .4em;
    padding-bottom: 0;
}

img {
    margin-top: .5em;
}
</style>
