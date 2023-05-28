<template>
  <div id="app">
    <h1 class="title">Accuenergy </h1>
    <h2 class="title">Pre-Interview Coding Test</h2>
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
  padding: 20px 0;
}

.title{
  color: #0c947c;
}
</style>
