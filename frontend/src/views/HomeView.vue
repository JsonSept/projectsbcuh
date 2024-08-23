<template>
  <div class="home">
    <h1>Home Page</h1>
    <p>dbahkdvhbaldisbvaljsbdvljbsdljv</p>
    <div>{{ message }}</div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      message: '',
    };
  },
  mounted() {
    axios.get('/')
      .then(response => {
        this.message = response.data;
      })
      .catch(error => {
        console.error(error);
      });
  },
}
</script>
