<script setup>
import { inject } from 'vue';

const { current_step, total_steps, click_next } = inject('app')

</script>

<template>
  <button class="button" @click="click_next">下一步</button>
</template>

<style scoped>
@import '@/assets/button.css';
</style>