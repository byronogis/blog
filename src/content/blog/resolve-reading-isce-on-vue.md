---
title: 解决 Vue 项目中的 reading 'isCE' 报错
description: Cannot read properties of null (reading 'isCE') 解决方案
publishedTime: 2024-01-16
modifiedTime: 2024-01-17
tags:
  - vue
keywords: js,vue,vite,fileURLToPath

---

# 解决 Vue 项目中的 reading 'isCE' 报错

## 问题场景

构建 Vue 项目时, 如果当前项目的依赖中也有依赖 Vue, 可能会出现如下报错: 

`Cannot read properties of null (reading 'isCE')`

## 解决方案

名称定义:
消费方: 当前项目
提供方: 当前项目中也同时依赖 Vue 的项目

以下均为基于 Vite 构建工具的配置展示, 并隐藏了其它非相关配置

> 包管理工具为 pnpm, 供参考

### 常规步骤

1. 提供方打包时排除 Vue

vite.config.js

```js
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    rollupOptions: {
      external: ['vue']
    }
  }
})
```

2. 提供方将 Vue 作为 devDependency 而不是 dependency

package.json

```json
{
  "devDependencies": {
    "vue": "^3.2.6"
  }
}
```

### 额外步骤

> 以下步骤为常规步骤无法解决问题时的额外步骤
> 
> 如, 有时以本地路径方式安装提供方时, 会出现提供方依旧使用提供方自己的 vue, 而不是消费方的 vue; 或其它导致依赖读取不统一的情况

- 消费方添加 vue 别名指定固定路径

vite.config.js

```js
import { fileURLToPath } from 'node:url'
import { defineConfig } from 'vite'

export default defineConfig({
  resolve: {
    alias: {
      vue: fileURLToPath(new URL('./node_modules/vue', import.meta.url))
    }
  }
})
```

- 消费方添加 vue 的重复数据消除

vite.config.js

```js
import { defineConfig } from 'vite'

export default defineConfig({
  resolve: {
    dedupe: ['vue']
  }
})
```

