<script setup>

import { computed, inject } from 'vue';

const { stdin_input, edit_mode, solution_output, get_step, current_step } = inject('app')

const stdin_placeholder = '你的测试用例\n(如果有的话)'
const stdout_placeholder = '程序的输出'
const display_solution_output = computed(()=>{
  return '答案:\n' + solution_output.value
})
const show_solution_output = computed(()=>{
  if(solution_output.value.length == 0){
    return false
  }else{
    return true
  }
})
const css_grid_template_columns = computed(()=>{
  if(show_solution_output.value){
    return '1fr 1fr 1fr'
  }else{
    return '1fr 1fr'
  }
})
const current_stack = computed(() => { // 当前要显示的帧（主要显示stack部分）
  return get_step(current_step.value)
})
const stdout = computed(() => { // 从开始到当前帧的累计输出
  try {
    return current_stack.value['stdout']
  } catch (e) { }
  return ''
})

</script>

<template>
  <div class="grid-container">
    <textarea class="code-input" v-model="stdin_input" :placeholder="stdin_placeholder" :readonly="!edit_mode"></textarea>
    <textarea class="code-input" v-model="stdout" :placeholder="stdout_placeholder" readonly></textarea>
    <textarea class="code-input" v-model="display_solution_output" v-if="show_solution_output" readonly></textarea>
  </div>
</template>

<style scoped>
@import '@/assets/code-input.css';

.grid-container{
  display: grid;
  grid-template-columns: v-bind(css_grid_template_columns);
  gap: 1rem;
  height: 100%;
}

</style>
