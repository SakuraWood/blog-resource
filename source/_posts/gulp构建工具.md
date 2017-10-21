---
title: Gulp构建工具
date: 2017-06-27 15:54:07
tags: gulp
categories:
- 前端
---

# Gulp构建工具

## gulp与webpack

* gulp是工具链、构建工具，可以配合各种插件做js压缩，css压缩，sass编译 替代手工实现自动化工作
* webpack是文件打包工具，可以把项目的各种js文、css文件等打包合并成一个或多个文件，主要用于模块化方案，预编译模块的方案

其实webpack很多功能gulp都能做，gulp甚至能够配置webpack，所以这两者之间是没有冲突的。

举个例子，说到js压缩，现在的人都用uglify，如果有多个要压缩的文件，咋办？你总不能一个一个去压缩吧。所以好事者就写脚本，写个.sh或者.bat什么的。幸运的是，gulp有uglify的插件，你就不用重复造轮子了。

```javascript
// 压缩js
gulp.task('jszip', function (callback) {
    pump(
        [
            gulp.src('app/**/*.js'),
            uglify({
                mangle: true,//类型：Boolean 默认：true 是否修改变量名
                compress: true//类型：Boolean 默认：true 是否完全压缩
            }),
            gulp.dest('dist')
        ],
        callback
    );
});
```
上面就是一个例子，把app文件夹下的js文件都压缩并打包到dist文件夹里。是不是很方便？

“看上去很牛逼，倒是讲讲怎么安装啊”

## nodejs

先稍微聊聊nodejs。gulp是基于nodejs的，而nodejs的流非常强大。

![gulp](/images/gulp_1.jpg)

“你说强大就强大，理由呢？”

当初的node还不叫nodejs，还在那纠结到底放到哪个语言上比较好。当时的语言，一遍观望下来，几乎都自带了一套I/O接口，而且是阻塞式的。
node的设计者对这些都不太满意，但他们又不想重新发明一种语言，然后他们瞄上了js，js几乎是完美的选择，天然的异步方式，加上没有I/O接口，node果断加入了js阵营。异步非阻塞的模型是nodejs强大的根本原因。

<!-- more -->

### I/O（题外话）
“异步非阻塞怎么就强大了？wtf？”

我想问的是，你明白I/O吗？
“I/O不就是输入输出嘛，有什么大不了的。”
所以说，写操作系统的人是真正的大神，深藏功与名，因为他们已经为你准备好了一切。

从gulp讲到了I/O，感觉完全跑题了。。。呃，我本来就不是为了单纯介绍gulp构建工具，有兴趣的可以看看，再说，复习复习I/O没什么不好，不是吗=。=

I/O操作大致可以分为两部分:
1. 发出请求
2. 结果完成

I/O在操作系统中的四种模型（图来自Unix网络编程）：

* Blocking I/O 阻塞式I/O
![gulp](/images/gulp_2.png)

计算机里要完成I/O操作，就会使用系统调用。

“系统调用是什么？”
额，这个就不讲了，否则越讲越偏。不要过多纠结在操作系统层面，这些模型是怎么实现的。
上图中的用户程序调用了recvfrom这个系统调用，kernel就开始了I/O的第一个阶段：准备数据。这个时候，kernel一直在等待足够的数据到达，与此同时，整个用户进程被阻塞。等准备好了之后，kernel返回结果，进程解除被block的状态。可以明显看到，kernel的两个状态，进程都是被阻塞的。

java里面的I/O处理：
```java
InputStream in = newConnection.getInputStream();
InputStreamReader reader = new InputStreamReader(in);
BufferedReader buffer = new BufferedReader(reader);
Request request = new Request();
while(!request.isComplete()) {
    String line = buffer.readLine();//阻塞
        request.addLine(line);
}
```

`readLine()`是一个阻塞操作。

* Non-Blocking I/O 非阻塞式I/O
![gulp](/images/gulp_3.png)

当用户进程发出read操作并调用系统调用，如果kernel中的数据还没准备好，此时它会马上返回一个结果，不会阻塞当前用户进程。一旦数据准备好，此时又再次收到了系统调用，那它就可以立即返回。唯一消耗的是，用户进程是在不断的主动询问kernel准备好数据没有。

