<script setup>
import { inject } from 'vue';

const { current_step, highlight_steps } = inject('app')

function click() {
  let target
  let success = false
  let distance = Infinity
  for(let element of highlight_steps.value){
    let this_distance = current_step.value - element
    if(this_distance > 0 && this_distance < distance){
      distance = this_distance
      target = element
      success = true
    }
  }
  if(success){
    current_step.value = target
  }
}

</script>

<template>
  <button class="button" @click="click">上个断点</button>
</template>

<style scoped>
@import '@/assets/button.css';
</style>