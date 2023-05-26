<template>
  <div id="app">
    <h4>Accuenergy - Pre-Interview Code test</h4>
    <MyMap/>
  </div>
</template>

<script>
import MyMap from "./components/MyMap.vue";

export default {
  name: "App",
  components: {
    MyMap,
  },
};
</script>

<style>
#app {
  text-align: center;
  padding: 20px;
}
</style>