* I/O multiplexing I/O复用

大名鼎鼎的I/O复用。
![gulp](/images/gulp_4.png)

I/O多路复用 (单个线程，通过记录跟踪每个I/O流(sock)的状态，来同时管理多个I/O流 。)
当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。
这个图和blocking IO的图其实并没有太大的不同，事实上，还更差一些。因为这里需要使用两个system call (select 和 recvfrom)，而blocking I/O只调用了一个system call (recvfrom)。但是，用select的优势在于它可以同时处理多个connection。

![gulp](/images/gulp_5.png)

就像拨码开关，哪里有数据拨向哪里。

* Asynchronous I/O 异步I/O

![gulp](/images/gulp_6.png)

可以看到，系统调用之后，并没有阻塞，而是依靠通知机制，当copy完了之后通知用户进程，可见这是纯异步的。发出后就不用任何等待，可以执行其他操作。

看到上面的解释，这些“异步”玩意儿，几乎都是操作系统层面的东西。那么nodejs所谓的异步非阻塞I/O是哪种？
还用说嘛，那不就是第四种。。。
但是你现在知道了，所谓的异步，是需要操作系统层面的支持的。如果没有或者有缺陷，就需要模拟。
Linux下有一个异步I/O的库，叫libeio，libeio实质上是采取线程池与阻塞式I/O模拟出来的异步I/O。
Windows下有独有的内核级异步I/O方案：IOCP。IOCP的思路是真正的异步I/O方案，调用异步方法，然后等待I/O完成通知。IOCP内部依旧是通过线程实现，不同在于这些线程由系统内核接手管理。IOCP的异步模型与Node.js的异步调用模型已经十分近似。
以上两种方案则正是Node.js选择的异步I/O方案。由于Windows平台和Unix平台的差异，Node.js提供了libuv来作为抽象封装层，使得所有平台兼容性的判断都由这一层次来完成，保证上层的Node.js与下层的libeio/libev及IOCP之间各自独立。Node.js在编译期间会判断平台条件，选择性编译unix目录或是win目录下的源文件到目标程序中。

### js中的异步

明确一点：
* js能异步是因为它用能调用的模块是异步的。js都是单线程的。而且只有一个事件队列（也可以理解成任务队列），他之所以异步是是因为某些的模块是异步的。当发送一个异步网络请求后，js的主线程不会一直等待这个请求返回，而是执行事件队列里下一个事件。请注意，js并没有实现如何发送网络请求，js只是调用了某个能发送网络请求的模块，而这个模块是通过c++或其他语言实现。然后这个模块在等待请求的结果，当得到响应后，便把响应成功这个事件添加到js的事件队列的队尾。网络请求发送的同时，js依然在执行，这显然是异步的。

可以通俗地认为，nodejs的函数基本都是默认异步的。

（ps：听上去很屌的样子，写个复制看看？不用写了，光是比复制怎么比得过嘛。再凶也凶不过系统原生的copy啊。。。有些人写复制来证明node很快，蛋疼不蛋疼？实测比系统复制慢一倍。）

## gulp

一般通用的web app项目结构:

![gulp](/images/gulp_10.png)

### nodejs安装

<http://nodejs.cn/download/>

装完之后，就有了NPM包管理器。像这样的：

可以执行：
```
npm init
```
这个用来初始化你的package.json。

这是我项目里的package.json配置：

