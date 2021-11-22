---
title: vue进阶面试题总结
date: 2021/11/18 13:06:53
tags: [前端]
---

## vue2中数据响应式原理

循环dom，当使用到这些属性时，进行依赖收集(组件watcher)。核心使用`object.defineProperty`重新定义属性，使得属性可监控。当数据发生变化，利用发布订阅模式更新相关watcher

## vue2如何检测数组变化

vue重写了data中数组方法，实现和`object.defineProperty`(只监听对象)一样的检测效果

## nextTick介绍和实现原理

主要特性是，在dom更新后延迟一段时间后回调函数，这个回调函数就是nextTick，可以来做dom更新完成后一些逻辑。原理是利用事件循环(event loop)中的微任务和宏任务(根据环境采用某一种方式)

## 组件中的data为什么是一个函数

组件属于一个实例，如果date是对象，引用类型会影响其他实例，为了保证不同组件的date不冲突。

## v-model原理

是:bind="value"与[@change](https://github.com/change)="value = $event.target.value"的结合

## vue事件绑定原理

原生事件通过addEventListener绑定到目标元素，自定义事件通过vue的$on实现的

## diff算法(2和3的区别)

一句话概括就是，vue在对新老dom进行更新采用的策略(比较 + 更新)算法

-  同级别比较，在比较子节点
- 一方有子节点，另外一方没有子节点(直接增加或者移除)
- 如果都有子节点。那么递归比较子节点(触发diff)

diff概括：

vue2中的diff采用双端比较，从新旧dom的前后开始，头头、尾尾、头尾、尾头。其中可以通过key实现可复用节点

vue3相比较vue2的diff实现，在创建vNode时候就确定类型，在patch的过程中采用位运算来判断类型，还使用动态规划求解最长递归子序列

