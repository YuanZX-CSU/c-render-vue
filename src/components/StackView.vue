<script setup>
import { computed, inject, provide, watch, ref } from 'vue';
import VarUnit from './VarUnit.vue';


const { get_step, current_step } = inject('app')

const current_stack = computed(() => {
  return get_step(current_step.value + 1)
})

const current_var_map = computed(() => {
  let map = {}
  function parse_array(array) {
    if (array[0] == 'C_DATA') {
      map[array[1]] = array[3]
    } else {
      for (let item of array) {
        if (Array.isArray(item)) {
          parse_array(item)
        }
      }
    }
  }
  try {
    for (let item of current_stack.value.stack_to_render) {
      for (let [key, value] of Object.entries(item.encoded_locals)) {
        parse_array(value)
      }
    }
    for (let [key, value] of Object.entries(current_stack.value.globals)) {
      parse_array(value)
    }
  } catch (e) { }
  return map
})

const prev_stack = computed(() => {
  return get_step(current_step.value)
})

const prev_var_map = computed(() => {
  let map = {}
  function parse_array(array) {
    if (array[0] == 'C_DATA') {
      map[array[1]] = array[3]
    } else {
      for (let item of array) {
        if (Array.isArray(item)) {
          parse_array(item)
        }
      }
    }
  }
  try {
    for (let item of prev_stack.value.stack_to_render) {
      for (let [key, value] of Object.entries(item.encoded_locals)) {
        parse_array(value)
      }
    }
    for (let [key, value] of Object.entries(prev_stack.value.globals)) {
      parse_array(value)
    }
  } catch (e) { }
  return map
})

function check_changed(value) {
  let address = value[1]
  if (address in current_var_map.value) {
    if (address in prev_var_map.value) {
      if (current_var_map.value[address] !== prev_var_map.value[address]) {
        return 'changed'
      }
    } else {
      return 'new'
    }
  }
  return 'x'
}


const has_globals = computed(() => {
  try {
    return Object.keys(current_stack.value['globals']).length > 0
  } catch (e) { }
  return false
})

const stack_to_render = computed(() => {
  try {
    return current_stack.value['stack_to_render']
  } catch (e) { }
})

const stdout = computed(() => {
  try {
    return current_stack.value['stdout']
  } catch (e) { }
  return ''
})

const has_stdout = computed(() => {
  return stdout.value.length > 0
})

const highlight_addresses = ref({})
watch(current_step, () => {
  highlight_addresses.value = {}
})
provide('stack_view',{highlight_addresses,check_changed,current_var_map})


// function click() {
//   // console.log(Object.keys(current_stack.value['globals']).length)
//   // console.log(current_var_map.value)
//   // for(let [key,value] of current_stack.value.globals){
//   //     console.log(value)
//   //   }
//   // console.log(has_stdout.value)
//   console.log(highlight_addresses.value)
// }


</script>

<template>
  <div class="flex-container">
    <div class="func-box" v-show="has_stdout">
      <div class="big-box">
        <div class="small-box head-box">输出</div>
      </div>
      <div class="big-box">
        <div class="small-box">
          <pre>{{ stdout }}</pre>
        </div>
      </div>
    </div>
    <div class="func-box" v-if="has_globals">
      <div class="big-box">
        <div class="small-box head-box">全局变量</div>
      </div>
      <VarUnit v-for="(value2, key2, index2) in current_stack.globals" :var_name="key2" :var_content="value2"
        :changed="check_changed(value2)" />
    </div>
    <div class="func-box" v-for="value1 in stack_to_render">
      <div class="big-box">
        <div class="small-box head-box">{{ value1['func_name'] }}</div>
      </div>
      <VarUnit v-for="(value2, key2, index2) in value1['encoded_locals']" :var_name="key2" :var_content="value2"
        :changed="check_changed(value2)" />
    </div>
  </div>
  <!-- <button @click="click">log</button> -->
  <!-- <div>{{ stack_to_render }}</div> -->

</template>

<style scoped>
.func-box {
  font-size: 0.8rem;
  background-color: white;
  border-radius: 0.5rem;
  padding: 0.2rem;
  margin-right: 0.5rem;
  /* width: min-content;
  height: min-content; */
}

.flex-container {
  display: flex;
  flex-direction: column;
  /* grid-auto-flow: column;
  grid-template-columns: auto;
  grid-template-rows: auto; */
  /* grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); */
  height: 100%;
  width: calc(100% + 0.5rem);
  gap: 1rem;
  overflow: auto;
  border-radius: 0.3rem;
}
</style>