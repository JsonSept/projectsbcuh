<template>
    <div>
        <canvas id="solarChart"></canvas>
    </div>
</template>

<script>
import { Chart } from 'chart.js/auto';

export default {
    name: 'SolarChart',
    props: {
        solarOutput: {
            type: Number,
            required: true,
        },
        energyConsumption: {
            type: Number,
            required: true,
        },
        surplusEnergy: {
            type: Number,
            required: true,
        },
    },
    mounted() {
        this.renderChart();
    },
    methods: {
        renderChart() {
            new Chart(document.getElementById('solarChart'), {
                type: 'bar', // or 'line', 'pie', etc.
                data: {
                    labels: ['Total Output', 'Energy Consumption', 'Surplus Energy'],
                    datasets: [
                        {
                            label: 'kWh',
                            data: [
                                this.solarOutput,
                                this.energyConsumption,
                                this.surplusEnergy,
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
/* Add any necessary styling here */
#solarChart {
    max-width: 600px;
    margin: auto;
}
</style>
