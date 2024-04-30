<script setup>
import { inject } from 'vue';

const { total_steps } = inject('app')

</script>

<template>
  <button class="button" disabled>步骤数超限!</button>
</template>

<style scoped>
@import '@/assets/button.css';

.button {
  color: red;
}

.button:hover {
  background-color: white;
}

.button:active {
  background-color: white;
}

button:disabled {
  opacity: 1;
}
</style>