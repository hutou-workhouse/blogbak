title: npm 简明使用教程
date: 2016-01-10 08:56:18
categories: 
-	javascript

tags: 
-	webpack
-	javascript
-	技术文章
- 工具

---

node.js的包管理工具

<!--more-->

## 简单介绍

NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：
1. 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
2. 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
3. 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

由于新版的nodejs已经集成了npm，所以之前npm也一并安装好了。同样可以通过输入 **"npm -v" **来测试是否成功安装。

## 常用命令
npm 安装 Node.js 模块语法格式如下：

**本地安装**
```
$ npm install <Module Name>
```
将安装包放在 ./node_modules 下（运行 npm 命令时所在的目录），如果没有 node_modules 目录，会在当前执行 npm 命令的目录下生成 node_modules 目录。 
可以通过 require() 来引入本地安装的包。 

**全局安装**
```
$ npm install <Module Name> -g
```
将安装包放置在如下位置：
Linux： /usr/local 
Windows ： C:\Users\Administrator\node_modules
***注意***：如果是在windows下，需要设置用户Path=C:\Users\Administrator\AppData\Roaming\npm。**如不设置，安装的命令无法执行，路径找不到**

**查看安装包**
```
# 查看安装包
$ npm ls   
# 查看全局安装包
$ npm ls -g 
```
**卸载模块**
```
$ npm uninstall <Module Name>
```
**更新模块**
```
$ npm update <Module Name>
```

**搜索模块**
```
$ npm search <Module Name>
```

**使用package.json**
package.json 位于模块的目录下，用于定义包的属性。

```
# 在当前模块目录下生产package.json文件
$ npm init
# 安装当前package.json中定义的模块
$ npm install
```

## 注册与提交
我们可以使用以下命令在 npm 资源库中注册用户（使用邮箱注册）：
```
$ npm adduser
Username: mcmohd
Password:
Email: (this IS public) mcmohd@gmail.com
```
然后就可以发表模块
```
$ npm publish
```

## 在墙内使用
由于众所周知的原因，很多时候npm使用出问题。这个时候可以选择使用[淘宝镜像](http://npm.taobao.org/)
镜像使用方法（三种办法任意一种都能解决问题，建议使用第三种，将配置写死，下次用的时候配置还在）:
1. 通过config命令
```
npm config set registry https://registry.npm.taobao.org 
npm info underscore （如果上面配置正确这个命令会有字符串response）
```
2. 命令行指定
```
npm --registry https://registry.npm.taobao.org info underscore 
```
3. 编辑 ~/.npmrc 加入下面内容
```
registry = https://registry.npm.taobao.org
```

有一点必须注意：**如果可能使用publish命令，需要将设置更换回来**

另外，可以选择使用**cnpm**

## NPM本地私服
需要一个简单的NPM本地管理
> 1. 下载依赖速度要快 
2. 不会因为npm官方镜像挂掉而影响开发
3. 私有模块管理

有两个模块可以解决这个问题：[npm_lazy](https://github.com/mixu/npm_lazy) 和 [sinopia ](https://github.com/rlidwka/sinopia) ，这两个模块的实现彻底消除了之前完整镜像npm官方的痛，几乎是零配置，他们基本的思路基本一致：在本地运行一个服务器实例，初始化一个空的“仓库”，我们无需关心它是什么样的仓库，当用户向本地服务器发起请求时，先检查本地是否有现成的已更新的包，有则从本地仓库返回，无则从官方或指定的镜像下载请求所需要的包缓存到本地并返回给用户。

### npm_lazy的使用
我们选择npm_lazy快速的架设一个本地npm cache服务
```
# 安装服务
$ npm install -g npm_lazy
# 运行程序
$ npm_lazy
```
如果在内网中作为私服使用，则需要进行一下配置
```
var path = require('path'),
    homePath = path.normalize(process.env[(process.platform == 'win32') ? 'USERPROFILE' : 'HOME']);

module.exports = {
  // Logging config
  loggingOpts: {
    // Print to stdout with colors
    logToConsole: true,
    // Write to file
    logToFile: false,

    // This should be a file path.
    filename: homePath + '/npm_lazy.log'
  },

  // Cache config

  // `cacheDirectory`: Directory to store cached packages.
  //
  // Note: Since any relative path is resolved relative to the current working
  // directory when the server is started, you should use a full path.

  cacheDirectory: homePath + '/.npm_lazy',

  // `cacheAge`: maximum age before an index is refreshed from remoteUrl
  // - negative value means no refresh (e.g. once cached, never update the package.json metadata)
  // - zero means always refresh (e.g. always ask the registry for metadata)
  // - positive value means refresh every n milliseconds
  //   (e.g. 60 * 60 * 1000 = expire metadata every 60 minutes)
  //
  // Note: if you want to use `npm star` and other methods which update
  // npm metadata, you will need to set cacheAge to 0. npm generally wants the latest
  // package metadata version so caching package metadata will interfere with it.

  // Recommended setting: 0
  cacheAge: 0,

  // Request config

  // max milliseconds to wait for each HTTP response
  httpTimeout: 10000,
  // maximum number of retries per HTTP resource to get
  maxRetries: 5,
  // whether or not HTTPS requests are checked against Node's list of CAs
  // set false if you are using your own npm mirror with a self-signed SSL cert
  rejectUnauthorized: true,

  // Remote and local URL

  // 这里的设置要和下面的地址和端口保持一致   
  externalUrl: 'http://192.168.1.6:8888',
  // registry url with trailing /
  remoteUrl: 'https://registry.npmjs.org/',
  // bind port and host
  // 这里是服务的端口，根据需要设置
  port: 8888,
  // 这里是内网地址，根据情况设置
  host: '192.168.1.6',

  // Proxy config
  // You can also configure this using the http_proxy and https_proxy environment variables
  // cf. https://wiki.archlinux.org/index.php/proxy_settings
  proxy: {
    // http: 'http://1.2.3.4:80/',
    // https: 'http://4.3.2.1:80/'
  }
};
```
当服务跑起来之后，需要将镜像地址进行设置
```
npm config set registry http://192.168.1.6:8888/
```