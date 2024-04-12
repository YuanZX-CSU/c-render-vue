<script setup>
import { computed, inject } from 'vue';
import Loading from './Loading.vue'


const { send_request, code_input, try_failed, loading } = inject('app')

const code_not_null = computed(() => {
  return code_input.value.length > 0
})

function click() {
  loading.value = true
  try_failed.value = false
  send_request()
}

</script>

<template>
  <button class="button" @click="click" :disabled="!code_not_null">运行代码<span v-show="loading">&nbsp</span>
    <Loading v-show="loading" />
  </button>
</template>

<style scoped>
@import '@/assets/button.css';

.button {
  display: flex;
  align-items: center;
  /* 垂直居中 */
}
</style>