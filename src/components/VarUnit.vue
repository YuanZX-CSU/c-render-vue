<script setup>
import { computed, inject, ref, watch } from 'vue';

const {  current_step } = inject('app')
const { check_changed, highlight_addresses,current_var_map } = inject('stack_view')

const props = defineProps(['var_name', 'var_content', 'changed'])
const var_name = computed(() => {
  return props.var_name
})
const var_content = computed(() => {
  return props.var_content
})
const changed = computed(() => {
  return props.changed
})
const is_changed = computed(() => {
  return changed.value == 'changed'
})
const is_new = computed(() => {
  return changed.value == 'new'
})
const display_name = computed(() => {
  if (is_element.value) {
    return var_name.value.substring(1)
  }
  const spaceIndex = var_name.value.indexOf(' '); // 认为仅有'var (static xxx)'的额外情况
  if (spaceIndex == -1) {
    if (is_array.value) {
      return '@' + var_name.value
    }
    if (is_struct.value) {
      return '#' + var_name.value + '(' + var_content.value[2] + ')'
    }
    return var_name.value
  } else {
    return '$' + var_name.value.substring(0, spaceIndex)
  }
})
const is_data = computed(() => {
  return var_content.value[0] == 'C_DATA'
})
const is_array = computed(() => {
  return var_content.value[0] == 'C_ARRAY'
})
const is_struct = computed(() => {
  return var_content.value[0] == 'C_STRUCT'
})
const is_element = computed(() => {
  return var_name.value.charAt(0) == '^'
})
const address = computed(() => {
  return var_content.value[1]
})
const display_content = computed(() => {
  if (is_data.value) {
    let v = var_content.value[3]
    if(is_pointer.value){
      if(v in current_var_map.value){
        v = v + ' → ' + current_var_map.value[v]
        return v
      }
    }
    return v != '<UNINITIALIZED>' ? v : '?'
  }
  return 'error'
})
const array_elements = computed(() => {
  if (is_array.value) {
    let array = var_content.value
    array.slice(2)
    return array
  }
})
const struct_elements = computed(() => {
  if (is_struct.value) {
    let struct = var_content.value
    struct.slice(3)
    return struct
  }
})
const is_pointer = computed(() => {
  if (is_data.value) {
    if (var_content.value[2] == 'pointer') {
      return true
    }
  }
  return false
})



const pointer_color = ref(-1)
const css_pointer_color = computed(() => {
  if (pointer_color.value < 0) {
    return 'none'
  } else {
    let c = pointer_color.value % 4
    if (c == 0) {
      return 'underline #ff0000 0.1rem'
    }
    if (c == 1) {
      return 'underline #ffcc00 0.1rem'
    }
    if (c == 2) {
      return 'underline #0066ff 0.1rem'
    }
    if (c == 3) {
      return 'underline #00cc00 0.1rem'
    }
  }
})

const css_var_color = computed(() => {
  if (address.value in highlight_addresses.value) {
    let c = highlight_addresses.value[address.value]
    c = c % 4
    if (c == 0) {
      return 'underline #ff0000 0.1rem'
    }
    if (c == 1) {
      return 'underline #ffcc00 0.1rem'
    }
    if (c == 2) {
      return 'underline #0066ff 0.1rem'
    }
    if (c == 3) {
      return 'underline #00cc00 0.1rem'
    }
  }
  return 'none'
})

function highlight_pointer() {
  if (is_data.value) {
    if (var_content.value[2] == 'pointer') {
      let pa = var_content.value[3]
      if (pa in highlight_addresses.value) {
        highlight_addresses.value[pa] += 1
        pointer_color.value = highlight_addresses.value[pa]
      } else {
        highlight_addresses.value[pa] = 0
        pointer_color.value = 0
      }
    }
  }
}



watch(current_step, () => {
  pointer_color.value = -1
})

</script>

<template>
  <!-- <br> -->
  <div v-if="is_data" class="big-box" :class="{ 'element-box': is_element }">
    <div class="small-box var-name">{{ display_name }}<span v-if="!is_element"> = </span><span v-else>: </span><span
        :class="{ changed: is_changed, new: is_new, pointer: is_pointer }" @click="highlight_pointer">{{ display_content
        }}</span></div>
  </div>
  <div v-if="is_array" class="big-box">
    <div class="array-box">
      <div class="array-box-header">{{ display_name }}</div>
      <VarUnit v-for="(item, index) in array_elements" :var_name="'^' + (index - 2)" :var_content="item"
        :changed="check_changed(item)" />
    </div>
  </div>
  <div v-if="is_struct" class="big-box">
    <div class="struct-box">
      <div class="struct-box-header">{{ display_name }}</div>
      <VarUnit v-for="item in struct_elements" :var_name="item[0]" :var_content="item[1]"
        :changed="check_changed(item[1])" />
    </div>
  </div>
</template>

<style>
.big-box {
  /* margin: 0rem; */
  /* border-radius: 0.3rem; */
  /* background-color: green; */
  padding: 0.2rem;
  /* width: auto; */
  /* display: inline-block; */
}

.small-box {
  border-radius: 0.2rem;
  background-color: #EEF4FA;
  padding: 0.2rem;
  /* width: auto; */
  /* display: inline-block; */
}

.head-box {
  background-color: white;
  padding-top: 0rem;
  padding-bottom: 0rem;
}

.array-box,
.struct-box {
  border-radius: 0.2rem;
  border: 0.1rem solid #C0E8FF;
  padding: 0.2rem;
}

.struct-box-header,
.array-box-header {
  padding-left: 0.2rem;
}

.element-box {
  display: inline-block;
}
</style>

<style scoped>
.changed {
  color: #00AEFF;
}

.new {
  color: #85d8ff;
}

.pointer {
  background-color: #00000000;
  border-radius: 0.3rem;
  transition-duration: 200ms;
  text-decoration: v-bind(css_pointer_color);
}

.pointer:hover {
  background-color: #00000020;
  transition-duration: 200ms;
}

.var-name {
  text-decoration: v-bind(css_var_color);
}
</style>