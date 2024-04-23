<script setup>

import { computed, inject } from 'vue';

const { stdin_input, feature_show_new_vars } = inject('app')

const command = computed(()=>{
  if(stdin_input.value.length <= 1){
    return ''
  }else{
    return stdin_input.value.slice(1)
  }
})

function click(){
  const cmd = command.value
  console.log(cmd)
  if(cmd == '?'){
    stdin_input.value = 'available commands:\n/show_new_vars'
  }
  else if(cmd == 'show_new_vars'){
    feature_show_new_vars.value = !feature_show_new_vars.value
    stdin_input.value = 'feature_show_new_vars: ' + feature_show_new_vars.value
  }
  else{
    stdin_input.value = 'unknown command\nuse /?'
  }
}

</script>

<template>
  <button class="button" @click="click">执行</button>
</template>

<style scoped>
@import '@/assets/button.css';

</style>