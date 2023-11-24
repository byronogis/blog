---
title: 添加 git 信息到 Vite 生成的 HTML 中
description: 使用 zx 在 js 中执行 git 命令并获取最新提交的 sha, 然后将 sha 写入到 Vite 生成的 HTML 中
publishedTime: 2023-11-24
tags:
  - vite
  - html
keywords: vite,html,git,zx
modifiedTime: 2023-11-24

---

# 添加 git 信息到 Vite 生成的 HTML 中

> 为了明确当前页面的版本情况, 可以将当前项目最新的 git 信息写入到 HTML 中

## 安装 zx

> [zx: A tool for writing better scripts](./node-package-zx)

```bash
# with pnpm
pnpm add zx -D
# or with other if you prefer
npm install zx -D
yarn add zx -D
```

## 写一个 Vite Plugin

```js
// vite.config.js
import { defineConfig, loadEnv } from 'vite'

export default defineConfig({
  // ...
  plugins: [
    // ...
    {
      name: 'vite-plugin-html-git-sha',
      transformIndexHtml(html) {
        return new Promise((resolve) => {
          import('zx/core').then(async (zx) => {
            const version = (await zx.$`git rev-parse --short HEAD`).stdout.trim()
            resolve([
              {
                tag: 'meta',
                injectTo: 'head',
                attrs: {
                  name: 'git:version',
                  content: version,
                },
              },
            ])
          })
        })
      },
    }
  ]
})
```