```json
{
  "name": "vrf",
  "version": "0.0.1",
  "description": "vrf project",
  "main": "gulpfile.js",
  "scripts": {
    "start": "webpack"
  },
  "repository": {
    "type": "git",
    "url": "http://xxxxx/susan/xxx.git"
  },
  "keywords": [
    "vrf"
  ],
  "author": "susan,hwb,leesure",
  "license": "ISC",
  "dependencies": {
    "babel-preset-es2015": "^6.24.1",
    "browser-sync": "^2.18.8",
    "cheerio": "^1.0.0-rc.1",
    "child_process": "^1.0.2",
    "css-loader": "^0.28.0",
    "del": "^2.2.2",
    "gulp": "^3.9.1",
    "gulp-babel": "^6.1.2",
    "gulp-clean-css": "^3.0.4",
    "gulp-concat": "^2.6.1",
    "gulp-htmlmin": "^3.0.0",
    "gulp-if": "^2.0.2",
    "gulp-imagemin": "^3.2.0",
    "gulp-jasmine": "^2.4.2",
    "gulp-jasmine-browser": "^1.7.1",
    "gulp-jsdoc3": "^1.0.1",
    "gulp-jsduck": "^1.0.0",
    "gulp-minify-css": "^1.2.4",
    "gulp-qunit": "^1.5.0",
    "gulp-rename": "^1.2.2",
    "gulp-sequence": "^0.4.6",
    "gulp-uglify": "^2.1.2",
    "gulp-useref": "^3.1.2",
    "gulp-util": "^3.0.8",
    "gulp-watch": "^4.3.11",
    "gulp-webserver": "^0.9.1",
    "jsdoc": "^3.4.3",
    "jsduck": "^1.1.2",
    "jsx-loader": "^0.13.2",
    "merge-stream": "^1.0.1",
    "pngquant": "^1.2.0",
    "pump": "^1.0.2",
    "qunit": "^1.0.0",
    "run-sequence": "^1.2.2",
    "sass-loader": "^6.0.3",
    "style-loader": "^0.16.1",
    "url-loader": "^0.5.8",
    "webpack-stream": "^3.2.0"
  },
  "directories": {
    "doc": "docs",
    "test": "test"
  },
  "devDependencies": {
    "gulp-obfuscate": "^0.2.9",
    "gulp-uglify": "^2.1.2",
    "uglify": "^0.1.5",
    "uglify-js": "^3.0.18"
  }
}
```
这个json相当于依赖配置文件，项目根据它里面定义所依赖的第三方包进行配置。
不明白？好，android的gradle懂吧？iOS的CocoaPods懂吧？

### gulp安装
1. 全局安装 gulp：

```
$ npm install --global gulp
```

2. 作为项目的开发依赖（devDependencies）安装：

```
$ npm install --save-dev gulp
```

3. 在项目根目录下创建一个名为 gulpfile.js 的文件：

```javascript
var gulp = require('gulp');

gulp.task('default', function() {
  // 将你的默认的任务代码放在这
});
```

4. 运行 gulp：

```
$ gulp
```

默认的名为 default 的任务（task）将会被运行，在这里，这个任务并未做任何事情。

想要单独执行特定的任务（task），请输入 gulp <*task*> <*othertask*>。

### gulp速览

其实，gulp的操作都是一些插件的运用，你只需要知道四个东西：

1. gulp.task()
2. gulp.src()
3. gulp.dest()
4. gulp.watch()

#### gulp.task(names[,deps,fn])
gulp.task方法用来定义任务，
name 为任务名，
deps 是当前定义的任务需要依赖的其他任务，
为一个数组。当前定义的任务会在所有依赖的任务执行完毕后才开始执行。
如果没有依赖，则可省略这个参数，
fn 为任务函数，我们把任务要执行的代码都写在里面。该参数也是可选的。

#### gulp.src(globs[,options])
gulp.src()方法正是用来获取流的，但要注意这个流里的内容不是原始的文件流，而是一个虚拟文件对象流，这个虚拟文件对象中存储着原始文件的路径、文件名、内容等信息，本文暂不对文件流进行展开，你只需简单的理解可以用这个方法来读取你需要操作的文件就行了，globs参数是文件匹配模式(类似正则表达式)，用来匹配文件路径(包括文件名)，当然这里也可以直接指定某个具体的文件路径。当有多个匹配模式时，该参数可以为一个数组。
options为可选参数。通常情况下我们不需要用到，暂不考虑。

文件匹配模式
Gulp内部使用了node-glob模块来实现其文件匹配功能。我们可以使用下面这些特殊的字符来匹配我们想要的文件：

