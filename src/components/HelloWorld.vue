<script setup lang="ts">
defineProps<{ msg: string }>();
</script>

<template>
  <div class="title">{{ msg }}</div>

  <a-button type="primary">按钮</a-button>
</template>

<style lang="less" scoped>
.title {
  color: @primary-color;
}
</style>
