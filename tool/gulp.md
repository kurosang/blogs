# gulp 学习笔记

## Gulp 主要API
**1 .gulp.src()**

gulp模块的src方法，用于产生数据流，表示要处理的文件，一般有以下几种格式：

&emsp;js/app.js：指定确切的文件名。

&emsp;js/*.js：某个目录所有后缀名为js的文件。

&emsp;js/**/*.js：某个目录及其所有子目录中的所有后缀名为js的文件。

&emsp;!js/app.js：除了js/app.js以外的所有文件。

&emsp;*.+(js|css)：匹配项目根目录下，所有后缀名为js或css的文件。

src方法的参数可以是一个数组，如：
```
gulp.src(['./js/*.js', './sass/*.scss']);
```
**2.pipe**

.pipe为每个任务的连接，执行完一个任务之后，再次连接执行下一个任务，如：
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

gulp模块的watch方法，用于指点需要监视的文件，一旦文件发生变动，就运行指点任务。
```
gulp.watch('./js/*.js', ['lint', 'script']);
```


## Gulp插件
* 1、编译Sass,生成雪碧图(gulp-compass);

* 2、编译sass(gulp-sass);

* 3、sass浏览器地图(gulp-sourcemaps);

* 4、重命名文件(gulp-rename);

* 5、JS语法检测(gulp-jshint);

* 6、JS丑化(gulp-uglify);

* 7、JS文件合并(gulp-concat);

* 8、图片压缩(gulp-imagemin);

* 9、缓存通知(gulp-cache);

* 10、web服务(gulp-connect);

* 11、压缩CSS(gulp-minify-css);

* 12、css文件引用URL图片加版本号(gulp-make-css-url-version);

* 13、清空文件夹(gulp-clean);

* 14、更新通知(gulp-notify);

* 15、html文件引用加版本号(gulp-rev-append);

* 16、web服务浏览器同步浏览(browser-sync);　　 // 推荐使用这个作为web服务

## gulpfile.js配置文件(举例)
旧版写法
```
const gulp = require('gulp');
const uglify = require('gulp-uglify')         // JS丑化

// 路径变量
var path = {
  // 开发环境
  src: {
      html: './src',
      js: './src/js',
      sass: './src/sass',
      css: './src/css',
      image: './src/images' 
  },
  // 发布环境
  dist: {
      html: './dist',
      js: './dist/js',
      css: './dist/css',
      image: './dist/images' 
  }
};

// 压缩HTML
gulp.task('html', () => {
  return gulp.src(path.src.html+"/*.html")
      .pipe(rev())                    // html 引用文件添加版本号
      .pipe(gulp.dest(path.dist.html))
  
});

// 合并压缩JS文件
gulp.task('script', () => {
  return gulp.src(path.src.js+'/*.js')
      //.pipe(concat('all.js'))            // 合并
      //.pipe(gulp.dest(path.dist.js))
      //.pipe(rename('all.min.js'))        // 重命名
      .pipe(uglify())                    // 压缩
      .pipe(gulp.dest(path.dist.js))
      //.pipe(notify({ message: 'JS合并压缩' }))
});

// 串行
gulp.task("default", gulp.series('html', 'script'))

// gulp.watch(path.src.html+'/*.*', gulp.series('html'));
// gulp.watch(path.src.js+'/*.js',  gulp.series('script'));
```

新版写法：
```
const gulp = require('gulp');
const uglify = require('gulp-uglify')         // JS丑化
const rev = require('gulp-rev-append')

// 压缩HTML
function html(done) {
  gulp.src("./src/*.html")
      .pipe(rev())                    // html 引用文件添加版本号
      .pipe(gulp.dest(path.dist.html))
  done()
}
// 合并压缩JS文件
function script(done) {
  gulp.src('./src/js/*.js')
      //.pipe(concat('all.js'))            // 合并
      //.pipe(gulp.dest(path.dist.js))
      //.pipe(rename('all.min.js'))        // 重命名
      .pipe(uglify())                    // 压缩
      .pipe(gulp.dest(path.dist.js))
      //.pipe(notify({ message: 'JS合并压缩' }))
  done()
}
// gulp.watch(path.src.html+'/*.*', gulp.series(html));
// gulp.watch(path.src.js+'/*.js',  gulp.series(script));
exports.html = gulp.series(html);
exports.default = gulp.series(html, script);
```

## 使用方法

gulp&emsp;&emsp;&emsp;&emsp;&emsp;// 运行default任务

gulp html&emsp;&emsp;&emsp;//运行html打包任务