title: 04-Istanbul代码覆盖率
date: 2016-01-19 19:15:07
categories: 
-	javascript
-	测试

tags: 
-	测试
-	javascript

---

讲述如何使用Istanbul

<!--more-->

测试的时候，我们常常关心，是否所有代码都测试到了。

这个指标就叫做"代码覆盖率"（code coverage）。它有四个测量维度。
>**行覆盖率**（line coverage）：是否每一行都执行了？
**函数覆盖率**（function coverage）：是否每个函数都调用了？
**分支覆盖率**（branch coverage）：是否每个if代码块都执行了？
**语句覆盖率**（statement coverage）：是否每个语句都执行了？

[官方地址](https://github.com/gotwarlost/istanbul)

## 起步
1. 安装
```
$ npm install -g istanbul
```
2. 简单例子
编写一个简单的程序 test.js
```
var test1 = "Hello ";
var test2 = "World! ";
console.info(test1 + test2);
function myfunc(){
	console.info('this is a test.');
}
```
运行程序，并且在coverage目录下生成报告
```
$ istanbul cover test.js
```

## 覆盖率设置
完美的覆盖率当然是 100%，但是现实中很难达到。需要有一个门槛，衡量覆盖率是否达标。
**istanbul check-coverage** 命令用来设置门槛，同时检查当前代码是否达标。
```
$ istanbul check-coverage --statement 90
```
上面命令设置语句覆盖率的门槛是 90% 
```
$ istanbul check-coverage --statement -1
```
我们还可以设置绝对值门槛，比如只允许有一个语句没有被覆盖到
```
$ istanbul check-coverage --statement -5 --branch -3 --function 100
```
上面命令设置了3个覆盖率门槛：5个语句、3个 if 代码块、100%的函数。注意，这三个门槛是"与"（and）的关系，只要有一个没有达标，就会报错。