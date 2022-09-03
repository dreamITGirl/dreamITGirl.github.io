---
title: el-form子组件表单校验无法传给父组件
date: 2022-04-26 10:12:44
tags:
- Project experience
- JS 
category: Project experience
keywords: 基于elementUI的表单校验
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我.
---
这篇博客主要记录这次在项目开发中，开发的公共组件无法将表单校验结果传给父组件

#### 问题原因

在父组件中引用封装的公共表单组件无法通过`this.$refs[formName].validate()`事件拿到返回的数据。也从网上尝试了各种方案，发现`validate`方法只是走到了`Pending`状态。`Promise` 的对象并没有返回结果.这是开始检查我之前自定义的校验规则。

#### 解决方法

**在校验规则中发现有一种情况没有执行callbac**k，所以导致表单校验一直卡在了`Pending`状态,没办法继续向下走。

