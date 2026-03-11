---
layout: post
title: "ref 与 reactive 的区别"
subtitle: "Vue3 中两种响应式 API 的对比与使用指南"
date: 2026-03-11
author: "b0t"
header-img: "img/post-bg-vue.jpg"
catalog: true
tags:
  - Vue3
  - 前端开发
  - 响应式数据
---

# 一、什么是响应式数据

所谓 **响应式数据**，指的是：

> 当数据发生变化时，页面会自动更新。

例如：

<template>  
  <div>{{ count }}</div>  
</template>

如果 `count` 改变，页面中的 `count` 会自动重新渲染。

Vue3 主要通过两种方式创建响应式数据：

- `ref`
    
- `reactive`
    

---

# 二、ref 的作用

`ref` 用来创建 **响应式变量**。

它既可以定义 **基本类型数据**，也可以定义 **对象类型数据**。

示例：

import { ref } from 'vue'  
  
const count = ref(0)

这里：

count -> ref对象  
count.value -> 实际数据

如果要修改数据：

count.value++

### ref 的特点

1️⃣ 必须使用 `.value` 访问数据

console.log(count.value)

2️⃣ Vue 会自动追踪 `.value` 的变化

当 `.value` 改变时，页面会更新。

---

# 三、reactive 的作用

`reactive` 用来创建 **响应式对象**。

示例：

import { reactive } from 'vue'  
  
const user = reactive({  
  name: "Tom",  
  age: 20  
})

访问数据：

console.log(user.name)

修改数据：

user.age++

与 `ref` 不同的是：

reactive 不需要 .value

---

# 四、ref 与 reactive 的区别

从宏观角度来看：

|特点|ref|reactive|
|---|---|---|
|数据类型|基本类型 + 对象|只能对象|
|访问方式|`.value`|直接访问|
|响应式原理|包装成 Ref 对象|Proxy 代理对象|

### 示例对比

#### ref

const count = ref(0)  
  
count.value++

#### reactive

const user = reactive({  
  name: "Tom"  
})  
  
user.name = "Jerry"

---

# 五、reactive 的一个注意点

如果 **直接给 reactive 重新赋值对象**，会失去响应式。

错误写法：

user = {  
  name: "Jack"  
}

这样 Vue 就无法追踪变化了。

正确写法：

Object.assign(user, {  
  name: "Jack"  
})

这样可以保持响应式。

---

# 六、使用原则

在实际开发中，一般遵循下面的规则：

### 1️⃣ 基本类型数据

必须使用 `ref`

例如：

number  
string  
boolean

示例：

const count = ref(0)

---

### 2️⃣ 对象类型（层级不深）

`ref` 或 `reactive` 都可以。

例如：

const user = ref({  
  name: "Tom",  
  age: 20  
})

或者

const user = reactive({  
  name: "Tom",  
  age: 20  
})

---

### 3️⃣ 对象层级较深

推荐使用 `reactive`

因为：

- 不需要 `.value`
    
- 操作更方便
    
- 代码更清晰
    

示例：

const user = reactive({  
  name: "Tom",  
  address: {  
    city: "Hangzhou",  
    street: "Xihu Road"  
  }  
})

---

# 七、总结

简单记忆可以用一句话：

> 基本类型用 **ref**，对象用 **reactive**。

完整理解：

- `ref`：适合 **基本数据类型**
    
- `reactive`：适合 **对象类型**
    
- `ref` 访问需要 `.value`
    
- `reactive` 直接访问