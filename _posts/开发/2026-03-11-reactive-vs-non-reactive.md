---
layout: post
title: "响应式数据和非响应式数据在本质上有什么区别"
subtitle: "深入理解 Vue3 响应式系统的核心原理"
date: 2026-03-11
author: "b0t"
header-img: "img/post-bg-vue.jpg"
catalog: true
tags:
  - Vue3
  - 前端开发
  - 响应式原理
---


这是 **Vue3 最核心的问题之一**。我们从 **本质（底层机制）**讲清楚，但用最容易理解的方式。

---

# 一、最简单一句话

**本质区别：**

| 类型     | 本质                                        |
| ------ | ----------------------------------------- |
| 非响应式数据 | 普通 JavaScript 变量                          |
| 响应式数据  | 被 Vue **代理（Proxy / getter / setter）** 的变量 |

也就是说：

响应式 = Vue 能知道你什么时候改了数据  
非响应式 = Vue 不知道

---

# 二、先看普通变量（非响应式）

let age = 20  
age = 30

JavaScript只是：

20 → 30

**Vue完全不知道发生了什么。**

所以：

页面不会更新

---

# 三、响应式数据

Vue会在数据外面 **包一层“监控系统”**。

比如：

const age = ref(20)

其实内部类似：

age = {  
  get value(){  
      // 读取数据  
  },  
  set value(newValue){  
      // 数据改变  
  }  
}

当你修改：

age.value = 30

Vue就能知道：

数据变了

于是：

重新渲染页面

---

# 四、核心区别（非常重要）

|特性|非响应式|响应式|
|---|---|---|
|数据变化|Vue不知道|Vue能检测|
|页面更新|不会更新|自动更新|
|实现方式|普通变量|Proxy / getter / setter|

---

# 五、举一个最简单例子

### 非响应式

let count = 0  
  
function add(){  
    count++  
}

模板：

{{ count }}

点击按钮：

count 变了  
但页面不会更新

因为 Vue 没监听它。

---

### 响应式

const count = ref(0)  
  
function add(){  
    count.value++  
}

点击按钮：

count.value 变了  
Vue检测到变化  
页面自动更新

---

# 六、Vue3 响应式原理（最核心流程）

Vue内部流程其实只有三步：

读取数据  
 ↓  
记录依赖  
 ↓  
修改数据  
 ↓  
触发更新

更专业一点：

track()  → 收集依赖  
trigger() → 触发更新

---

# 七、再讲一个本质（很多人不知道）

Vue3响应式核心就是：

Proxy

例如：

const user = reactive({  
  name: "张三"  
})

其实变成：

Proxy({  
  name: "张三"  
})

Proxy会：

拦截读取  
拦截修改

所以 Vue 能知道：

什么时候访问数据  
什么时候修改数据

---

# 八、最简单理解方式

可以这样理解：

|数据|状态|
|---|---|
|普通变量|Vue看不见|
|ref/reactive|Vue在监视|

就像：

普通数据 → 没摄像头  
响应式数据 → 有摄像头

Vue一直在看。

---

# 九、Vue3里两种响应式

|API|用途|
|---|---|
|ref()|基本类型|
|reactive()|对象|

例子：

const age = ref(20)  
  
const user = reactive({  
  name: "张三",  
  age: 20  
})

---

# 十、一句话总结

**响应式数据的本质：**

> Vue在数据外面加了一层代理，使得数据变化时可以自动触发页面更新。