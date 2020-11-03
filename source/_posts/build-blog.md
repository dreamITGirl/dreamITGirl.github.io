---
title: 搭建个人博客（hexo框架为例）
date: 2020-10-30 14:18:24
tags:
- hexo
cover: true
keywords: hexo搭建个人博客
img: 2020/10/30/build-blog/build1.png
category: 
- 框架学习
summary: 欢迎大家访问我的博客[dreamITGirl](https://github.com/dreamITGirl)，不要吝啬你们的小星星，点个star～ 有问题的话，你可以将问题在 [GitHub](https://github.com/dreamITGirl/dreamITGirl.github.io/issues)问我.
---
这篇文章适合有前端基础及会使用git及gitHub的人阅读.[hexo](https://hexo.bootcss.com/)是快速、简洁且高效的博客框架，适合新手。如果平时用vue开发的话，可以使用[vuePress](https://www.vuepress.cn/);如果使用react开发的话，可使用[Gatsby](https://github.com/destinytaoer/gatsby-start)；如果喜欢自己造轮子的话，也可以自己从0搭建。

## 安装

### 安装博客框架Hexo
代码中的blog指的是项目的名称，可以更换成自己的项目名称

```
npm install hexo-cli -g 
hexo init blog
cd blog
npm install
hexo server
```
### 配置主题
如果喜欢系统默认的博客主题的话，可以不修改主题；如果不喜欢的话，可以去[github](https://github.com/)官网中
搜索**hexo-theme**，选择自己喜欢的主题；
这里以[blinkfox/hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md)为例

```
git clone https://github.com/blinkfox/hexo-theme-matery.git 
```
2. 将克隆到本地的主题移动到hexo根目录themes目录中，将博客中的其他主题删掉
3. 删除下载的主题中的.git目录，这样做的目的是为了让下载后的主题和我们的博客放在同一个git地址下
4. 修改Hexo根目录下的_config.yml中的theme，将它改成 hexo-theme-matery

###  Hexo根目录下的_config.yml
Hexo根目录下的_config.yml中可以设置博客站点的标题、副标题、描述、作者、语言等

## 上传到github

### 创建库

1. 命名博客的方式

    命名方式一：https://[username].github.io
    仓库名必须是[username].github.io,将打包的版本放在master
    命名方式二: https://[username].github.io/[repo]
    这种方式可以自定义库名，打包的版本放在gh-pages(对应的库名)
    第一种方式更适合用来部署博客，第二种方式更适合部署开源项目或者作品展示等

2. 创建库
    创建库的方式如图所示
    {% asset_link build1.png 点击查看图片 %}
    创建成功后，我们需要将本地的代码传到我们创建的库，我习惯使用命令行的方式，把命令写在下面，需要的话自取
    
    ```
    git init 
    git add remote '创建成功后提示的路径'
    ```
    这两行代码是初始化git项目，并给项目添加远程仓库
    在使用git管理代码之前，先安装hexo-deployer-git依赖，这个库会帮助我们将生成好的代码放到具体的分支上，
    安装好后，打开**_config.yml**文件，找到deploy,设置对应内容
    ```
    deploy:
        type: git
        repo: https://github.com/dreamITGirl/dreamITGirl.github.io
        branch: master
    ```

   接下来运行
    ```
    npm run deploy 
    // 或者运行
    hexo deploy

    ```
    部署代码；代码提交完成后，我们就可以在远程仓库中看到我们提交的代码了。我们提交的master的代码就是打包后public目录下的文件。
    然后找到这个远程库的setting，找到GitHub Pages,这里面显示了我们的站点已经部署到了我们命名的仓库地址上了，点击就可以看到我们的博客了
    到这里为止，我们就完成了一个最基本的博客的部署

    ### 自动化部署
    
    在做自动化部署之前，我们需要将我们的源代码提交到远程仓库,master分支现在已经被占用了，我们需要将源代码提交到另一个分支上

    ```
    git add .
    git commit -m '提交源代码'
    git branch my-blog // my-blog是分支名称
    git push --set-upstream origin my-blog

    ```
    我们也可以在仓库首页放上我们的博客地址，便于访问

    接下来就是自动化部署了，这里用到github中的actions,我们这里会实现项目的代码部署和自动打包
    1. 在根目录创建.github文件夹及子文件夹workflows
    2. 创建deploy.yml文件
    3. deploy.yml代码如下

    ```
name: Build and Deploy
on: [push]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 🛎️
        uses: actions/checkout@v2 # If you're using actions/checkout@v2 you must set persist-credentials to false in most cases for the deployment to work correctly.
        with:
          persist-credentials: false

      - name: Install and Build 🔧 # This example project is built using npm and outputs the result to the 'build' folder. Replace with the commands required to build your project, or remove this step entirely if your site is pre-built.
        run: |
          npm install
          npm run build
        env:
          CI: false

      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@releases/v3
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          BRANCH: master # The branch the action should deploy to.
          FOLDER: public # The folder the action should deploy.
    ```
    我们将代码设置好之后，将源码提交到刚才创建的分支上后，代码就可以自动打包，生成新的版本


