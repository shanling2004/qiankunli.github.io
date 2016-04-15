---

layout: post
title: 以新的角度看数据结构
category: 技术
tags: Algorithm
keywords: 数据结构

---


## 前言（未完待续） ##



## 队列

教科书上一说队列，就是四个字“先进先出”

### 生产者消费者模式，以及这个模式的意义（在多线程领域）

### 轮询的一种实现

假设我们有一个集合，对集合中的元素实现轮询效果。

1. 我们可以标记一个index，记住上一次使用的元素，进而实现轮询。
2. 集合最开始就是用队列存储，即天然具备轮询效果。

这两种方式，在多线程环境下，还需注意线程安全问题。

## 图

深度优先和广度优先遍历，{初始状态，目标状态，规则集合}在寻找最佳策略上的应用
