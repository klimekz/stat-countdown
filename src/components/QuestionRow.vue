<script setup >
import { onMounted, ref } from "vue";

const { question, time } = defineProps(['question', 'time'])
const clicked = ref(false);
const emit = defineEmits();
const guess = ref("");
const seconds = ref(time);
const showA = ref(false);
const showB = ref(false);
let timerId = 0;
let startTime = undefined;
let endTime = undefined;
let diff = time + .2;


function emitQuestionCompleted() {
    emit('question-completed', null);
}

function emitQuestionCorrect(diff) {
    emit('question-correct', diff);
}

function emitQuestionIncorrect(diff) {
    emit('question-incorrect', diff);
}

function emitQuestionMissed(diff) {
    emit('question-missed', diff)
}

function getCardStyle(name, answer) {
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

function getPlayerPath(playerId) {
    return "https://ak-static.cms.nba.com/wp-content/uploads/headshots/nba/latest/260x190/" +
        playerId + ".png";
}

function getTimerStyle(time) {
    return time > 2 ? " timer" : time == 2 ? " timer timerClose" : time == 1 ? " timer timerCloser" : " timer timerExpired"
}

function onCardClick(name) {
    let clickEnd = new Date()
    if (endTime === undefined) {
        endTime = clickEnd
        diff = ((endTime.getTime() - startTime.getTime()) / 1000)
    }

    if (!clicked.value && seconds.value > 0) {
        guess.value = name;
        clearInterval(timerId);
        clicked.value = !clicked.value;
        if (guess.value === question.answer) {
            emitQuestionCorrect(diff);
        }
        else {
            emitQuestionIncorrect(diff);
        }
        emitQuestionCompleted();
    }
}

function runTimer() {
    while (!showA || !showB)
        continue

    startTime = new Date();
    timerId = setInterval(updateTime, 1000);
}

function updateTime() {
    seconds.value--;
    if (seconds.value === 0) {
        let clickEnd = new Date()
        if (endTime === undefined) {
            endTime = clickEnd
            diff = ((endTime.getTime() - startTime.getTime()) / 1000)
        }
        seconds.value = 0;
        clearInterval(timerId);
        emitQuestionCompleted();
        emitQuestionMissed(diff);
    }
}


onMounted(() => {
    runTimer();
})

</script>

<template>
    <div class="qCol">
        <div v-show="showA && showB" class="row">
            <div class="col">
                <h2>{{ question.statCategory }}</h2>
                <div class="playerCard" @click="onCardClick(question.playerAName)">
                    <div class="imgContainer"><img class="playerHeadshot" :src="getPlayerPath(question.playerAId)"
                            @load="showA = true" />
                        <h5 class="teamTxt">{{ question.playerATeam }}</h5>
                    </div>
                    <div :class="getCardStyle(question.playerAName, question.answer)">
                        <div class="infoRow">
                            <div class="nameDiv">
                                <h4 class="disableSelect nameText">{{ question.playerAName.split(" ")[0] }} </h4>
                                <h4 class="disableSelect nameText">{{ question.playerAName.split(" ")[1] }} </h4>
                            </div>
                            <div class="nameDiv stat">
                                <p v-show="clicked || seconds == 0" class="disableSelect ">{{
                                    question.playerAStat.toFixed(1)
                                }}
                                </p>
                                <p v-show="clicked || seconds == 0" class="disableSelect"> {{
                                    question.statCategory.substring(0, 1) }}PG
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
                <h2>{{ question.season }} </h2>
                <div class="playerCard" @click="onCardClick(question.playerBName)">
                    <div class="imgContainer"><img class="playerHeadshot" :src="getPlayerPath(question.playerBId)"
                            @load="showB = true" />
                        <h5 class="teamTxt">{{ question.playerBTeam }}</h5>
                    </div>
                    <div :class="getCardStyle(question.playerBName, question.answer)">
                        <div class="infoRow">
                            <div class="nameDiv">
                                <h4 class="disableSelect nameText">{{ question.playerBName.split(" ")[0] }} </h4>
                                <h4 class="disableSelect nameText">{{ question.playerBName.split(" ")[1] }} </h4>
                            </div>
                            <div class="nameDiv stat">
                                <p v-show="clicked || seconds == 0" class="disableSelect ">{{
                                    question.playerBStat.toFixed(1)
                                }}
                                </p>
                                <p v-show="clicked || seconds == 0" class="disableSelect"> {{
                                    question.statCategory.substring(0, 1) }}PG
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

.row {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}


.imgContainer {
    position: relative;
    text-align: center;
}

.clicked {
    background-color: rgba(54, 69, 79, 0.3);
    transition: background-color 0.33s ease;
}

.clickedCorrect {
    background-color: #0245204d;
    transition: background-color 0.167s ease;
}

.clickedIncorrect {
    background-color: rgba(181, 0, 22, .3);
    transition: background-color 0.167s ease;
}

.noClick {
    background-color: rgba(200, 132, 1, .3);
    transition: background-color 0.33s ease;
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

.eight {
    color: #552583
}

.info {
    border-top: solid 6px black;
    padding-top: .5em;
    padding-bottom: .5em;
    border-bottom-left-radius: 6px;
    border-bottom-right-radius: 6px;
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
    background-color: rgb(232, 232, 232);
    min-width: 23%;
    border-radius: 6px;
    box-shadow: 1px 2px 2px hsl(0deg 0% 0% / 0.18);
    margin-bottom: 4em;
}

.playerCard:hover {
    transform: scale(1.01);
    box-shadow: 2px 4px 4px hsl(0deg 0% 0% / 0.24);
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
    margin-right: .5em;
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
    border-radius: 3px;
}

.timerClose {
    color: white;
    outline: solid 3px rgba(255, 150, 0, .7);
    border-radius: 1.5px;
    transition: background-color 0.167s ease;
}

.timerCloser {
    outline: solid 3px rgba(128, 0, 0, 0.5);
    color: white;
    border-radius: 1.5px;
    transition: background-color 0.167s ease;
}

.timerExpired {
    background-color: rgba(181, 0, 22, .3);
    outline: solid 4px white;
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

@media (max-width:500px) {
    .timer {
        margin-left: .5em;
        margin-right: .5em;
    }
}

@media((min-width: 375px) and (max-width: 415px)) {
    h4 {
        font-size: .8em;
    }

    p {
        font-size: .8em;
    }
}
</style>
