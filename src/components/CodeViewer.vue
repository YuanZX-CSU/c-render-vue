<script setup>

import { computed, inject } from 'vue';
import LineUnit from './LineUnit.vue';

const { data } = inject('app');
const code = computed(() => {
  return data.value['code'];
})
const code_split = computed(() => {
  return String(code.value).split('\n');
})



</script>

<template>
  <div class="container">
    <LineUnit v-for="(item, index) in code_split" :code="item" :line="index + 1" />
  </div>
</template>

<style scoped>
.container {
  font-size: 0.8rem;
  background-color: white;
  overflow: auto;
  width: calc(100% - 0.8rem);
  height: calc(100% - 0.8rem);
  border: 0rem;
  border-radius: 0.3rem;
  /* padding: 0.4rem; */
}
</style>
