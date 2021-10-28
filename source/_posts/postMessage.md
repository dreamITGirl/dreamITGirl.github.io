---
title: 关于新开页面的传值问题
date: 2021-05-20 18:31:54
tags: 
- H5
- js
category:
- JavaScript
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我
---
欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.

### 新开浏览器页签的传值问题

#### 1. 业务背景
在新建/修改/详情等二级或者三级新打开的页签中，需要实现传递参数的功能，并同时刷新上一级页面

#### 2. 思考方式：

1. 使用 `localStorage` 的方式
这种方式的最大的弊端是当项目较大的时候存储的信息量极易超过浏览器中要求的存储空间。所以我们在第一次迭代时放弃了这种方式。
2. 使用`window.postMessage`
这种方式在列表页面中打开一个新建页面时是没有问题的，但是当打开多个新建页面，这个传的值就会发生问题。导致信息接收失败。在调研这个方式后也放弃了这种思路
3. 自封装插件
思路：采用了JS打开页面的初始化事件，利用H5的新特性`history`来实现的。

#### 3. 代码解析
主要是利用了`history`中的`pushState`和 `replaceState`这两个API，通过监听页面的渲染顺序，将函数设计成发布/订阅者模式，从而达到想要的效果

```
function parseConfig (config) { // 将传递的参数String化
    let str = ''
    for (let key in config) {
        if (config[key] !== null) {
            str+=`${key}=${config[key]},`
        }
    }
    return str
}

const _historyWrap = function(type) { 
    const orig = history[type];
    const e = new Event(type);
    return function() {
        const rv = orig.apply(this, arguments);
        e.arguments = arguments;
        window.dispatchEvent(e); // 向一个指定的事件目标派发一个事件,  并以合适的顺序同步调用目标元素相关的事件处理函数
        return rv;
    }
};

history.pushState = _historyWrap('pushState');
history.replaceState = _historyWrap('replaceState');

class Win {
    constructor (url, config) {
        config = config || {}
        if (!url) {
            throw Error('url is not defined!')
        }
        let name = '_'+Date.now() // 确保name的唯一性
        this.win = window.open(url, name, parseConfig(config))
        window.addEventListener('beforeunload', ()=>{this.win.close()}) // 监听window，document 及其资源即将被载。
        window.addEventListener('pushState', ()=>{this.win.close()}); // history在会话历史堆栈顶部插入一条记录
        window.addEventListener('replaceState', ()=>{this.win.close()}); // history更新会话历史堆栈顶部记录信息
        window.addEventListener('popstate', ()=>{this.win.close()}); //history在会话历史堆栈顶部弹出一条记录
        if (this.win === null) {
            alert('请将“弹出式窗口和重定向”设置为允许')
            throw Error('新窗口创建失败，窗口被拦截！')
        }
        return this
    }
    on(eventName, cb) {
        if (eventName === 'load') {
            this.win.onload = (ev) => {
                cb(ev, this)
            }
        }
        if (eventName === 'message') {
            let _this = this
            function closeListener() {
                setTimeout(()=>{
                    _this.win.send = function(message){
                        cb(message)
                    }
                    _this.win.addEventListener('unload', closeListener)
                })
            }
            this.win.addEventListener('unload', closeListener)
        }
    }
    close(cb) {
        cb && cb()
        window.name = 'Win_'+Date.now()
        this.win.open('',window.name)
        window.name = ''
        this.win.close()
    }
}
```

#### 4. 知识点链接

* [history MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/History)
* [history API ](https://juejin.cn/post/6844903602641862663#heading-17)
* [前端路由实现原理](https://juejin.cn/post/6844903634954633224)


   


    
