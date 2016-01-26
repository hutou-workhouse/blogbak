title: Glup 简明使用教程
date: 2016-01-04 20:17:51
categories: 
-	javascript
-	Glup

tags: 
-	Glup
-	javascript
-	技术文章
-	工具

---

[Glup](http://www.gulpjs.com.cn/)用自动化构建工具增强你的工作流程！

<!--more-->


同类型的软件还有[Grunt](http://www.gruntjs.net/)。关于两者的区别可以参考这篇文章[Grunt VS Gulp](http://www.w3ctech.com/topic/673)

### 安装：
```
$ npm install gulp -g
$ npm install gulp --save-dev
```
### 安装gulp插件
安装一些插件，完成下列任务：
1. 编译Sass ([gulp-ruby-sass](https://github.com/sindresorhus/gulp-ruby-sass))
2. Autoprefixer ([gulp-autoprefixer](https://github.com/Metrime/gulp-autoprefixer))
3. 缩小化(minify)CSS ([gulp-minify-css](https://github.com/jonathanepollack/gulp-minify-css))
4. JSHint ([gulp-jshint](https://github.com/wearefractal/gulp-jshint))
5. 拼接 ([gulp-concat](https://github.com/wearefractal/gulp-concat))
6. 丑化(Uglify) ([gulp-uglify](https://github.com/terinjokes/gulp-uglify))
1. 图片压缩 ([gulp-imagemin](https://github.com/sindresorhus/gulp-imagemin))
1. 即时重整(LiveReload) ([gulp-livereload](https://github.com/vohof/gulp-livereload))
1. 清理档案 ([gulp-clean](https://github.com/peter-vilja/gulp-clean))
1. 图片快取，只有更改过得图片会进行压缩 ([gulp-cache](https://github.com/jgable/gulp-cache/))
1. 更动通知 ([gulp-notify](https://github.com/mikaelbr/gulp-notify))

```
$ npm install gulp-ruby-sass gulp-autoprefixer gulp-minify-css gulp-jshint gulp-concat gulp-uglify gulp-imagemin gulp-clean gulp-notify gulp-rename gulp-livereload gulp-cache --save-dev
```

### 载入插件
接下来，我们需要建立一个gulpfile.js档案，并且载入这些插件：
```
var gulp = require('gulp'),  
    sass = require('gulp-ruby-sass'),
    autoprefixer = require('gulp-autoprefixer'),
    minifycss = require('gulp-minify-css'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    imagemin = require('gulp-imagemin'),
    rename = require('gulp-rename'),
    clean = require('gulp-clean'),
    concat = require('gulp-concat'),
    notify = require('gulp-notify'),
    cache = require('gulp-cache'),
    livereload = require('gulp-livereload');
```

### 建立任务
下面是任务的最基本形态，在gulpfile.js文件中
```
var gulp = require('gulp');

gulp.task('default', function() {
  // 将你的默认的任务代码放在这
});
```
可以通过如下命令来执行任务：
```
$ gulp default
```
我们设置编译Sass。我们将编译Sass、接著通过Autoprefixer，最后储存结果到我们的目的地。接著产生一个缩小化的.min版本、自动重新整理页面及通知任务已经完成：
```
gulp.task('styles', function() {  
  return gulp.src('src/styles/main.scss')
    .pipe(sass({ style: 'expanded' }))
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(rename({suffix: '.min'}))
    .pipe(minifycss())
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(notify({ message: 'Styles task complete' }));
});
```
### gulp基本API
> + gulp.src(globs[, options])
> + gulp.dest(path[, options])
> + gulp.task(name[, deps], fn)
> + gulp.watch(glob [, opts], tasks) 

#### gulp.src(globs[, options])
输出（Emits）符合所提供的匹配模式（glob）或者匹配模式的数组（array of globs）的文件。 将返回一个 [Vinyl files](https://github.com/wearefractal/vinyl-fs) 的 [stream](http://nodejs.org/api/stream.html) 它可以被 [piped](http://nodejs.org/api/stream.html#stream_readable_pipe_destination_options) 到别的插件中。

#### gulp.dest(path[, options])
能被 pipe 进来，并且将会写文件。并且重新输出（emits）所有数据，因此你可以将它 pipe 到多个文件夹。如果某文件夹不存在，将会自动创建它。
```
gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));
```
文件被写入的路径是以所给的相对路径根据所给的目标目录计算而来。类似的，相对路径也可以根据所给的 base 来计算。 请查看上述的 gulp.src 来了解更多信息。

#### gulp.task(name[, deps], fn)

#### gulp.watch(glob [, opts], tasks) 或 gulp.watch(glob [, opts, cb])
监视文件，并且可以在文件发生改动时候做一些事情。它总会返回一个 EventEmitter 来发射（emit） change 事件。
例如：需要在文件变动后执行的一个或者多个通过 gulp.task() 创建的 task 的名字
```
var watcher = gulp.watch('js/**/*.js', ['uglify','reload']);
watcher.on('change', function(event) {
  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
});
```
```
gulp.watch('js/**/*.js', function(event) {
  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
});
```

### 使用例子：
1. 编译Sass，Autoprefix及缩小化
```
gulp.task('styles', function() {  
  return gulp.src('src/styles/main.scss')
    .pipe(sass({ style: 'expanded' }))
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(rename({suffix: '.min'}))
    .pipe(minifycss())
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(notify({ message: 'Styles task complete' }));
});
```
2. JSHint，拼接及缩小化JavaScript
```
gulp.task('scripts', function() {  
  return gulp.src('src/scripts/**/*.js')
    .pipe(jshint('.jshintrc'))
    .pipe(jshint.reporter('default'))
    .pipe(concat('main.js'))
    .pipe(gulp.dest('dist/assets/js'))
    .pipe(rename({suffix: '.min'}))
    .pipe(uglify())
    .pipe(gulp.dest('dist/assets/js'))
    .pipe(notify({ message: 'Scripts task complete' }));
});
```
3. 图片压缩
```
gulp.task('images', function() {  
  return gulp.src('src/images/**/*')
    .pipe(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true }))
    .pipe(gulp.dest('dist/assets/img'))
    .pipe(notify({ message: 'Images task complete' }));
});
```
4. 收拾乾淨!
在我们进行佈署之前，清除目的地目录并重建档案是一个好主意–以防万一任何已经被删除的来源档案遗留在目的地目录之中:
```
gulp.task('clean', function() {  
  return gulp.src(['dist/assets/css', 'dist/assets/js', 'dist/assets/img'], {read: false})
    .pipe(clean());
});
```
我们可以传入一个目录(或档案)阵列到gulp.src。因为我们不想要读取已经被删除的档案，我们可以加入read: false选项来防止gulp读取档案内容–让它快一些。
5. 预设任务
我们可以建立一个预设任务，当只输入$ gulp指令时执行的任务，这裡执行三个我们所建立的任务:
```
gulp.task('default', ['clean'], function() {  
    gulp.start('styles', 'scripts', 'images');
});
```
注意额外传入gulp.task的阵列。这裡我们可以定义任务相依(task dependencies)。在这个范例中，gulp.start开始任务前会先执行清理(clean)任务。Gulp中所有的任务都是并行(concurrently)执行，并没有先后顺序哪个任务会先完成，所以我们需要确保clean任务在其他任务开始前完成。
6. 看守
为了能够看守档案，并在更动发生后执行相关任务，首先需要建立一个新的任务，使用gulp.watchAPI来看守档案:
```
gulp.task('watch', function() {
    // 看守所有.scss档
    gulp.watch('src/styles/**/*.scss', ['styles']);
    // 看守所有.js档
    gulp.watch('src/scripts/**/*.js', ['scripts']);
    // 看守所有图片档
    gulp.watch('src/images/**/*', ['images']);
});
```
7. 即时重整(LiveReload)
Gulp也可以处理档案更动后，自动重新整理页面。我们需要修改watch任务，设置即时重整伺服器。
```
// 载入外挂
var gulp = require('gulp'),  
    sass = require('gulp-ruby-sass'),
    autoprefixer = require('gulp-autoprefixer'),
    minifycss = require('gulp-minify-css'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    imagemin = require('gulp-imagemin'),
    rename = require('gulp-rename'),
    clean = require('gulp-clean'),
    concat = require('gulp-concat'),
    notify = require('gulp-notify'),
    cache = require('gulp-cache'),
    livereload = require('gulp-livereload');
// 样式
gulp.task('styles', function() {  
  return gulp.src('src/styles/main.scss')
    .pipe(sass({ style: 'expanded', }))
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('dist/styles'))
    .pipe(rename({ suffix: '.min' }))
    .pipe(minifycss())
    .pipe(gulp.dest('dist/styles'))
    .pipe(notify({ message: 'Styles task complete' }));
});
// 脚本
gulp.task('scripts', function() {  
  return gulp.src('src/scripts/**/*.js')
    .pipe(jshint('.jshintrc'))
    .pipe(jshint.reporter('default'))
    .pipe(concat('main.js'))
    .pipe(gulp.dest('dist/scripts'))
    .pipe(rename({ suffix: '.min' }))
    .pipe(uglify())
    .pipe(gulp.dest('dist/scripts'))
    .pipe(notify({ message: 'Scripts task complete' }));
});
// 图片
gulp.task('images', function() {  
  return gulp.src('src/images/**/*')
    .pipe(cache(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true })))
    .pipe(gulp.dest('dist/images'))
    .pipe(notify({ message: 'Images task complete' }));
});
// 清理
gulp.task('clean', function() {  
  return gulp.src(['dist/styles', 'dist/scripts', 'dist/images'], {read: false})
    .pipe(clean());
});
// 预设任务
gulp.task('default', ['clean'], function() {  
    gulp.start('styles', 'scripts', 'images');
});
// 看手
gulp.task('watch', function() {
  // 看守所有.scss档
  gulp.watch('src/styles/**/*.scss', ['styles']);
  // 看守所有.js档
  gulp.watch('src/scripts/**/*.js', ['scripts']);
  // 看守所有图片档
  gulp.watch('src/images/**/*', ['images']);
  // 建立即时重整伺服器
  var server = livereload();
  // 看守所有位在 dist/  目录下的档案，一旦有更动，便进行重整
  gulp.watch(['dist/**']).on('change', function(file) {
    server.changed(file.path);
  });
});
```