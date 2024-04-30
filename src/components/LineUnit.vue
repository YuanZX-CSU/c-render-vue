<script setup>
import { computed, inject, watch, getCurrentInstance } from 'vue';

const props = defineProps(['line', 'code'])

const { current_line, click_line, highlight_lines, code_view_split, prev_line, current_step } = inject('app')

const line = computed(() => { // 行号
  return props.line
})
const line_string = computed(()=>{
  const zero_count = String(code_view_split.value.length).length - String(line.value).length
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
  if (line.value == prev_line.value) {
    return '↗'
  }
  return ' '
})
const is_current = computed(() => { // 是当前行
  return line.value == current_line.value
})
const is_prev = computed(()=>{
  if(is_current.value){
    return false
  }
  return line.value == prev_line.value
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

watch(current_step, () => {
  if(is_current.value){
    current_instance.proxy.$el.scrollIntoView({
      behavior: 'smooth',
      inline: 'start',
      block: 'center'
    });
  }
})

const current_instance = getCurrentInstance()

</script>

<template>
  <div class="code" @click="click">
    <pre :class="{ blue: is_current, 'light-blue': is_prev }"> {{ mark }} </pre>
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

.light-blue {
  color: #c0e8ff;
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
  color: #ff2700;
}
</style>