`*` 匹配文件路径中的0个或多个字符，但不会匹配路径分隔符，除非路径分隔符出现在末尾
`**` 匹配路径中的0个或多个目录及其子目录,需要单独出现，即它左右不能有其他东西了。如果出现在末尾，也能匹配文件。
`?`匹配文件路径中的一个字符(不会匹配路径分隔符)
`[...]` 匹配方括号中出现的字符中的任意一个，当方括号中第一个字符为^或!时，则表示不匹配方括号中出现的其他字符中的任意一个，类似js正则表达式中的用法!(pattern|pattern|pattern)匹配任何与括号中给定的任一模式都不匹配的
`?(pattern|pattern|pattern)`匹配括号中给定的任一模式0次或1次，类似于js正则中的(pattern|pattern|pattern)?
`+(pattern|pattern|pattern)`匹配括号中给定的任一模式至少1次，类似于js正则中的(pattern|pattern|pattern)+
`*(pattern|pattern|pattern)`匹配括号中给定的任一模式0次或多次，类似于js正则中的(pattern|pattern|pattern)*
`@(pattern|pattern|pattern)`匹配括号中给定的任一模式1次，类似于js正则中的(pattern|pattern|pattern)
文件匹配列子：

`*` 能匹配 reeoo.js,reeoo.css,reeoo,reeoo/,但不能匹配reeoo/reeoo.js
`*.*`能匹配 reeoo.js,reeoo.css,reeoo.html
`*/*/*.js`能匹配 r/e/o.js,a/b/c.js,不能匹配a/b.js,a/b/c/d.js
`**`能匹配 reeoo,reeoo/reeoo.js,reeoo/reeoo/reeoo.js,reeoo/reeoo/reeoo,reeoo/reeoo/reeoo/reeoo.co,能用来匹配所有的目录和文件
`**/*.js`能匹配 reeoo.js,reeoo/reeoo.js,reeoo/reeoo/reeoo.js,reeoo/reeoo/reeoo/reeoo.js
`reeoo/**/co`能匹配 reeoo/co,reeoo/ooo/co,reeoo/a/b/co,reeoo/d/g/h/j/k/co
`reeoo/**b/co`能匹配 reeoo/b/co,reeoo/sb/co,但不能匹配reeoo/x/sb/co,因为只有单**单独出现才能匹配多级目录
`?.js`能匹配 reeoo.js,reeoo1.js,reeoo2.js
`reeoo??`能匹配 reeoo.co,reeooco,但不能匹配reeooco/,因为它不会匹配路径分隔符
`[reo].js`只能匹配 r.js,e.js,o.js,不会匹配re.js,reo.js等,整个中括号只代表一个字符
`[^reo].js`能匹配 a.js,b.js,c.js等,不能匹配r.js,e.js,o.js
当有多种匹配模式时可以使用数组

```javascript
    //使用数组的方式来匹配多种文
    gulp.src(['js/*.js','css/*.css','*.html'])
```

使用数组的方式还有一个好处就是可以很方便的使用排除模式，在数组中的单个匹配模式前加上!即是排除模式，它会在匹配的结果中排除这个匹配，要注意一点的是不能在数组中的第一个元素中使用排除模式

`gulp.src([*.js,'!r*.js'])` 匹配所有js文件，但排除掉以r开头的js文件
`gulp.src(['!r*.js',*.js])` 不会排除任何文件，因为排除模式不能出现在数组的第一个元素中
此外，还可以使用展开模式。展开模式以花括号作为定界符，根据它里面的内容，会展开为多个模式，最后匹配的结果为所有展开的模式相加起来得到的结果。展开的例子如下：

`r{e,o}c`会展开为 rec,roc
`r{e,}o`会展开为 reo,ro
`r{0..3}o`会展开为 r0o,r1do,r2o,r3o

#### gulp.dest(path[,options])

gulp.dest()方法是用来写文件的，path为写入文件的路径,options为一个可选的参数对象，通常我们不需要用到，暂不考虑。
要想使用好gulp.dest()这个方法，就要理解给它传入的路径参数与最终生成的文件的关系。
gulp的使用流程一般是这样子的：首先通过gulp.src()方法获取到我们想要处理的文件流，然后把文件流通过pipe方法导入到gulp的插件中，最后把经过插件处理后的流再通过pipe方法导入到gulp.dest()中，gulp.dest()方法则把流中的内容写入到文件中，这里首先需要弄清楚的一点是，我们给gulp.dest()传入的路径参数，只能用来指定要生成的文件的目录，而不能指定生成文件的文件名，它生成文件的文件名使用的是导入到它的文件流自身的文件名，所以生成的文件名是由导入到它的文件流决定的，即使我们给它传入一个带有文件名的路径参数，然后它也会把这个文件名当做是目录名，例如：
```javascript
var gulp = require('gulp');
gulp.src('script/jquery.js')
    .pipe(gulp.dest('dist/foo.js'));
//最终生成的文件路径为 dist/foo.js/jquery.js,而不是dist/foo.js
```
要想改变文件名，可以使用插件gulp-rename

