---
title: 浅谈JS中this关键字指向的对象
date: 2017-05-6 09:42:07
tags: javascript
categories:
- 前端
---

# 浅谈JS中this关键字指向的对象

学习js三个月了，现在来谈一下`this`关键字，做个总结。

## java中的this

因为之前是码`java`的，所以我想和之前的`OO`语言做个比较。所以如果之前你敲过`java`，对这样的代码应该不会陌生：

```
public class Test1{
    private int i=1;

    public void Test1(){
        this.i++;           //①这里的this指向什么？
    }
}

```

<!-- more -->


当一个对象创建后，`Java`虚拟机（`JVM`）就会给这个对象分配一个引用自身的指针，这个指针的名字就是 `this`。上个例子中，如果你创建了一个`Test`的实例对象，那它就是`Test`对象的引用。

当然还有这样的：

```
public class Test2{
    private int i=1;

    public void Test2(){
        this(2);           //②这里的this用来干嘛？
    }

    public void Test2(int i){
        this.i++;
    }
}
```
这个例子是用于在构造方法中引用满足指定参数类型的构造器（其实也就是构造方法）。

## js中的this
因为之前的概念，所以搞了很久才接受`js`中的`this`。总之，就是一句话，关键字 `this` 总是指向调用该方法的对象。这句话，看上去很简单，但我想举几个例子，以便有个更好的记忆。

* example 1：

```
function Person(){
    this.name="wtf";        //①这里的this指的是什么？
}

Person();
```

* example 2:

```
var Person= {
  hehe:function () {
      this.name="wtf";        //②这里的this指的是什么？
  }
};

Person.hehe();
```

* example 3:

```
function Person() {
    this.name = "wtf";        //③这里的this指的是什么？
}

var person = new Person();

console.log(person.name);
```

* example 4:

```
function hehe() {
    console.log(this.name);  //④这里的this指的是什么？
}

node.addEventListener('click',hehe);
```

* example 5:

```
var Person = {
    name: 'wtf',

    hehe: function () {
        console.log(this.name); //⑤这里的this指的是什么？
    }
};

node.addEventListener('click',Person.hehe);
```

* example 6：

```
var Person = {
    name: 'wtf',

    hehe: function () {
        console.log(this.name); //⑥这里的this指的是什么？
    }
};

node.addEventListener('click', function () {
    Person.hehe();
});
```

先说一下答案：

1. global
2. Object
3. Person
4. global
5. global
6. Object

始终记住一句话，关键字 `this` 总是指向调用该方法的对象！你可以像记右手法则来记它，并用它来判断。

* 首先看第一个，你可以理解为无人调用它（函数调用），如果无人调用，那它指的就是`global`，就这样。

* 第二个，因为是`Person`调用了`hehe`方法（方法调用），`Person`是一个`Object`，所以是`Object`，完毕。

* 第三个，new关键字来了，其实new关键字是做了一些工作的，这里简单说一下，比如var person =new Person(),那么它其实这样做了：
```
var Person=function () {
    this=Object.create(Person.prototype);  //new的时候创建一个Person的原型对象并返回
    return this;
};
```
可能你会问怎么能允许给`this`赋值呢？是的，我们没有这样的能力，但js的解释器是有这样的能力的。
所以答案是`Person`。

* 第四个，放到一个事件方法里了，有些可能就以为有些“猫腻”了，甚至觉得指向`node`=。=,其实前面加再多都没卵用，始终一句话，`this` 总是指向调用该方法的对象。无人调用（函数调用）`hehe`，那就是`global`。

* 第五个，还是在一个事件方法里，只不过，`hehe`前面加了个`Person.`。所以如果觉得这里的`this`指的是`Person`的话，还是先去把`方法`和`方法被调用`这两个概念搞清楚。之前码java的，都没接触过方法可以作为参数，但`Person.hehe`和`Person.hehe()`不是一码事。

* 最后一个，也许就是就是上一个例子里，你想要的答案=。=，这次不是直接写了，而是放到一个`function`内，再由`Person`来调用`hehe`，这次，没得说了吧，`this`指的就是`Object`了。

说了这些，其实都不如这一句话，`this` 总是指向调用该方法的对象。







