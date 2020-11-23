---
title: proxy的应用场景
date: 2020-11-17 17:14:55
cover: true
tags:
- js
- ES6
- vue
category: ES6系列
img: 2020/11/16/promise-de-xue-xi/es6.jpeg
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我.
---
欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.

**写在前面**
随着Vue3越来越被大家学习，关于Proxy的使用也是不断收到关注。`Proxy`和`Object.defineProperty`的对比也备受关注。这篇文章适合有vue开发经验的同学阅读。

#### 回顾`Object.defineProperty`

在vue2.x版本中，是通过`Object.defineProperty`进行双向数据绑定的，接下来看一下`Object.defineProperty`是如何进行双向绑定的。接下来简单实现一下。
```
function observe(obj) {
    if (typeof obj === 'object' ) {
        for (const key in obj) {
            defineReactive(obj,key,obj[key])
        }
    }
}
function defineReactive(obj,key,value) { 
    // 针对value是对象，进行递归
    observe(value)
    Object.defineProperty(obj,key,{
        get:function () {
            console.log(`获取${key}:value是${value}`)
            return value
        },
        set:function (v) {
            observe(v)
            console.log(`${key}数据改变了`)
            value = v
        }
    })
}

let obj = {  
    name: '守候',  
    flag: {  
        book: {  
            name: 'js',  
            page: 325  
        },  
        interest: ['火锅', '旅游'],  
    }  
}
observe(obj)
```

![测试结果](https://i.loli.net/2020/11/23/tLFbcE46hAfMIXw.png)

**问题一：`Object.defineProperty`无法监听增加或删除的属性**

我们知道`Object.defineProperty`无法监听增加或者删除的属性，
在我们使用vue2.x中，增加属性我们一般会通过`$set`的方法进行操作，`$set`内部也是通过使用`Object.defineProperty`实现的

![测试结果1](https://i.loli.net/2020/11/23/tVkfeP6QoA8b1C4.png)

**问题二：`Object.defineProperty`无法监听到数组的变化**

![测试结果2](https://i.loli.net/2020/11/23/EaUgiGTpwh8H3vA.png)

从上面的图片中我们可以看到修改数组中的值是可以的，但是不能被监听到，输出的interest数组还是改之前的数组

**问题三：如果对象的层级很多的情况下，不断的递归，会导致时间过长，影响性能**


#### 回顾`Proxy`

vue3的版本中，采用了`Proxy`的方式。MDN 中的定义如下：
>对象用于定义基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）

简单来说就是，对目标对象设置一层拦截，任何对这个对象的操作都会经常这层拦截。这就相当于Axios的拦截器。

接下来写个案例，理解一下Proxy

```
function observerProxy(obj) {  
    let handler = {  
        get(target, key, receiver) {  
            console.log('获取：' + key) // 如果是对象，就递归添加 proxy 拦截  
            if (typeof target[key] === 'object' && target[key] !== null) {  
                return new Proxy(target[key], handler)  
            }  
            return Reflect.get(target, key, receiver)  
        },  
        set(target, key, value, receiver) {  
            console.log(key + "-数据改变了") 
            return Reflect.set(target, key, value, receiver)  
        }  
    }  
    return new Proxy(obj, handler)  
}  
let obj = {  
    name: '守候',  
    flag: {  
        book: {  
            name: 'js',  
            page: 325  
        },  
        interest: ['火锅', '旅游'],  
    }  
}  
let objTest = observerProxy(obj)  
```
![Proxy输出结果](https://i.loli.net/2020/11/23/TJg1GciB2FuPE9x.png)

操作数组的时候也会监听到

![修改数组](https://i.loli.net/2020/11/23/3hUBpJHeKoX6vbl.png)

操作对象属性中是对象的时候也会监听到

![修改对象](https://i.loli.net/2020/11/23/CqVv39JjmSt2MoE.png)

#### 比较两者的区别
通过上面的代码，我们不难看出来它们的区别，`proxy`还是很强大的，有11种方法。能够处理`Object.defineProperty`这个方法中无法处理的。

#### Proxy的处理场景
##### 负索引数组
##### 表单校验
##### 增加附属属性
##### 数据格式化
##### 负索引数组

