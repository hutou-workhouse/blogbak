title: Vue.JS入门篇--过滤器
date: 2015-12-28 17:46:56
categories: 
-	javascript
-	Vue

tags: 
-	Vue
-	javascript
-	技术文章

---

类似AngularJS的过滤器概念

<!--more-->

# 概要
一个 Vue.js 的过滤器本质上是一个函数，这个函数会接收一个值，将其处理并返回。过滤器在指令中由一个管道符 (|) 标记，并可以跟随一个或多个参数：
```
<element directive="expression | filterId [args...]"></element>
```
# 示例
过滤器必须放置在一个指令的值的最后：
```
<span v-text="message | capitalize"></span>
```
你也可以用在 mustache 风格的绑定的内部：
```
<span>{{message | uppercase}}</span>
```
可以串联多个过滤器：
```
<span>{{message | lowercase | reverse}}</span>
```
# 参数
一些过滤器是可以接受参数的。参数用空格分隔开：
```
<span>{{order | pluralize 'st' 'nd' 'rd' 'th'}}</span>
```
```
<input v-on="keyup: submitForm | key 'enter'">
```
纯字符串参数需要用引号包裹。无引号的参数会作为表达式在当前数据作用域内动态计算。