---
title: python与java比较
date: 2017-02-12 19:36:07
tags: python java
categories:
- python
---
# python对比java


1. python函数可返回多个

函数返回的是一个tuple


2. 函数参数还有默认参数这么一说

def xxx(a,b=2):
    ...

3. 可变参数的写法

def xxx(*num):

注意，list和tuple前面加上*也可以表示可变参数

<!-- more -->

4. 关键字参数
def person(name, age, **kw):
    print 'name:', name, 'age:', age, 'other:', kw

参数定义的顺序必须是：必选参数、默认参数、可变参数和关键字参数。
默认参数一定要用不可变对象，如果是可变对象，运行会有逻辑错误！

5. 切片
太灵活了。
举个不同的，L[:10:2]代表前10个，每两个取一个

6. 列表生成式

7. 生成器generator
