<script setup>
import { computed, inject } from 'vue';

const props = defineProps(['line', 'code'])

const { current_line, next_line, click_line, highlight_lines, feature_breakpoint_style } = inject('app')

const line = computed(() => { // 行号
  return props.line
})
const code = computed(() => { // 该行的代码
  return props.code
})
const mark = computed(() => { // 代码左侧的标记
  if (line.value == current_line.value) {
    return '→'
  }
  if (line.value == next_line.value) {
    return '→'
  }
  return ' '
})
const is_current = computed(() => { // 是当前行
  return line.value == current_line.value
})
const is_next = computed(() => { // 是下一行
  return line.value == next_line.value
})
const css_code_background = computed(() => { // 对应的style
  if(feature_breakpoint_style.value){
    return 'white'
  }
  if (Object.values(highlight_lines.value).includes(line.value)) {
    return '#FFDAD6'
  } else {
    return 'white'
  }
})
const breakpoint = computed(() => {
  // 
  if (Object.values(highlight_lines.value).includes(line.value)) {
    return ''
  } else {
    return ' '
  }
})
function click() {
  click_line(line.value)
}

</script>

<template>
  <div>
    <pre :class="{ blue: is_current, 'light-blue': is_next }">{{ mark }} </pre>
    <pre class="breakpoint" v-show="feature_breakpoint_style">{{ breakpoint }} </pre>
    <pre :class="{ blue: is_current }" class="code" @click="click">{{ code }}</pre>
  </div>
</template>

<style scoped>
pre {
  display: inline;
}

.blue {
  color: #00aeff;
}

.light-blue {
  color: #c0e8ff;
}

.code {
  border: 0rem;
  border-radius: 0.3rem;
  background-color: v-bind(css_code_background);
  transition-duration: 200ms;
}

.code:hover {
  /* background-color: #FFB4AB; */
  filter: brightness(0.9);
  transition-duration: 200ms;
}

.code:active {
  filter: brightness(0.8);
  transition-duration: 200ms;
}

.breakpoint{
  color: red;
}
</style>