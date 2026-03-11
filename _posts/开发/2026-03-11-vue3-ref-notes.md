---
layout: post
title: "Vue3 响应式数据 —— ref 笔记"
subtitle: "深入理解 Vue3 的 ref 响应式系统"
date: 2026-03-11
author: "b0t"
header-img: "img/post-bg-vue.jpg"
catalog: true
tags:
  - Vue3
  - 前端开发
  - 响应式数据
---

## 一、什么是 `ref`

`ref` 是 Vue3 提供的一个函数，用来 **创建响应式数据（Reactive Data）**。

作用：

定义可以被 Vue 监听的数据  
数据变化 → 页面自动更新

---

# 二、`ref` 的基本语法

let xxx = ref(初始值)

例如：

import { ref } from 'vue'  
  
let name = ref('张三')  
let age = ref(18)

---

# 三、`ref` 返回的是什么

`ref()` 返回的是一个 **RefImpl 实例对象**。

可以理解为：

ref对象

内部结构类似：

{  
  value: 初始值  
}

例如：

let name = ref('张三')

实际结构可以理解为：

name = {  
   value: '张三'  
}

---

# 四、为什么要 `.value`

因为 `ref()` 返回的是一个对象：

name.value

所以：

|操作|写法|
|---|---|
|读取|`name.value`|
|修改|`name.value = '李四'`|

例如：

function changeName(){  
    name.value = '李四'  
}

---

# 五、模板中为什么不需要 `.value`

在 **Vue模板中**：

<h2>姓名：{{name}}</h2>

Vue 会 **自动解包 ref**。

所以：

模板里  
name === name.value

不需要写：

{{name.value}}

---

# 六、完整示例代码

<template>  
  <div class="person">  
    <h2>姓名：{{name}}</h2>  
    <h2>年龄：{{age}}</h2>  
  
    <button @click="changeName">修改名字</button>  
    <button @click="changeAge">年龄+1</button>  
    <button @click="showTel">点我查看联系方式</button>  
  </div>  
</template>  
  
<script setup lang="ts" name="Person">  
import {ref} from 'vue'  
  
// 创建响应式数据  
let name = ref('张三')  
let age = ref(18)  
  
// 普通变量（不是响应式）  
let tel = '13888888888'  
  
function changeName(){  
  name.value = '李四'  
}  
  
function changeAge(){  
  age.value += 1  
}  
  
function showTel(){  
  alert(tel)  
}  
</script>

---

# 七、响应式与非响应式对比

|变量|是否响应式|修改后页面是否更新|
|---|---|---|
|`name = ref('张三')`|是|会更新|
|`age = ref(18)`|是|会更新|
|`tel = '13888888888'`|否|不会更新|

例如：

tel = '123'

页面不会变化。

---

# 八、重要理解

对于：

let name = ref('张三')

要注意：

name 不是响应式  
name.value 才是响应式

所以：

正确写法：

name.value = '李四'

错误写法：

name = ref('李四')

因为：

Vue监听的是 value

---

# 九、`ref` 的使用场景

适用于：

基本类型数据

例如：

|类型|
|---|
|string|
|number|
|boolean|

例如：

const count = ref(0)  
const title = ref('Vue')  
const isLogin = ref(true)