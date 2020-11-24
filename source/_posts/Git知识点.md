---
title: Git知识点
date: 2020-11-24 15:38:04
tags:
- Git
category: Git
keywords: Git知识点
summary: 欢迎大家访问我的博客，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在留言板留言问我.
---
欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.

#### 初次使用Git

###### Git的全局配置
```
git config --global user.name '用户名'
git comfig --global user.email '邮箱'
```
**注意点：**
1. 用户名需要是英文的，不可以是中文
2. 输入配置完成后，可输入`git list`查看自己刚才的配置。

###### Git理论

1. Git保存的是什么？

   - SVN保存的是每次版本更新的内容
   - Git则是将每个版本都独立保存

   工作区域、暂存区域、Git仓库

2. Git的工作流程：
    - 在工作目录中添加、修改文件
    - 将需要进行版本管理的文件放在暂存区域
    - 将暂存区域的文件提交到Git仓库

3. Git管理的文件有三种状态：
    - 已修改(modified)
    - 已暂存(staged)
    - 已提交(committed)

###### Git相关命令行

1. 创建Git库

```
git init // 初始化仓库
git clone '库的地址'  // 克隆远程库
```
