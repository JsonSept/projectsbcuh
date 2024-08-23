<template>
    <div>
        <canvas id="powerOutputChart"></canvas>
    </div>
</template>

<script>
import { Chart } from 'chart.js/auto';

export default {
    name: 'PowerOutputChart',
    props: {
        powerGenerated: {
            type: Number,
            required: true,
        },
        dailyConsumption: {
            type: Number,
            required: true,
        },
        savingsInRand: {
            type: Number,
            required: false,
        },
    },
    mounted() {
        this.renderChart();
    },
    methods: {
        renderChart() {
            new Chart(document.getElementById('powerOutputChart'), {
                type: 'bar', // or 'line', 'pie', etc.
                data: {
                    labels: ['Generated Power (kWh/day)', 'Daily Consumption (kWh/day)', 'Potential Savings (R)'],
                    datasets: [
                        {
                            label: 'Values',
                            data: [
                                this.powerGenerated.toFixed(2),
                                this.dailyConsumption.toFixed(2),
                                this.savingsInRand ? this.savingsInRand.toFixed(2) : 0,
                            ],
                            backgroundColor: ['#ff6384', '#36a2eb', '#cc65fe'],
                        },
                    ],
                },
                options: {
                    responsive: true,
                    scales: {
                        y: {
                            beginAtZero: true,
                        },
                    },
                },
            });
        },
    },
};
</script>

<style scoped>
#powerOutputChart {
    max-width: 600px;
    margin: auto;
}
</style>
