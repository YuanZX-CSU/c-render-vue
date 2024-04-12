<script setup>

import { inject } from 'vue';

const { code_input } = inject('input')

const placeholder = '将你的代码粘贴到这里'

</script>

<template>
  <textarea class="code-input" v-model="code_input" :placeholder="placeholder"></textarea>
</template>

<style scoped>
@import '@/assets/code-input.css';

.code-input {
  white-space: nowrap;
}
</style>
