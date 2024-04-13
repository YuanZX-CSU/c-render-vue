<script setup>
import { inject } from 'vue';


const { exception_msg, try_failed } = inject('app')

</script>

<template>
  <div class="info-view" v-show="!try_failed">
    <h1>欢迎</h1>
  </div>
  <div class="info-view" v-show="try_failed">
    <p class="big-face">🤔</p>
    <p>好像出了点问题？</p><br>
    <p>{{ exception_msg }}</p>
  </div>
</template>

<style scoped>
.info-view {
  width: 100%;
  height: 100%;
  overflow: auto;
}

.big-face {
  font-size: 7rem;
}
</style>