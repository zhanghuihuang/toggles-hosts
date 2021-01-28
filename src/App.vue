<template>
  <div id="app">
    <ToggleHost/>
  </div>
</template>

<script>
import ToggleHost from "views/index.vue";

export default {
  name: "App",
  components: {
    ToggleHost
  }
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 5px;
}
</style>
