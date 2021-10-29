---
title: vue3学习二(深入理解响应性基础)
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

### Vue3.x中的响应性 

#### Vue是如何知道哪些代码在运行的？

> Vue 通过一个副作用(effect) 来跟踪当前正在运行的函数。副作用是一个函数的包裹器，在函数被调用之前就启动跟踪。Vue知道哪个副作用在何时运行，并能再次执行它

```
let name = 'Alice' , age = 20 
let mySelfInfo = `${name}今年${age}岁了`

console.log(mySelfInfo) // Alice今年20岁了

age = 30 // 我们预期接下来输出的是：Alice今年30岁了;

console.log(mySelfInfo) // Alice今年20岁了;
```
如果我们想要让第二次输出的`mySelfInfo`中的age发生变化，就需要再次执行一下拼接的`mySelfInfo`
```
let name = 'Alice' , age = 20 
let mySelfInfo = `${name}今年${age}岁了`

console.log(mySelfInfo) // Alice今年20岁了

age = 30

mySelfInfo = `${name}今年${age}岁了`

console.log(mySelfInfo) // Alice今年30岁了
```
##### effect

在Vue2 中，通过副作用`effect`来追踪当前正常运行的函数。接下来对上述代码进行改进;
*案例1*

```
let name = 'Alice' , age = 20 
let mySelfInfo = ''
const effectInfo = () => mySelfInfo = `${name}今年${age}岁了`

effectInfo()
console.log(mySelfInfo) // Alice今年20岁了

age = 30

effectInfo()

console.log(mySelfInfo) // Alice今年30岁了
```
案例1的这个例子，每次更新了`age`,都要执行一下effect的方法；在Vue3中是设置了一个执行副作用的栈，然后当副作用被调用时，在调用`effectInfo`这个函数之前，会将自身推到一个数组中，这个数组可以用来检查当前正在运行的副作用
```
// 维持一个执行副作用的栈
const runningEffects = []

const createEffect = fn => {
  // 将传来的 fn 包裹在一个副作用函数中
  const effect = () => {
    runningEffects.push(effect)
    effectInfo()
    runningEffects.pop()
  }

  // 立即自动执行副作用
  effect()
}
```


#### Vue如何追踪变化

> Vue3 中使用了 Proxy，将对象包裹在一个带有`get`和`set`处理程序的 `Proxy` 中。


Proxy是ES6的新特性，该博客也有对该API的介绍,大家可以简单看一下。[proxy的理解](https://dreamitgirl.github.io/2020/11/03/proxy/)和[proxy的应用场景](https://dreamitgirl.github.io/2020/11/17/proxy-de-ying-yong-chang-jing/)。
 
> 在使用proxy的一个难点是`this`的问题。我们希望任何方法都绑定在`Proxy`，而不是目标对象。这样我们就可以拦截他们。在ES6中有一个新特性`Reflect`，它允许我们一最小的代价消除这个问题

```
const dinner = {
  meal: 'apple'
}

const handler = {
  get(target, property, receiver) {
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal) // apple
```

使用`Proxy`实现响应性的第一步就是跟踪一个属性何时被读取，在`track`的处理器函数中执行此操作。这个函数可以传入`target` 和 `property`两个参数

```
const dinner = {
  meal: 'apple'
}

const handler = {
  get(target, property, receiver) {
    track(target, property)
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal) // apple
```
`track`将检查当前运行的是哪个副作用，并将其与 target 和 property 记录在一起。这就是Vue能知道`property`是该副作用的依赖项
重新修改一下`handle`函数
```
const dinner = {
  meal: 'apple'
}

const handler = {
  get(target, property, receiver) {
    track(target, property)
    return Reflect.get(...arguments)
  },
  set(target,property,value,receiver){
      trigger(target,property)
      return Reflect.set(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal) // apple
```
通过上述的例子，我们可以对Vue如果追踪属性的变化有了了解
> 1. 当一个值被读取时进行追踪：`proxy`中的`get`处理函数中，`track`函数记录了该`property`和当前副作用
2. 当某个值改变时进行检测: 在`proxy`上调用`set`处理函数
3. 重新运行代码来读取原始值 `trigger`函数查找哪些副作用依赖于该`propery`并执行。

我们可以通过重写一个组件来写一个例子

```
const vm = createApp({
    data(){
        return {
            val1:1,
            val2:2
        }
    },
    computed:{
        sum(){
            return this.val1 + this.val2
        }
    }
}).mount('#app')

console.log(vm.sum) // 3
vm.val1 = 4
console.log(vm.sum) // 6
```

