---
title: vue3的学习1
date: 2020-11-02 10:41:10
cover: true
toc: false
tags:
- vue
- js
category: vue
img: 2020/11/02/vue3/vue.png
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我.
---
欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.
这篇文章适合有vue基础和js基础的人学习[vue3.0](https://v3.vuejs.org/guide/installation.html)官方关于vue3的文档是英文的，大家可以慢慢阅读。这里给大家提供一个中文版的[vue3.0](https://www.bookstack.cn/books/vue-3.0-zh)版本的文档,建议阅读英文.也可以看视频学习，推荐一个视频[李江南vue3正式版的学习](https://space.bilibili.com/305684376/video)

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

### v-for Array Refs

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
- 执行时机：
   在beforeCreate 和 Created 生命周期之间执行 
- 注意点
   - 由于在执行setup函数时，组件中的data和methods还没有被创建好，所以无法使用data/mehtods中的属性的
   - setup函数中的this被修改为了undefined
   - setup函数只能是同步的不能是异步的


## Reactivity API 

The Reactivity API contains the following sections:
1. Basic Reactivity APIs
2. Refs
3. Computed and watch

### reactive的理解
  >Returns a reactive copy of the object.
    The reactive conversion is "deep"—it affects all nested properties. In the ES2015 Proxy based implementation, the returned proxy is not equal to the original object. It is recommended to work exclusively with the reactive proxy and avoid relying on the original object.

- reactive 是vue3中提供的实现响应式数据的方法
    - 在vue2中，通过defineProperty的属性来监听数据的变化
    - vue3中的响应式数据通过ES6的[proxy的学习](https://dreamitgirl.github.io/2020/11/03/proxy/) 来实现的.
- reactive的注意点
    - reactive的参数必须是Object(json/Array) 
    - 如果给reactive传了其他的对象，默认情况下，不会更新视图界面。如果想要更新，需要重新赋值

  **案例**
  ```
<template>
    <div>
        <p>{{state}}</p> 
        <p>{{state.time}}</p>
        <button @click="changeVal">按钮</button>
    </div>
</template>
<script>
import { reactive } from 'vue'
export default {
    name:'App',
    setup(){
      // 当给reactive传的参数是其他类型的时候，页面上是可以正常显示的。但是控制台会有一句警告
      // vue.js:948 value cannot be made reactive: 2 
      let state = reactive(2)
      function changeVal() {
          state = 34
          // 输出的state的值改变了，但是视图并未更新，说明当传递给reactive的参数不是对象的时候，vue无法监听到数据的变化
          console.log(state) // 34
      }
      return {state,changeVal} 
     // 当传给reactive的参数是一个数组的时候，修改数据的值，视图中的数据是可以变化的
        let state = reactive([1,2,3])
        function changeVal() {
            state[2] = 666
        }
        return {state,changeVal} 
    // 当传递的参数是不是一个对象时
    let state = reactive({
        time:new Date()
    })
    function changeVal() {
        // 直接修改的话，视图没有发生改变 , 可以通过重新赋值的方式 
        // state.time.setDate(state.time.getDate() + 1)
        // 将新值重新赋值给state.time,视图会跟着变化
        const time2 = new Date(state.time.getTime())
        state.time = new Date(time2.setDate(state.time.getDate() + 1))
        console.log(state.time) // App.vue:38 Wed Nov 04 2020 10:09:02 GMT+0800 (中国标准时间)
    }
    return {state,changeVal} 
    }
}
</script>
```

### ref的理解
  >Takes an inner value and returns a reactive and mutable ref object. The ref object has a single property .value that points to the inner value.

  - 从官网上的解释我们可以知道，ref也是用来实现响应式数据的，它的参数不是对象，是简单值的监听
  - ref的本质仍然是返回reactive，只不过在系统会自动将我们传的参数，改写成以下这种形式
   reactive({
     value: xx
   })
  - ref的注意点：
  在js中只能通过value的形式更新或者获取值
  在vue的template中，可以直接使用变量获取，系统会默认加上value属性

  **案例**
  ```
<template>
    <div>
        <p>{{count}}</p>
        <button @click="changeVal">按钮</button>
    </div>
</template>
<script>
import {ref} from 'vue'
export default {
    name:'App',
    setup(){
        let count = ref(2)
        console.log(count) //RefImpl {_rawValue: 2, _shallow: false, __v_isRef: true, _value: 2}
        function changeVal() {
            // count 的本质是一个对象，直接将count赋值为3，是无法使视图改变的
            // count = 3
            //  console.log(count) // 3
            count.value = 3
            console.log(count) 
            // RefImpl {_rawValue: 3, _shallow: false, __v_isRef: true, _value: 3
        }

        return {count,changeVal}
    }
}
</script>
```

### ref和reactive的区别
- 在template中使用的数据类型如果是ref类型，Vue会自动帮我们添加.value
- 在template中使用的数据类型如果是reactive类型，Vue不会自动帮我们添加.value

- Vue是如何决定是否需要自动添加.value?
  Vue在解析数据之前，会自动判断这个数据是ref类型还是reactive类型

- Vue是根据什么属性判断的
通过当前数据的__v_ref属性判断的。这是vue的私有属性，如果这个属性存在且值为true，则当前数据是一个ref

- Vue中提供了两个API，isReactive 和 isRef 来判断数据是ref还是reactive；
```
let count = ref(0)
console.log(isRef(count)) // true
console.log(isReactive(count)) // false
```



### 递归监听和非递归监听
  - 默认情况下，ref 和 reactive都是递归监听的。如果数据量大且结构复杂的话，会耗费性能，
    所以，vue3引入了shallowReactive 和 shallowRef 两个API
  - 先举个例子说一下，ref和reactive 是如何进行递归监听的
  **案例一**
  下面的实例告诉我们reactive通过递归监听，每次输出的结果都是一个Proxy对象；在数据量很大的情况下这种会耗费性能
  ```
<template>
    <div>
        <p>{{state.a}}</p>
        <p>{{state.b.c}}</p>
        <p>{{state.b.d.s}}</p>
        <button @click="changeVal">按钮</button>
    </div>
</template>
<script>
import { reactive } from 'vue'
export default {
    name:'App',
    setup(){
        let state = reactive({
            a:'a',
            b:{
                c:'c',
                d:{
                    s:'d'
                }
            }
        })
        function changeVal() {
            state.a = '1'
            state.b.c = '2'
            state.b.d.s = '3'
            console.log(state) // Proxy {a: "1", b: {…}}
            console.log(state.b) // Proxy {c: "2", d: {…}}
            console.log(state.b.d) // Proxy {s: "3"}
        }
        return {state , changeVal}
    }
}
</script>
  ```

  **案例二 shallowReactive**
  ```
<template>
    <div>
        <p>{{state.a}}</p>
        <p>{{state.b.c}}</p>
        <p>{{state.b.d.s}}</p>
        <button @click="changeVal">按钮</button>
    </div>
</template>
<script>
import { shallowReactive } from 'vue'
export default {
    name:'App',
    setup(){
    let state = shallowReactive({
            a:'a',
            b:{
                c:'c',
                d:{
                    s:'d'
                }
            }
        })
        function changeVal() {
            //state.a = 1
            console.log(state) //  Proxy {a: 1, b: {…}}
            console.log(state.b) // {c: 2, d: {…}}
            console.log(state.b.d) //  {s: 3}
            // 这种情况下只有state是Proxy的对象
            // 如果我不修改state.a,直接修改state.b.c,会不会监听到变化，视图随之改变呢？
            state.b.c = 2
            // 可以自己写一个demo试一下。结果是不会改变。
        
        }
        return {state , changeVal}  

    }
}
</script>
  ```

  **案例三 shallowRef 和 triggerRef**
  ```
  let state = shallowRef({
    a:'a',
    b:{
        c:'c',
        d:{
            s:'d'
        }
    }
  })
  function changeVal() {
      // 这样直接改值的话，视图不会更新的
       state.value.a = 2 
      // 正确方式：
       state.value = {
           a:1,
           b:{
               c:2,
               d:{
                   s:3
               }
           }
       }
      // 如果只改动state.b.d中的值，要如何实现呢？
      // 这里引入了triggerRef的API
      /** 
      state.value.b.c = 5
      triggerRef(state) 
      **/
  }
  ```

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