下面说说生成的文件路径与我们给gulp.dest()方法传入的路径参数之间的关系。
gulp.dest(path)生成的文件路径是我们传入的path参数后面再加上gulp.src()中有通配符开始出现的那部分路径。例如：
```javascript
var gulp = reruire('gulp');
//有通配符开始出现的那部分路径为 **/*.js
gulp.src('script/**/*.js')
    .pipe(gulp.dest('dist')); //最后生成的文件路径为 dist/**/*.js
//如果 **/*.js 匹配到的文件为 jquery/jquery.js ,则生成的文件路径为 dist/jquery/jquery.js
```
再举更多一点的例子
```javascript
gulp.src('script/avalon/avalon.js') //没有通配符出现的情况
    .pipe(gulp.dest('dist')); //最后生成的文件路径为 dist/avalon.js

//有通配符开始出现的那部分路径为 **/underscore.js
gulp.src('script/**/underscore.js')
    //假设匹配到的文件为script/util/underscore.js
    .pipe(gulp.dest('dist')); //则最后生成的文件路径为 dist/util/underscore.js

gulp.src('script/*') //有通配符出现的那部分路径为 *
    //假设匹配到的文件为script/zepto.js
    .pipe(gulp.dest('dist')); //则最后生成的文件路径为 dist/zepto.js
```
通过指定gulp.src()方法配置参数中的base属性，我们可以更灵活的来改变gulp.dest()生成的文件路径。
当我们没有在gulp.src()方法中配置base属性时，base的默认值为通配符开始出现之前那部分路径，例如：
```javascript
gulp.src('app/src/**/*.css') //此时base的值为 app/src
```
上面我们说的gulp.dest()所生成的文件路径的规则，其实也可以理解成，用我们给gulp.dest()传入的路径替换掉gulp.src()中的base路径，最终得到生成文件的路径。
```javascript
gulp.src('app/src/**/*.css') //此时base的值为app/src,也就是说它的base路径为app/src
     //设该模式匹配到了文件 app/src/css/normal.css
    .pipe(gulp.dest('dist')) //用dist替换掉base路径，最终得到 dist/css/normal.css
```
所以改变base路径后，gulp.dest()生成的文件路径也会改变

```javascript
gulp.src(script/lib/*.js) //没有配置base参数，此时默认的base路径为script/lib
    //假设匹配到的文件为script/lib/jquery.js
    .pipe(gulp.dest('build')) //生成的文件路径为 build/jquery.js
gulp.src(script/lib/*.js, {base:'script'}) //配置了base参数，此时base路径为script
    //假设匹配到的文件为script/lib/jquery.js
    .pipe(gulp.dest('build')) //此时生成的文件路径为 build/lib/jquery.js
```
用gulp.dest()把文件流写入文件后，文件流仍然可以继续使用。


#### gulp.watch(glob[,opts],tasks)

gulp.watch()用来监视文件的变化，当文件发生变化后，我们可以利用它来执行相应的任务，例如文件压缩等。
glob 为要监视的文件匹配模式，规则和用法与gulp.src()方法中的glob相同。
opts 为一个可选的配置对象，通常不需要用到，暂不考虑。
tasks 为文件变化后要执行的任务，为一个数组，
```javascript
gulp.task('uglify',function(){
  //do something
});
gulp.task('reload',function(){
  //do something
});
gulp.watch('js/**/*.js', ['uglify','reload']);
```

gulp.watch()还有另外一种使用方式：

```javascript
gulp.watch(glob[, opts, cb])
```

