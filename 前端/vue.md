# Vue 语法、事件、表单

- 开发版本
  - 包含完整的警告和调试模式：https://vuejs.org/js/vue.js
  - 删除了警告，30.96KB min + gzip：https://vuejs.org/js/vue.min.js
- CDN
  - `<script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>`
  - `<script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.min.js"></script>`

```java
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>第一个 Vue 应用程序</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
</head>
<body>
    <div id="vue">
        {{message}}
 		<h1 v-if="ok">YES</h1>
    <h1 v-else>NO</h1>
    
    <h1 v-if="type === 'A'">A</h1>
    <h1 v-else-if="type === 'B'">B</h1>
    <h1 v-else-if="type === 'C'">C</h1>
    <h1 v-else>你看不见我</h1>
      
    <li v-for="item in items">
        {{ item.message }}
    </li>
    
    <button v-on:click="sayHi">点我</button>
      
      单行文本：<input type="text" v-model="message" />&nbsp;&nbsp;单行文本是：{{message}}
			多行文本：<textarea v-model="message"></textarea>&nbsp;&nbsp;多行文本是：{{message}}
			单复选框：<input type="checkbox" id="checkbox" v-model="checked">&nbsp;&nbsp;<label for="checkbox">{{ checked }}</label>
       多复选框：
    <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
    <label for="jack">Jack</label>
    <input type="checkbox" id="john" value="John" v-model="checkedNames">
    <label for="john">John</label>
    <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
    <label for="mike">Mike</label>
    <span>选中的值: {{ checkedNames }}</span>
      
     单选按钮：
    <input type="radio" id="one" value="One" v-model="picked">
    <label for="one">One</label>
    <input type="radio" id="two" value="Two" v-model="picked">
    <label for="two">Two</label>
    <span>选中的值: {{ picked }}</span>
      
      下拉框：
    <select v-model="selected">
        <option disabled value="">请选择</option>
        <option>A</option>
        <option>B</option>
        <option>C</option>
    </select>
    <span>选中的值: {{ selected }}</span>
      **注意：如果 `v-model` 表达式的初始值未能匹配任何选项，`` 元素将被渲染为“未选中”状态。在 iOS 中，这会使用户无法选择第一个选项。因为这样的情况下，iOS 不会触发 change 事件。因此，更推荐像上面这样提供一个值为空的禁用选项。**
    </div>

<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data: {
            message: 'Hello Vue!',
            ok: true,
          	type: 'A',
            items: [
                {message: 'Foo'},
                {message: 'Bar'}
            ],
          	checked: false,
         	 	checkedNames: [],
          	picked: '',
          	selected: ''
        },
        // 在 `methods` 对象中定义方法
        methods: {
            sayHi: function (event) {
                // `this` 在方法里指向当前 Vue 实例
                alert(this.message);
            }
        }
    });
</script>
</body>
</html>
```

- `el:'#vue'`：绑定元素的 ID
- `data:{message:'Hello Vue!'}`：数据对象中有一个名为 `message` 的属性，并设置了初始值 `Hello Vue!`



# Vue 实例的生命周期

## 什么是生命周期

Vue 实例有一个完整的生命周期，也就是从开始创建、初始化数据、编译模板、挂载 DOM、渲染→更新→渲染、卸载等一系列过程，我们称这是 Vue 的生命周期。通俗说就是 Vue 实例从创建到销毁的过程，就是生命周期。

在 Vue 的整个生命周期中，它提供了一系列的事件，可以让我们在事件触发时注册 JS 方法，可以让我们用自己注册的 JS 方法控制整个大局，在这些事件响应方法中的 this 直接指向的是 Vue 的实例。

![img](https://www.funtl.com/assets1/6aedb651gy1fmncxvp4doj20xc2cfaim.jpg)

> 特别值得注意的是 `created` 钩子函数和 `mounted` 钩子函数的区别

## 钩子函数的触发时机

### beforeCreate

在实例初始化之后，数据观测(data observer) 和 event/watcher 事件配置之前被调用。

### created

实例已经创建完成之后被调用。在这一步，实例已完成以下的配置：数据观测(data observer)，属性和方法的运算， watch/event 事件回调。然而，挂载阶段还没开始，$el 属性目前不可见。

### beforeMount

在挂载开始之前被调用：相关的 render 函数首次被调用。

### mounted

el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。

### beforeUpdate

数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。 你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。

### updated

由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。

当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。然而在大多数情况下，你应该避免在此期间更改状态，因为这可能会导致更新无限循环。该钩子在服务器端渲染期间不被调用。

### beforeDestroy

实例销毁之前调用。在这一步，实例仍然完全可用。

### destroyed

Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。该钩子在服务器端渲染期间不被调用。

#Axios 

Axios 是一个开源的可以用在浏览器端和 NodeJS 的异步通信框架，她的主要作用就是实现 AJAX 异步通信，其功能特点如下：

- 从浏览器中创建 `XMLHttpRequests`
- 从 `node.js` 创建 `http` 请求
- 支持 `Promise API`
- 拦截请求和响应
- 转换请求数据和响应数据
- 取消请求
- 自动转换 `JSON` 数据
- 客户端支持防御 `XSRF`（跨站请求伪造）

GitHub：https://github.com/axios/axios

先在项目里模拟一段 JSON 数据，创建名为data.json数据内容如下：

```json
{
  "name": "广州千锋",
  "url": "http://www.funtl.com",
  "page": 88,
  "isNonProfit": true,
  "address": {
    "street": "元岗路.",
    "city": "广东广州",
    "country": "中国"
  },
  "links": [
    {
      "name": "Google",
      "url": "http://www.google.com"
    },
    {
      "name": "Baidu",
      "url": "http://www.baidu.com"
    },
    {
      "name": "SoSo",
      "url": "http://www.SoSo.com"
    }
  ]
}
```

### 创建 HTML

```html
<div id="vue">
    <div>名称：{{info.name}}</div>
    <div>地址：{{info.address.country}}-{{info.address.city}}-{{info.address.street}}</div>
    <div>链接：<a v-bind:href="info.url" target="_blank">{{info.url}}</a> </div>
</div>
```



注：在这里使用了 `v-bind` 将 `a:href` 的属性值与 Vue 实例中的数据进行绑定

### 引入 JS 文件

```html
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

### JavaScript

```javascript
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data() {
            return {
                info: {
                    name: null,
                    address: {
                        country: null,
                        city: null,
                        street: null
                    },
                    url: null
                }
            }
        },
        mounted() {
            axios
                .get('data.json')
                .then(response => (this.info = response.data));
        }
    });
</script>
```



使用 `axios` 框架的 `get` 方法请求 AJAX 并自动将数据封装进了 Vue 实例的数据对象中

#### 数据对象

这里的数据结构与 JSON 数据结构是匹配的

```json
info: {
    name: null,
    address: {
        country: null,
        city: null,
        street: null
    },
    url: null
}
```



#### 调用 `get` 请求

调用 `axios` 的 `get` 请求并自动装箱数据

```javascript
axios
    .get('data.json')
    .then(response => (this.info = response.data));
```



# 组件

## 什么是组件

组件是可复用的 Vue 实例，说白了就是一组可以重复使用的模板，跟 `JSTL` 的自定义标签、`Thymeleaf` 的 `th:fragment` 以及 `Sitemesh3` 框架有着异曲同工之妙。通常一个应用会以一棵嵌套的组件树的形式来组织：

![img](https://www.funtl.com/assets/Lusifer201812220001.png)

例如，你可能会有页头、侧边栏、内容区等组件，每个组件又包含了其它的像导航链接、博文之类的组件。

## 第一个 Vue 组件

**注意：在实际开发中，我们并不会用以下方式开发组件，而是采用 `vue-cli` 创建 `.vue` 模板文件的方式开发，以下方法只是为了让大家理解什么是组件。**

### [#](https://www.funtl.com/zh/vue/组件基础.html#使用-vue-component-方法注册组件)使用 `Vue.component()` 方法注册组件

#### [#](https://www.funtl.com/zh/vue/组件基础.html#javascript)JavaScript

```javascript
<script type="text/javascript">
    // 先注册组件
    Vue.component('my-component-li', {
        template: '<li>Hello li</li>'
    });

    // 再实例化 Vue
    var vm = new Vue({
        el: '#vue'
    });
