title: Vue.JS入门篇--列表渲染
date: 2015-12-28 18:26:55
categories: 
-	javascript
-	Vue

tags: 
-	Vue
-	javascript
-	技术文章

---

如何渲染列表

<!--more-->

>你可以使用 v-repeat 指令来基于 ViewModel 上的对象数组渲染列表。对于数组中的每个对象，该指令将创建一个以该对象作为其 $data 对象的子 Vue 实例。这些子实例继承父实例的数据作用域，因此在重复的模板元素中你既可以访问子实例的属性，也可以访问父实例的属性。此外,你还可以通过 $index 属性来获取当前实例对应的数组索引。

```
	<ul id="demo">
	  <li v-repeat="items" class="item-{{$index}}">
	    {{$index}} - {{parentMsg}} {{childMsg}}
	  </li>
	</ul>
```
```
	var demo = new Vue({
	  el: '#demo',
	  data: {
	    parentMsg: 'Hello',
	    items: [
	      { childMsg: 'Foo' },
	      { childMsg: 'Bar' }
	    ]
	  }
	})
```
查看一下效果，应该很容易得到结果

# 块级重复
有时我们可能需要重复一个包含多个节点的块，这时可以用 <template> 标签包裹这个块。这里的 <template> 标签只起到语义上的包裹作用，其本身不会被渲染出来。例如：
```
	<ul>
	  <template v-repeat="list">
	    <li>{{msg}}</li>
	    <li class="divider"></li>
	  </template>
	</ul>
```
# 简单值数组
简单值 (primitive value) 即字符串、数字、boolean 等并非对象的值。对于包含简单值的数组，你可用 $value 直接访问值:
```
	<ul id="tags">
	  <li v-repeat="tags">
	    {{$value}}
	  </li>
	</ul>
```
```
	new Vue({
	  el: '#tags',
	  data: {
	    tags: ['JavaScript', 'MVVM', 'Vue.js']
	  }
	})
```
# 使用别名
有时我们可能想要更明确地访问当前作用域的变量而不是隐式地回退到父作用域。你可以通过提供一个参数给 v-repeat 指令并用它作为将被迭代项的别名:
```
    <ul id="users">
        <li v-repeat="user : users">
            {{user.name}} - {{user.email}}
        </li>
    </ul>
```
```
    var users = new Vue({
        el: '#users',
        data: {
            users: [
                { name: 'Foo Bar', email: 'foo@bar.com' },
                { name: 'John Doh', email: 'john@doh.com' }
            ]
        }
    });
```
# 变异方法
Vue.js 内部对被观察数组的变异方法 (mutating methods，包括 push(), pop(), shift(), unshift(), splice(), sort() 和 reverse()) 进行了拦截，因此调用这些方法也将自动触发视图更新。
```
// 以下操作会触发 DOM 更新
demo.items.unshift({ childMsg: 'Baz' })
demo.items.pop()
```
下面是一个演示的例子，点击按钮的时候数据项会被移除
```
	<!DOCTYPE html>
	<html>
	<head lang="en">
	<meta charset="UTF-8">
	<title></title>
	<script src="http://cdnjs.cloudflare.com/ajax/libs/vue/0.12.16/vue.min.js"></script>
	</head>
	<body>
	    <ul id="tags">
	        <li v-repeat="tags">
	            {{$value}}
	        </li>
	    </ul>
	
	    <input type="button" value="测试" onclick="myClick();">
	<script>
	    var tag = new Vue({
	        el: '#tags',
	        data: {
	            tags: ['标签一', '标签二', '标签三']
	        }
	    });
	
	    function myClick(){
	        tag.tags.pop();
	    }
	</script>
	</body>
	</html>
```
# 扩展方法
Vue.js 给被观察数组添加了两个便捷方法：$set() 和 $remove() 。
你应该避免直接通过索引来设置数据绑定数组中的元素，比如 demo.items[0] = {}，因为这些改动是无法被 Vue.js 侦测到的。你应该使用扩展的 $set() 方法:
```
// 不要用 `demo.items[0] = ...`
demo.items.$set(0, { childMsg: 'Changed!'})
```
$remove() 只是 splice()方法的语法糖。它将移除给定索引处的元素。当参数不是数值时，$remove() 将在数组中搜索该值并删除第一个发现的对应元素。
```
// 删除索引为 0 的元素。
demo.items.$remove(0)
```
```
<!DOCTYPE html>
<html>
<head lang="en">
<meta charset="UTF-8">
<title></title>
<script src="http://cdnjs.cloudflare.com/ajax/libs/vue/0.12.16/vue.min.js"></script>
</head>
<body>
    <ul id="tags">
        <li v-repeat="tags">
            {{$value}}
        </li>
    </ul>

    <input type="button" value="测试" onclick="myClick();">
<script>
    var tag = new Vue({
        el: '#tags',
        data: {
            tags: ['标签一', '标签二', '标签三']
        }
    });

    function myClick(){
        //tag.tags.pop();
        //tag.tags[0] = '修改后的内容不生效';
        tag.tags.$set(0, '修改后的内容生效');
        tag.tags.$remove(1);
    }
</script>
</body>
</html>
```
# 替换数组
当你使用非变异方法，比如filter(), concat() 或 slice()，返回的数组将是一个不同的实例。在此情况下，你可以用新数组替换旧的数组:
```
demo.items = demo.items.filter(function (item) {
  return item.childMsg.match(/Hello/)
})
```
你可能会认为这将导致整个列表的 DOM 被销毁并重新渲染。但别担心，Vue.js 能够识别一个数组元素是否已有关联的 Vue 实例， 并尽可能地进行有效复用。
# 使用track-by
在某些情况下，你可能需要以全新的对象（比如 API 调用所返回的对象）去替换数组。如果你的每一个数据对象都有一个唯一的 id 属性，那么你可以使用 track-by 特性参数给 Vue.js 一个提示，这样它就可以复用已有的具有相同 id 的 Vue 实例和 DOM 节点。
例如：你的数据是这个样子的
```
{
  items: [
    { _uid: '88f869d', ... },
    { _uid: '7496c10', ... }
  ]
}
```
那么你可以像这样给出提示:
```
<div v-repeat="items" track-by="_uid">
  <!-- content -->
</div>
```
在替换 items 数组时，Vue.js 如果碰到一个有 _uid: '88f869d' 的新对象，它就会知道可以直接复用有同样 _uid 的已有实例。**在使用全新数据重新渲染大型 v-repeat 列表时，合理使用 track-by 能极大地提升性能。**
# 遍历对象
你也可以使用 v-repeat 遍历一个对象的所有属性。每个重复的实例会有一个特殊的属性 $key。对于简单值，你也可以象访问数组中的简单值那样使用 $value 属性。
```
    <ul id="repeat-object">
        <li v-repeat="primitiveValues">{{$key}} : {{$value}}</li>
        <li>===</li>
        <li v-repeat="objectValues">{{$key}} : {{msg}}</li>
    </ul>
```
```
    new Vue({
        el: '#repeat-object',
        data: {
            primitiveValues: {
                FirstName: 'John',
                LastName: 'Doe',
                Age: 30
            },
            objectValues: {
                one: {
                    msg: 'Hello'
                },
                two: {
                    msg: 'Bye'
                }
            }
        }
    });
```
在 ECMAScript 5 中，对于给对象添加一个新属性，或是从对象中删除一个属性时，没有机制可以检测到这两种情况。针对这个问题，Vue.js 中的被观察对象会被添加三个扩展方法: $add(key, value), $set(key, value) 和 $delete(key)。这些方法可以被用于在添加 / 删除观察对象的属性时触发对应的视图更新。方法 $add 和 $set 的不同之处在于当指定的键已经在对象中存在时 $add 将提前返回，所以调用 obj.$add(key) 并不会以 undefined 覆盖已有的值。
# 迭代值域
v-repeat 也可以接受一个整数。在这种情况下，它将重复显示一个模板多次。**下面的例子将迭代3次。**
```
    <div id="range">
        <div v-repeat="val">Hi! {{$index}}</div>
    </div>
```
```
    new Vue({
        el: '#range',
        data: {
            val: 3
        }
    });
```

# 数组过滤器
有时候我们只需要显示一个数组的过滤或排序过的版本，而不需要实际改变或重置原始数据。Vue 提供了两个内置的过滤器来简化此类需求： filterBy 和 orderBy。
```
<input v-model="searchText">
<ul>
  <li v-repeat="users | filterBy searchText">{{name}}</li>
</ul>
```