---
layout: post
title: "Vue 模板只追踪原来的 ref 对象，为什么？"
subtitle: "深入解析 Vue3 响应式系统的依赖收集原理"
date: 2026-03-11
author: "b0t"
header-img: "img/post-bg-vue.jpg"
catalog: true
tags:
  - Vue3
  - 前端开发
  - 响应式原理
---
这个问题其实涉及 **Vue3 响应式系统的核心原理：依赖收集（dependency tracking）**。我按 **真实执行流程**给你讲一遍，你就会彻底明白为什么。

---

# 一、Vue 是怎么“追踪变量”的

Vue 并不是监听 **变量名**，而是监听 **对象的属性访问**。

Vue3 的响应式核心是：

Proxy / getter / setter

当模板使用数据时，会发生一件事：

读取数据 → Vue记录依赖

---

# 二、看一个例子

const age = ref(20)

实际创建的是一个对象：

age = {  
  value: 20  
}

---

# 三、模板执行时发生了什么

模板：

{{ age }}

Vue 编译后其实是：

age.value

此时 Vue 会执行：

读取 age.value

这一步会触发 **getter**

Vue就会做一件事：

记录：当前组件依赖 age.value

内部大概是这样：

依赖表  
  
target(age对象)  
   └── key: value  
           └── effect(当前组件渲染函数)

图示：

age对象  
   │  
   └─ value  
        │  
        └─ 组件render函数

---

# 四、当数据改变

如果你写：

age.value = 99

会触发：

setter

Vue就会：

找到依赖 age.value 的组件  
→ 重新渲染

页面更新。

---

# 五、关键问题来了

如果你写：

age = ref(99)

发生什么？

其实只是：

变量指向改变

原来：

age ───► 对象A { value:20 }

现在：

age ───► 对象B { value:99 }

但是 Vue 的依赖表是：

对象A.value → render函数

Vue **只知道对象A**。

对象B它完全不知道。

---

# 六、所以页面不会更新

图示：

### Vue依赖关系

对象A.value  
     │  
     └─ render

### 你做的事情

age → 对象B

但是 Vue 仍然监听：

对象A

---

# 七、核心原因总结

Vue追踪的是：

对象 + 属性

不是：

变量名

所以：

|操作|Vue是否能追踪|
|---|---|
|age.value = 99|✅|
|age = ref(99)|❌|

---

# 八、为什么 Vue 不追踪变量名

因为 **JavaScript 本身无法监听变量赋值**。

JS没有类似：

watch(age)

的能力。

唯一能监听的是：

对象属性

所以 Vue 才设计：

ref.value

---

# 九、理解到这里其实就掌握 Vue3 响应式原理了

Vue3本质是：

读取属性 → 收集依赖  
修改属性 → 触发更新

流程：

render()  
   ↓  
读取 age.value  
   ↓  
track(age,value)  
   ↓  
age.value = 99  
   ↓  
trigger(age,value)  
   ↓  
重新render