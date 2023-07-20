<script setup lang="ts">
import { Ref, onMounted, ref } from "vue";

type PropType = {
    time: number
}

const clicked: Ref<boolean> = ref(false);
const guess: Ref<string> = ref("");
const props = defineProps<PropType>();
const seconds: Ref<number> = ref(props.time);
var timerId = 0;


const gameData = {
    playerAId: "1628418",
    playerAName: "Thomas Bryant",
    playerAPts: 9.8,
    playerBId: "201565",
    playerBName: "Derrick Rose",
    playerBPts: 5.6,
    season: "22-23",
    answer: "Thomas Bryant",
    statCategory: "PTS"
}

const updateTime = () => {
    seconds.value--;
    if (seconds.value === 0) {
        seconds.value = 0;
        clearInterval(timerId);
    }
}

const runTimer = () => {
    timerId = setInterval(updateTime, 1000);
}

const getPlayerPath = (playerId: string): string => {
    return "https://ak-static.cms.nba.com/wp-content/uploads/headshots/nba/latest/260x190/" +
        playerId + ".png";
}

const onCardClick = (name: string) => {
    if (!clicked.value) {
        guess.value = name;
        clearInterval(timerId);
        clicked.value = !clicked.value;
    }
    // Add next question
}

const getCardStyle = (name: string, answer: string) => {
    if (seconds.value === 0)
        return "playerCard clicked"
    if (!clicked.value) {
        return "playerCard";
    } else {
        if (guess.value === answer) {
            return name == guess.value ? "playerCard clickedCorrect" : "playerCard";
        }
        else {
            return name == guess.value ? "playerCard clickedIncorrect" : "playerCard";
        }

    }
}

onMounted(() => {
    runTimer();
})
</script>

<template>
    <div class="row qRow">
        <div class="qCol">
            <h3 class="rowText">{{ gameData.statCategory }}</h3>
            <p class="yearText">{{ gameData.season }}</p>
        </div>
        <div :class="getCardStyle(gameData.playerAName, gameData.answer)" @click="onCardClick(gameData.playerAName)">
            <div class="row">
                <p v-show="clicked || seconds === 0">{{ gameData.playerAPts }}</p>
            </div>
            <img class="playerHeadshot" :src="getPlayerPath(gameData.playerAId)" />
            <h5>{{ gameData.playerAName }}</h5>

        </div>
        <p class="rowText">or</p>
        <div :class="getCardStyle(gameData.playerBName, gameData.answer)" @click="onCardClick(gameData.playerBName)">
            <div class="row">
                <p v-show="clicked || seconds == 0">{{ gameData.playerBPts }}</p>
            </div>
            <img class="playerHeadshot" :src="getPlayerPath(gameData.playerBId)" />
            <h5>{{ gameData.playerBName }}</h5>
        </div>
        <p @click="runTimer" class="rowText timer">{{ seconds }}</p>
    </div>
</template>

<style scoped>
.row {
    display: flex;
    flex-direction: row;
}

.qRow {
    margin-top: 1.6em;
    margin-bottom: 1.6em;
}

h5 {
    margin: 0;

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
    outline: solid 2px black;
    border-radius: 2px;
    padding-top: 1em;
    padding-right: 1em;
    padding-left: 1em;
    padding-bottom: .5em;
    margin-left: 1em;
    margin-right: 1em;
}

.clicked {
    background-color: rgba(54, 69, 79, 0.2)
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

.playerHeadshot {
    max-width: 100%;
    max-height: 100%;
}

.yearText {
    font-size: .8em;
    margin-bottom: auto;
}

.timer {
    outline: solid 1px black;
    border-radius: 2px;
    padding: .2em;
    margin-left: 1.5em;
    padding-right: .6em;
    padding-left: .6em;
}
</style>
