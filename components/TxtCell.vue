<script setup lang="ts">
interface Props {
  value?: any
}defineProps<Props>()
</script>

<template>
  <!-- 解决 td 内元素撑满 td 高度后，上下无法居中的问题 -->
  <div class="flex flex-col justify-center h-100%!" >
    <span>{{ value }}</span>
  </div>
</template>