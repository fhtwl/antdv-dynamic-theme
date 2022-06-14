# 基于 vue3、vite、antdv、css 变量实现在线主题色切换

## 1、前言

动态切换主题是一个很常见的需求. 实现方案也有很多, 如:

- 编译多套 css 文件, 然后切换类名(需要预设主题, 不够灵活)

- less 在线编译(不兼容 ie, 性能较差)

- css 变量(不兼容 ie)

但是这些基本都是针对 vue2 的, 我在网上并没有找到比较完整的解决 vue3 换肤的方案, 大多只处理了自定义样式或者 ui 框架(比如 antdv)二者之一的主题切换, <a href="https://www.antdv.com/docs/vue/customize-theme-variable-cn" >antdv 官网</a>对动态主题的说明也不够清晰, 且与推荐的按需加载插件 unplugin-vue-components 有冲突 <br />

我最终放弃了 unplugin-vue-components 的样式的按需加载, 采取组件按需加载, 样式全量加载, 并通过 css 变量和 antdv 的 ConfigProvider 实现了在线主题色切换 <br />

下面是具体是实现

## 2、基础环境搭建

### 1、项目创建

根据<a href="https://vitejs.cn/guide/#scaffolding-your-first-vite-project" >vite 官方文档</a>, 使用社区模板, 即可轻松创建基于 vue3 和 ts 的项目模板

```sh
npm init vite@latest
```

然后按照提示, 依次选择 _vue_ 、 _vue-ts_ , 即可创建 vue3 + ts + vite 项目

### 2、eslint 和 prettier 配置

安装依赖

```sh
yarn add eslint eslint-config-prettier eslint-plugin-prettier eslint-plugin-vue @typescript-eslint/eslint-plugin @typescript-eslint/parser prettier -D
```

添加配置

1. 新增.eslintrc.json

```json
// .eslintrc.json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true,
    "vue/setup-compiler-macros": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:vue/vue3-recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:prettier/recommended"
  ],
  "parser": "vue-eslint-parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "parser": "@typescript-eslint/parser",
    "sourceType": "module"
  },
  "plugins": ["vue", "@typescript-eslint"],
  "rules": {
    "vue/comment-directive": "off",
    "prettier/prettier": "off",
    // 允许单字单词作为组件名
    "vue/multi-word-component-names": "off",
    "@typescript-eslint/no-non-null-assertion": "off"
  }
}
```

2. 新增.prettierrc.js

```js
// .prettierrc.js
module.exports = {
  printWidth: 80, //单行长度
  tabWidth: 2, //缩进长度
  useTabs: false, //使用空格代替tab缩进
  semi: true, //句末使用分号
  singleQuote: true, //使用单引号
};
```

3. 安装 vscode 插件

安装下列 vscode 插件(已安装可跳过)

- Vue Language Features (Volar)
- Prettier - Code formatter

Volar 可以简单理解为是 vue3 的 Vetur, 如果是既有 vue2 项目又有 vue3 项目的, 可在工作区修改设置 <br />

新建.vscode 文件夹, 在.vscode 新建 extensions.json 和 settings.json

```json
// .vscode/extensions.json
{
  "recommendations": ["johnsoncodehk.volar"]
}
```

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  "vetur.format.enable": true,
  "vetur.validation.script": false,
  "vetur.validation.style": false,
  "vetur.validation.template": false
}
```

4. 配置 lint 相关命令

修改 package.json

```json
// package.json
...
"scripts": {
  "dev": "vite",
  "build": "vue-tsc --noEmit && vite build",
  "preview": "vite preview",
  "lint": "eslint . --ext .vue,.ts,.jsx,.tsx --fix",
  "format": "prettier --write ./**/*.{vue,ts,tsx,js,jsx,css,less,scss,json,md}"
},
...
```

执行 lint 和 format 即可校验和格式化项目文件

## 3、自定义主题色切换

### 1、引入 less

这套方案里, less 不是必须的, 使用 css、sass、postCSS 都可以, 其核心原理是使用了 css 变量, 但是项目需要使用 less 的类名嵌套、变量、函数等功能, 且 antd 本身是基于 less 的, 因此项目的样式这里也统一使用 less. <br/>

vite 本身是支持 less 等 css 预编译器的, 只需要安装 less, 然后直接在 style 添加属性 lang = 'ts' 即可使用 less <br/>

```
yarn add less -D
```

在 HelloWorld.vue 里使用 less

```vue
<!-- src/components/HelloWorld.vue -->
<script setup lang="ts">
defineProps<{ msg: string }>();
</script>

<template>
  <div class="title">{{ msg }}</div>
</template>

