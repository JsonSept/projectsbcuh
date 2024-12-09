<template>
  <div>
    <div class="solar-calculator bg-dark text-light">
      <h1>Simple Solar Power Calculator</h1>

      <!-- Input for Location -->
      <h1>Solar Irradiance Calculator</h1>
      <form @submit.prevent="handleSubmit">
        <input type="text" v-model="address" placeholder="Enter your address or suburb" required />
        <button type="submit" class="btn">Get Results</button>
      </form>

      <div v-if="loading">
        <p>Loading...</p>
      </div>

      <div v-else-if="error">
        <p>Error: {{ error }}</p>
      </div>

      <div v-else-if="irradiance !== null">
        <p>Location: {{ locationName }}</p>
        <p>Solar Irradiance: {{ irradiance.toFixed(2) }} W/m²</p>
      </div>
      <!-- Simplified Panel Area Input -->
      <div class="form-group">
        <label for="panelArea">Solar Array Surface Area (m²)</label>
        <input type="number" id="panelArea" v-model="panelArea" placeholder="e.g., 10">
      </div>

      <!-- Simplified Panel Efficiency with Default -->
      <div class="form-group">
        <label for="efficiency">Panel Efficiency</label>
        <select id="efficiency" v-model="efficiency">
          <option value="15">Standard (15%)</option>
          <option value="18">High Efficiency (18%)</option>
          <option value="20">Premium (20%)</option>
        </select>
      </div>

      <!-- Button to Calculate -->
      <button class="btn" @click="calculatePower">Calculate Power Output</button>

      <!-- Display the Results -->
      <div v-if="powerGenerated !== null">
        <h2 class="out text-light">Estimated Power Output: {{ powerGenerated.toFixed(2) }} kWh/day</h2>
        <p class="out text-light">This is equivalent to {{ powerInWatts.toFixed(2) }} W/day</p>
        <p class="out text-light">(This is an estimate based on average sunlight hours for your location)</p>
      </div>

      <!-- Input for Daily Consumption -->
      <div class="form-group" v-if="powerGenerated !== null">
        <label for="consumption">Your Daily Consumption (kWh/day)</label>
        <input type="number" id="consumption" v-model="dailyConsumption" placeholder="e.g., 5">
      </div>

      <!-- Button to Compare with Consumption -->
      <button class="btn" v-if="powerGenerated !== null && dailyConsumption !== null"
        @click="compareConsumption">Compare
        Consumption</button>

      <!-- Display the Consumption Comparison and Currency Calculation -->
      <div v-if="consumptionResult !== null">
        <h3 class="out text-light">{{ consumptionResult }}</h3>
        <h4 v-if="savingsInRand !== null" class="out text-light">
          Potential Savings: R{{ savingsInRand.toFixed(2) }}
        </h4>
      </div>

      <!-- Chart for Visualizing the Results -->
      <PowerOutputChart v-if="powerGenerated !== null && dailyConsumption !== null" :powerGenerated="powerGenerated"
        :dailyConsumption="dailyConsumption" :savingsInRand="savingsInRand" />
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import PowerOutPutChart from '../components/PowerOutPutChart.vue';

const API_KEY = 'e945d7f71eb0e5e621a7dfcce2cb1a43';

export default {
  components: {
    PowerOutputChart,
  },
  data() {
    return {
      address: '',
      irradiance: null,
      locationName: '',
      loading: false,
      error: null,
      panelArea: 10, // Default value
      efficiency: 18, // Default value (in percentage)
      sunlightHours: 5, // Estimated average sunlight hours
      powerGenerated: null,
      powerInWatts: null,
      dailyConsumption: null,
      consumptionResult: null,
      savingsInRand: null,
      conversionRate: 1.33, // Conversion rate
      randPerKWh: 1.5, // Example Rand rate per kWh
    };
  },
  methods: {
    async handleSubmit() {
      if (!this.address) {
        this.error = 'Please enter an address or suburb.';
        return;
      }

      this.loading = true;
      this.error = null;
      this.irradiance = null;

      try {
        const geocodeData = await this.getCoordinates(this.address);
        if (geocodeData.length === 0) {
          throw new Error('Location not found.');
        }

        const { lat, lon, name, country, state } = geocodeData[0];
        this.locationName = `${name}${state ? ', ' + state : ''}, ${country}`;

        const weatherData = await this.getWeatherData(lat, lon);

        this.irradiance = this.calculateSolarIrradiance(weatherData);

        // Calculate the power output directly after fetching the solar irradiance
        this.calculatePower();
      } catch (err) {
        console.error(err);
        this.error = err.message || 'An error occurred.';
      } finally {
        this.loading = false;
      }
    },

    async getCoordinates(address) {
      const geocodeUrl = `https://api.openweathermap.org/geo/1.0/direct?q=${encodeURIComponent(
        address
      )}&limit=1&appid=${API_KEY}`;
      const response = await axios.get(geocodeUrl);
      return response.data;
    },

    async getWeatherData(lat, lon) {
      const weatherUrl = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}`;
      const response = await axios.get(weatherUrl);
      return response.data;
    },

    calculateSolarIrradiance(weatherData) {
      try {
        const cloudCover = weatherData.clouds.all / 100; // Convert percentage to fraction
        const solarConstant = 1361; // Solar constant in W/m²

        const currentTime = weatherData.dt; // Current time in Unix UTC
        const sunrise = weatherData.sys.sunrise; // Sunrise time in Unix UTC
        const sunset = weatherData.sys.sunset; // Sunset time in Unix UTC

        let solarElevationAngle;
        if (currentTime < sunrise || currentTime > sunset) {
          solarElevationAngle = -1; // Night time
        } else {
          const dayProgress = (currentTime - sunrise) / (sunset - sunrise);
          solarElevationAngle = 90 * Math.sin(dayProgress * Math.PI);
        }

        if (solarElevationAngle <= 0) {
          return 0;
        }

        const clearSkyIrradiance = solarConstant * Math.sin(Math.PI * solarElevationAngle / 180);
        const irradiance = clearSkyIrradiance * (1 - cloudCover);

        return irradiance;
      } catch (error) {
        console.error('Error calculating solar irradiance:', error);
        return null;
      }
    },

    calculatePower() {
      if (this.irradiance !== null) {
        const panelEfficiencyDecimal = this.efficiency / 100;
        // Use the actual solar irradiance value in the calculation
        this.powerGenerated = (this.panelArea * this.irradiance * panelEfficiencyDecimal * this.sunlightHours) / 1000;
        this.powerInWatts = this.powerGenerated * 1000;
      } else {
        this.error = 'Failed to calculate power output due to missing solar irradiance data.';
      }
    },

    compareConsumption() {
      if (this.dailyConsumption > this.powerGenerated) {
        this.consumptionResult = `Your daily consumption of ${this.dailyConsumption} kWh exceeds the generated power of ${this.powerGenerated.toFixed(2)} kWh/day.`;
      } else {
        this.consumptionResult = `Your generated power of ${this.powerGenerated.toFixed(2)} kWh/day is sufficient for your daily consumption of ${this.dailyConsumption} kWh.`;

        const differenceInKWh = this.powerGenerated - this.dailyConsumption;
        const savings = differenceInKWh * this.conversionRate;
        this.savingsInRand = savings * this.randPerKWh;
      }
    }
  }
}
</script>
