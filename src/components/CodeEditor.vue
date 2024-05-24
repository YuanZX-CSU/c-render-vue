<script setup>

import { inject } from 'vue';
import { Codemirror } from 'vue-codemirror';
import { cpp } from '@codemirror/lang-cpp';
import { EditorView } from '@codemirror/view';

const { code_input } = inject('app')

let custom_theme = EditorView.theme({
  "&": {
    "font-size": "0.8rem"
  },
  "&.cm-focused": {
    outline: "none"
  },
  ".cm-gutters": {
    backgroundColor: "#EEF4FA"
  }
}, { dark: false })

</script>

<template>
  <Codemirror v-model="code_input" placeholder="在此处输入你的代码" style="height: 100%;" :extensions="[cpp(), custom_theme]" :tab-size="4"/>
</template>

<style scoped></style>
