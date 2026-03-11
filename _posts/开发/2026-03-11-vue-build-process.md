---
layout: post
title: "Vue 项目的整个构建流程"
subtitle: "从创建项目到页面渲染的完整流程解析"
date: 2026-03-11
author: "b0t"
header-img: "img/post-bg-vue.jpg"
catalog: true
tags:
  - Vue3
  - 前端开发
  - 项目构建
---

# 一、Vue 项目的整体流程

最核心的流程其实只有一条：

创建项目  
   ↓  
启动开发服务器  
   ↓  
加载 index.html  
   ↓  
执行 main.ts  
   ↓  
创建 Vue 应用  
   ↓  
加载 App.vue  
   ↓  
加载 components  
   ↓  
渲染页面

我们一步一步解释。

---

# 二、第一步：创建 Vue 项目

你之前运行的命令类似：

npm create vue@latest

或者：

npm init vue@latest

然后创建了一个项目：

hello_vue3

项目结构大概是：

hello_vue3  
 ├─ node_modules  
 ├─ public  
 ├─ src  
 │   ├─ assets  
 │   ├─ components  
 │   ├─ App.vue  
 │   └─ main.ts  
 ├─ index.html  
 ├─ package.json  
 └─ vite.config.ts

这里最重要的文件：

|文件|作用|
|---|---|
|index.html|网页入口|
|main.ts|Vue启动文件|
|App.vue|根组件|
|components|子组件|

---

# 三、第二步：启动项目

你运行：

npm run dev

实际上执行的是：

Vite 开发服务器

Vite 会：

1 启动本地服务器  
2 编译 Vue 文件  
3 打开浏览器

浏览器访问：

http://localhost:5173

---

# 四、第三步：浏览器加载 index.html

浏览器首先读取：

index.html

例如：

<body>  
  <div id="app"></div>  
  
  <script type="module" src="/src/main.ts"></script>  
</body>

这里关键是：

```
<script src="/src/main.ts">
```

意思是：

加载 main.ts

---

# 五、第四步：执行 main.ts

main.ts 代码通常是：

import { createApp } from 'vue'  
import App from './App.vue'  
  
createApp(App).mount('#app')

执行流程：

导入 Vue  
↓  
导入 App.vue  
↓  
创建 Vue 应用  
↓  
挂载到 # app
### createApp(App)
- 功能 ：创建一个新的 Vue 应用实例
- 参数 ： App 是根组件（在这个项目中是从 ./App.vue 导入的）
- 返回值 ：返回一个应用实例，该实例包含了应用的配置和方法
### mount('#app')
- 功能 ：将创建的 Vue 应用实例挂载到指定的 DOM 元素上
- 参数 ： '#app' 是一个 CSS 选择器，指向 HTML 中 id 为 "app" 的元素
- 作用 ：将 Vue 组件渲染到指定的 DOM 元素中，使其成为应用的容器
### 整个过程的工作原理：
1. Vue 加载根组件 App
2. 解析组件的模板、数据和逻辑
3. 将组件渲染为真实的 DOM 元素
4. 将渲染后的 DOM 元素插入到 HTML 中 id 为 "app" 的元素内
### 为什么需要这样做？
- 分离关注点 ：将应用的创建和挂载分开，使代码更清晰
- 灵活性 ：可以在挂载前对应用实例进行配置（如添加插件、全局属性等）
- 模块化 ：便于组织大型应用的代码结构
### 实际效果
当执行这行代码后，你在 App.vue 中定义的模板会被渲染到 HTML 页面中，替换掉原来的 <div id="app"></div> 元素，从而在浏览器中显示你的 Vue 应用。

示例 ：

- 如果你在 App.vue 中有 <h1>hello</h1> ，执行后浏览器会显示这个标题
- 如果你在 App.vue 中使用了其他组件（如 <Person /> ），它们也会被正确渲染
这是 Vue 3 应用的标准启动方式，是每个 Vue 3 项目的入口点。


# 六、第五步：加载 App.vue

`App.vue` 是 **整个 Vue 应用的根组件**。

例如：

<template>  
  <h1>Hello Vue</h1>  
</template>

Vue 会把这个组件渲染到：

<div id="app"></div>

变成：

<div id="app">  
  <h1>Hello Vue</h1>  
</div>

---

# 七、第六步：加载子组件

如果 `App.vue` 使用组件：

<template>  
  <Person />  
</template>

Vue会：

加载 components/Person.vue

然后渲染：

组件树

例如：

App  
 ├─ Person  
 ├─ Navbar  
 └─ Footer

---

# 八、Vue文件是怎么运行的

浏览器其实 **不能直接运行 `.vue` 文件**。

所以 Vite 会：

.vue 文件  
   ↓  
编译  
   ↓  
JavaScript  
   ↓  
浏览器执行

例如：

<template>  
<h1>{{ name }}</h1>  
</template>

会被编译成：

render(){  
   return h("h1", this.name)  
}

---

# 九、开发模式 vs 生产模式

### 开发模式

npm run dev

特点：

- 实时刷新
    
- 快速编译
    
- 调试
    

---

### 生产模式

npm run build

Vite 会：

压缩代码  
打包JS  
优化资源

生成：

dist/  
 ├─ index.html  
 ├─ assets  
 └─ js

然后部署到服务器。

---

# 十、完整流程图（最重要）

npm create vue  
        ↓  
创建项目结构  
        ↓  
npm install  
        ↓  
npm run dev  
        ↓  
Vite启动  
        ↓  
浏览器加载 index.html  
        ↓  
执行 main.ts  
        ↓  
createApp(App)  
        ↓  
加载 App.vue  
        ↓  
加载 components  
        ↓  
渲染页面