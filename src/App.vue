<script setup lang="ts">
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import HelloWorld from './components/HelloWorld.vue';
import ThemeSetting from './components/ThemeSetting.vue';
import { ConfigProvider } from 'ant-design-vue';
ConfigProvider.config({
  theme: {
    primaryColor: '#18a058',
  },
});
</script>

<template>
  <ThemeSetting />
  <HelloWorld msg="Hello Vue 3 + TypeScript + Vite" />
</template>

<style lang="less">
@import url('ant-design-vue/dist/antd.variable.less');
@import url('@/assets/styles/common/var.css');
@import url('@/assets/styles/common/common.less');
</style>
