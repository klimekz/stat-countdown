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
const showA: Ref<boolean> = ref(false);
const showB: Ref<boolean> = ref(false);
let timerId: any = 0;
let startTime: Date | undefined = undefined;
let endTime: Date | undefined = undefined;
let diff: number = props.time + .2;


function emitQuestionCompleted() {
    emit('question-completed', null);
}

function emitQuestionCorrect(diff: number) {
    emit('question-correct', diff);
}

function emitQuestionIncorrect(diff: number) {
    emit('question-incorrect', diff);
}

function emitQuestionMissed(diff: number) {
    emit('question-missed', diff)
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
    let clickEnd = new Date()
    if (endTime === undefined) {
        endTime = clickEnd
        diff = ((endTime.getTime() - startTime!.getTime()) / 1000)
    }

    if (!clicked.value && seconds.value > 0) {
        guess.value = name;
        clearInterval(timerId);
        clicked.value = !clicked.value;
        if (guess.value === props.question.answer) {
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
            diff = ((endTime.getTime() - startTime!.getTime()) / 1000)
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
                <h2>{{ props.question.statCategory }}</h2>
                <div class="playerCard" @click="onCardClick(props.question.playerAName)">
                    <div class="imgContainer"><img class="playerHeadshot" :src="getPlayerPath(props.question.playerAId)"
                            @load="showA = true" />
                        <h5 class="teamTxt">{{ props.question.playerATeam }}</h5>
                    </div>
                    <div :class="getCardStyle(props.question.playerAName, props.question.answer)">
                        <div class="infoRow">
                            <div class="nameDiv">
                                <h4 class="disableSelect nameText">{{ props.question.playerAName.split(" ")[0] }} </h4>
                                <h4 class="disableSelect nameText">{{ props.question.playerAName.split(" ")[1] }} </h4>
                            </div>
                            <div class="nameDiv stat">
                                <p v-show="clicked || seconds == 0" class="disableSelect ">{{
                                    props.question.playerAStat.toFixed(1)
                                }}
                                </p>
                                <p v-show="clicked || seconds == 0" class="disableSelect"> {{
                                    props.question.statCategory.substring(0, 1) }}PG
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
                <h2>{{ props.question.season }} </h2>
                <div class="playerCard" @click="onCardClick(props.question.playerBName)">
                    <div class="imgContainer"><img class="playerHeadshot" :src="getPlayerPath(props.question.playerBId)"
                            @load="showB = true" />
                        <h5 class="teamTxt">{{ props.question.playerBTeam }}</h5>
                    </div>
                    <div :class="getCardStyle(props.question.playerBName, props.question.answer)">
                        <div class="infoRow">
                            <div class="nameDiv">
                                <h4 class="disableSelect nameText">{{ props.question.playerBName.split(" ")[0] }} </h4>
                                <h4 class="disableSelect nameText">{{ props.question.playerBName.split(" ")[1] }} </h4>
                            </div>
                            <div class="nameDiv stat">
                                <p v-show="clicked || seconds == 0" class="disableSelect ">{{
                                    props.question.playerBStat.toFixed(1)
                                }}
                                </p>
                                <p v-show="clicked || seconds == 0" class="disableSelect"> {{
                                    props.question.statCategory.substring(0, 1) }}PG
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
    transition: background-color 0.33s ease;
}

.clickedCorrect {
    background-color: rgba(2, 69, 32, .3);
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
    background-color: white;
    min-width: 23%;
    border-radius: 6px;
    box-shadow: 3px 6px 6px hsl(0deg 0% 0% / 0.25);
    margin-bottom: 4em;
}

.playerCard:hover {
    transform: scale(1.01);
    box-shadow: 4.5px 9px 9px hsl(0deg 0% 0% / 0.21);
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

@media((min-width: 375px) and (max-width: 415px)) {
    h4 {
        font-size: .8em;
    }

    p {
        font-size: .8em;
    }
}
</style>
