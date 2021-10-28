---
title: vue3学习二
date: 2021-10-25 14:25:34
cover: true
toc: false
tags:
- vue
- js
category: vue
img: https://i.loli.net/2021/10/23/AG8oT9q1pMCj2Z5.jpg
summary: 本篇文章主要讲Vue3.x版本的响应性基础及原理。适合有vue基础和js基础的人学习
---
这篇文章适合有vue基础和js基础的人学习[vue3.0](https://v3.vuejs.org/guide/installation.html)官方关于vue3的文档是英文的，大家可以慢慢阅读。这里给大家提供一个中文版的[vue3.0](https://www.bookstack.cn/books/vue-3.0-zh)版本的文档,建议阅读英文.也可以看视频学习，推荐一个视频[李江南vue3正式版的学习](https://space.bilibili.com/305684376/video)
欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.

#### Vue3.x中的响应性 

##### Vue是如何知道哪些代码在运行的？

> Vue 通过一个副作用(effect) 来跟踪当前正在运行的函数。副作用是一个函数的包裹器，在函数被调用之前就启动跟踪。Vue知道哪个副作用在何时运行，并能再次执行它

