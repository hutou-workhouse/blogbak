title: Vue.JS入门篇--初体验
date: 2015-12-28 17:38:45
categories: 
-	javascript
-	Vue

tags: 
-	Vue
-	javascript
-	技术文章

---

这是一个系列文章的第一篇，先来体验一下

<!--more-->

这是一个我感兴趣的东东，所以准备好好的看看[官方网站](http://vuejs.org/) 。下面的内容基本上都是从官网获取
0. 概念
  这是一个用于创建Web交互界面的库，不是一个大而全的框架，仅仅是一个简单灵活的视图层。

1. 安装：部分环境，诸如 Google Chrome Apps，强制要求内容安全策略 (CSP) 并且不允许使用new Function()来进行表达式求值。在此情况下，你可以用 [CSP 兼容版本](https://github.com/yyx990803/vue/tree/csp/dist)代替
```
$ npm install vue 
$ npm install vue@csp 
```

2. 概念综述
![图片地址](http://012-cn.vuejs.org/images/mvvm.png)
3.1 ViewModel
  一个同步View和Model的对象，使用new Vue()来创建。
```
var vm = new Vue({ /* options */ })
```
3.2 View
被 Vue 实例管理的 DOM 节点
```
vm.$el // The View
```
3.3 Model
一个轻微改动过的原生 JavaScript 对象。Vue.js 中的模型就是普通的 JavaScript 对象 —— 也可以称为**数据对象**。
```
vm.$data // The Model
```
3.4 Directives 
带特殊前缀的 HTML 特性，可以让 Vue.js 对一个 DOM 元素做各种处理。
```
<div v-text="message"></div>
```
3.5 mustache风格绑定
你也可以使用 mustache 风格的绑定，不管在文本中还是在属性中。它们在底层会被转换成 v-text 和 v-attr 的指令
```
<div id="person-{{id}}">Hello {{name}}!</div>
```
**	Internet Explorer 在解析 HTML 时会移除非法的内联 style 属性，所以如果你想支持 IE，请在绑定内联 CSS 的时候始终使用 v-style**
**	一个 image 的 src 属性在赋值时会产生一个 HTTP 请求，所以当模板在第一次被解析时会产生一个 404 请求。这种情况下更适合用 v-attr。**
3.6 Filters
过滤器是用于在更新视图之前处理原始值的函数。它们通过一个“管道”在指令或绑定中进行处理。
```
<div>{{message | capitalize}}</div>
```
3.7 组件Components
在 Vue.js，每个组件都是一个简单的 Vue 实例。一个树形嵌套的各种组件就代表了你的应用程序的各种接口。通过 Vue.extend 返回的自定义构造函数可以把这些组件实例化，不过更推荐的声明式的用法是通过 Vue.component(id, constructor) 注册这些组件。
```
<my-component> <!-- internals handled by my-component --></my-component>
```

3. 简单使用例子
创建HTML文件
```html
<!DOCTYPE html>
        <html>
        <head lang="en">
            <meta charset="UTF-8">
            <title></title>
            <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/0.12.16/vue.min.js"></script>
        </head>
        <body>
        <div id="demo">
            <h1>{{title | uppercase}}</h1>
            <ul>
                <li
                        v-repeat="todos"
                        v-on="click: done = !done"
                        class="{{done ? 'done' : ''}}">
                    {{content}}
                </li>
            </ul>
        </div>
        
        <script>
            var demo = new Vue({
                el: '#demo',
                data: {
                    title: 'todos',
                    todos: [
                        {
                            done: true,
                            content: 'Learn JavaScript'
                        },
                        {
                            done: false,
                            content: 'Learn Vue.js'
                        }
                    ]
                }
            })
        </script>
        </body>
        </html>
```