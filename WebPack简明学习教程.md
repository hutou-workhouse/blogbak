title: WebPack 简明学习教程
date: 2015-12-27 22:33:35
categories: 
-	javascript

tags: 
-	webpack
-	javascript
-	技术文章
- 工具

---

简单介绍WebPack的使用方法，未在系统中真实用过，仅仅研究过

<!--more-->

# WebPack是什么
> 1. 一个打包工具
> 2. 一个模块加载工具
> 3. 各种资源都可以当成模块来处理
> 4. 网站 http://webpack.github.io/

如今，越来越多的JavaScript代码被使用在页面上，我们添加很多的内容在浏览器里。如何去很好的组织这些代码，成为了一个必须要解决的难题。

对于模块的组织，通常有如下几种方法：
1. 通过书写在不同文件中，使用script标签进行加载
2. CommonJS进行加载（NodeJS就使用这种方式）
3. AMD进行加载（require.js使用这种方式）
4. ES6模块

**思考：为什么只有JS需要被模块化管理，前台的很多预编译内容，不需要管理吗？**

基于以上的思考，WebPack项目有如下几个目标：
*  将依赖树拆分，保证按需加载
*  保证初始加载的速度
*  所有静态资源可以被模块化
*  可以整合第三方的库和模块
*  可以构造大系统

从下图可以比较清晰的看出WebPack的功能
![这是一个示意图](http://webpack.github.io/assets/what-is-webpack.png)

-------------
# WebPack的特点
1. 丰富的插件，方便进行开发工作
2. 大量的加载器，包括加载各种静态资源
3. 代码分割，提供按需加载的能力
4. 发布工具

---------------

# WebPack的优势
> * webpack 是以 commonJS 的形式来书写脚本滴，但对 AMD/CMD 的支持也很全面，方便旧项目进行代码迁移。
> * 能被模块化的不仅仅是 JS 了。
> * 开发便捷，能替代部分 grunt/gulp 的工作，比如打包、压缩混淆、图片转base64等。
> * 扩展性强，插件机制完善，特别是支持 React 热插拔（见 react-hot-loader ）的功能让人眼前一亮。
--------------

# WebPack的安装

1. 安装命令
```
$ npm install webpack -g
```
2. 使用webpack
```
$ npm init  # 会自动生成一个package.json文件
$ npm install webpack --save-dev #将webpack增加到package.json文件中
```
3. 可以使用不同的版本
```
$ npm install webpack@1.2.x --save-dev
```
4. 如果想要安装开发工具
```
$ npm install webpack-dev-server --save-dev
```

# WebPack的配置
>  每个项目下都必须配置有一个 webpack.config.js ，它的作用如同常规的 gulpfile.js/Gruntfile.js ，就是一个配置项，告诉 webpack 它需要做什么。

下面是一个例子
```
var webpack = require('webpack');
var commonsPlugin = new webpack.optimize.CommonsChunkPlugin('common.js');
module.exports = {
	//插件项
	plugins: [commonsPlugin],
	//页面入口文件配置
	entry: {
		index : './src/js/page/index.js'
	},
	//入口文件输出配置
	output: {
		path: 'dist/js/page',
		filename: '[name].js'
	},
	module: {
		//加载器配置
		loaders: [
			{ test: /\.css$/, loader: 'style-loader!css-loader' },
			{ test: /\.js$/, loader: 'jsx-loader?harmony' },
			{ test: /\.scss$/, loader: 'style!css!sass?sourceMap'},
			{ test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'}
		]
	},
	//其它解决方案配置
	resolve: {
		root: 'E:/github/flux-example/src', //绝对路径
		extensions: ['', '.js', '.json', '.scss'],
		alias: {
			AppStore : 'js/stores/AppStores.js',
			ActionType : 'js/actions/ActionType.js',
			AppAction : 'js/actions/AppAction.js'
		}
	}
}; 
```
1.  plugins 是插件项，这里我们使用了一个 CommonsChunkPlugin的插件，它用于提取多个入口文件的公共脚本部分，然后生成一个 common.js 来方便多页面之间进行复用。
2.   entry 是页面入口文件配置，output 是对应输出项配置 （即入口文件最终要生成什么名字的文件、存放到哪里）
3.  module.loaders 是最关键的一块配置。它告知 webpack 每一种文件都需要使用什么加载器来处理。 **所有加载器需要使用npm来加载**
4.  最后是 resolve 配置，配置查找模块的路径和扩展名和别名（方便书写）

# WebPack开始使用
> 这里有最基本的使用方法，给大家一个感性的认识

1. 正确安装了WebPack，方法可以参考上面
2. 书写entry.js文件
```
document.write("看看如何让它工作！");
```
3. 书写index.html文件
```
<html>
 <head>
 <meta charset="utf-8">
 </head>
 <body>
 <script type="text/javascript" src="bundle.js" charset="utf-8"></script>
 </body>
</html>
```
4. 执行命令，生成bundle.js文件
```
$ webpack ./entry.js bundle.js
```
5. 在浏览器中打开index.html文件，可以正常显示出预期
6. 增加一个content.js文件
```
module.exports = "现在的内容是来自于content.js文件！";
```
7. 修改entry.js文件
```
document.write(require("./content.js"));
```
8. 执行第四步的命令

**进行加载器试验**
1. 增加style.css文件
```
body {
 background: yellow;
}
```
2. 修改entry.js文件
```
require("!style!css!./style.css");
document.write(require("./content.js"));
```
3. 执行命令，安装加载器
```
$ npm install css-loader style-loader   # 安装的时候不使用 -g
```
4. 执行webpack命令，运行看效果
5. 可以在命令行中使用loader 
```
$ webpack ./entry.js bundle.js --module-bind "css=style!css"
```

**使用配置文件**
默认的配置文件为webpack.config.js
1. 增加webpack.config.js文件
```
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```
2. 执行程序
```
$ webpack
```

**发布服务器**
1. 安装服务器
```
$ npm install webpack-dev-server -g
$ webpack-dev-server --progress --colors
```
2. 服务器可以自动生成和刷新，修改代码保存后自动更新画面
```
http://localhost:8080/webpack-dev-server/bundle
```