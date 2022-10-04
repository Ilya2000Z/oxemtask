<template>
  <div>
    <CalculatorAvto></CalculatorAvto>
  </div>
</template>

<script>
import CalculatorAvto from './components/CalculatorAvto.vue';

export default {
  name: 'App',
  components: {
    CalculatorAvto
}
}
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
}
  
</style>