<style lang="less" scoped>
.title {
  color: red;
}
</style>
```

### 2、自定义全局 less 变量

在 assets 新建 styles 目录存放样式, 在 styles 下新建 common 目录存放公共样式和变量, 在 common 下新建 common.less、var.css 和 var.less <br/>

```css
/* src/assets/styles/common/var.css 存放css变量 */
:root {
  --ant-primary-color: #18a058;
}
```

```less
/* src/assets/styles/common/var.less 存放less变量 */
@primary-color: var(--ant-primary-color, #18a058);
```

```less
/* src/assets/styles/common/common.less 存放公共样式 */
a {
  color: @primary-color;
}
```

在 vite.config.ts 引入 var.less, var.less 里定义的 less 变量便可以在全局使用

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import path from 'path';

function resolve(url: string): string {
  return path.resolve(__dirname, url);
}

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve('./src'),
      '~@': resolve('./src'),
    },
  },
  css: {
    preprocessorOptions: {
      less: {
        // 全局添加less
        additionalData: `@import '@/assets/styles/common/var.less';`,
        javascriptEnabled: true,
      },
    },
  },
});
```

此时 ts 并不能识别 nodejs 的模块, path 会报错, 需要安装@types/node

```sh
yarn add @types/node -D
```

然后修改 tsconfig.node.json

```json
// tsconfig.node.json
{
  "compilerOptions": {
    "composite": true,
    "module": "esnext",
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

在 App.vue 里引入 var.css 和 common.less

```vue
<!-- src/App.vue -->
<script setup lang="ts">
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import HelloWorld from './components/HelloWorld.vue';
</script>

<template>
  <HelloWorld msg="Hello Vue 3 + TypeScript + Vite" />
</template>

<style>
@import url('@/assets/styles/common/var.css');
@import url('@/assets/styles/common/common.less');
</style>
```

然后在 HelloWorld 里使用 less 变量, 可以看到配置生效了

```vue
<!-- src/components/HelloWorld.vue -->
<script setup lang="ts">
defineProps<{ msg: string }>();
</script>

<template>
  <div class="title">{{ msg }}</div>
</template>

<style lang="less" scoped>
.title {
  color: @primary-color;
}
</style>
```

### 3、切换主题色

新建 ThemeSetting 组件, 用于修改全局的主题色

```vue
<!-- src/components/ThemeSetting.vue -->
<script lang="ts">
import { defineComponent } from 'vue';
export default defineComponent({
  setup() {
    return {
      color: '#18a058',
      setColor(color: string) {
        document.documentElement.style.setProperty(
          '--ant-primary-color',
          color
        );
      },
    };
  },
  methods: {
    handleChange(color: string) {
      this.color = color;
      this.setColor(this.color);
    },
  },
});
</script>

<template>
  <input type="color" v-model="color" @change="handleChange(color)" />
</template>

<style lang="less" scoped></style>
```

当通过颜色选择器修改颜色后, 可以看到 HelloWorld.vue 的颜色同时被修改了

## 4、antdv 主题色切换

### 1、引入 antdv

```sh
yarn add ant-design-vue
```

在 main.ts 里引入非组件模块

```ts
// src/main.ts
import { createApp } from 'vue';
import App from './App.vue';

import { message } from 'ant-design-vue';

const app = createApp(App);
app.mount('#app');
app.config.globalProperties.$message = message;
```

### 2、按需加载

安装 unplugin-vue-components

```sh
yarn add unplugin-vue-components -D
```

在 vite 里使用插件

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import path from 'path';
import Components from 'unplugin-vue-components/vite';
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers';

function resolve(url: string): string {
  return path.resolve(__dirname, url);
}

export default defineConfig({
  plugins: [
    vue(),
    // 按需加载
    Components({
      resolvers: [
        AntDesignVueResolver({
          // 不加载css, 而是手动加载css. 通过手动加载less文件并将less变量绑定到css变量上, 即可实现动态主题色
          importStyle: false,
        }),
      ],
    }),
  ],
  ...
});


```

### 3、主题色切换

全局引入 css 文件, 并通过 ConfigProvider 设置主题色

```vue
<!-- src/App.vue -->
<script setup lang="ts">
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
```

在 HelloWorld 里使用 antd 组件

```vue
<!-- src/components/HelloWorld.vue -->
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
```

修改 setColor, 添加设置 antd 主题色功能

```vue
<script lang="ts">
import { ConfigProvider } from 'ant-design-vue';
import { defineComponent } from 'vue';
export default defineComponent({
  setup() {
    return {
      color: '#18a058',
      setColor(color: string) {
        document.documentElement.style.setProperty(
          '--ant-primary-color',
          color
        );
        ConfigProvider.config({
          theme: {
            primaryColor: color,
          },
        });
      },
    };
  },
});
</script>
...
```

## 5. 总结

至此, 基于 vue3 和 antdv 的主题色切换功能完成了. <br />

有好的建议，请在下方输入你的评论。<br />

完整代码可以访问<a href="https://github.com/fhtwl/antdv-dynamic-theme" > github </a>