</script>
```



#### [#](https://www.funtl.com/zh/vue/组件基础.html#html)HTML

```html
<div id="vue">
    <ul>
        <my-component-li></my-component-li>
    </ul>
</div>
```



#### [#](https://www.funtl.com/zh/vue/组件基础.html#说明)说明

- `Vue.component()`：注册组件
- `my-component-li`：自定义组件的名字
- `template`：组件的模板

#### [#](https://www.funtl.com/zh/vue/组件基础.html#测试效果)测试效果

![img](https://www.funtl.com/assets/Lusifer_20181222232831.png)

### [#](https://www.funtl.com/zh/vue/组件基础.html#使用-props-属性传递参数)使用 `props` 属性传递参数

像上面那样用组件没有任何意义，所以我们是需要传递参数到组件的，此时就需要使用 `props` 属性了

**注意：默认规则下 `props` 属性里的值不能为大写；感谢来自 [Java微服务技术交流群2](https://shang.qq.com/wpa/qunwpa?idkey=7a848842e23f9a9178a7ec3c85c579b9c7acb41ac7c2ee65451152bb36b125d6) 的群友 [CV战士蛋蛋面] 帮助大家踩坑；**

#### [#](https://www.funtl.com/zh/vue/组件基础.html#javascript-2)JavaScript

```javascript
<script type="text/javascript">
    // 先注册组件
    Vue.component('my-component-li', {
        props: ['item'],
        template: '<li>Hello {{item}}</li>'
    });

    // 再实例化 Vue
    var vm = new Vue({
        el: '#vue',
        data: {
            items: ["张三", "李四", "王五"]
        }
    });
</script>
```



#### [#](https://www.funtl.com/zh/vue/组件基础.html#html-2)HTML

```html
<div id="vue">
    <ul>
        <my-component-li v-for="item in items" v-bind:item="item"></my-component-li>
    </ul>
</div>
```



#### [#](https://www.funtl.com/zh/vue/组件基础.html#说明-2)说明

- `v-for="item in items"`：遍历 Vue 实例中定义的名为 `items` 的数组，并创建同等数量的组件

- `v-bind:item="item"`：将遍历的 `item` 项绑定到组件中 `props` 定义的名为 `item` 属性上；`=` 号左边的 `item` 为 `props` 定义的属性名，右边的为 `item in items` 中遍历的 `item` 项的值

  

### 完整的 HTML

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>布局篇 组件基础</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
</head>
<body>

<div id="vue">
    <ul>
        <my-component-li v-for="item in items" v-bind:item="item"></my-component-li>
    </ul>
</div>

<script type="text/javascript">
    // 先注册组件
    Vue.component('my-component-li', {
        props: ['item'],
        template: '<li>Hello {{item}}</li>'
    });

    // 再实例化 Vue
    var vm = new Vue({
        el: '#vue',
        data: {
            items: ["张三", "李四", "王五"]
        }
    });
</script>
</body>
</html>
```

# 计算属性

## 什么是计算属性

计算属性的重点突出在 `属性` 两个字上（**属性是名词**），首先它是个 `属性` 其次这个属性有 `计算` 的能力（**计算是动词**），这里的 `计算` 就是个函数；简单点说，它就是一个能够将计算结果缓存起来的属性（**将行为转化成了静态的属性**），仅此而已；

## [#](https://www.funtl.com/zh/vue/计算属性.html#计算属性与方法的区别)计算属性与方法的区别

### [#](https://www.funtl.com/zh/vue/计算属性.html#完整的-html)完整的 HTML

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>布局篇 计算属性</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
</head>
<body>

<div id="vue">
    <p>调用当前时间的方法：{{currentTime1()}}</p>
    <p>当前时间的计算属性：{{currentTime2}}</p>
</div>

<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data: {
            message: 'Hello Vue'
        },
        methods: {
            currentTime1: function () {
                return Date.now();
            }
        },
        computed: {
            currentTime2: function () {
                this.message;
                return Date.now();
            }
        }
    });
</script>
</body>
</html>
```



#### [#](https://www.funtl.com/zh/vue/计算属性.html#说明)说明

- `methods`：定义方法，调用方法使用 `currentTime1()`，需要带括号
- `computed`：定义计算属性，调用属性使用 `currentTime2`，不需要带括号；`this.message` 是为了能够让 `currentTime2` 观察到数据变化而变化

**注意：`methods` 和 `computed` 里不能重名**

调用方法时，每次都需要进行计算，既然有计算过程则必定产生系统开销，那如果这个结果是不经常变化的呢？此时就可以考虑将这个结果缓存起来，采用计算属性可以很方便的做到这一点；**计算属性的主要特性就是为了将不经常变化的计算结果进行缓存，以节约我们的系统开销**



# 内容分发与自定义事件

## [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-Vue-布局篇-内容分发与自定事件](https://www.bilibili.com/video/av43659820/)
- [【视频】Vue 渐进式 JavaScript 框架-Vue-入门总结](https://www.bilibili.com/video/av43660011/)

## [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#vue-中的内容分发)Vue 中的内容分发

在 Vue.js 中我们使用 `` 元素作为承载分发内容的出口，作者称其为 `插槽`，可以应用在组合组件的场景中

### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#利用插槽功能实现一个组合组件)利用插槽功能实现一个组合组件

比如准备制作一个待办事项组件（`todo`），该组件由待办标题（`todo-title`）和待办内容（`todo-items`）组成，但这三个组件又是相互独立的，该如何操作呢？

#### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#定义一个名为-todo-的待办事项组件)定义一个名为 `todo` 的待办事项组件

```javascript
Vue.component('todo', {
    template: '<div>\
                    <slot name="todo-title"></slot>\
                    <ul>\
                        <slot name="todo-items"></slot>\
                    </ul>\
               </div>'
});
```



该组件中放置了两个插槽，分别为 `todo-title` 和 `todo-items`

#### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#定义一个名为-todo-title-的待办标题组件)定义一个名为 `todo-title` 的待办标题组件

```javascript
Vue.component('todo-title', {
    props: ['title'],
    template: '<div>{{title}}</div>'
});
```



#### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#定义一个名为-todo-items-的待办内容组件)定义一个名为 `todo-items` 的待办内容组件

```javascript
Vue.component('todo-items', {
    props: ['item', 'index'],
    template: '<li>{{index + 1}}. {{item}}</li>'
});
```



#### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#实例化-vue-并初始化数据)实例化 Vue 并初始化数据

```javascript
var vm = new Vue({
    el: '#vue',
    data: {
        todoItems: ['《刀剑神域3》', '《关于我转生成为史莱姆这件事》', '《实力至上主义教室》']
    }
});
```



#### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#html)HTML

```html
<div id="vue">
    <todo>
        <todo-title slot="todo-title" title="今日动漫推荐"></todo-title>
        <todo-items slot="todo-items" v-for="(item, index) in todoItems" v-bind:item="item" v-bind:index="index" :key="index"></todo-items>
    </todo>
