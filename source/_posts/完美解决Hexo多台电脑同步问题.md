---
title: 解决Hexo多台电脑同步问题
date: 2017-02-13 15:54:07
tags: hexo
categories:
- 其他
---

## 问题 
使用`Hexo`的时候遇到这样一个问题，如何在多台电脑之间同步自己的博客？
要知道`Hexo`在部署博客到`GitHub`上时，只推了`/public`文件夹中的内容，如果从办公室回到家中，
想要继续写博客，该怎么办呢？

## 思路

事实上，`Hexo`博客文件夹里，从`.gitignore`中可以看到，
它刻意屏蔽掉了`/public`文件夹，如果你懂Git，相信聪明的你已经有解决办法了。

## 解决办法
从你的本地博客文件夹下，创建一个新的分支，随便取个名字，比如：`hexo`，把它推到远程仓库去。

<!-- more -->

`
git checkout hexo
`

`
git add .
`

`
git commit -m "create hexo branch"
`

`
git push origin hexo
`

这样，你的远程仓库下就已经有了hexo分支，它是干嘛用的？没错，它就是用来保存你的
博客原始文件，而用这些原始文件，你可以通过
`
hexo d -g
`
来部署博客！这个命令，用过hexo的你一定很熟悉了吧。

所以你在别的电脑`clone`了这个仓库之后，以后可以执行：

`
git pull origin hexo
`

这样就能更新你的原始文件了，从而生成和部署博客。
