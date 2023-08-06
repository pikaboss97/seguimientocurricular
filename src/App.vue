<template>
  <img alt="FIIS logo" src="https://fiis.unas.edu.pe/sites/all/themes/business_responsive_theme/logo.png">
  <Record :code=code />
</template>

<script>
import Record from './components/Record.vue'

export default {
  name: 'App',
  components: {
    Record
  },
  data(){
    return {
      code:""
    }

  },
  created() {
    const params = this.getURLParams();
    this.code = params.code;
  },
  methods: {
    getURLParams() {
      const url = window.location.href;
      const params = {};
      const queryString = url.split("?")[1];
      if (queryString) {
        const keyValuePairs = queryString.split("&");
        keyValuePairs.forEach(keyValuePair => {
          const [key, value] = keyValuePair.split("=");
          params[key] = decodeURIComponent(value);
        });
      }

      return params;
    },
  }

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
