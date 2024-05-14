<script setup>
import { computed, inject, ref, watch, getCurrentInstance, nextTick } from 'vue';

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

const is_changed = computed(() => { // 相对于上一帧，发生了变化
  return changed.value == 'changed'
})
const is_new = computed(() => { // 相对于上一帧，是新增的
  return changed.value == 'new'
})
const display_name = computed(() => { // 最终显示的变量名称
  if (is_element.value) {
    let index = var_name.value.substring(1)
    const caretIndex = index.indexOf('^');
    if(caretIndex == -1){
      return index
    }else{
      const [partBeforeCaret, partAfterCaret] = index.split('^', 2);
      function getIndexInMultiDimArray(part1, part2Str) {
        const dimensions = part2Str.split(',').map(Number);
        const totalElements = dimensions.reduce((product, dim) => product * dim, 1);
        if (part1 < 0 || part1 >= totalElements) {
          return 'error'
        }
        const indices = [];
        for (let dim of dimensions.slice().reverse()) {
          indices.unshift(part1 % dim);
          part1 = Math.floor(part1 / dim);
        }
        return indices.join(',');
      }
      return getIndexInMultiDimArray(Number(partBeforeCaret),partAfterCaret)
    }
  }
  const spaceIndex = var_name.value.indexOf(' '); // 认为仅有'var (static xxx)'的额外情况
  if (spaceIndex == -1) {
    if (is_array.value) {
      return '@' + var_name.value
    }
    if (is_struct.value) {
      return '#' + var_name.value + '(' + var_content.value[2] + ')'
    }
    if(is_pointer.value){
      return '*' + var_name.value
    }
    return var_name.value
  } else {
    return '$' + var_name.value.substring(0, spaceIndex)
  }
})
const is_data = computed(() => { // 是普通数据
  return var_content.value[0] == 'C_DATA'
})
const is_array = computed(() => { // 是数组
  return var_content.value[0] == 'C_ARRAY'
})
const is_struct = computed(() => { // 是结构体
  return var_content.value[0] == 'C_STRUCT'
})
const is_multi_array = computed(()=>{
  return var_content.value[0] == 'C_MULTIDIMENSIONAL_ARRAY'
})
const is_element = computed(() => { // 是数组里的元素
  return var_name.value.charAt(0) == '^'
})
const address = computed(() => { // 该变量的地址
  return var_content.value[1]
})
const display_content = computed(() => { // 最终显示的变量的值
  if (is_data.value) {
    let v = var_content.value[3]
    if(is_pointer.value){
      if(v in current_var_map.value){
        v = v + ' → ' + current_var_map.value[v]
        return v
      }
    }
    return v != '<UNINITIALIZED>' && v != '<UNALLOCATED>' ? v : '?'
  }
  return 'error'
})
const array_elements = computed(() => { // 如果是数组，获取数组的子元素
  if (is_array.value) {
    let array = var_content.value
    array.slice(2)
    return array
  }
})
const multi_array_head = computed(() => { // 如果是数组，获取数组的子元素
  if (is_multi_array.value) {
    let array = var_content.value
    return array[2]
  }
})
const multi_array_elements = computed(() => { // 如果是数组，获取数组的子元素
  if (is_multi_array.value) {
    let array = var_content.value
    array.slice(3)
    return array
  }
})
const struct_elements = computed(() => { // 如果是结构体，获取结构体的子元素
  if (is_struct.value) {
    let struct = var_content.value
    struct.slice(3)
    return struct
  }
})
const is_pointer = computed(() => { // 判断该变量类型是否为指针
  if (is_data.value) {
    if (var_content.value[2] == 'pointer') {
      return true
    }
  }
  return false
})
const pointer_color = ref(-1) // 用数字储存对应的style
const css_pointer_color = computed(() => { // 获取对应的style
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
const css_var_color = computed(() => { // 获取对应的style
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
function highlight_pointer() { // 点击某个地址，使其和对应元素改变style
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
function unhighlight_pointer() { // 点击某个地址，使其和对应元素改变style
  if (is_data.value) {
    if (var_content.value[2] == 'pointer') {
      highlight_addresses.value[var_content.value[3]] = -1
      pointer_color.value = -1
    }
  }
}

watch(current_step, () => {
  pointer_color.value = -1
})

watch(changed,(new_value)=>{
  if(new_value == 'changed' || new_value == 'new'){
    console.log('1')
    nextTick(()=>{
      current_instance.proxy.$el.nextElementSibling.scrollIntoView({
        behavior: 'smooth',
        block: 'center'
      });
    })
  }
})

const current_instance = getCurrentInstance()

</script>

<template>
  <div v-if="is_data" class="big-box inline-block" :class="{ 'element-box': is_element }">
    <div class="small-box var-name">
      <span v-if="!is_element">{{ display_name }} = </span>
      <div v-else class="element-index">{{ display_name }}<br></div>
      <span :class="{ changed: is_changed, new: is_new, pointer: is_pointer }" @click="highlight_pointer" @contextmenu.prevent="unhighlight_pointer">
        {{ display_content}}
      </span>
    </div>
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
  <div v-if="is_multi_array" class="big-box">
    <div class="array-box">
      <div class="array-box-header">{{ display_name }}</div>
      <VarUnit v-for="(item, index) in multi_array_elements" :var_name="'^' + (index - 3) + '^' + multi_array_head" :var_content="item"
        :changed="check_changed(item)" />
    </div>
  </div>
</template>

<style>
.big-box {
  padding: 0.2rem;
}

.small-box {
  border-radius: 0.2rem;
  background-color: #EEF4FA;
  padding: 0.2rem;
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

.element-index{
  font-size: 0.4rem;

}

.inline-block{
  display: inline-block;
}
</style>