<script setup>
import { computed, inject } from 'vue';

const props = defineProps(['step'])
const this_step = computed(() => {
  return props.step
})

const { current_step, click_step, highlight_steps } = inject('app')
const finished = computed(() => {
  return this_step.value <= current_step.value
})
const css_background_color = computed(() => {
  if (Object.values(highlight_steps.value).includes(this_step.value)) {
    if (finished.value) {
      return '#ffb4ab' // 已完成 高亮
    } else {
      return '#ffdad6' // 未完成 高亮
    }
  } else {
    if (finished.value) {
      return '#71d2ff' // 已完成
    } else {
      return '#c0e8ff' // 未完成
    }
  }
})
const css_filter = computed(() => {
  if (finished.value) {
    return 'brightness(0.8)' // 暗色情况下改为1.5
  } else {
    return 'brightness(0.8)' // 暗色情况下改为1.5
  }
})


function click_step_unit() {
  click_step(this_step.value)
}


</script>

<template>
  <div class="step-unit" @click="click_step_unit">
  </div>
</template>

<style scoped>
.step-unit {
  background-color: v-bind(css_background_color);
  transition-duration: 200ms;
}

.step-unit:hover {
  filter: v-bind(css_filter);
  transition-duration: 200ms;
}
</style>