</div>
```



此时，我们的 `todo-title` 和 `todo-items` 组件分别被分发到了 `todo` 组件的 `todo-title` 和 `todo-items` 插槽中

## 使用自定义事件删除待办事项

通过以上代码不难发现，数据项在 Vue 的实例中，但删除操作要在组件中完成，那么组件如何才能删除 Vue 实例中的数据呢？此时就涉及到参数传递与事件分发了，Vue 为我们提供了自定义事件的功能很好的帮助我们解决了这个问题；使用 `this.$emit('自定义事件名', 参数)`，操作过程如下

### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#修改创建-vue-实例代码)修改创建 Vue 实例代码

```javascript
var vm = new Vue({
    el: '#vue',
    data: {
        todoItems: ['《刀剑神域3》', '《关于我转生成为史莱姆这件事》', '《实力至上主义教室》']
    },
    methods: {
        // 该方法可以被模板中自定义事件触发
        removeTodoItems: function (index) {
            console.log("删除 " + this.todoItems[index] + " 成功");
            // splice() 方法向/从数组中添加/删除项目，然后返回被删除的项目，其中 index 为添加/删除项目的位置，1 表示删除的数量
            this.todoItems.splice(index, 1);
        }
    }
});
```



增加了 `methods` 对象并定义了一个名为 `removeTodoItems` 的方法

### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#修改-todo-items-待办内容组件的代码)修改 `todo-items` 待办内容组件的代码

```javascript
Vue.component('todo-items', {
    props: ['item', 'index'],
    template: '<li>{{index + 1}}. {{item}} <button @click="remove">删除</button></li>',
    methods: {
        remove: function (index) {
            // 这里的 remove 是自定义事件的名称，需要在 HTML 中使用 v-on:remove 的方式指派
            this.$emit('remove', index);
        }
    }
});
```



增加了 `删除` 元素并绑定了组件中定义的 `remove` 事件

### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#修改-todo-items-待办内容组件的-html-代码)修改 `todo-items` 待办内容组件的 HTML 代码

```html
<todo-items slot="todo-items" v-for="(item, index) in todoItems" v-bind:item="item" v-bind:index="index" :key="index" v-on:remove="removeTodoItems(index)"></todo-items>
```



增加了 `v-on:remove="removeTodoItems(index)"` 自定义事件，该事件会调用 Vue 实例中定义的名为 `removeTodoItems` 的方法

### [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#测试效果-2)测试效果

![img](https://www.funtl.com/assets/Lusifer_20181223160154.png)

![img](https://www.funtl.com/assets/Lusifer_20181223160252.png)

## [#](https://www.funtl.com/zh/vue/内容分发与自定义事件.html#完整的-html)完整的 HTML

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>布局篇 内容分发与自定义事件</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
</head>
<body>

<div id="vue">
    <todo>
        <todo-title slot="todo-title" title="今日动漫推荐"></todo-title>
        <todo-items slot="todo-items" v-for="(item, index) in todoItems" v-bind:item="item" v-bind:index="index" :key="index" v-on:remove="removeTodoItems(index)"></todo-items>
    </todo>
</div>

<script type="text/javascript">
    // 定义一个待办事项组件
    Vue.component('todo', {
        template: '<div>\
                        <slot name="todo-title"></slot>\
                        <ul>\
                            <slot name="todo-items"></slot>\
                        </ul>\
                   </div>'
    });

    // 定义一个待办事项标题组件
    Vue.component('todo-title', {
        props: ['title'],
        template: '<div>{{title}}</div>'
    });

    // 定义一个待办事项内容组件
    Vue.component('todo-items', {
        props: ['item', 'index'],
        template: '<li>{{index + 1}}. {{item}} <button @click="remove">删除</button></li>',
        methods: {
            remove: function (index) {
                this.$emit('remove', index);
            }
        }
    });

    var vm = new Vue({
        el: '#vue',
        data: {
            todoItems: ['《刀剑神域3》', '《关于我转生成为史莱姆这件事》', '《实力至上主义教室》']
        },
        methods: {
            // 该方法可以被模板中自定义事件触发
            removeTodoItems: function (index) {
                console.log("删除 " + this.todoItems[index] + " 成功");
                this.todoItems.splice(index, 1);
            }
        }
    });
</script>
</body>
</html>
```

# 第一个 vue-cli 应用程序

## [#](https://www.funtl.com/zh/vue-cli/#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-Vuecli-第一个 vuecli 应用程序](https://www.bilibili.com/video/av43993793/)

## [#](https://www.funtl.com/zh/vue-cli/#什么是-vue-cli)什么是 vue-cli

