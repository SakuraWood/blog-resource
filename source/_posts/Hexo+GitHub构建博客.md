---
title: Hexo+GitHub构建个人博客
date: 2017-02-12 19:36:07
tags:
---

## 什么是Hexo？

Hexo是一个快速，简单和强大的博客框架。你可以在短时间内利用markdown标记语言通过Hexo生成一份拥有漂亮主题的博客。

## 安装Hexo

### 需要的安装环境
安装Hexo非常容易。但是，你需要先安装几个其他的东西：
* Node.js
* Git

### 安装Node.js
最好是通过nvm来安装Node.js。

cURL:

`
$ curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | sh
`

Wget:

`
$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/master/install.sh | sh
`

nvm安装完毕后，重启终端输入以下命令安装Node.js:
`
$ nvm install stable
`

如果你的电脑已经有这些，恭喜你！只需用npm安装Hexo：

` 
$ npm install -g hexo-cli
`

## 创建博客
### 初始化工作
首先创建一个博客文件夹，比如：
`
C:\Users\Administrator\Desktop\blog
`
然后对其进行初始化：

`
hexo init blog
`

进入博客文件夹：

`
cd blog
`

接着安装必须的node依赖库：

`
npm install
`

运行：

`
hexo server
`

此时打开浏览器`localhost:4000`可以看到本地的一个运行效果，如果你打不开，有可能是端口冲突，可以通过更改端口解决，比如：

`
hexo server -p 5000
`

### 添加新文章
可以通过以下命令创建一篇新的文章：

`
hexo new title
`

title就是你的文章名字，hexo会在你的博客文件夹/source/_posts/下创建相应的title.md文件。

## 部署到GitHub
登录GitHub账号，新建一个仓库，名字格式如下：

`
yourname.github.io
`

其中yourname就是你的Git账号名。进入你的博客文件夹，修改_config.yml参数，找到deploy选项，键入以下参数：

```
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master

```

终端输入：

`
hexo d -g      //d代表部署 g代表生成
`

完成后等待一段时间，浏览器打开yourname.github.io, 可以看到博库已经部署到GitHub上。

