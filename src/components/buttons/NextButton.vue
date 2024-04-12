<script setup>
import { inject } from 'vue';

const { current_step, total_steps } = inject('step')

function click() {
  if (current_step.value < total_steps.value) {
    current_step.value += 1
  }
}

</script>

<template>
  <button class="button" @click="click">下一步</button>
</template>

<style scoped>
@import '@/assets/button.css';
</style>