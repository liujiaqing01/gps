<template>
  <div class="app-container home">
    <h2>GPS调度系统</h2>
    <el-button :class="classValue" @click="ck">dh</el-button>
  </div>
</template>

<script setup name="Index">
import {ref} from 'vue'

const classValue = ref([])

function ck() {
  classValue.value = ['animate__animated', 'animate__hinge']
}
</script>

<style scoped lang="scss">
h2 {
  text-align: center;
}
</style>

