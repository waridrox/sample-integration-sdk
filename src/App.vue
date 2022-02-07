<template>
  <div class="bg">
    <Calculator/>
  </div>
</template>

<script>
import Calculator from './components/calculator/Calculator.vue';
export default {
  components: {
    Calculator
  }
}
</script>

<style >
  .bg {
    background-color: #3C1687;
    width: 100vw;
    height: 100vh;
    padding: 2rem;
  }
  html, body {
      font-family: 'Sora', sans-serif;
  }
</style>