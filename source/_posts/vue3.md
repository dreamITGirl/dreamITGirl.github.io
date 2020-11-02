---
title: vue3的学习（未更新完）
date: 2020-11-02 10:41:10
tags:
- vue
- js
category: vue3系列文章
img: 2020/11/02/vue3/vue.png
---
欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我. 这篇文章适合有vue基础和js基础的人学习

[vue3.0](https://v3.vuejs.org/guide/installation.html)官方关于vue3的文档是英文的，大家可以慢慢阅读。
这里给大家提供一个中文版的[vue3.0](https://www.bookstack.cn/books/vue-3.0-zh)版本的文档,建议阅读英文.
也可以看视频学习，推荐一个视频[李江南vue3正式版的学习](https://space.bilibili.com/305684376/video)

## vue3的亮点
1.  Performance 性能比vue 2.x快1.2～2倍
2.  Tree shanking support 按需编译，体积比vue2.x更小
3.  Composition API（组合API）
4.  Better TypeScript support 更好的TS支持
5.  Cunstom Render API : 暴露了自定义渲染的API
6.  Fragment,Teleport(Protal),Suspense: 更先进的组件

## vue3 是如何变快的？

### diff算法优化

在vue2中的虚拟dom是进行全量对比的，也就是说，更新一处，所有的dom节点都会重新对比一遍。
在vue3中增加了静态标记，只需要进行比较flag的节点，并且可以通过flag的标志符判断更新的具体内容
```
<div>
    <p>测试</p>
    <p>{{msg}}</p>
</div>
// vue2x中：编译后的js代码
function anonymous() {
  with(this) {
    return _c('div', [_ssrNode("<p>测试</p> <p>" + _ssrEscape(_s(msg)) + "</p>")])
  }
}
// vue3x中：编译后的代码
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("p", null, "测试"),
    _createVNode("p", null, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}
```
我们可以看到在vue3中在渲染虚拟dom的时候就标记了一个flag用于后面更新比较
### 静态提升
vue2 中，无论创建的元素是否参与更新，每次都会被重新创建，然后再渲染
vue3 中，对于不参与更新的元素，会做静态提升，只会被创建一次，在渲染的时候直接复用
```
<div>
  <p>测试</p>
  <p>测试</p>
  <p>测试</p>
  <p>{{msg}}</p>
</div>

//静态提升之前：
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("p", null, "测试"),
    _createVNode("p", null, "测试"),
    _createVNode("p", null, "测试"),
    _createVNode("p", null, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}
//静态提升之后
const _hoisted_1 = /*#__PURE__*/_createVNode("p", null, "测试", -1 /* HOISTED */)
const _hoisted_2 = /*#__PURE__*/_createVNode("p", null, "测试", -1 /* HOISTED */)
const _hoisted_3 = /*#__PURE__*/_createVNode("p", null, "测试", -1 /* HOISTED */)

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", null, [
    _hoisted_1,
    _hoisted_2,
    _hoisted_3,
    _createVNode("p", null, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}
```
### cancleHandlers 事件侦听器缓存

默认情况下，onClick会被认定为动态绑定，每次都会追踪它的变化
但是因为是同一个函数，在没有追踪到变化时，直接缓存起来复用即可
```
<div>
  <button @click="clickFunc">点击</button>
</div>
// 开启事件侦听器缓存之前：
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("button", { onclick: _ctx.clickFunc }, "点击", 32 /* PROPS, HYDRATE_EVENTS */, ["onOnclick"])
  ]))
}
// 开启事件侦听器缓存之后：
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("button", {
      onOnclick: _cache[1] || (_cache[1] = (...args) => (_ctx.clickFunc(...args)))
    }, "点击", 32 /* HYDRATE_EVENTS */)
  ]))
}
```
**⚠️:在vue的diff算法中，只有静态标记的才会进行比较，才会进行追踪**




## vue3项目的创建
创建vue3的项目可以和vue2的方式一样，这里说的是vue3中新推出的vite的方式
### 什么是Vite
Vite 是一个由原生 ESM 驱动的 Web 开发构建工具。在开发环境下基于浏览器原生 ES imports 开发，在生产环境下基于 Rollup 打包。
它主要具有以下特点：
1. 快速的冷启动
2. 即时的模块热更新
3. 真正的按需编译
### 如何使用
```
npm install -g create-vite-app // 全局安装vite
create-vite-app vue3Project // 创建项目名为vue3Project的项目
cd vue3Project
npm i
npm run dev // 测试环境启动项目 
```
这里就不再仔细讲构建的项目中的目录结构了











## 组合API
### 1. v-for Array Refs
> In Vue 2, using the ref attribute inside v-for will populate the corresponding $refs property with an array of refs. This behavior becomes ambiguous and inefficient when there are nested v-fors present.
在vue2中，在v-for中使用的ref属性，会用ref数组填充相应的$refs的属性。如果遇到嵌套的v-for的时候，行动就会不明确且效率就会变得很低

>In Vue 3, such usage will no longer automatically create an array in $refs. To retrieve multiple refs from a single binding, bind ref to a function which provides more flexibility (this is a new feature):
在vue3中，这样使用不会在$refs中自动创建数组。要从单个绑定绑定获取多个ref时，需要将ref绑定到一个更灵活的函数上

**案例一**
```
<template>
  <div>
    <p>count:{{ count }}</p>
    <button @click="testFn">按钮</button>
  </div>
</template>

<script>
// ref函数注意点：
// ref 只能监听简单类型的数据变化，不能监听复杂类型的数据变化(对象/数组)
import {ref} from 'vue'
export default {
  name: 'App',
  // setup函数是组合API的入口函数
  setup(){
    let count = ref(0) //是一个对象
    // 在组合API中想定义方法，不用写在methods中，直接定义抛出就可以了
    function testFn() {
        count.value += 1
    }
    // 注意点：
    // 在组合API中定义的变量/方法，要想在外界使用，需要通过return{xxx,xxx}暴露出去
    return {count,testFn}
  }
}
</script>
```
**案例二**
```
<template>
    <div>
        <form>
            <label>
                id: <input type="text" v-model="stuForm.stu.id">
            </label>
            <label>
                name: <input type="text" v-model="stuForm.stu.name">
            </label>
            <label>
                age: <input type="text" v-model="stuForm.stu.age">
            </label>
            <button type="button" @click.stop.prevent="addStu"> 添加</button>
        </form>
        <h5>列表展示</h5>
        <p v-for="(item,index) in state.stus" :key="item.id" @click="removeStu(index)">
            {{item.name}} - {{item.age}}
        </p>
        
    </div>
</template>
<script>
import {reactive} from 'vue'
export default {
    name: 'App',
    setup(){
        /** 
         * let state = reactive({
            stus:[
                {id:1,name:'lucy',age:10},
                {id:2,name:'john',age:12},
                {id:3,name:'lucy',age:15},
                {id:4,name:'alice',age:14}
            ]
        })
        console.log('state',state)
        function removeStu(index) {
            state.stus = state.stus.filter((stu,ind) => ind !== index)
        }
        return {state,removeStu} 
        **/
        let {state,removeStu} = stuFunc()
        let stuForm = reactive({
            stu:{
                id:'',
                name:'',
                age:''
            }
        })
        function addStu() { 
            let stuObj = Object.assign({},stuForm.stu) 
            state.stus.push(stuObj) 
            stuForm.stu.id = ''
            stuForm.stu.name = ''
            stuForm.stu.age = ''
        }

        return {state,removeStu,stuForm,addStu}
    }
}

function stuFunc() {
    let state = reactive({
        stus:[
            {id:1,name:'lucy',age:10},
            {id:2,name:'john',age:12},
            {id:3,name:'lucy',age:15},
            {id:4,name:'alice',age:14}
        ]
    })
    
    function removeStu(index) {
        state.stus = state.stus.filter((stu,ind) => ind !== index)
    }
    return {state,removeStu}
}
</script>
```

这两个案例，讲了如何使用组合API，以及我们可以将我们的函数封装成一个js文件，通过export的方式引入；这样在开发中，使得我们的代码更简洁，更便于维护

**Composition API(组合API/注入API)的本质**
将setup中定义的函数或变量，注入到Option API 中的methods或者data属性中

**setup函数**
1. 执行时机：
   在beforeCreate 和 Created 生命周期之间执行 
2. 注意点
   - 由于在执行setup函数时，组件中的data和methods还没有被创建好，所以无法使用data/mehtods中的属性的
   - setup函数中的this被修改为了undefined
   - setup函数只能是同步的不能是异步的


### reactive的理解
### ref的理解
### ref和reactive的区别

## 附录 （vue PatchFlags 标识符）
```
export const enum PatchFlags {
  // 动态文字内容
  TEXT = 1,

  // 动态 class
  CLASS = 1 << 1, // 2

  // 动态样式
  STYLE = 1 << 2, // 4

  // 动态 props
  PROPS = 1 << 3, // 8

  // 有动态的key，也就是说props对象的key不是确定的
  FULL_PROPS = 1 << 4, // 16

  // 合并事件
  HYDRATE_EVENTS = 1 << 5, // 32

  // children 顺序确定的 fragment
  STABLE_FRAGMENT = 1 << 6, // 64

  // children中有带有key的节点的fragment
  KEYED_FRAGMENT = 1 << 7, // 128

  // 没有key的children的fragment
  UNKEYED_FRAGMENT = 1 << 8, // 256

  // 只有非props需要patch的，比如`ref`
  NEED_PATCH = 1 << 9, // 512

  // 动态的插槽
  DYNAMIC_SLOTS = 1 << 10, // 1024

  // SPECIAL FLAGS -------------------------------------------------------------

  // 以下是特殊的flag，不会在优化中被用到，是内置的特殊flag

  // 表示他是静态节点，他的内容永远不会改变，对于hydrate的过程中，不会需要再对其子节点进行diff
  HOISTED = -1,

  // 用来表示一个节点的diff应该结束
  BAIL = -2,
}
```
