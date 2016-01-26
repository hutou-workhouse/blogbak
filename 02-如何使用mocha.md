title: 02-如何使用mocha
date: 2016-01-19 19:13:54
categories: 
-	javascript
-	测试

tags: 
-	测试
-	javascript

---

讲述JavaScript的测试框架mocha

<!--more-->

mocha 是一个功能丰富的*javascript测试框架*，可以运行在nodejs和浏览器环境，使异步测试变得简单有趣。mocha *串联运行测试*，允许灵活和精确地报告结果，同时映射未捕获的异常用来纠正测试用例。
支持*TDD/BDD* 的 开发方式，结合 should.js/expect/chai/better-assert *断言库*，能轻松构建各种风格的*测试用例*。

### 安装
```
npm install -g mocha
```

### 起步
```
//模块依赖
var assert = require("assert");
 
//断言条件
describe('Array', function(){
  describe('#indexOf()', function(){
    it('当值不存在时应该返回 -1', function(){
      assert.equal(-1, [1,2,3].indexOf(5));
      assert.equal(-1, [1,2,3].indexOf(0));
    });
  });
});

```
重要的有二步：
1. 加载模块
2. 设置断言： assert.equal(实际值，期望值，返回信息)。当实际与期望相同的时候，显示错误

测试驱动开发  **TDD**
行为驱动开发  **BDD**
exports风格

### 简要说明BDD的使用

BDD认为，不应该针对代码的实现细节写测试，而是要针对行为写测试。BDD测试的是*行为*，即软件应该怎样运行。
BDD接口提供以下方法：
>**describe()**：测试套件
**it()**：测试用例
**before()**：所有测试用例的统一前置动作
**after()**：所有测试用例的统一后置动作
**beforeEach()**：每个测试用例的前置动作
**afterEach()**：每个测试用例的后置动作

```
var expect = require('chai').expect;
 
describe('Array', function(){
  before(function(){
    console.log('在测试之前运行');
  });
 
  describe('#indexOf()', function(){
    it('当值不存在时应该返回 -1', function(){
      expect([1,2,3].indexOf(4)).to.equal(-1);
    });
  });
});
```
运行的命令
```
$ mocha ./step2.js
```
### **hook机制**
hook 就是在测试流程的不同时段触发，比如在整个测试流程之前，或在每个独立测试之前等。
hook也可以理解为是一些逻辑，通常表现为一个函数或者一些声明，当特定的事件触发时 hook 才执行。

提供方法有：before()、beforeEach() after() 和 afterEach()。

**方法解析**：
>**before()**：所有测试用例的统一前置动作
**after()**：所有测试用例的统一后置动作
**beforeEach()**：每个测试用例的前置动作
**afterEach()**：每个测试用例的后置动作

```
var expect = require('chai').expect;

describe('hooks', function() {
  before('可以加入描述',function() {
    //在执行本区块的所有测试之前执行
  });
 
  after('可以加入描述',function() {
    //在执行本区块的所有测试之后执行
  });
 
  beforeEach('可以加入描述',function() {
    //在执行本区块的每个测试之前都执行
  });
 
  afterEach('可以加入描述',function() {
    //在执行本区块的每个测试之后都执行
  });
 
  //测试用例
  describe('#indexOf()', function(){
    it('当值不存在时应该返回 -1', function(){
      expect([1,2,3].indexOf(4)).to.equal(-1);
    });
  }); 
});
```
测试用例占位只要添加一个没有回调的 it() 方法即可：
```
var assert = require('assert');
 
describe('Array', function() {
 
  describe('#indexOf()', function() {
    //同步测试
    it('当值不存在时应该返回 -1', function() {
      assert.equal(-1, [1,2,3].indexOf(5));
      assert.equal(-1, [1,2,3].indexOf(0));
    });
  });
 
  describe('Array', function() {
    describe('#indexOf()', function() {
      //下面是一个挂起的测试
      it('当值不存在时应该返回 -1');
    });
  });
});
```
仅执行指定测试的特性可以让你通过添加 .only() 来指定唯一要执行的测试套件或测试用例：
```
describe('Array', function(){
  describe.only('#indexOf()', function(){
    ...
  })
})
```
或一个指定的测试用例：
```
describe('Array', function(){
  describe('#indexOf()', function(){
    it.only('当值不存在时应该返回 -1', function(){
 
    })
 
    it('当值不存在时应该返回 -1', function(){
 
    })
  })
})
```
**忽略指定测试**
该特性和 .only() 非常相似，通过添加 .skip() 你可以告诉 Mocha 忽略的测试套件或者测试用例（可以有多个）。该操作使得这些操作处于挂起的状态，这比使用注释来的要好，因为你可能会忘记把注释给取消掉。
```
describe('Array', function(){
  describe.skip('#indexOf()', function(){
    ...
  })
})
```
或一个指定的测试用例：
```
describe('Array', function(){
  describe('#indexOf()', function(){
    it.skip('当值不存在时应该返回 -1', function(){
 
    })
 
    it('当值不存在时应该返回 -1', function(){
 
    })
  })
})
```
**动态生成测试**
由于mocha 可以使用 function.prototype.call 和function 表达式*定义测试套件和测试用例*，所以可以动态生成测试用例。
```
var assert = require('assert');
 
function add() {
  return Array.prototype.slice.call(arguments).reduce(function(prev, curr) {
    return prev + curr;
  }, 0);
}
 
describe('add()', function() {
  var tests = [
    {args: [1, 2],       expected: 3},
    {args: [1, 2, 3],    expected: 6},
    {args: [1, 2, 3, 4], expected: 10}
  ];
 
  tests.forEach(function(test) {
    it('correctly adds ' + test.args.length + ' args', function() {
      var res = add.apply(null, test.args);
      assert.equal(res, test.expected);
    });
  });
});
```
### **hook机制**


### 简要说明TDD的使用
TDD（测试驱动开发）组织方式是使用*测试集（suite）和测试（test）*。
每个测试集都有* setup 和 teardown 函数*。这些方法会在测试集中的测试执行前执行，它们的作用是为了避免代码重复以及最大限度使得测试之间相互独立。

TDD接口：
>**suite**：类似BDD中 describe()
**test**：类似BDD中 it()
**setup**：类似BDD中 before()
**teardown**：类似BDD中 after()
**suiteSetup**：类似BDD中 beforeEach()
**suiteTeardown**：类似BDD中 afterEach()

```
var assert = require("assert");
 
suite('Array', function(){
  setup(function(){
    console.log('测试执行前执行');
  });
 
  suite('#indexOf()', function(){
    test('当值不存在时应该返回 -1', function(){
      assert.equal(-1, [1,2,3].indexOf(4));
    });
  });
});
```
运行的命令
```
$ mocha --ui tdd ./step2.js
```
**这里需要明确指定tdd方式**

### 简要说明exports风格的使用
exports类似于nodejs里的*模块语法*,关键字 before, after, beforeEach, 和 afterEach 是特殊保留的，值为对象时是一个*测试套件*，为函数时则是一个*测试用例*。
```
```
