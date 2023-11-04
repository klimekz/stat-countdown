<script setup >
import { Line } from "vue-chartjs";
import {
    Chart as ChartJS,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend
} from 'chart.js'

ChartJS.register(
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend
)

const options = {
    responsive: true,
    backgroundColor: '#c9082a',
    type: 'line',
    plugins: {
        legend: {
            display: false,
        },
    }
};
const { answers } = defineProps(['answers'])

</script>
<template>
    <Line id="result-times" :options="options" :data="{
        labels: answers.map((_, index) => (index + 1).toString()),
        datasets: [{
            data: answers.map((t, index) => ({
                x: index,
                y: t,
            })),
        }],
    }" class="lineGraph center"></Line>
    <h5 class="center">Response Times</h5>
    <br />
</template>
<style>
.lineGraph {
    max-width: 65%;
    max-height: 20vh;
}

.center {
    align-self: center;
    margin-top: .5em;
    margin-bottom: 0;
}
</style>