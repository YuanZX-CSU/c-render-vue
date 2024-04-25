<script setup>
import { computed, inject } from 'vue';

const { current_step, highlight_steps } = inject('app')

const clickable = computed(()=>{
  return highlight_steps.value.length == 0
})

function click() {
  let target
  let success = false
  let distance = Infinity
  for(let element of highlight_steps.value){
    let this_distance = element - current_step.value
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
  <button class="button" @click="click" :disabled="clickable">下个断点</button>
</template>

<style scoped>
@import '@/assets/button.css';
</style>