glob和opts参数与第一种用法相同
cb参数为一个函数。每当监视的文件发生变化时，就会调用这个函数,并且会给它传入一个对象，该对象包含了文件变化的一些信息，type属性为变化的类型，可以是added,changed,deleted；path属性为发生变化的文件的路径
```javascript
gulp.watch('js/**/*.js', function(event){
    console.log(event.type); //变化类型 added为新增,deleted为删除，changed为改变 
    console.log(event.path); //变化的文件的路径
});     
```

### gulp插件的使用

安装gulp插件都是通过npm安装：
```
npm install --save-dev xxxxx
```

#### gulp-sourcemaps

source map是什么？

* 从源码转换讲起
JavaScript脚本正变得越来越复杂。大部分源码（尤其是各种函数库和框架）都要经过转换，才能投入生产环境。

常见的源码转换，主要是以下三种情况：

1. 压缩，减小体积。比如jQuery 1.9的源码，压缩前是252KB，压缩后是32KB。
2. 多个文件合并，减少HTTP请求数。
3. 其他语言编译成JavaScript。最常见的例子就是CoffeeScript。
这三种情况，都使得实际运行的代码不同于开发代码，除错（debug）变得困难重重。

通常，JavaScript的解释器会告诉你，第几行第几列代码出错。但是，这对于转换后的代码毫无用处。举例来说，jQuery 1.9压缩后只有3行，每行3万个字符，所有内部变量都改了名字。你看着报错信息，感到毫无头绪，根本不知道它所对应的原始位置。

这就是Source map想要解决的问题。

简单说，Source map就是一个信息文件，里面储存着位置信息。也就是说，转换后的代码的每一个位置，所对应的转换前的位置。

有了它，出错的时候，除错工具将直接显示原始代码，而不是转换后的代码。这无疑给开发者带来了很大方便。

#### gulp-uglify
```
npm install --save-dev gulp-uglify
```

```javascript
// 压缩js
gulp.task('jszip', function (callback) {
    pump(
        [
            gulp.src('app/**/*.js'),
            sourcemaps.init(),
            uglify({
                mangle: true,//类型：Boolean 默认：true 是否修改变量名
                compress: true//类型：Boolean 默认：true 是否完全压缩
            }),
            sourcemaps.write(),
            // obfuscate(),
            gulp.dest('dist')
        ],
        callback
    );
});
```
这其中用到了pump插件：
```
cnpm install --save-dev pump
```
它是用来取代nodejs原生的pipe，主要是它能定位错误信息。

#### gulp-clean-css
```
cnpm install --save-dev gulp-clean-css
```

```javascript
//压缩css
gulp.task('csszip', function (callback) {
    pump(
        [
            gulp.src(['app/**/*.css']),
            sourcemaps.init(),
            minicss(),
            sourcemaps.write(),
            gulp.dest('dist')
        ],
        callback
    );
});
```

#### gulp-htmlmin
```
cnpm install --save-dev gulp-htmlmin
```

```javascript
//压缩html
gulp.task('htmlzip', function (callback) {
    pump(
        [
            gulp.src('app/**/*.html'),
            htmlmin({
                removeComments: true,//清除HTML注释
                collapseWhitespace: true,//压缩HTML
                collapseBooleanAttributes: true,//省略布尔属性的值 <input checked="true"/> ==> <input />
                removeEmptyAttributes: true,//删除所有空格作属性值 <input id="" /> ==> <input />
                removeScriptTypeAttributes: true,//删除<script>的type="text/javascript"
                removeStyleLinkTypeAttributes: true,//删除<style>和<link>的type="text/css"
                minifyJS: true,//压缩页面JS
                minifyCSS: true//压缩页面CSS
            }),
            gulp.dest('dist')
        ],
        callback
    );
});
```

#### gulp-imagemin与pngquant

```javascript
//压缩图片
gulp.task('imgzip', function (callback) {
    pump(
        [
            gulp.src(['app/**/*.png', 'app/**/*.jpeg', 'app/**/*.gif']),
            imagemin(
                {
                    optimizationLevel: 5, //类型：Number  默认：3  取值范围：0-7（优化等级）
                    progressive: true,
                    svgoPlugins: [{removeViewBox: false}],//不要移除svg的viewbox属性
                    use: [pngquant()] //使用pngquant深度压缩png图片的imagemin插件
                }
            ),
            gulp.dest('dist')
        ],
        callback
    );
});
```

