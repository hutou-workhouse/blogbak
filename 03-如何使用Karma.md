title: 03-如何使用Karma
date: 2016-01-19 19:14:29
categories: 
-	javascript
-	测试

tags: 
-	测试
-	javascript

---

讲述如何使用Karma

<!--more-->

**Karma** 是Google 开源的一个基于Node.js 的 JavaScript 测试执行过程管理工具（Test Runner）。该工具可用于测试所有主流Web浏览器，也可集成到 CI （Continuous integration）工具，也可和其他代码编辑器一起使用。

##  起步
1. 安装
```
$ npm install karma -g
$ npm install -g karma-cli
```
进行配置，最终生成karma-conf.js配置文件
```
$ mkdir karma-sample
$ cd karma-sample
$ karma init karma-conf.js
```
在生成配置文件的过程中，有几个选择是比较重要的！假设我们的源码都在**js**目录下，测试用例都在**test**目录下。
回答测试文件位于哪里这个问题时, 我们键入如下命令:
```
js/**/*.js
test/**/*.js
```
在回答测试框架的时候可以自行选择，例子中使用jasmine框架。
2. 安装包
```
$ npm install
```
2. 生成代码
写一个简单的待测试文件add.js
```
function add(a,b){
  return a+b;
}
```
写一个测试用例add-test.js
```
describe("my great and huge math lib", function () {
  it("should perfectly complete complex addition", function () {
    var result = add(3, 5);
    expect(result).toEqual(9);
  });
});
```
4. 启动运行
```
$ karma start karma-conf.js
```
##  配置文件
下面是一个标准的配置文件，设置在Firefox和Chorme中进行测试
```
// Karma configuration
// Generated on Mon Mar 17 2014 12:46:02 GMT+0800 (CST)
module.exports = function(config) {
  config.set({
    // base path, that will be used to resolve files and exclude
    basePath: '',
    // frameworks to use
    frameworks: ['mocha'],
    // list of files / patterns to load in the browser
    files: [
      'src/**/*.js',
      'test-lib/**/*.js',
      'test/**/*.js'
    ],
    // reporters : ['progress'],
    // list of files to exclude
    exclude: [
    ],
    // test results reporter to use
    // possible values: 'dots', 'progress', 'junit', 'growl', 'coverage'
    reporters: ['spec'],
    // web server port
    port: 9876,
    // enable / disable colors in the output (reporters and logs)
    colors: true,
    // level of logging
    // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
    logLevel: config.LOG_INFO,
    // enable / disable watching file and executing tests whenever any file changes
    autoWatch: true,
    // Start these browsers, currently available:
    // - Chrome
    // - ChromeCanary
    // - Firefox
    // - Opera (has to be installed with `npm install karma-opera-launcher`)
    // - Safari (only Mac; has to be installed with `npm install karma-safari-launcher`)
    // - PhantomJS
    // - IE (only Windows; has to be installed with `npm install karma-ie-launcher`)
    browsers: ['Chrome', 'Firefox'],
    // If browser does not capture in given timeout [ms], kill it
    captureTimeout: 60000,
    // Continuous Integration mode
    // if true, it capture browsers, run tests and exit
    singleRun: false
  });
};
```