<template>
  <div>
    <navbar-comp />
    <router-view />
    <h1>Solar Energy Output Visualization</h1>
    <SolarChart :solarOutput="solarOutput" :energyConsumption="energyConsumption" :surplusEnergy="surplusEnergy" />
  </div>
</template>

<script>
import NavbarComp from './components/NavbarComp.vue'
import SolarChart from './components/SolarChart.vue';

export default {
  name: 'App',
  components: {
    SolarChart,
    NavbarComp,
  },
  data() {
    return {
      solarOutput: 1200, // Replace with actual data
      energyConsumption: 800, // Replace with actual data
      surplusEnergy: 400, // Replace with actual data
    };
  },
};
</script>

<style>
/* Add any necessary styling here */
</style>



