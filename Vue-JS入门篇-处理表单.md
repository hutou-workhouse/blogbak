title: Vue.JS入门篇--处理表单
date: 2015-12-28 18:28:38
categories: 
-	javascript
-	Vue

tags: 
-	Vue
-	javascript
-	技术文章

---

讲述表单的处理方法

<!--more-->

# 基本用法
```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/0.12.16/vue.min.js"></script>
</head>
<body>
<form id="demo">
    <!-- text -->
    <p>
        <input type="text" v-model="msg">
        {{msg}}
    </p>
    <!-- checkbox -->
    <p>
        <input type="checkbox" v-model="checked">
        {{checked ? "yes" : "no"}}
    </p>
    <!-- radio buttons -->
    <p>
        <input type="radio" name="picked" value="one" v-model="picked">
        <input type="radio" name="picked" value="two" v-model="picked">
        {{picked}}
    </p>
    <!-- select -->
    <p>
        <select v-model="selected">
            <option>one</option>
            <option>two</option>
        </select>
        {{selected}}
    </p>
    <!-- multiple select -->
    <p>
        <select v-model="multiSelect" multiple>
            <option>one</option>
            <option>two</option>
            <option>three</option>
        </select>
        {{multiSelect}}
    </p>
    <p><pre>data: {{$data | json 2}}</pre></p>
</form>
<script>
    new Vue({
        el: '#demo',
        data: {
            msg      : 'hi!',
            checked  : true,
            picked   : 'one',
            selected : 'two',
            multiSelect: ['one', 'three']
        }
    });

</script>
</body>
</html>
```
# 惰性更新
默认情况下，v-model 会在每个 input 事件之后同步输入的数据。你可以添加一个 lazy 特性，将其改变为在 change 事件之后才进行同步。
```
<!-- 在 "change" 而不是 "input" 事件触发后进行同步 -->
<input v-model="msg" lazy>
```
# 转换为数字
如果你希望将用户的输入自动转换为数字，你可以在 v-model 所在的 input 上添加一个 number 特性。**没有试验成功，不知道什么原因**
```
<input v-model="age" number>
```

# 绑定表达式
当使用 v-model 在单选框和复选框时，被绑定的值可以是布尔值或字符串：
```
<!-- toggle 是 true 或 false -->
<input type="checkbox" v-model="toggle">

<!-- 当单选框被选中时 pick 是 "red" -->
<input type="radio" v-model="pick" value="red">
```
这里有一点小的局限性——有的时候我们想把背后的值绑定到一些别的东西上。你可以按下面这个例子实现：
1. **复选框**
```
<input type="checkbox" v-model="toggle" true-exp="a" false-exp="b">
```
```
// 被选中时：
vm.toggle === vm.a
// 被取消选中时：
vm.toggle === vm.b
```
2. **单选框**
```
<input type="radio" v-model="pick" exp="a">
```
```
// 被选中时：
vm.pick === vm.a
```

# 动态select选项
当你需要为一个 <select> 元素动态渲染列表选项时，推荐将 options 特性和 v-model 指令配合使用，这样当选项动态改变时，v-model 会正确地同步：
```
<select v-model="selected" options="myOptions"></select>
```
在你的数据里，myOptions 应该是一个指向选项数组的路径或是表达式。
这个可选的数组可以包含简单的数组：
```
options = ['a', 'b', 'c']
```
或者可以包含格式如 {text:'', value:''} 的对象。该对象格式允许你设置可选项，让文本展示不同于背后的值：
```
options = [
  { text: 'A', value: 'a' },
  { text: 'B', value: 'b' }
]
```
会被渲染成为
```
<select>
  <option value="a">A</option>
  <option value="b">B</option>
</select>
```
1. **选项组**
另外，数组里对象的格式也可以是 {label:'', options:[...]}。这样的数据会被渲染成为一个 <optgroup>：
```
[
  { label: 'A', options: ['a', 'b']},
  { label: 'B', options: ['c', 'd']}
]
```
```
<select>
  <optgroup label="A">
    <option value="a">a</option>
    <option value="b">b</option>
  </optgroup>
  <optgroup label="B">
    <option value="c">c</option>
    <option value="d">d</option>
  </optgroup>
</select>
```

2. **选项过滤**
你的原始数据很有可能不是这里所要求的格式，因此在动态生成选项时必须进行一些数据转换。为了简化这种转换，options特性支持过滤器。将数据的转换逻辑做成一个可复用的 ***自定义过滤器*** 通常来说是个好主意：
```
Vue.filter('extract', function (value, keyToExtract) {
  return value.map(function (item) {
    return item[keyToExtract]
  })
})
```
```
<select
  v-model="selectedUser"
  options="users | extract 'name'">
</select>
```
上述过滤器将像 [{ name: 'Bruce' }, { name: 'Chuck' }] 这样的原始数据转化为 ['Bruce', 'Chuck']，从而符合动态选项的格式要求。

3. **静态默认选项**
除了动态生成的选项之外，你还可以提供一个静态的默认选项：
```
<select v-model="selectedUser" options="users">
  <option value="">Select a user...</option>
</select>
```
基于 users 动态生成的选项将会被添加到这个静态选项后面。如果 v-model 的绑定值为除 0 之外的伪值，则会自动选中该默认选项。

# 输入debounce 
在一次输入被同步到模型之前，debounce 特性允许你设置一个每次用户事件后的等待延迟。如果在这个延迟到期之前用户再次输入，则不会立刻触发更新，而是重置延迟的等待时间。当每次更新前你要执行繁重作业时会很有用，例如一个基于 ajax 的自动补全功能。**有效的减少重复无用的提交**
```
<input v-model="msg" debounce="500">
```
注意 debounce 参数并不对用户的输入事件进行 debounce：它只对底层数据的 “写入” 操作起作用。因此当使用 debounce 时，你应该用 vm.$watch() 而不是 v-on 来响应数据变化。
