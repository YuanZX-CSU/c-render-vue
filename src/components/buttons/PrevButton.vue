<script setup>

import { inject } from 'vue';

const { current_step } = inject('step')

function click() {
  if (current_step.value > 1) {
    current_step.value -= 1
  }
}

</script>

<template>
  <button class="button" @click="click">上一步</button>
</template>

<style scoped>
@import '@/assets/button.css';
</style>