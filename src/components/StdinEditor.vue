<script setup>

import { computed, inject } from 'vue';

const { stdin_input } = inject('input')

const edit_mode = inject('edit_mode')
const readonly = computed(() => {
  return !(edit_mode.value)
})

const placeholder = '将你的测试用例粘贴到这里\n(如果有的话)'

</script>

<template>
  <textarea class="code-input" v-model="stdin_input" :placeholder="placeholder" :readonly="!edit_mode"></textarea>
</template>

<style scoped>
@import '@/assets/code-input.css';
</style>
