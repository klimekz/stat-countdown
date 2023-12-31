<script setup>
import { Bar } from "vue-chartjs"
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'
import { firebaseConfig } from "./FirebaseConfig.js";
import { getFirestore, collection, getDocs } from "firebase/firestore";
import { initializeApp } from "firebase/app";
import { onMounted, ref } from "vue";
const app = initializeApp(firebaseConfig)
const db = getFirestore(app);
const loaded = ref(false);
const results = ref([]);
let numCorrectValues = {};
let numStraightValue = {};

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale);

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

function getDate() {
    const d = new Date()
    const month = d.getMonth() < 9 ? "0" + (d.getMonth() + 1).toString() : (d.getMonth() + 1).toString();
    const date = d.getDate() < 10 ? "0" + d.getDate() : d.getDate().toString();
    return d.getFullYear() + "-" + month + "-" + date;
}

onMounted(() => {
    getResults().then(() => {
        loaded.value = true;
    })
})
</script>

<template>
    <div class="statCol">
        <div class="colRow row">
            <h3>Daily Stats</h3>
            <p class="dateText">{{ new Date().toLocaleDateString("en-us") }}</p>
        </div>
        <br />
        <h4>Correct Answer Distribution</h4>
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
}" class=" graph" />
        <br />
        <h4>Guesses Until an Incorrect Answer Distribution</h4>
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
}" class=" graph" />
    </div>
</template>

<style scoped>
.graph {
    max-width: 75%;
    max-height: 55%;
}

.colRow {
    min-width: 100%;
}

.statCol {
    margin-top: 1.66em;
    display: flex;
    flex-direction: column;
    align-items: center;
    align-self: center;
}

.dateText {
    align-self: center;
}
</style>