#### gulp-sass
sass是一个css预编译器，这里不做过多介绍。

```javascript
var sass = require('gulp-sass');
gulp.task('sass', function () {
 return gulp.src('./sass/**/*.scss')
  .pipe(sourcemaps.init())
  .pipe(sass().on('error', sass.logError))
  .pipe(sourcemaps.write('./maps'))
  .pipe(gulp.dest('./css'));
});
```

#### gulp-sequence

```javascript
var runSequence=require('gulp-sequence');

//异步执行压缩任务
gulp.task('default',
    runSequence('clear', ['parsezip', 'imgzip', 'htmlzip', 'csszip', 'jszip', 'font', 'doc'], 'copy'
    )
);
```

#### gulp-concat

```javascript
//合并js
gulp.task('parsezip', function (callback) {
    pump(
        [
            gulp.src(['app/js/json/state-inner.js',
                'app/js/json/state-outdoor.js',
                'app/js/json/state-basic.js',
                'app/js/json/hex-unit.js',
                'app/js/json/hex-parse-inner.js',
                'app/js/json/hex-parse-outdoor.js',
                'app/js/json/hex-parse-basic.js',
                'app/js/json/json-parse-frame.js',
                'app/js/json/json-send-frame.js']),
            concat('parsedata.min.js'),
            gulp.dest('app/js/json')
        ],
        callback
    );
});
```

#### gulp-jsdoc3

```javascript
//生成注释文档
gulp.task('doc', function (cb) {
    var config = require('./jsdoc-config.json');
    gulp.src(['./app/**/*.js'], {read: false})
        .pipe(jsdoc(config, cb));

});
```

#### gulp-jasmine

这是一个单元测试插件，jasmine是一个单元测试框架，类似的还有Qunit。

```javascript
var jasmineBrowser = require('gulp-jasmine-browser');

gulp.task('jasmine', function () {
    var files = ['./test/src/**/*.js', './test/spec/**/*.js', 'app/js/**/*.js'];
    return gulp.src(files)
        .pipe(watch(files))
        .pipe(jasmineBrowser.specRunner())
        .pipe(jasmineBrowser.server({port: 9199}))
});
```

#### gulp-webpack
gulp是可以和webpack配合的，而且gulp的uglify不能直接压缩ES6的代码。
什么，你跟我说gulp-babel？是的，我说的是gulp-babel，实际过程中，gulp-babel也遇上一些问题，不能转码。而且出问题的一些第三方库，又不想去改它，心里想也许切换成webpack的方案来进行转码或许就可以。

gulp-webpack就是这样的插件，它可以让webpack的配置嵌入到gulp中。
```javascript
var webpack = require('gulp-webpack');
gulp.task('webpack', function() {
  return gulp.src('./xxxx.js')
    .pipe(webpack({
        module: {
        loaders: [
            { test: /\.js$/, loader: 'babel-loader' },
        ],
        },
    }
    ))
    .pipe(rename('/xxxx.js'))
    .pipe(gulp.dest('./dist'));
});
```
同样地，你可以使用`webpack.config.js`:

```javascript
return gulp.src('src/entry.js')
  .pipe(webpack( require('./webpack.config.js') ))
  .pipe(gulp.dest('dist/'));
```

### jenkins与gulp配合

jenkins是一个持续集成工具，就是一个监控持续重复工作的工具。

#### jenkins安装
下载地址：
<https://jenkins.io/download/>

```
java -jar jenkins.war
```
静静等待安装完毕。。。

#### jenkins配置

首先，新建一个项目。
![gulp](/images/gulp_11.png)

设置源码仓库地址：
![gulp](/images/gulp_12.png)

触发器设置：
![gulp](/images/gulp_13.png)

* */10 * * * * 代表每10分钟监测源码库，并执行构建。

构建环境配置：
![gulp](/images/gulp_14.png)

构建配置：
![gulp](/images/gulp_15.png)
![gulp](/images/gulp_16.png)

因为在项目里面有gulpfile.js，所以这里唯一需要的东西就是node环境了。其他都可以用cnpm install然后执行gulp任务即可。

关于gulp的基本使用到此介绍完毕。