<script setup>
import { inject } from 'vue';


const { edit_mode, try_failed, data } = inject('app')
function click() {
  edit_mode.value = true
  try_failed.value = false
  data.value = {}
}

</script>

<template>
  <button class="button" @click="click">修改代码</button>
</template>

<style scoped>
@import '@/assets/button.css';
</style>