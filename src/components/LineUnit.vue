<script setup>
import { computed, inject, ref } from 'vue';

const props = defineProps(['line', 'code'])

const { current_line, next_line } = inject('data')

const { click_line, highlight_lines } = inject('step')

const { success } = inject('data')

const line = computed(() => {
  return props.line
})
const code = computed(() => {
  return props.code
})

const mark = computed(() => {
  if (line.value == current_line.value) {
    return '→'
  }
  if (line.value == next_line.value) {
    return '→'
  }
  return ' '
})

const is_current = computed(() => {
  return line.value == current_line.value
})

const is_next = computed(() => {
  return line.value == next_line.value
})

const css_code_background = computed(() => {
  if (Object.values(highlight_lines.value).includes(line.value)) {
    return '#FFDAD6'
  } else {
    return 'white'
  }
})

function click() {
  click_line(line.value)
}

</script>

<template>
  <div>
    <pre :class="{ blue: is_current, 'light-blue': is_next }">{{ mark }} </pre>
    <pre :class="{ blue: is_current }" class="code" @click="click">{{ code }}</pre>
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
  background-color: v-bind(css_code_background);
  transition-duration: 200ms;
}

.code:hover {
  /* background-color: #FFB4AB; */
  filter: brightness(0.9);
  transition-duration: 200ms;
}

.code:active {
  filter: brightness(0.8);
  transition-duration: 200ms;
}
</style>