`vue-cli` 官方提供的一个脚手架（预先定义好的目录结构及基础代码，咱们在创建 Maven 项目时可以选择创建一个骨架项目，这个骨架项目就是脚手架；可以参考我以前写的 [LeeSite 项目骨架生成工具](https://github.com/topsale/leesite-archetype-webapp)），用于快速生成一个 vue 的项目模板

### [#](https://www.funtl.com/zh/vue-cli/#主要功能)主要功能

- 统一的目录结构
- 本地调试
- 热部署
- 单元测试
- 集成打包上线

### [#](https://www.funtl.com/zh/vue-cli/#环境准备)环境准备

- Node.js（>= 6.x，首选 8.x）
- git

### [#](https://www.funtl.com/zh/vue-cli/#安装-vue-cli)安装 vue-cli

- 安装 Node.js

请自行前往 http://nodejs.cn/download 官网下载安装，此处不再赘述

![img](https://www.funtl.com/assets/Lusifer_20181224052651.png)

- 安装 Node.js 淘宝镜像加速器（`cnpm`）

```bash
npm install cnpm -g

# 或使用如下语句解决 npm 速度慢的问题
npm install --registry=https://registry.npm.taobao.org
```

![img](https://www.funtl.com/assets/Lusifer_20181224053021.png)

- 安装 vue-cli

```bash
cnpm install vue-cli -g
```

- 测试是否安装成功

```bash
# 查看可以基于哪些模板创建 vue 应用程序，通常我们选择 webpack
vue list
```

![img](https://www.funtl.com/assets/Lusifer_20181224053315.png)

## [#](https://www.funtl.com/zh/vue-cli/#第一个-vue-cli-应用程序-2)第一个 vue-cli 应用程序

### [#](https://www.funtl.com/zh/vue-cli/#创建一个基于-webpack-模板的-vue-应用程序)创建一个基于 webpack 模板的 vue 应用程序

```bash
# 这里的 myvue 是项目名称，可以根据自己的需求起名
vue init webpack myvue
```

![img](https://www.funtl.com/assets/Lusifer_20181224054035.png)

#### [#](https://www.funtl.com/zh/vue-cli/#说明)说明

- `Project name`：项目名称，默认 `回车` 即可
- `Project description`：项目描述，默认 `回车` 即可
- `Author`：项目作者，默认 `回车` 即可
- `Install vue-router`：是否安装 `vue-router`，选择 `n` 不安装（后期需要再手动添加）
- `Use ESLint to lint your code`：是否使用 `ESLint` 做代码检查，选择 `n` 不安装（后期需要再手动添加）
- `Set up unit tests`：单元测试相关，选择 `n` 不安装（后期需要再手动添加）
- `Setup e2e tests with Nightwatch`：单元测试相关，选择 `n` 不安装（后期需要再手动添加）
- `Should we run npm install for you after the project has been created`：创建完成后直接初始化，选择 `n`，我们手动执行

### [#](https://www.funtl.com/zh/vue-cli/#初始化并运行)初始化并运行

```bash
cd myvue
npm install
npm run dev
```

# vue-cli 目录结构

## [#](https://www.funtl.com/zh/vue-cli/vue-cli-目录结构.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-Vuecli-目录结构](https://www.bilibili.com/video/av43993862/)

## [#](https://www.funtl.com/zh/vue-cli/vue-cli-目录结构.html#概述)概述

![img](https://www.funtl.com/assets/Lusifer_20190101111159.png)

- build 和 config：WebPack 配置文件

- node_modules：用于存放 `npm install` 安装的依赖文件

- **src：** 项目源码目录

- static：静态资源文件

- .babelrc：Babel 配置文件，主要作用是将 ES6 转换为 ES5

- .editorconfig：编辑器配置

- eslintignore：需要忽略的语法检查配置文件

- .gitignore：git 忽略的配置文件

- .postcssrc.js：css 相关配置文件，其中内部的 `module.exports` 是 NodeJS 模块化语法

- index.html：首页，仅作为模板页，实际开发时不使用

- package.json：项目的配置文件

  - name：项目名称
  - version：项目版本
  - description：项目描述
  - author：项目作者
  - scripts：封装常用命令
  - dependencies：生产环境依赖
  - devDependencies：开发环境依赖

# vue-cli src 目录

  ## [#](https://www.funtl.com/zh/vue-cli/vue-cli-src.html#概述)概述

  `src` 目录是项目的源码目录，所有代码都会写在这里

  ![img](https://www.funtl.com/assets1/Lusifer_20190106170155.png)

  ## [#](https://www.funtl.com/zh/vue-cli/vue-cli-src.html#main-js)main.js

  项目的入口文件，我们知道所有的程序都会有一个入口

  ```javascript
  // The Vue build version to load with the `import` command
  // (runtime-only or standalone) has been set in webpack.base.conf with an alias.
  import Vue from 'vue'
  import App from './App'
  
  Vue.config.productionTip = false
  
  /* eslint-disable no-new */
  new Vue({
    el: '#app',
    components: { App },
    template: '<App/>'
  })
  ```



  - `import Vue from 'vue'`：ES6 写法，会被转换成 `require("vue");` （require 是 NodeJS 提供的模块加载器）

  - `import App from './App'`：意思同上，但是指定了查找路径，`./` 为当前目录

  - `Vue.config.productionTip = false`：关闭浏览器控制台关于环境的相关提示

  - ```
    new Vue({...})
    ```

    ：实例化 Vue

    - `el: '#app'`：查找 index.html 中 id 为 app 的元素
    - `template: ''`：模板，会将 index.html 中 `` 替换为 ``
    - `components: { App }`：引入组件，使用的是 `import App from './App'` 定义的 App 组件

  ## [#](https://www.funtl.com/zh/vue-cli/vue-cli-src.html#app-vue)App.vue

  组件模板

  ```vue
  <template>
    <div id="app">
      <img src="./assets/logo.png">
      <HelloWorld/>
    </div>
  </template>
  
  <script>
  import HelloWorld from './components/HelloWorld'
  
  export default {
    name: 'App',
    components: {
      HelloWorld
    }
  }
  </script>
  
  <style>
  #app {
    <!-- 字体 -->
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    <!-- 文字平滑效果 -->
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  </style>
  ```



  - `template`：HTML 代码模板，会替换 `` 中的内容

  - `import HelloWorld from './components/HelloWorld'`：引入 HelloWorld 组件，用于替换 `template` 中的 ``

  - ```
    export default{...}
    ```

    ：导出 NodeJS 对象，作用是可以通过

     

    ```
    import
    ```

     

    关键字导入

    - `name: 'App'`：定义组件的名称
    - `components: { HelloWorld }`：定义子组件



# Webpack 简介

## [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-Webpack-简介](https://www.bilibili.com/video/av43993942/)

## [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#概述)概述

本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

## [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#简介)简介

Webpack 是当下最热门的前端资源模块化管理和打包工具，它可以将许多松散耦合的模块按照依赖和规则打包成符合生产环境部署的前端资源。还可以将按需加载的模块进行代码分离，等到实际需要时再异步加载。通过 loader 转换，任何形式的资源都可以当做模块，比如 CommonsJS、AMD、ES6、CSS、JSON、CoffeeScript、LESS 等；

## [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#现状)现状

伴随着移动互联网的大潮，当今越来越多的网站已经从网页模式进化到了 WebApp 模式。它们运行在现代浏览器里，使用 HTML5、CSS3、ES6 等新的技术来开发丰富的功能，网页已经不仅仅是完成浏览器的基本需求；WebApp 通常是一个 SPA （单页面应用），每一个视图通过异步的方式加载，这导致页面初始化和使用过程中会加载越来越多的 JS 代码，这给前端的开发流程和资源组织带来了巨大挑战。

前端开发和其他开发工作的主要区别，首先是前端基于多语言、多层次的编码和组织工作，其次前端产品的交付是基于浏览器的，这些资源是通过增量加载的方式运行到浏览器端，如何在开发环境组织好这些碎片化的代码和资源，并且保证他们在浏览器端快速、优雅的加载和更新，就需要一个模块化系统，这个理想中的模块化系统是前端工程师多年来一直探索的难题。

## [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#模块化的演进)模块化的演进

### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#script-标签)Script 标签

```html
<script src="module1.js"></scirpt>
<script src="module2.js"></scirpt>
<script src="module3.js"></scirpt>
<script src="module4.js"></scirpt>
```

这是最原始的 JavaScript 文件加载方式，如果把每一个文件看做是一个模块，那么他们的接口通常是暴露在全局作用域下，也就是定义在 `window` 对象中，不同模块的调用都是一个作用域。

这种原始的加载方式暴露了一些显而易见的弊端：

- 全局作用域下容易造成变量冲突
- 文件只能按照 `` 的书写顺序进行加载
- 开发人员必须主观解决模块和代码库的依赖关系
- 在大型项目中各种资源难以管理，长期积累的问题导致代码库混乱不堪

### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#commonsjs)CommonsJS

服务器端的 NodeJS 遵循 CommonsJS 规范，该规范核心思想是允许模块通过 `require` 方法来同步加载所需依赖的其它模块，然后通过 `exports` 或 `module.exports` 来导出需要暴露的接口。

```javascript
require("module");
require("../module.js");
export.doStuff = function() {};
module.exports = someValue;
```

1
2
3
4

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#优点)优点

- 服务器端模块便于重用
- NPM 中已经有超过 45 万个可以使用的模块包
- 简单易用

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#缺点)缺点

- 同步的模块加载方式不适合在浏览器环境中，同步意味着阻塞加载，浏览器资源是异步加载的
- 不能非阻塞的并行加载多个模块

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#实现)实现

- 服务端的 NodeJS
- Browserify，浏览器端的 CommonsJS 实现，可以使用 NPM 的模块，但是编译打包后的文件体积较大
- modules-webmake，类似 Browserify，但不如 Browserify 灵活
- wreq，Browserify 的前身

### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#amd)AMD

Asynchronous Module Definition 规范其实主要一个主要接口 `define(id?, dependencies?, factory);` 它要在声明模块的时候指定所有的依赖 `dependencies`，并且还要当做形参传到 `factory` 中，对于依赖的模块提前执行。

```javascript
define("module", ["dep1", "dep2"], function(d1, d2) {
  return someExportedValue;
});
require(["module", "../file.js"], function(module, file) {});
```



#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#优点-2)优点

- 适合在浏览器环境中异步加载模块
- 可以并行加载多个模块

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#缺点-2)缺点

- 提高了开发成本，代码的阅读和书写比较困难，模块定义方式的语义不畅
- 不符合通用的模块化思维方式，是一种妥协的实现

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#实现-2)实现

- RequireJS
- curl

### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#cmd)CMD

Commons Module Definition 规范和 AMD 很相似，尽量保持简单，并与 CommonsJS 和 NodeJS 的 Modules 规范保持了很大的兼容性。

```javascript
define(function(require, exports, module) {
  var $ = require("jquery");
  var Spinning = require("./spinning");
  exports.doSomething = ...;
  module.exports = ...;
});
```



#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#优点-3)优点

- 依赖就近，延迟执行
- 可以很容易在 NodeJS 中运行

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#缺点-3)缺点

- 依赖 SPM 打包，模块的加载逻辑偏重

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#实现-3)实现

- Sea.js
- coolie

### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#es6-模块)ES6 模块

EcmaScript6 标准增加了 JavaScript 语言层面的模块体系定义。 ES6 模块的设计思想，是尽量静态化，使编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonsJS 和 AMD 模块，都只能在运行时确定这些东西。

```javascript
import "jquery";
export function doStuff() {}
module "localModule" {}
```



#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#优点-4)优点

- 容易进行静态分析
- 面向未来的 EcmaScript 标准

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#缺点-4)缺点

- 原生浏览器端还没有实现该标准
- 全新的命令，新版的 NodeJS 才支持

#### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#实现-4)实现

- Babel

### [#](https://www.funtl.com/zh/vue-cli/WebPack-简介.html#期望的模块系统)期望的模块系统

可以兼容多种模块风格，尽量可以利用已有的代码，不仅仅只是 JavaScript 模块化，还有 CSS、图片、字体等资源也需要模块化。



# 安装 WebPack

## [#](https://www.funtl.com/zh/vue-cli/安装-WebPack.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-Webpack-安装和使用](https://www.bilibili.com/video/av43994018/)

## [#](https://www.funtl.com/zh/vue-cli/安装-WebPack.html#概述)概述

WebPack 是一款模块加载器兼打包工具，它能把各种资源，如 JS、JSX、ES6、SASS、LESS、图片等都作为模块来处理和使用。

## [#](https://www.funtl.com/zh/vue-cli/安装-WebPack.html#安装)安装

```bash
npm install webpack -g
npm install webpack-cli -g
```



## [#](https://www.funtl.com/zh/vue-cli/安装-WebPack.html#配置)配置

创建 `webpack.config.js` 配置文件

- entry：入口文件，指定 WebPack 用哪个文件作为项目的入口
- output：输出，指定 WebPack 把处理完成的文件放置到指定路径
- module：模块，用于处理各种类型的文件
- plugins：插件，如：热更新、代码重用等
- resolve：设置路径指向
- watch：监听，用于设置文件改动后直接打包

```javascript
module.exports = {
    entry: "",
    output: {
        path: "",
        filename: ""
    },
    module: {
        loaders: [
            {test: /\.js$/, loader: ""}
        ]
    },
    plugins: {},
    resolve: {},
    watch: true
}
```



## [#](https://www.funtl.com/zh/vue-cli/安装-WebPack.html#执行)执行

直接运行 `webpack` 命令打包



# 使用 WebPack

## [#](https://www.funtl.com/zh/vue-cli/使用-WebPack.html#概述)概述

使用 WebPack 打包项目非常简单，主要步骤如下：

- 创建项目
- 创建一个名为 `modules` 的目录，用于放置 JS 模块等资源文件
- 创建模块文件，如 `hello.js`，用于编写 JS 模块相关代码
- 创建一个名为 `main.js` 的入口文件，用于打包时设置 `entry` 属性
- 创建 `webpack.config.js` 配置文件，使用 `webpack` 命令打包
- 创建 HTML 页面，如 `index.html`，导入 WebPack 打包后的 JS 文件
- 运行 HTML 看效果

## [#](https://www.funtl.com/zh/vue-cli/使用-WebPack.html#目录结构)目录结构

![img](https://www.funtl.com/assets1/Lusifer_20190212015555.png)

## [#](https://www.funtl.com/zh/vue-cli/使用-WebPack.html#模块代码)模块代码

创建一个名为 `hello.js` 的 JavaScript 模块文件，代码如下：

```javascript
exports.sayHi = function () {
  document.write("<div>Hello WebPack</div>");
};
```



## [#](https://www.funtl.com/zh/vue-cli/使用-WebPack.html#入口代码)入口代码

创建一个名为 `main.js` 的 JavaScript 入口模块，代码如下：

```javascript
var hello = require("./hello");
hello.sayHi();
```

1
2

## [#](https://www.funtl.com/zh/vue-cli/使用-WebPack.html#配置文件)配置文件

创建名为 `webpack.config.js` 的配置文件，代码如下：

```javascript
module.exports = {
    entry: "./modules/main.js",
    output: {
        filename: "./js/bundle.js"
    }
};
```



## [#](https://www.funtl.com/zh/vue-cli/使用-WebPack.html#html)HTML

创建一个名为 `index.html`，代码如下：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
<script src="dist/js/bundle.js"></script>
</body>
</html>
```



## [#](https://www.funtl.com/zh/vue-cli/使用-WebPack.html#打包)打包

```bash
# 用于监听变化
webpack --watch
```



## [#](https://www.funtl.com/zh/vue-cli/使用-WebPack.html#运行)运行

运行 HTML 文件，你会在浏览器看到：

```html
Hello WebPack
```

# 第一个 vue-router 路由

## [#](https://www.funtl.com/zh/vue-router/#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-VueRouter-第一个路由](https://www.bilibili.com/video/av43994080/)

## [#](https://www.funtl.com/zh/vue-router/#概述)概述

Vue Router 是 Vue.js 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。包含的功能有：

- 嵌套的路由/视图表
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 Vue.js 过渡系统的视图过渡效果
- 细粒度的导航控制
- 带有自动激活的 CSS class 的链接
- HTML5 历史模式或 hash 模式，在 IE9 中自动降级
- 自定义的滚动条行为

## [#](https://www.funtl.com/zh/vue-router/#安装)安装

vue-router 是一个插件包，所以我们还是需要用 npm/cnpm 来进行安装的。打开命令行工具，进入你的项目目录，输入下面命令。

```bash
npm install vue-router --save-dev
```

1

如果在一个模块化工程中使用它，必须要通过 `Vue.use()` 明确地安装路由功能：

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter);
```



## [#](https://www.funtl.com/zh/vue-router/#使用)使用

以下案例在 vue-cli 项目中使用 vue-router

### [#](https://www.funtl.com/zh/vue-router/#创建组件页面)创建组件页面

创建一个名为 `src/components` 的目录专门放置我们开发的 Vue 组件，在 `src/components` 目录下创建一个名为 `Content.vue` 的组件，代码如下：

```vue
<template>
    <div>
      我是内容页
    </div>
</template>

<script>
    export default {
        name: "Content"
    }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
```



### [#](https://www.funtl.com/zh/vue-router/#安装路由)安装路由

创建一个名为 `src/router` 的目录专门放置我们的路由配置代码，在 `src/router` 目录下创建一个名为 `index.js` 路由配置文件，代码如下：

```javascript
import Vue from 'vue'
// 导入路由插件
import Router from 'vue-router'
// 导入上面定义的组件
import Content from '@/components/Content'

// 安装路由
Vue.use(Router);

// 配置路由
export default new Router({
  routes: [
    {
      // 路由路径
      path: '/content',
      // 路由名称
      name: 'Content',
      // 跳转到组件
      component: Content
    }
  ]
});
```



### [#](https://www.funtl.com/zh/vue-router/#配置路由)配置路由

修改 `main.js` 入口文件，增加配置路由的相关代码

```javascript
import Vue from 'vue'
import App from './App'
// 导入上面创建的路由配置目录
import router from './router'

Vue.config.productionTip = false;

new Vue({
  el: '#app',
  // 配置路由
  router,
  components: { App },
  template: '<App/>'
});
```



### [#](https://www.funtl.com/zh/vue-router/#使用路由)使用路由

修改 `App.vue` 组件，代码如下：

```vue
<template>
  <div id="app">
    <router-link to="/">首页</router-link>
    <router-link to="/content">内容</router-link>
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
```



说明：

- **router-link：** 默认会被渲染成一个 `` 标签，`to` 属性为指定链接
- **router-view：** 用于渲染路由匹配到的组件

### [#](https://www.funtl.com/zh/vue-router/#效果演示)效果演示

![img](https://www.funtl.com/assets1/Lusifer_2019021504350001.gif)

# 第一个 Vue 工程项目

## [#](https://www.funtl.com/zh/vue-router/第一个-Vue-工程项目.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-VueRouter-创建工程项目](https://www.bilibili.com/video/av43994143/)

## [#](https://www.funtl.com/zh/vue-router/第一个-Vue-工程项目.html#概述)概述

从本章节开始，我们采用实战教学模式并结合 [**ElementUI**](http://element-cn.eleme.io/#/zh-CN) 组件库，将所需知识点应用到实际中，以最快速度带领大家掌握 Vue 的使用

## [#](https://www.funtl.com/zh/vue-router/第一个-Vue-工程项目.html#创建工程)创建工程

> 使用 NPM 安装相关组件依赖时可能会遇到权限问题，此时使用 PowerShell 管理员模式运行即可；开始菜单 -> 鼠标右击 -> Windows PowerShell (管理员)

![img](https://www.funtl.com/assets1/Lusifer_20190216235700.png)

创建一个名为 `hello-vue` 的工程

```bash
# 使用 webpack 打包工具初始化一个名为 hello-vue 的工程
vue init webpack hello-vue
```



![img](https://www.funtl.com/assets1/Lusifer_20190217002847.png)

## [#](https://www.funtl.com/zh/vue-router/第一个-Vue-工程项目.html#安装依赖)安装依赖

我们需要安装 `vue-router`、`element-ui`、`sass-loader` 和 `node-sass` 四个插件

```bash
# 进入工程目录
cd hello-vue
# 安装 vue-router
npm install vue-router --save-dev
# 安装 element-ui
npm i element-ui -S
# 安装 SASS 加载器
npm install sass-loader node-sass --save-dev
```



![img](https://www.funtl.com/assets1/Lusifer_20190217003944.png)

```bash
# 安装依赖
npm install
```



![img](https://www.funtl.com/assets1/Lusifer_20190217004352.png)

## [#](https://www.funtl.com/zh/vue-router/第一个-Vue-工程项目.html#启动工程)启动工程

```bash
npm run dev
```

1

![img](https://www.funtl.com/assets1/Lusifer_20190217004622.png)

## [#](https://www.funtl.com/zh/vue-router/第一个-Vue-工程项目.html#运行效果)运行效果

在浏览器打开 http://localhost:8080 你会看到如下效果

![img](https://www.funtl.com/assets1/Lusifer_20190217004659.png)

## [#](https://www.funtl.com/zh/vue-router/第一个-Vue-工程项目.html#附：npm-相关命令说明)附：NPM 相关命令说明

- `npm install moduleName`：安装模块到项目目录下
- `npm install -g moduleName`：-g 的意思是将模块安装到全局，具体安装到磁盘哪个位置，要看 npm config prefix 的位置
- `npm install -save moduleName`：--save 的意思是将模块安装到项目目录下，并在 package 文件的 dependencies 节点写入依赖，`-S` 为该命令的缩写
- `npm install -save-dev moduleName`：--save-dev 的意思是将模块安装到项目目录下，并在 package 文件的 devDependencies 节点写入依赖，`-D` 为该命令的缩写



# 第一个 ElementUI 页面 (登录页)

## [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-VueRouter-创建登录页](https://www.bilibili.com/video/av43994228/)

## [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#目录结构)目录结构

在源码目录中创建如下结构：

- assets：用于存放资源文件
- components：用于存放 Vue 功能组件
- views：用于存放 Vue 视图组件
- router：用于存放 vue-router 配置

![img](https://www.funtl.com/assets1/Lusifer_20190217012252.png)

## [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#创建视图)创建视图

### [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#创建首页视图)创建首页视图

在 `views` 目录下创建一个名为 `Main.vue` 的视图组件；该组件在当前章节无任何作用，主要用于登录后展示登录成功的跳转效果；

```vue
<template>
    <div>
      首页
    </div>
</template>

<script>
    export default {
        name: "Main"
    }
</script>

<style scoped>

</style>
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15

### [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#创建登录页视图)创建登录页视图

在 `views` 目录下创建一个名为 `Login.vue` 的视图组件，其中 `el-*` 的元素为 ElementUI 组件；

```vue
<template>
  <div>
    <el-form ref="loginForm" :model="form" :rules="rules" label-width="80px" class="login-box">
      <h3 class="login-title">欢迎登录</h3>
      <el-form-item label="账号" prop="username">
        <el-input type="text" placeholder="请输入账号" v-model="form.username"/>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input type="password" placeholder="请输入密码" v-model="form.password"/>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" v-on:click="onSubmit('loginForm')">登录</el-button>
      </el-form-item>
    </el-form>

    <el-dialog
      title="温馨提示"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose">
      <span>请输入账号和密码</span>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: "Login",
    data() {
      return {
        form: {
          username: '',
          password: ''
        },

        // 表单验证，需要在 el-form-item 元素中增加 prop 属性
        rules: {
          username: [
            {required: true, message: '账号不可为空', trigger: 'blur'}
          ],
          password: [
            {required: true, message: '密码不可为空', trigger: 'blur'}
          ]
        },

        // 对话框显示和隐藏
        dialogVisible: false
      }
    },
    methods: {
      onSubmit(formName) {
        // 为表单绑定验证功能
        this.$refs[formName].validate((valid) => {
          if (valid) {
            // 使用 vue-router 路由到指定页面，该方式称之为编程式导航
            this.$router.push("/main");
          } else {
            this.dialogVisible = true;
            return false;
          }
        });
      }
    }
  }
</script>

<style lang="scss" scoped>
  .login-box {
    border: 1px solid #DCDFE6;
    width: 350px;
    margin: 180px auto;
    padding: 35px 35px 15px 35px;
    border-radius: 5px;
    -webkit-border-radius: 5px;
    -moz-border-radius: 5px;
    box-shadow: 0 0 25px #909399;
  }

  .login-title {
    text-align: center;
    margin: 0 auto 40px auto;
    color: #303133;
  }
</style>
```



## [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#创建路由)创建路由

在 `router` 目录下创建一个名为 `index.js` 的 vue-router 路由配置文件

```javascript
import Vue from 'vue'
import Router from 'vue-router'

import Login from "../views/Login"
import Main from '../views/Main'

Vue.use(Router);

export default new Router({
  routes: [
    {
      // 登录页
      path: '/login',
      name: 'Login',
      component: Login
    },
    {
      // 首页
      path: '/main',
      name: 'Main',
      component: Main
    }
  ]
});
```



## [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#配置路由)配置路由

### [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#修改入口代码)修改入口代码

修改 `main.js` 入口代码

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'
import router from './router'

// 导入 ElementUI
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

import App from './App'

// 安装路由
Vue.use(VueRouter);

// 安装 ElementUI
Vue.use(ElementUI);

new Vue({
  el: '#app',
  // 启用路由
  router,
  // 启用 ElementUI
  render: h => h(App)
});
```



修改 `App.vue` 组件代码

```vue
<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
  export default {
    name: 'App',
  }
</script>
```



## [#](https://www.funtl.com/zh/vue-router/第一个-ElementUI-页面.html#效果演示)效果演示

在浏览器打开 http://localhost:8080/#/login 你会看到如下效果

![img](https://www.funtl.com/assets1/Lusifer_201902170001.gif)

# 配置嵌套路由

## [#](https://www.funtl.com/zh/vue-router/配置嵌套路由.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-VueRouter-嵌套路由](https://www.bilibili.com/video/av44056776/)

## [#](https://www.funtl.com/zh/vue-router/配置嵌套路由.html#什么是嵌套路由)什么是嵌套路由

嵌套路由又称子路由，在实际应用中，通常由多层嵌套的组件组合而成。同样地，URL 中各段动态路径也按某种结构对应嵌套的各层组件，例如：

```text
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

1
2
3
4
5
6
7
8

## [#](https://www.funtl.com/zh/vue-router/配置嵌套路由.html#创建嵌套视图组件)创建嵌套视图组件

### [#](https://www.funtl.com/zh/vue-router/配置嵌套路由.html#用户信息组件)用户信息组件

在 `views/user` 目录下创建一个名为 `Profile.vue` 的视图组件；该组件在当前章节无任何作用，主要用于展示嵌套效果；

```vue
<template>
    <div>
      个人信息
    </div>
</template>

<script>
    export default {
        name: "UserProfile"
    }
</script>

<style scoped>

</style>
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15

### [#](https://www.funtl.com/zh/vue-router/配置嵌套路由.html#用户列表组件)用户列表组件

在 `views/user` 目录下创建一个名为 `List.vue` 的视图组件；该组件在当前章节无任何作用，主要用于展示嵌套效果；

```vue
<template>
    <div>
      用户列表
    </div>
</template>

<script>
    export default {
        name: "UserList"
    }
</script>

<style scoped>

</style>
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15

## [#](https://www.funtl.com/zh/vue-router/配置嵌套路由.html#配置嵌套路由-2)配置嵌套路由

修改 `router` 目录下的 `index.js` 路由配置文件，代码如下：

```javascript
import Vue from 'vue'
import Router from 'vue-router'

import Login from "../views/Login"
import Main from '../views/Main'

// 用于嵌套的路由组件
import UserProfile from '../views/user/Profile'
import UserList from '../views/user/List'

Vue.use(Router);

export default new Router({
  routes: [
    {
      // 登录页
      path: '/login',
      name: 'Login',
      component: Login
    },
    {
      // 首页
      path: '/main',
      name: 'Main',
      component: Main,
      // 配置嵌套路由
      children: [
        {path: '/user/profile', component: UserProfile},
        {path: '/user/list', component: UserList},
      ]
    }
  ]
});
```

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33

说明：主要在路由配置中增加了 `children` 数组配置，用于在该组件下设置嵌套路由

## [#](https://www.funtl.com/zh/vue-router/配置嵌套路由.html#修改首页视图)修改首页视图

接着上一节的代码，我们修改 `Main.vue` 视图组件，此处使用了 [ElementUI 布局容器组件](http://element-cn.eleme.io/#/zh-CN/component/container)，代码如下：

```vue
<template>
    <div>
      <el-container>
        <el-aside width="200px">
          <el-menu :default-openeds="['1']">
            <el-submenu index="1">
              <template slot="title"><i class="el-icon-caret-right"></i>用户管理</template>
              <el-menu-item-group>
                <el-menu-item index="1-1">
                  <router-link to="/user/profile">个人信息</router-link>
                </el-menu-item>
                <el-menu-item index="1-2">
                  <router-link to="/user/list">用户列表</router-link>
                </el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu index="2">
              <template slot="title"><i class="el-icon-caret-right"></i>内容管理</template>
              <el-menu-item-group>
                <el-menu-item index="2-1">分类管理</el-menu-item>
                <el-menu-item index="2-2">内容列表</el-menu-item>
              </el-menu-item-group>
            </el-submenu>
          </el-menu>
        </el-aside>

        <el-container>
          <el-header style="text-align: right; font-size: 12px">
            <el-dropdown>
              <i class="el-icon-setting" style="margin-right: 15px"></i>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item>个人信息</el-dropdown-item>
                <el-dropdown-item>退出登录</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
            <span>Lusifer</span>
          </el-header>

          <el-main>
            <router-view />
          </el-main>
        </el-container>
      </el-container>
    </div>
</template>

<script>
    export default {
        name: "Main"
    }
</script>

<style scoped lang="scss">
  .el-header {
    background-color: #B3C0D1;
    color: #333;
    line-height: 60px;
  }

  .el-aside {
    color: #333;
  }
</style>
```

1
2

说明：

- 在 `` 元素中配置了 `` 用于展示嵌套路由
- 主要使用 `个人信息` 展示嵌套路由内容

## [#](https://www.funtl.com/zh/vue-router/配置嵌套路由.html#效果演示)效果演示

![img](https://www.funtl.com/assets1/Lusifer_201902180001.gif)

# 参数传递

## [#](https://www.funtl.com/zh/vue-router/参数传递.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-VueRouter-参数传递](https://www.bilibili.com/video/av44056836/)

## [#](https://www.funtl.com/zh/vue-router/参数传递.html#概述)概述

我们经常需要把某种模式匹配到的所有路由，全都映射到同个组件。例如，我们有一个 User 组件，对于所有 ID 各不相同的用户，都要使用这个组件来渲染。此时我们就需要传递参数了；

## [#](https://www.funtl.com/zh/vue-router/参数传递.html#使用路径匹配的方式)使用路径匹配的方式

### [#](https://www.funtl.com/zh/vue-router/参数传递.html#修改路由配置)修改路由配置

```javascript
{path: '/user/profile/:id', name:'UserProfile', component: UserProfile}
```

1

说明：主要是在 `path` 属性中增加了 `:id` 这样的占位符

### [#](https://www.funtl.com/zh/vue-router/参数传递.html#传递参数)传递参数

#### [#](https://www.funtl.com/zh/vue-router/参数传递.html#router-link)router-link

```vue
<router-link :to="{name: 'UserProfile', params: {id: 1}}">个人信息</router-link>
```

1

说明：此时我们将 `to` 改为了 `:to`，是为了将这一属性当成对象使用，注意 **router-link 中的 name 属性名称** 一定要和 **路由中的 name 属性名称** 匹配，因为这样 Vue 才能找到对应的路由路径；

#### [#](https://www.funtl.com/zh/vue-router/参数传递.html#代码方式)代码方式

```javascript
this.$router.push({ name: 'UserProfile', params: {id: 1}});
```

1

### [#](https://www.funtl.com/zh/vue-router/参数传递.html#接收参数)接收参数

在目标组件中使用

```vue
{{ $route.params.id }}
```

1

来接收参数

## [#](https://www.funtl.com/zh/vue-router/参数传递.html#使用-props-的方式)使用 `props` 的方式

### [#](https://www.funtl.com/zh/vue-router/参数传递.html#修改路由配置-2)修改路由配置

```javascript
{path: '/user/profile/:id', name:'UserProfile', component: UserProfile, props: true}
```

1

说明：主要增加了 `props: true` 属性

### [#](https://www.funtl.com/zh/vue-router/参数传递.html#传递参数-2)传递参数

同上

### [#](https://www.funtl.com/zh/vue-router/参数传递.html#接收参数-2)接收参数

为目标组件增加 `props` 属性，代码如下：

```javascript
  export default {
    props: ['id'],
    name: "UserProfile"
  }
```

1
2
3
4

模板中使用

```vue
{{ id }}
```

1

接收参数

# 组件重定向

## [#](https://www.funtl.com/zh/vue-router/组件重定向.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-VueRouter-组件重定向](https://www.bilibili.com/video/av44081446/)

## [#](https://www.funtl.com/zh/vue-router/组件重定向.html#概述)概述

重定向的意思大家都明白，但 Vue 中的重定向是作用在路径不同但组件相同的情况下

## [#](https://www.funtl.com/zh/vue-router/组件重定向.html#配置重定向)配置重定向

### [#](https://www.funtl.com/zh/vue-router/组件重定向.html#修改路由配置)修改路由配置

```javascript
    {
      path: '/main',
      name: 'Main',
      component: Main
    },
    {
      path: '/goHome',
      redirect: '/main'
    }
```

1
2
3
4
5
6
7
8
9

说明：这里定义了两个路径，一个是 `/main` ，一个是 `/goHome`，其中 `/goHome` 重定向到了 `/main` 路径，由此可以看出重定向不需要定义组件；

### [#](https://www.funtl.com/zh/vue-router/组件重定向.html#重定向到组件)重定向到组件

设置对应路径即可

```vue
<router-link to="/goHome">回到首页</router-link>
```

1

## [#](https://www.funtl.com/zh/vue-router/组件重定向.html#带参数的重定向)带参数的重定向

### [#](https://www.funtl.com/zh/vue-router/组件重定向.html#修改路由配置-2)修改路由配置

```javascript
    {
      // 首页
      path: '/main/:username',
      name: 'Main',
      component: Main
    },
    {
      path: '/goHome/:username',
      redirect: '/main/:username'
    }
```



### [#](https://www.funtl.com/zh/vue-router/组件重定向.html#重定向到组件-2)重定向到组件

```vue
<router-link to="/goHome/Lusifer">回到首页</router-link>
```

# 路由模式与 404

## [#](https://www.funtl.com/zh/vue-router/路由模式与-404.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-VueRouter-路由模式与404](https://www.bilibili.com/video/av44082362/)

## [#](https://www.funtl.com/zh/vue-router/路由模式与-404.html#路由模式)路由模式

路由模式有两种

- hash：路径带 `#` 符号，如 `http://localhost/#/login`
- history：路径不带 `#` 符号，如 `http://localhost/login`

修改路由配置，代码如下：

```javascript
export default new Router({
  mode: 'history',
  routes: [
  ]
});
```



## [#](https://www.funtl.com/zh/vue-router/路由模式与-404.html#处理-404)处理 404

创建一个名为 `NotFound.vue` 的视图组件，代码如下：

```vue
<template>
    <div>
      页面不存在，请重试！
    </div>
</template>

<script>
    export default {
        name: "NotFount"
    }
</script>

<style scoped>

</style>
```



修改路由配置，代码如下：

```javascript
    {
      path: '*',
      component: NotFound
    }
```

# 路由钩子与异步请求

## [#](https://www.funtl.com/zh/vue-router/路由钩子与异步请求.html#本节视频)本节视频

- [【视频】Vue 渐进式 JavaScript 框架-VueRouter-路由钩子与异步请求](https://www.bilibili.com/video/av44082689/)

## [#](https://www.funtl.com/zh/vue-router/路由钩子与异步请求.html#路由中的钩子函数)路由中的钩子函数

- `beforeRouteEnter`：在进入路由前执行
- `beforeRouteLeave`：在离开路由前执行

案例代码如下：

```javascript
  export default {
    props: ['id'],
    name: "UserProfile",
    beforeRouteEnter: (to, from, next) => {
      console.log("准备进入个人信息页");
      next();
    },
    beforeRouteLeave: (to, from, next) => {
      console.log("准备离开个人信息页");
      next();
    }
  }
```



参数说明：

- `to`：路由将要跳转的路径信息

- `from`：路径跳转前的路径信息

- ```
  next
  ```

  ：路由的控制参数

  - `next()` 跳入下一个页面
  - `next('/path')` 改变路由的跳转方向，使其跳到另一个路由
  - `next(false)` 返回原来的页面
  - `next((vm)=>{})` **仅在 beforeRouteEnter 中可用，vm 是组件实例**

## [#](https://www.funtl.com/zh/vue-router/路由钩子与异步请求.html#在钩子函数中使用异步请求)在钩子函数中使用异步请求

安装 Axios

```bash
npm install axios -s
```

1

引用 Axios

```javascript
import axios from 'axios'
Vue.prototype.axios = axios;
```

1
2

在 `beforeRouteEnter` 中进行异步请求，案例代码如下：

```javascript
  export default {
    props: ['id'],
    name: "UserProfile",
    beforeRouteEnter: (to, from, next) => {
      console.log("准备进入个人信息页");
      // 注意，一定要在 next 中请求，因为该方法调用时 Vue 实例还没有创建，此时无法获取到 this 对象，在这里使用官方提供的回调函数拿到当前实例
      next(vm => {
        vm.getData();
      });
    },
    beforeRouteLeave: (to, from, next) => {
      console.log("准备离开个人信息页");
      next();
    },
    methods: {
      getData: function () {
        this.axios({
          method: 'get',
          url: 'http://localhost:8080/data.json'
        }).then(function (repos) {
          console.log(repos);
        }).catch(function (error) {
          console.log(error);
        });
      }
    }
  }
```

# Vuex 快速入门

## [#](https://www.funtl.com/zh/vuex/#简介)简介

Vuex 是一个专为 Vue.js 应用程序开发的 **状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

## [#](https://www.funtl.com/zh/vuex/#目标)目标

继续之前 [vue-router](https://www.funtl.com/zh/vue-router/) 章节做的案例项目，我们通过完善登录功能将用户信息保存至 Vuex 中来体会它的作用；

## [#](https://www.funtl.com/zh/vuex/#安装)安装

在项目根目录执行如下命令来安装 Vuex

```bash
npm install vuex --save
```



修改 `main.js` 文件，导入 Vuex，关键代码如下：

```javascript
import Vuex from 'vuex'
Vue.use(Vuex);
```



## [#](https://www.funtl.com/zh/vuex/#判断用户是否登录)判断用户是否登录

我们利用路由钩子 `beforeEach` 来判断用户是否登录，期间会用到 `sessionStorage` 存储功能

### [#](https://www.funtl.com/zh/vuex/#修改-login-vue)修改 `Login.vue`

在表单验证成功方法内增加如下代码：

```javascript
// 设置用户登录成功
sessionStorage.setItem('isLogin', 'true');
```



### [#](https://www.funtl.com/zh/vuex/#修改-main-js)修改 `main.js`

利用路由钩子 `beforeEach` 方法判断用户是否成功登录，关键代码如下：

```javascript
// 在跳转前执行
router.beforeEach((to, form, next) => {
  // 获取用户登录状态
  let isLogin = sessionStorage.getItem('isLogin');

  // 注销
  if (to.path == '/logout') {
    // 清空
    sessionStorage.clear();

    // 跳转到登录
    next({path: '/login'});
  }

  // 如果请求的是登录页
  else if (to.path == '/login') {
    if (isLogin != null) {
      // 跳转到首页
      next({path: '/main'});
    }
  }

  // 如果为非登录状态
  else if (isLogin == null) {
    // 跳转到登录页
    next({path: '/login'});
  }

  // 下一个路由
  next();
});
```

## [#](https://www.funtl.com/zh/vuex/#配置-vuex)配置 `vuex`

### [#](https://www.funtl.com/zh/vuex/#创建-vuex-配置文件)创建 Vuex 配置文件

在 `src` 目录下创建一个名为 `store` 的目录并新建一个名为 `index.js` 文件用来配置 Vuex，代码如下：

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex);

// 全局 state 对象，用于保存所有组件的公共数据
const state = {
  // 定义一个 user 对象
  // 在组件中是通过 this.$store.state.user 来获取
  user: {
    username: ''
  }
};

// 实时监听 state 值的最新状态，注意这里的 getters 可以理解为计算属性
const getters = {
  // 在组件中是通过 this.$store.getters.getUser 来获取
  getUser(state) {
    return state.user;
  }
};

// 定义改变 state 初始值的方法，这里是唯一可以改变 state 的地方，缺点是只能同步执行
const mutations = {
  // 在组件中是通过 this.$store.commit('updateUser', user); 方法来调用 mutations
  updateUser(state, user) {
    state.user = user;
  }
};

// 定义触发 mutations 里函数的方法，可以异步执行 mutations 里的函数
const actions = {
  // 在组件中是通过 this.$store.dispatch('asyncUpdateUser', user); 来调用 actions
  asyncUpdateUser(context, user) {
    context.commit('updateUser', user);
  }
};

export default new Vuex.Store({
  state,
  getters,
  mutations,
  actions
});
```



修改 `main.js` 增加刚才配置的 `store/index.js`，关键代码如下：

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import store from './store'

Vue.use(Vuex);

new Vue({
  el: '#app',
  store
});
```



### [#](https://www.funtl.com/zh/vuex/#解决浏览器刷新后-vuex-数据消失问题)解决浏览器刷新后 Vuex 数据消失问题

#### [#](https://www.funtl.com/zh/vuex/#问题描述)问题描述

Vuex 的状态存储是响应式的，当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。但是有一个问题就是：vuex 的存储的数据只是在页面的中，相当于我们定义的全局变量，刷新之后，里边的数据就会恢复到初始化状态。但是这个情况有时候并不是我们所希望的。

#### [#](https://www.funtl.com/zh/vuex/#解决方案)解决方案

监听页面是否刷新，如果页面刷新了，将 state 对象存入到 sessionStorage 中。页面打开之后，判断 sessionStorage 中是否存在 state 对象，如果存在，则说明页面是被刷新过的，将 sessionStorage 中存的数据取出来给 vuex 中的 state 赋值。如果不存在，说明是第一次打开，则取 vuex 中定义的 state 初始值。

#### [#](https://www.funtl.com/zh/vuex/#修改代码)修改代码

在 `App.vue` 中增加监听刷新事件

```javascript
  export default {
    name: 'App',
    mounted() {
      window.addEventListener('unload', this.saveState);
    },
    methods: {
      saveState() {
        sessionStorage.setItem('state', JSON.stringify(this.$store.state));
      }
    }
  }
```



修改 `store/index.js` 中的 state

```javascript
const state = sessionStorage.getItem('state') ? JSON.parse(sessionStorage.getItem('state')) : {
  user: {
    username: ''
  }
};
```



## [#](https://www.funtl.com/zh/vuex/#模块化)模块化

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。为了解决以上问题，Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割

### [#](https://www.funtl.com/zh/vuex/#创建-user-模块)创建 `user` 模块

在 `store` 目录下创建一个名为 `modules` 的目录并创建一个名为 `user.js` 的文件，代码如下：

```javascript
const user = {
  // 因为模块化了，所以解决刷新问题的代码需要改造一下
  state: sessionStorage.getItem('userState') ? JSON.parse(sessionStorage.getItem('userState')) : {
    user: {
      username: ''
    }
  },
  getters: {
    getUser(state) {
      return state.user;
    }
  },
  mutations: {
    updateUser(state, user) {
      state.user = user;
    }
  },
  actions: {
    asyncUpdateUser(context, user) {
      context.commit('updateUser', user);
    }
  }
};

export default user;
```



### [#](https://www.funtl.com/zh/vuex/#修改-store-index-js)修改 `store/index.js`

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

import user from './modules/user'

Vue.use(Vuex);

export default new Vuex.Store({
  modules: {
    // this.$store.state.user
    user
  }
});
```



**备注：由于组件中使用的是 `getters` 和 `actions` 处理，所以调用代码不变**

### [#](https://www.funtl.com/zh/vuex/#修改-app-vue)修改 `App.vue`

```
  export default {
    name: 'App',
    mounted() {
      window.addEventListener('unload', this.saveState);
    },
    methods: {
      saveState() {
        // 模块化后，调用 state 的代码修改为 this.$store.state.user
        sessionStorage.setItem('userState', JSON.stringify(this.$store.state.user));
      }
    }
  }
```

