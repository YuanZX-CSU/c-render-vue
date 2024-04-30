<script setup>

import { inject } from 'vue';
import LineUnit from './LineUnit.vue';

const { code_view_split } = inject('app');
 
</script>

<template>
  <div class="container">
    <LineUnit v-for="(item, index) in code_view_split" :code="item" :line="index + 1" />
  </div>
</template>

<style scoped>
.container {
  font-size: 0.8rem;
  background-color: white;
  overflow: auto;
  width: 100%;
  height: 100%;
  border: 0rem;
  border-radius: 0.3rem;
}
</style>
