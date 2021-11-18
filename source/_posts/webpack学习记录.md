---
title: webpack学习记录(基础入门篇)
date: 2021-11-02 15:47:33
img: https://i.loli.net/2021/10/23/ATlKvnwmgZ1YS78.jpg
tags:
- webpack
category:
- 构建工具
keywords: webpack的学习；适用于对webpack不了解想要学习的前端同学。内容比较基础
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我.
---

#### webpack 核心概念

##### 入口(entry)
入口起点(entry point) 指示webpack 应该使用哪个模块，作为项目的入口。一般情况下是`./src/index.js` ,我们也可以直接在webpakck.configuration 中配置 `entry` 的属性。来指定项目一个或多个入口起点

用法: `entry:{ <entryChunkName> string | [string]} | {}`

案例一：

```
modules.export = {
    entry:'./src/main.js'
}
```
案例二：
```
modules.export = {
    entry:{
        a:'./src/a.html'，
        b:'./src/b.html'
    }
}
```

###### 描述入口的对象

- `dependon` : 当前入口所依赖的入口，必须在加载入口文件之前加载
- `filename` : 指定要输出的文件名称
- `import` : 启动时需要加载的模块
- `library` : 指定library选项，为当前entry 构建一个library
- `runtime` : 运行时chunk 的名字，如果设置了 就会创建一个新的运行时chunk.在webpack5.43.0之后可将其设置为false,以避免一个新的运行时的chunk
- `publicPath` : 当该入口的输出文件在浏览器中被引用时，为它们指定一个公共的URL地址

**注意：**
1. `runtime` 和 `dependon` 不应该出现在同一个入口上使用。
2. 确保 `runtime` 不能指向已存在的入口名称，
```
modules.export = {
    entry:{
        a:'./a',
        b:{
            runtime:'x',
            dependon:'a', // 报错 本次配置无效
            import :'./b1'
        }
    }
}
```
3. `dependOn` 不能是循环引用的
```
modules.export = {
    entry:{
        a:{
            import :'./a2',
            dependon:'b'
        },
        b:{
            runtime:'x',
            dependon:'a', // 报错 本次配置无效
            import :'./b1'
        }
    }
}
```







##### 出口(output)
