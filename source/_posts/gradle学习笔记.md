---
title: gradle 学习笔记
date: 2017-08-15 15:54:07
tags: 
- gradle 
- android
categories:
- Android开发
---
# gradle上路篇

作为写`Android`的人（虽然现在重心已经偏向`js`），当遇上`gradle`编译时各种各样的错误，，你却不知道怎么办，有时虽然解决了却不知道所以然，我想这样的经历绝不只有我一个。

如果说为什么对`gradle`始终很糊涂，那么`Android Studio`将要背很大的锅，因为`IDE`为你生成好了构建脚本，所以你无需花很多精力去了解它，你的任务更多只是在业务代码本身，事实上，学习gradle你还可以更加地了解`android`程序整个构建过程，在这一点上，恐怕很多人也跟当初我一样是一知半解的。

## 为什么是build.gradle?
对呀，为什么是这个文件？其实它没有什么特别的，只不过因为它是默认的构建脚本。所以好事者说，我想执行自己的怎么办？比如`a.gradle`？ 好，那么其实你可以执行以下命令： 
```
$ gradle -b a.gradle yourtask
```

这样子你可以将构建脚本替换成你想执行的`a.gradle`。可能对于一直点击![run](/images/run.png)图标的同学来说，`gradle`命令行甚至都没用过，这个`yourtask`是什么鬼？其实在你点击了![run](/images/run.png)图标之后，`gradle`的构建脚本可是做了大量的工作，之后我会细说。

## Groovy语言
`gradle`脚本其实是基于一个叫`Groovy`的语言编写的，它的语法相当友好，兼容`java`语法，有着一些`java`语言并没有的特性，比如函数传参。但事实上，你并不需要过多了解这门语言，对于`gradle`脚本来说，掌握部分就够用了。

我并不想在这里介绍`Groovy`语言，有闲时间我建议直接看`Groovy`的官方指南：

<http://groovy-lang.org/documentation.html>

<!-- more -->
## 项目示例
有些人不想看也不想了解`Groovy`语言，那也没事，我直接拿个例子，看完你就想去看了，拿个android工程的构建脚本，先来最简单的，用`Android Studio`创建完一个新的`android`工程之后，`IDE`给你准备了大致上这样的东西：
![project](/images/android_new_project.png)

刚才不是说`build.gradle`是默认的吗，怎么两个啊？所以，很多人在开发时会切到真实的目录下去，像这样的：
![project](/images/real_project.png)

来看看项目根目录下的`build.gradle`:
```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

整个`gradle`构建脚本看上去具有拆分式`JSON`语法，先看最下面，这里定义了一个`task`，叫做`clean`，`task`是`gradle`内置的定义任务的关键字，不用我说你也知道`clean`这是干嘛用的，当你点击![project](/images/clean_project.png)这个的时候，它就在执行这个任务。

另外这个，
```
dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
```
`dependencies`表示的是项目依赖，而这里定义了所引用的`gradle`插件， 好吧，这个`'com.android.tools.build:gradle:2.2.3'`它来自哪？当然是从项目存储库里取，`repositories{}`定义了依赖的源链接库。

注意，`gradle`并不是专门为`android`而生，根目录下的`build.gradle`使用的可是`gradle`自带的`Build script blocks`,
像`allprojects{}`,`repositories{}`,`dependencies{}`等等（wtf? 请参阅<https://docs.gradle.org/current/dsl/>）。

好了，我们再继续看看`app`目录下的`build.gradle`：
```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "com.example.administrator.myapplication"
        minSdkVersion 15
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.1.0'
    testCompile 'junit:junit:4.12'
}

```
第一行说明你引用了一个`gradle`插件`com.android.application`,就好比你在写`java`代码时所引用的包一样，那这个包在哪取？就是之前根目录下的`build.gradle`所定义的`com.android.tools.build:gradle:2.2.3`插件。

既然引用了插件，当然就可以开始用插件里的东西了，所以`android{}`标签来了，注意只有`compileSdkVersion`跟`buildToolsVersion`是必须的，除此之外你可以定义一切不超出插件语法范围的配置。进一步的`android gradle`插件配置，可以参考<http://tools.android.youdaxue.com/tech-docs/new-build-system/user-guide>。

```
defaultConfig {
        applicationId "com.example.administrator.myapplication"         //app的id
        minSdkVersion 15                                                //最低sdk版本号
        targetSdkVersion 25                                             //目标sdk版本号
        versionCode 1                                                   //应用版本号
        versionName "1.0"                                               //应用版本名
    }
```

你会发现`defaultConfig{}`里面定义的其实都是可以在`manifest.xml`中可以定义的。

`buildTypes`代表构建类型，用于控制构建和包装应用的方式。`android`工程默认有两个构建类型，一个是`release`,一个是`debug`。有些人跃跃欲试了，这是不是就是多渠道打包了啊？嗯，有点接近，但并不是。插件额外定义了另外一个`block`，用以构建面向用户的版本，这就是`productFlavors`(产品风格)。

假如发布应用，一个收费版和一个免费版的，则可以这样定义：
```
   productFlavors {
        free {
            applicationId "com.example.administrator.myapplication.free"
        }
        paid {
            applicationId "com.example.administrator.myapplication.paid"
        }
    }
```

你也看到了，其实所有可以放进`defaultConfig{}`里的东西，都可以放到`productFlavors`中，如有重复项则会覆盖掉默认配置。

