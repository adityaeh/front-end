<template>
  <div id="app">
    <div class="text-5xl text-red-400">test</div>
    <div class="container-fluid max-w-[800px] mx-auto">
      <SimpleTable title="User Table"></SimpleTable>
    </div>
  </div>
</template>

<script>
import SimpleTable from "@/components/SimpleTable.vue";
export default {
  name: "App",
  components: {
    SimpleTable,
  },
};
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
