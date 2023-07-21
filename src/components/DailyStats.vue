<script setup lang="ts">
import { Ref, onMounted, ref } from "vue";
import { getFirestore, collection, getDocs } from "firebase/firestore";
import { initializeApp } from "firebase/app";
import { Bar } from "vue-chartjs"
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'
ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)


const firebaseConfig = {
    apiKey: "AIzaSyBg4jyjrEXh02M3iFpbsXGx9hsta-3Mfd0",
    authDomain: "vue-nba-guess.firebaseapp.com",
    projectId: "vue-nba-guess",
    storageBucket: "vue-nba-guess.appspot.com",
    messagingSenderId: "608907352543",
    appId: "1:608907352543:web:36f7fc4d78d70bcf7acc6a"
};
const results: Ref<any> = ref([]);
const loaded: Ref<boolean> = ref(false);
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

function getDate() {
    const d: Date = new Date()
    const month: string = d.getMonth() < 9 ? "0" + (d.getMonth() + 1).toString() : (d.getMonth() + 1).toString();
    const date: string = d.getDate() < 10 ? "0" + d.getDate() : d.getDate().toString();
    return d.getFullYear() + "-" + month + "-" + date;
}

let numCorrectValues: any = {};
let numStraightValue: any = {};


async function getResults() {
    const querySnapshot = await getDocs(collection(db, "daily-challenge", getDate(), "results"));
    querySnapshot.forEach((doc) => {
        const result = {
            numCorrect: doc.data().numCorrectScore,
            numStraight: doc.data().streakCorrectScore
        }

        if (numCorrectValues[result.numCorrect]) {
            numCorrectValues[result.numCorrect]++
        } else {
            numCorrectValues[result.numCorrect] = 1
        }

        if (numStraightValue[result.numStraight]) {
            numStraightValue[result.numStraight]++
        } else {
            numStraightValue[result.numStraight] = 1
        }

        results.value.push(result)
    });
}

onMounted(() => {
    getResults().then(() => {
        loaded.value = true;
    })
})
</script>

<template>
    <div class="statCol">
        <h2>Today's Stats</h2>
        <h4>Number of Questions Guessed Correctly</h4>
        <Bar v-if="loaded" id="correct-graph" :options="{
            responsive: true,
            backgroundColor: '#17408b',
            plugins: {
                legend: {
                    display: false,
                    position: 'bottom'
                }
            }
        }" :data="{
    labels: Object.keys(numCorrectValues),
    datasets: [{
        data: Object.values(numCorrectValues),
    }]
}" class="media graph" />
        <br />
        <h4>Guesses Until an Incorrect Answer</h4>
        <Bar v-if="loaded" id="streak-graph" :options="{
            responsive: true,
            backgroundColor: '#c9082a',
            plugins: {
                legend: {
                    display: false,
                    position: 'bottom'
                }
            }
        }" :data="{
    labels: Object.keys(numStraightValue),
    datasets: [{
        data: Object.values(numStraightValue),
    }]
}" class="media graph" />
    </div>
</template>

<style scoped>
.statCol {
    margin-top: 3em;
    display: flex;
    flex-direction: column;
    align-items: center;
    /* max-width: 66%; */
}

.graph {
    max-width: 60%;
    max-height: 60%;
}
</style>