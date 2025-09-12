---
title: "PrimevueV4+TailwindV4配置" # 文章标题
date: 2025-09-12T14:30:00+08:00 # 文章创建日期
draft: false # 是否为草稿，true 表示草稿，不会在网站上显示
author: "runshell" # 文章作者
description: "介绍PrimevueV4+TailwindV4配置" # 文章描述
tags: ["Primevue", "vue3", "前端"] # 文章标签
categories: ["WEB开发"] # 文章分类
---

# 初始化项目

```bash
npm create vue@latest .
```

# 安装 primevue 及其相关组件

```bash
npm install primevue primeicons
npm install @primevue/themes
```

# 安装自动导入插件

```bash
npm install unplugin-vue-components -D
npm install @primevue/auto-import-resolver -D
```

# 安装 tailwindcss

```bash
npm install tailwindcss-primeui
npm install tailwindcss @tailwindcss/vite
```

# src/main.js 配置

```javascript
import { createApp } from "vue";
import App from "./App.vue";
import PrimeVue from "primevue/config";
import Aura from "@primevue/themes/aura";
import "primeicons/primeicons.css";

const app = createApp(App);

app.use(router);
app.use(PrimeVue, {
  theme: {
    preset: Aura,
    options: {
      // 深色模式开关
      darkModeSelector: false,
    },
  },
});

app.mount("#app");
```

# src/style.css 配置

```css
/* 导入tailwindcss */
@import "tailwindcss";
@import "tailwindcss-primeui";
```

# index.html

```html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <!-- 加载css，使style.css中导入的tailwindcss生效 -->
    <link href="/src/style.css" rel="stylesheet" />
    <title>Vite App</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

# vite.config.js 配置

```javascript
import { fileURLToPath, URL } from "node:url";
import vue from "@vitejs/plugin-vue";
import { defineConfig } from "vite";
// 导入tailwindcss
import tailwindcss from "@tailwindcss/vite";
// 导入自动导入插件
import Components from "unplugin-vue-components/vite";
import { PrimeVueResolver } from "@primevue/auto-import-resolver";

// https://vite.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    // 加载tailwindcss
    tailwindcss(),
    Components({
      // 自动导入组件
      resolvers: [PrimeVueResolver()],
    }),
  ],
  resolve: {
    alias: {
      "@": fileURLToPath(new URL("./src", import.meta.url)),
    },
  },
});
```
