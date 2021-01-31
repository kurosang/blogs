# Gulp 简介与实战

### 一、项目背景

这个月有一个需求是要迭代`光环助手app`里面论坛发布和编辑帖子的`编辑器`，于是翻出了好久没改过的代码，做着做着才发现不对劲。为什么不对劲呢？因为这个编辑器在 web 前端这里只负责内容显示的区域，并且只有一个 js 文件提供一些方法，连 index.html 都是放在客户端里的。

举个例子，插入图片等操作按钮，是属于客户端的，web 这边只负责写一个方法 function，参数是接收一个图片地址 url，然后生成 dom 插入到 html 中。因此，当跟客服端联调时，每改完一次代码，我就需要把文件拖到 oss 上，然后打开 app 重新进入查看效果。

那么，有没有方法去提高效率呢？有的，引入 gulp 进行改造。

### 二、Gulp 的简介

gulp 是前端开发过程中对代码进行`构建`的工具，是`自动化`项目的构建利器；它不仅能对网站资源进行`优化`，而且在开发过程中很多`重复`的任务能够使用正确的工具自动完成；使用它，我们不仅可以很愉快的编写代码，而且大大提高我们的工作效率。

#### Gulp 主要 API

**1 .gulp.src()**

gulp 模块的 src 方法，用于产生数据流，表示要处理的文件，一般有以下几种格式：

&emsp;js/app.js：指定确切的文件名。

&emsp;js/\*.js：某个目录所有后缀名为 js 的文件。

&emsp;js/\*_/_.js：某个目录及其所有子目录中的所有后缀名为 js 的文件。

&emsp;!js/app.js：除了 js/app.js 以外的所有文件。

&emsp;\*.+(js|css)：匹配项目根目录下，所有后缀名为 js 或 css 的文件。

src 方法的参数可以是一个数组，如：

```
gulp.src(['./js/*.js', './sass/*.scss']);
```

**2.pipe**

.pipe 为每个任务的连接，执行完一个任务之后，再次连接执行下一个任务，如：

```
.pipe(minifycss())                // 压缩CSS
.pipe(gulp.dest('output/'));  // 发布到线上版本
```

**3.gulp.dest()**

创建一个用于将 Vinyl 对象写入到文件系统的流。

```
.pipe(minifycss())                // 压缩CSS
.pipe(gulp.dest('output/'));  // 发布到线上版本
```

**4.gulp.watch()**

gulp 模块的 watch 方法，用于指点需要监视的文件，一旦文件发生变动，就运行指点任务。

```
gulp.watch('./js/*.js', ['lint', 'script']);
```

#### Gulp 插件常用：

- 1、编译 Sass,生成雪碧图(gulp-compass);

- 2、编译 sass(gulp-sass);

- 3、sass 浏览器地图(gulp-sourcemaps);

- 4、重命名文件(gulp-rename);

- 5、JS 语法检测(gulp-jshint);

- 6、JS 丑化(gulp-uglify);

- 7、JS 文件合并(gulp-concat);

- 8、图片压缩(gulp-imagemin);

- 9、缓存通知(gulp-cache);

- 10、web 服务(gulp-connect);

- 11、压缩 CSS(gulp-minify-css);

- 12、css 文件引用 URL 图片加版本号(gulp-make-css-url-version);

- 13、清空文件夹(gulp-clean);

- 14、更新通知(gulp-notify);

- 15、html 文件引用加版本号(gulp-rev-append);

- 16、web 服务浏览器同步浏览(browser-sync);　　 // 推荐使用这个作为 web 服务

### 实现思路与代码

我们首先在`package.json`里，新增两条命令，一条是测试环境，一条是正是环境。

```
{
  "gulp:js": "cross-env NODE_ENV=production gulp --gulpfile gulpfile-android-js.js",
  "gulp:watch:js": "gulp --gulpfile gulpfile-android-js.js"
}
```

通过上面命令可以看出，`gulpfile-android-js.js`就是我们重点要写的文件，除此之外，我们通过`cross-env NODE_ENV=production`来设定正式环境。

在`gulpfile-android-js.js`文件中，我们可以通过 process.env.NODE_ENV 如果等于 production 就是正式环境，否则就是测试环境。

**测试环境**

```
gulp.watch("./app-android-js/halo_app_test.js", gulp.series(uploadTest))
```

测试环境主要使用 gulp.watch，来实现：检测文件改动，就将文件上传到 oss。

之前我们想要输出看某个值，我们需要先改动 test.js 文件，然后打开资源文件夹，把 test.js 文件拖到 oss，再打开 app 重新进入页面。

经过调整之后，我们只需要改动完 test.js 文件，保存，然后在 app 重新进入就可以看到效果了。开发效率提高是非常显著的。

**正式环境**

```
    fs.copyFileSync(
      "./app-android-js/halo.js",
      "./app-android-js/halo_backup.js"
    )
    console.log("copy file succeed")
    fs.copyFileSync(
      "./app-android-js/halo_app_test.js",
      "./app-android-js/halo.js"
    )
    console.log("copy file succeed")
    upload()
```

正式环境的流程主要是：首先把 halo.js 这个正式环境的文件复制到 halo_backup.js 中，然后把 test.js 测试文件复制到 halo.js 正式文件中，最后，把这些文件一起上传到 oss 上。

这里的`fs.copyFileSync`方法是，fs 是 node.js 中用于操作文件系统的模块，copyFileSync 是其中自带的复制文件 api。

`upload()`就是 web 直传 oss 的函数了。

### 总结

现在提到前端，我们总会提到前端构建，自动化构建等词语。我今天分享的这个案例不难，但很好的诠释了这点：从手动部署到自动化部署。

这次改造主要使用了 gulp，npm scripts，node 等知识，实际上前端工程化，前端构建还有许多知识和工具，除了 webpack 以外，还有 gulp，rollup，parcel 等等，不同的工具都是有对应的特性以及应用场景。作为一个合格的前端工程师，应该要去熟悉不同工具的差别，然后根据需求场景来选择最优的工具，以此来提高自己与组的工、开发效率。
