---
title: js语法
date: 2017-02-12 19:36:07
tags: javascript
categories:
- 前端
---
# JS 语法突击检查

1. 知道`continue`吧？那你知道`label`语句吗？请说明使用方法。
    `continue`后跟label可以跳出到label所在的循环。

2. 基本类型有哪些？
    Undefined,Null,Boolean,Number,String

3. js的字符串是引用类型吗？
    js的字符串不是，在大多数语言中，字符串被当做对象，是引用类型，但js放弃了这一传统。

4. 函数传参是按什么传递？
    全部是按值传递，不存在按引用传递。

5. 如何检测类型？
    检测基本数据类型用typeof，检测对象用instanceof。根据规定，所有引用类型都是Object的实例。

6. 了解过ES6吗？简单说明let操作符。解释什么是暂时性死区。
    简单地说，let操作符提供了块级作用域{}，暂时性死区是指在let作用域内，所声明的变量不受外部影响。

<!-- more -->