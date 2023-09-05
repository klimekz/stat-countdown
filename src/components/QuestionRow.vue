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

const clicked: Ref<boolean> = ref(false);
const emit = defineEmits();
const guess: Ref<string> = ref("");
const props = defineProps<PropType>();
const seconds: Ref<number> = ref(props.time);
let timerId: any = 0;

function emitQuestionCompleted() {
    emit('question-completed', "Payload");
}

function emitQuestionCorrect() {
    emit('question-correct', "Payload");
}

function emitQuestionIncorrect() {
    emit('question-incorrect', "Payload");
}

function emitQuestionMissed() {
    emit('question-missed', "Payload")
}

function getCardStyle(name: string, answer: string): string {
    if (seconds.value === 0)
        return "info clicked noClick"
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

function getPlayerPath(playerId: string): string {
    return "https://ak-static.cms.nba.com/wp-content/uploads/headshots/nba/latest/260x190/" +
        playerId + ".png";
}

function getTimerStyle(time: number): string {
    return time > 2 ? " timer" : time == 2 ? " timer timerClose" : time == 1 ? " timer timerCloser" : " timer timerExpired"
}

function onCardClick(name: string) {
    if (!clicked.value && seconds.value > 0) {
        guess.value = name;
        clearInterval(timerId);
        clicked.value = !clicked.value;
        if (guess.value === props.question.answer) {
            console.log('correct')
            emitQuestionCorrect();
        }
        else
            emitQuestionIncorrect();
        emitQuestionCompleted();
    }
}

function runTimer() {
    timerId = setInterval(updateTime, 1000);
}

function updateTime() {
    seconds.value--;
    if (seconds.value === 0) {
        seconds.value = 0;
        clearInterval(timerId);
        emitQuestionCompleted();
        emitQuestionMissed();
    }
}

onMounted(() => {
    runTimer();
})

</script>

<template>
    <div class="qCol">
        <div class="row">
            <div class="col">
                <h2>{{ props.question.statCategory }}</h2>
                <div class="playerCard" @click="onCardClick(props.question.playerAName)">
                    <div class="imgContainer"><img class="playerHeadshot" :src="getPlayerPath(props.question.playerAId)" />
                        <h5 class="teamTxt">{{ props.question.playerATeam }}</h5>
                    </div>
                    <div :class="getCardStyle(props.question.playerAName, props.question.answer)">
                        <div class="infoRow">
                            <div class="nameDiv">
                                <h4 class="disableSelect nameText">{{ props.question.playerAName.split(" ")[0] }} </h4>
                                <h4 class="disableSelect nameText">{{ props.question.playerAName.split(" ")[1] }} </h4>
                            </div>
                            <div class="row stat">
                                <p v-show="clicked || seconds === 0" class="disableSelect">{{
                                    props.question.playerAStat.toFixed(1)
                                }} {{ props.question.statCategory.substring(0, 1) }}PG
                                </p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div :class="getTimerStyle(seconds)">
                <h4 class="timerText disableSelect" @click="runTimer">0{{ seconds }}</h4>
            </div>
            <div class="col">
                <h2>{{ props.question.season }}</h2>
                <div class="playerCard" @click="onCardClick(props.question.playerBName)">
                    <div class="imgContainer"><img class="playerHeadshot" :src="getPlayerPath(props.question.playerBId)" />
                        <h5 class="teamTxt">{{ props.question.playerBTeam }}</h5>
                    </div>
                    <div :class="getCardStyle(props.question.playerBName, props.question.answer)">
                        <div class="infoRow">
                            <div class="nameDiv">
                                <h4 class="disableSelect nameText">{{ props.question.playerBName.split(" ")[0] }} </h4>
                                <h4 class="disableSelect nameText">{{ props.question.playerBName.split(" ")[1] }} </h4>
                            </div>
                            <div class="row stat">
                                <p v-show="clicked || seconds == 0" class="disableSelect">{{
                                    props.question.playerBStat.toFixed(1)
                                }} {{ props.question.statCategory.substring(0, 1) }}PG
                                </p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>

<style scoped>
.teamTxt {
    position: absolute;
    top: .66em;
    right: 1em;
}

.imgContainer {
    position: relative;
    text-align: center;
}

.clicked {
    background-color: rgba(54, 69, 79, 0.3);
}

.clickedCorrect {
    background-color: rgba(2, 69, 32, .3)
}

.clickedIncorrect {
    background-color: rgba(181, 0, 22, .3);
}

.noClick {
    background-color: rgba(200, 132, 1, .3);
}

.clicked:hover,
.clickedIncorrect:hover,
.clickedCorrect:hover {
    transform: scale(1.0);
}

.disableSelect {
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    user-select: none;
}

.expired {
    background-color: rgba(181, 0, 22, .3);
}

.headerRow {
    justify-content: space-around;
}

.info {
    border-top: solid 6px black;
    padding-top: .5em;
    padding-bottom: .5em;
    border-bottom-left-radius: 12px;
    border-bottom-right-radius: 12px;
}

.nameText {
    margin: 0;
    padding-right: .5em;
    padding-left: .5em;
}

.orText {
    margin-top: auto;
    margin-bottom: auto;
}

.playerCard {
    /* outline: solid 7px black; */
    margin-right: .5em;
    margin-left: .5em;
    background-color: white;
    min-width: 23%;
    border-radius: 12px;
    box-shadow: 6px 12px 12px hsl(0deg 0% 0% / 0.25);
    margin-bottom: 4em;
}

.playerCard:hover {
    transform: scale(1.02);
    box-shadow: 6px 12px 12px hsl(0deg 0% 0% / 0.21);
}

.playerHeadshot {
    max-width: 100%;
    max-height: 100%;
}

.qCol {
    display: flex;
    flex-direction: column;
    margin-right: .5em;
    margin-top: 1em;
    margin-bottom: 1em;
}

.infoRow {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    padding-left: 0.2em;
    padding-right: 0.2em;
}


.stat {
    justify-content: right;
    padding-right: .5em;
}

.timer {
    /* outline: solid 1px black; */
    padding: .2em;
    min-width: 2.5em;
    min-height: 2.5em;
    max-width: 2.5em;
    max-height: 2.5em;
    margin-left: 1.33em;
    margin-right: 1.33em;
    background-color: black;
    color: white;
    display: flex;
    flex-direction: column;
    justify-content: center;
    margin-top: auto;
    margin-bottom: auto;
}

.timerClose {
    color: white;
    outline: solid 3px rgba(255, 150, 0, .7);
}

.timerCloser {
    outline: solid 3px rgba(128, 0, 0, 0.5);
    color: white;
}

.timerExpired {
    background-color: rgba(181, 0, 22, .3);
    /* outline: solid 3px rgba(255, 150, 0, .7); */
    color: black;
}

.timerText {
    align-self: center;
}

h2 {
    margin-top: 0;
}

h5 {
    margin: 0;
    padding: 0;
}


img {
    margin-top: .5em;
}

p {
    margin: 0;
    padding: 0;
}
</style>
