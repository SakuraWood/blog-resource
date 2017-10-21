---
title: gulp简单用法
date: 2017-04-18 09:36:07
tags:
---

# gulp简单用法

不说原理，直接开干

**需了解node**
稍微介绍一下node：
* package.json(node项目配置文件)，形式如下：

``` json
   {

  "name": "32000", //项目名
  "version": "0.0.1", //版本号
  "description": "vrf project", //项目描述
  "main": "gulpfile.js",//项目入口文件
  "scripts": {          //脚本命令，可自定义，像这个，我定义了一个start命令用来指                          //向webpack命令
    "start": "webpack"
  },
  "repository": {       //git目录
    "type": "git",
    "url": "http://greesmartapp02/susan/air-h5.git"
  },
  "keywords": [         //关键词
    "vrf"
  ],
  "author": "susan,hwb,leesure",    //作者
  "license": "ISC",
  "devDependencies": {              //依赖插件，用到的所有依赖，相当于android                                       //gradle里面的依赖，懂了吧
    "browser-sync": "^2.18.8",
    "css-loader": "^0.28.0",
    "gulp": "^3.9.1",               //gulp也是node的插件，相当于你用的第三方包，                                    //懂？
    "gulp-concat": "^2.6.1",
    "gulp-htmlmin": "^3.0.0",
    "gulp-if": "^2.0.2",
    "gulp-minify-css": "^1.2.4",
    "gulp-uglify": "^2.1.2",
    "gulp-useref": "^3.1.2",
    "gulp-util": "^3.0.8",
    "jsx-loader": "^0.13.2",
    "run-sequence": "^1.2.2",
    "sass-loader": "^6.0.3",
    "style-loader": "^0.16.1",
    "url-loader": "^0.5.8",
    "webpack-stream": "^3.2.0"
  }
}
```

* 接下来再说一下gulpfile.js,这个呢，就是gulp的配置文件；

```javascript

var gulp = require('gulp');
//合并文件
var concat = require('gulp-concat');
//串行执行顺序task
var runSequence = require('gulp-sequence');
//webpack插件
var webpack = require('webpack');
//深度压缩图片插件
var pngquant = require('pngquant');
//删除文件插件
var del = require('del');
// 合并stream
var mergeStream=require('merge-stream');
//子任务
var exec=require('child_process').exec;

//压缩图片
var imagemin = require('gulp-imagemin');
//压缩html
var htmlmin = require('gulp-htmlmin');
//压缩css
var minicss = require('gulp-clean-css');
//压缩js
var uglify = require('gulp-uglify');
//代替pipe
var pump = require('pump');
//重命名插件
var rename = require('gulp-rename');
//文档注释插件
var jsdoc = require('gulp-jsdoc3');
//单元测试插件jasmine
var jasmineBrowser = require('gulp-jasmine-browser');
//监视文件变化插件
var watch = require('gulp-watch');

//生成注释文档
gulp.task('doc', function (cb) {
    var config = require('./jsdoc-config.json');
    gulp.src(['./app/**/*.js'], {read: false})
    // gulp.src(['./app/**/*.js'], {read: false})
        .pipe(jsdoc(config, cb));

});

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

//压缩css
gulp.task('csszip', function (callback) {
    pump(
        [
            gulp.src(['app/**/*.css']),
            minicss(),
            gulp.dest('dist')
        ],
        callback
    );
});

//压缩js
gulp.task('jszip', function (callback) {
    pump(
        [
            gulp.src('app/**/*.js'),
            uglify(),
            gulp.dest('dist')
        ],
        callback
    );
});

gulp.task('clear', function () {
    return del(['dist']);
});

//复制字体
gulp.task('font', function () {
    gulp.src(['app/**/*.ttf', 'app/**/*.otf', 'app/**/*.jar', 'app/**/*.xml'])
        .pipe(gulp.dest('dist'))
});

//异步执行压缩任务
gulp.task('default',
    runSequence('clear',['parsezip', 'imgzip', 'htmlzip', 'csszip', 'jszip', 'font','doc'],'copy'
    )

);

//复制到android工程
//测试用
gulp.task('copyto', function () {
    gulp.src('app/**/*')
        .pipe(gulp.dest('../assets/release/Plugins/32000'))
});
//正式用
gulp.task('copy', function () {
    gulp.src('dist/**/*')
        .pipe(gulp.dest('../assets/release/Plugins/32000'))
});

//单元测试
gulp.task('jasmine', function () {
    var files = ['./test/src/**/*.js', './test/spec/**/*.js', 'app/js/**/*.js'];
    return gulp.src(files)
        .pipe(watch(files))
        .pipe(jasmineBrowser.specRunner())
        .pipe(jasmineBrowser.server({port: 9199}))
});

//合并json中的js
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

* 执行npm install


