<script setup>

import { inject } from 'vue';

const { current_step, click_prev } = inject('app')

</script>

<template>
  <button class="button" @click="click_prev">上一步</button>
</template>

<style scoped>
@import '@/assets/button.css';
</style>