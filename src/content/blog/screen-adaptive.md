---
title: 大屏自适应实践记录
description: 利用 scale 实现屏幕自适应
publishedTime: 2023-11-03
modifiedTime: 2023-11-03
tags: [js]
keywords: js,vue,react,大屏,自适应,scale
---


# 大屏自适应实践记录

> 本文档记录了大屏实践过程中的一些经验和技巧  
> 现有技术栈参考: vite@3.2.3 + vue@3.2.45 + element-plus@2.2.27 + v-scale-screen@2.2.0

## 依赖安装

> Vue: [v-scale-screen](https://github.com/Alfred-Skyblue/v-scale-screen)(有 React 版本)

```bash
npm install v-scale-screen
```

## 配置使用

```vue
<!-- ~/src/App.vue -->
<script setup>
import VScaleScreen from 'v-scale-screen'

const designWidth = 1920
const designHeight = 1080

const scaleScreenProps = {
  width: designWidth,
  height: designHeight,
  delay: 0,
  boxStyle: {
    backgroundColor: '#000',
  },
  wrapperStyle: {
  },
  fullScreen: true,
}
</script>

<template>
  <VScaleScreen v-bind="scaleScreenProps">
    <router-view />
  </VScaleScreen>
</template>
```

## 优化

### element-plus 弹层弹窗适配

> 这一个优化需要修改的比较多, 目的是要使弹层弹窗的位置及大小保持和上面缩放后的项目协调一致  
> 新建预备插入位置  
> 对`element-plus`依赖进行patch(本地打补丁), 指向插入位置  
> 对`v-scale-screen`进行patch, 使其在缩放完成后触发一个事件, 此时对上面新建的插入位置进行位置及比例同步调整  

1. 新建预备插入位置

```diff
# ~/index.html

+ <div id="app-teleport"></div>
<div id="app"></div>
```

并设置如下样式:
```css
body {
  position: relative;
}

#app-teleport {
  position: absolute;
  z-index: 1000;
  width: 1920px; /* designWidth */
  height: 1080px; /* designHeight */
  pointer-events: none;
}

#app-teleport > * {
  pointer-events: auto;
}
```

2. 对`element-plus`依赖进行patch

```bash
pnpm patch element-plus
```

打开终端输出的目录, 目录中搜索代码`openBlock(), createBlock(Teleport`, 找到所有文件对应的位置, 修改如下属性

```diff
- to: ***,
+ to: '#app-teleport',
```

patch 参考:

```patch
diff --git a/es/components/dialog/src/dialog2.mjs b/es/components/dialog/src/dialog2.mjs
index a28b7f0e978c7e78f342a174879057dac0d75baf..8b76a9dd072bffba70171cf7daa8071c7f1ae6c0 100644
--- a/es/components/dialog/src/dialog2.mjs
+++ b/es/components/dialog/src/dialog2.mjs
@@ -78,7 +78,7 @@ const _sfc_main = /* @__PURE__ */ defineComponent({
     });
     return (_ctx, _cache) => {
       return openBlock(), createBlock(Teleport, {
-        to: "body",
+        to: '#app-teleport',
         disabled: !_ctx.appendToBody
       }, [
         createVNode(Transition, {
diff --git a/es/components/drawer/src/drawer2.mjs b/es/components/drawer/src/drawer2.mjs
index 135978dfeb2958cd845cf0dbeffba0813ff1ede9..a5be5de4c319c6a9709ed1f175fb2185605810c9 100644
--- a/es/components/drawer/src/drawer2.mjs
+++ b/es/components/drawer/src/drawer2.mjs
@@ -69,7 +69,7 @@ function _sfc_render(_ctx, _cache, $props, $setup, $data, $options) {
   const _component_el_focus_trap = resolveComponent("el-focus-trap");
   const _component_el_overlay = resolveComponent("el-overlay");
   return openBlock(), createBlock(Teleport, {
-    to: "body",
+    to: '#app-teleport',
     disabled: !_ctx.appendToBody
   }, [
     createVNode(Transition, {
diff --git a/es/components/image-viewer/src/image-viewer2.mjs b/es/components/image-viewer/src/image-viewer2.mjs
index 39fdeaaab69f91b47089c5c3c8cc34c53abcc528..333e85df2ac4f7dd5fa987ee8bca198585d487b1 100644
--- a/es/components/image-viewer/src/image-viewer2.mjs
+++ b/es/components/image-viewer/src/image-viewer2.mjs
@@ -259,7 +259,7 @@ const _sfc_main = /* @__PURE__ */ defineComponent({
     });
     return (_ctx, _cache) => {
       return openBlock(), createBlock(Teleport, {
-        to: "body",
+        to: '#app-teleport',
         disabled: !_ctx.teleported
       }, [
         createVNode(Transition, {
diff --git a/es/components/teleport/src/teleport2.mjs b/es/components/teleport/src/teleport2.mjs
index a86c399e19e703cf4d38d4774e8269043a2bde0c..8fd7581ae86f55cfd3ec6d5dead0b7b95e38660b 100644
--- a/es/components/teleport/src/teleport2.mjs
+++ b/es/components/teleport/src/teleport2.mjs
@@ -28,7 +28,7 @@ const _sfc_main = /* @__PURE__ */ defineComponent({
     return (_ctx, _cache) => {
       return _ctx.container ? (openBlock(), createBlock(Teleport$1, {
         key: 0,
-        to: _ctx.container,
+        to: '#app-teleport',
         disabled: _ctx.disabled
       }, [
         createElementVNode("div", {
diff --git a/es/components/tooltip/src/content2.mjs b/es/components/tooltip/src/content2.mjs
index 605353f93a55b0d0a09a30ab921e1288ff2c8bea..f991c8796f13c49f84f1d8a8abbd1e4b408aaae2 100644
--- a/es/components/tooltip/src/content2.mjs
+++ b/es/components/tooltip/src/content2.mjs
@@ -120,7 +120,7 @@ const _sfc_main = /* @__PURE__ */ defineComponent({
     return (_ctx, _cache) => {
       return openBlock(), createBlock(Teleport, {
         disabled: !_ctx.teleported,
-        to: unref(appendTo)
+        to: '#app-teleport',
       }, [
         createVNode(Transition, {
           name: _ctx.transition,
diff --git a/es/components/tooltip-v2/src/tooltip2.mjs b/es/components/tooltip-v2/src/tooltip2.mjs
index 1b13000ff57a9c519f83330fd0c50a94ccc4c2a6..52c158ad092387a277964339d9610413801d9312 100644
--- a/es/components/tooltip-v2/src/tooltip2.mjs
+++ b/es/components/tooltip-v2/src/tooltip2.mjs
@@ -34,7 +34,7 @@ const _sfc_main = /* @__PURE__ */ defineComponent({
             _: 3
           }, 16),
           (openBlock(), createBlock(Teleport, {
-            to: _ctx.to,
+            to: '#app-teleport',
             disabled: !_ctx.teleported
           }, [
             _ctx.fullTransition ? (openBlock(), createBlock(Transition, normalizeProps(mergeProps({ key: 0 }, _ctx.transitionProps)), {
```

3. 对`v-scale-screen`进行patch

```bash
pnpm patch v-scale-screen
```

对缩放的元素添加一个`transitionend`事件, 用于及时触发上面新建的插入位置的位置及比例同步调整

patch 参考:

```patch
diff --git a/dist/v-scale-screen.js b/dist/v-scale-screen.js
index 6253bfdd6d7de74378319b07e3aca50d3ae7b267..6bc811497383ba64c819d343f67afa8aea66bdc4 100644
--- a/dist/v-scale-screen.js
+++ b/dist/v-scale-screen.js
@@ -44,9 +44,10 @@ const N = H({
     bodyOverflowHidden: {
       type: Boolean,
       default: !0
-    }
+    },
   },
-  setup(t, { slots: l }) {
+  emits: ["transitionend"],
+  setup(t, { slots: l, emit, }) {
     let o;
     const e = p({
       width: 0,
@@ -108,7 +110,9 @@ const N = H({
     };
     return O(() => {
       m(), b(async () => {
-        await f(), g(), w(), window.addEventListener("resize", h), x();
+        await f();
+        n.value?.addEventListener("transitionend", (e) => emit('transitionend', e)),
+        g(), w(), window.addEventListener("resize", h), x();
       });
     }), $(() => {
       var i;
```

4. 修改`v-scale-screen`配置

```diff
const scaleScreenProps = {
  width: designWidth,
  height: designHeight,
  delay: 0,
  boxStyle: {
    backgroundColor: '#000',
  },
  wrapperStyle: {
  },
  fullScreen: true,
+  onTransitionend(e) {
+    if (e.currentTarget !== e.target) return
+    console.log('transitionend')
+    const { x, y } = e.target.getBoundingClientRect()
+    const teleportEl = document.querySelector('#app-teleport')
+    teleportEl.style.transform = window.getComputedStyle(e.target).transform
+    teleportEl.style.transformOrigin = 'top left'
+    teleportEl.style.top = `${y}px`
+    teleportEl.style.left = `${x}px`ndow.getComputedStyle(e.target).transform.split(',')[0].slice(7))
+  },
}
```

### 保持比例

> 禁用`fullScreen`属性, 同时可选的设置留白背景色  
> 同时, 适配`element-plus`弹层弹窗的背景范围至屏幕全部区域

1. 修改`v-scale-screen`配置

```diff
const scaleScreenProps = {
  width: designWidth,
  height: designHeight,
  delay: 0,
  boxStyle: {
-    backgroundColor: '#000',
+    backgroundColor: '#000', // change to your project design background color
  },
  },
  wrapperStyle: {
  },
-  fullScreen: true,
+  fullScreen: false,
  onTransitionend(e) {
    if (e.currentTarget !== e.target) return
    console.log('transitionend')
    const { x, y } = e.target.getBoundingClientRect()
    const teleportEl = document.querySelector('#app-teleport')
    teleportEl.style.transform = window.getComputedStyle(e.target).transform
    teleportEl.style.transformOrigin = 'top left'
    teleportEl.style.top = `${y}px`
    teleportEl.style.left = `${x}px`ndow.getComputedStyle(e.target).transform.split(',')[0].slice(7))
  },
}
```

2. 适配`element-plus`弹层弹窗的背景范围至屏幕全部区域

```diff
# ~/index.html
- <div id="app-teleport"></div>
+ <div id="app-teleport__wrapper">
+   <div id="app-teleport"></div>
+ </div>
```

同时修改之前的样式为:
  
```css
#app-teleport__wrapper, #app-teleport {
  position: absolute;
  z-index: 1000;
  pointer-events: none;
}

#app-teleport__wrapper {
  width: 100vw;
  height: 100vh;
}

#app-teleport {
  width: 1920px;
  height: 1080px;
}

#app-teleport__wrapper:has(.el-overlay:not(:is([style*="display: none"], [style*="display:none"]))) {
  background-color: var(--el-overlay-color-lighter); /* or change to your mask color */
}

#app-teleport .el-overlay {
  background-color: transparent;
}

#app-teleport > * {
  pointer-events: auto;
}
```

### 特定路由缩放

1. 将`v-scale-screen`的组件包裹调整到的需要缩放的页面路由中

```diff
# ~/src/App.vue

- <VScaleScreen v-bind="scaleScreenProps">
-   <router-view />
- </VScaleScreen>
+ <router-view />
```

```vue
<!-- ~/src/path/to/view.vue -->

<script setup>
import VScaleScreen from 'v-scale-screen'

// 同时记得把之前的配置移过来
</script>

<template>
  <VScaleScreen v-bind="scaleScreenProps">
    <!-- ... -->
  </VScaleScreen>
</template>
```

2. 调整`v-scale-screen`的配置

```diff
onTransitionend(e) {
  if (e.currentTarget !== e.target) return
  console.log('transitionend')
  const { x, y } = e.target.getBoundingClientRect()
-  const teleportEl = document.querySelector('#app-teleport')
-  teleportEl.style.transform = window.getComputedStyle(e.target).transform
-  teleportEl.style.transformOrigin = 'top left'
-  teleportEl.style.top = `${y}px`
-  teleportEl.style.left = `${x}px`
+  document.documentElement.style.setProperty('--design-transform', window.getComputedStyle(e.target).transform)
+  document.documentElement.style.setProperty('--design-transform-origin', 'top left')
+  document.documentElement.style.setProperty('--design-top', `${y}px`)
+  document.documentElement.style.setProperty('--design-left', `${x}px`)
},
```

3. 添加样式

```diff
#app-teleport {
  width: 1920px;
  height: 1080px;
+  transform: var(--design-transform);
+  transform-origin: var(--design-transform-origin);
+  top: var(--design-top);
+  left: var(--design-left);
+}
+
+#app-teleport.not-scale-screen {
+  width: 100%;
+  height: 100%;
+  transform: unset;
+  transform-origin: unset;
+  top: unset;
+  left: unset;
}
```

4. 在`App.vue`中添加针对路由路径的监听器, 用于判断是否需要缩放, 以调整弹层弹窗插入点的样式

```diff
# ~/src/App.vue

+ const scaleScreenRoutes = [
+   '/your/path/to/scale/screen',
+ ]
+ 
+ const route = useRoute()
+ 
+ watch(() => route.path, (newValue) => {
+   if (scaleScreenRoutes.includes(newValue) || scaleScreenRoutes.some(i => newValue.startsWith(i))) {
+     document.querySelector('#app-teleport')?.classList.remove('not-scale-screen')
+   } else {
+     document.querySelector('#app-teleport')?.classList.add('not-scale-screen')
+   }
+ }, { immediate: true })
```


