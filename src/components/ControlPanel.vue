<script setup>
import { computed, inject } from 'vue';
import StepUnit from './StepUnit.vue'
import EditButton from './buttons/EditButton.vue';
import RunButton from './buttons/RunButton.vue';
import PrevButton from './buttons/PrevButton.vue';
import NextButton from './buttons/NextButton.vue';
import CommandButton from './buttons/CommandButton.vue';
import NextBreakpoint from './buttons/NextBreakpoint.vue';
import PrevBreakpoint from './buttons/PrevBreakpoint.vue';

const { total_steps, edit_mode, stdin_input } = inject('app')

const css_grid_template_columns = computed(() => {
  return `repeat(${total_steps.value}, 1fr)`;
})
const command_available = computed(()=>{
  return stdin_input.value.startsWith("/")
})


</script>

<template>
  <div class="container">
    <div class="process-bar" v-show="!edit_mode">
      <StepUnit v-for="(item, index) in total_steps" :step="index + 1" />
    </div>
    <EditButton v-show="!edit_mode" />
    <RunButton v-show="edit_mode" />
    <PrevButton v-show="!edit_mode" />
    <NextButton v-show="!edit_mode" />
    <CommandButton v-show="command_available && edit_mode" />
    <NextBreakpoint v-show="!edit_mode" />
    <PrevBreakpoint v-show="!edit_mode" />
  </div>
</template>

<style scoped>
.container {
  height: 100%;
}

.process-bar {
  display: grid;
  grid-template-rows: 0.5rem;
  grid-template-columns: v-bind(css_grid_template_columns);
  border-radius: 100rem;
  /* 足够大 */
  overflow: hidden;
  margin-bottom: 0.8rem;
}
</style>