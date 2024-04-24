<script setup>
import { computed, inject } from 'vue';

const props = defineProps(['line', 'code'])

const { current_line, click_line, highlight_lines, total_steps } = inject('app')

const line = computed(() => { // 行号
  return props.line
})
const line_string = computed(()=>{
  const zero_count = String(total_steps.value).length - String(line.value).length
  if(zero_count <= 0){
    return line.value
  }else{
    let padding = '';
    for (let i = 0; i < zero_count; i++) {
        padding += '0';
    }
    return padding + String(line.value)
  }
})
const code = computed(() => { // 该行的代码
  return props.code
})
const mark = computed(() => { // 代码左侧的标记
  if (line.value == current_line.value) {
    return '→'
  }
  return ' '
})
const is_current = computed(() => { // 是当前行
  return line.value == current_line.value
})
const breakpoint = computed(()=>{
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
  <div class="code" @click="click">
    <pre :class="{ blue: is_current }">{{ mark }} </pre>
    <pre class="breakpoint">{{ breakpoint }} </pre>
    <pre>{{ line_string }} </pre>
    <pre :class="{ blue: is_current }">{{ code }}</pre>
  </div>
</template>

<style scoped>
pre {
  display: inline;
}

.blue {
  color: #00aeff;
}

.code {
  border: 0rem;
  border-radius: 0.3rem;
  background-color: white;
  transition-duration: 200ms;
  width: fit-content;
}

.code:hover {
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