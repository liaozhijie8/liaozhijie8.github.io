# Vue实践规范

<!--more-->
##### 1、为列表渲染设置属性key

**key这个特殊属性主要用在Vue.js的虚拟DOM算法中，在对比新旧虚拟节点时辨识虚拟节点。**
在虚拟DOM中，更新子节点时，需要从旧虚拟节点列表中查找与新虚拟节点相同的节点进行更新。如果这个查找过程设置了属性key，那么查找速度会快很多。所以无论何时，建议大家尽可能地在使用v-for时提供key。示例如下:
```Vue
01  <div v-for="item in items" :key="item.id">
02    <!-- 内容 -->
03  </div>
```
##### 2、在v-if/v-if-else/v-else中使用key
如果一组v-if+v-else的元素类型相同，最好使用属性key（比如两个`<div>`元素）。
v-if指令在编译后：
```js
01  (has)
02    ? _c('li',[_v("if")])
03    : _c('li',[_v("else")])
```
当状态发生变化时，生成的虚拟节点既有可能是v-if上的虚拟节点，也有可能是v-else上的虚拟节点。
>默认情况下，Vue.js会尽可能高效地更新DOM。这意味着，当它在相同类型的元素之间切换时，会修补已存在的元素，而不是将旧的元素移除，然后在同一位置添加一个新元素。如果本不相同的元素被识别为相同，则会出现意料之外的副作用。如果添加了属性key，那么在比对虚拟DOM时，则会认为它们是两个不同的节点，于是会将旧元素移除并在相同的位置添加一个新元素，从而避免意料之外的副作用。

不添加key:
```Vue
01  <div v-if="error">
02    错误：{{ error }}
03  </div>
04  <div v-else>
05    {{ results }}
06  </div>
```
添加key：
```Vue
01  <div
02    v-if="error"
03    key="search-status"
04  >
05    错误：{{ error }}
06  </div>
07  <div
08    v-else
09    key="search-results"
10  >
11    {{ results }}
12  </div>
```
##### 3、解决路由切换组件不变
>在使用Vue.js开发项目时，最常遇到的一个典型问题就是，当页面切换到同一个路由但不同参数的地址时，组件的生命周期钩子并不会重新触发。

例如，路由是下面这样的：
```js
01  const routes = [
02    {
03      path: '/detail/:id',
04      name: 'detail',
05      component: Detail
06    }
07  ]
```
当我们从路由/detail/1切换到/detail/2时，组件是不会发生任何变化的。
这是因为vue-router会识别出两个路由使用的是同一个组件从而进行复用，并不会重新创建组件，因此组件的生命周期钩子自然也不会被触发。
**组件本质上是一个映射关系**，所以先销毁再重建一个相同的组件会存在很大程度上的性能浪费，复用组件才是正确的选择。但是这也意味着组件的生命周期钩子不会再被调用。

###### 方法一:路由导航守卫beforeRouteUpdate
>vue-router提供了导航守卫beforeRouteUpdate，该守卫在当前路由改变且组件被复用时调用，所以可以在组件内定义路由导航守卫来解决这个问题。

组件的生命周期钩子虽然不会重新触发，但是路由提供的beforeRouteUpdate守卫可以被触发。因此，只需要把每次切换路由时需要执行的逻辑放到beforeRouteUpdate守卫中即可。例如，在beforeRouteUpdate守卫中发送请求拉取数据，更新状态并重新渲染视图。

###### 方法二:观察 $route对象的变化
通过watch可以监听到路由对象发生的变化，从而对路由变化作出响应。例如：
```js
01  const User = {
02    template: '...',
03    watch: {
04      '$route' (to, from) {
05        // 对路由变化作出响应
06      }
07    }
08  }
```
这种方式也可以解决上述问题，但代价是**组件内多了一个watch，这会带来依赖追踪的内存开销。**

如果最终选择使用watch解决这个问题，那么在某些场景下我推荐在组件里**只观察自己需要的query**，这样有利于减少不必要的请求。

>假设有这样一个场景，页面中有两部分内容，上面是个人的描述信息，下面一个带翻页的列表，这时假设路由中的参数是 /user?id=4&page=1时，说明用户ID是4，列表是第一页。
我们可以断定每次翻页时只需要发送列表的请求，而个人的描述信息只需要第一次进入组件时请求一次即可。当翻到第二页时，路由应该是这样的：/user?id=4&page=2。

可以看到，参数中的id没有变，只有page变了。所以为了避免发送多余的请求，应该这样去观察路由：
```js
01  const User = {
02    template: '...',
03    watch: {
04      '$route.query.id' () {
05        // 请求个人描述信息
06      },
07      '$route.query.page' () {
08        // 请求列表
09      }
10    }
11  }
```
不好的做法是统一观察 $route：
```js
01  const User = {
02    template: '...',
03    watch: {
04      '$route' (to, from) {
05        // 请求个人描述信息
06        // 请求列表
07      }
08    }
09  }
```
这种做法之所以不好，是因为如果路由参数中只是页码变了，那么只需要请求列表信息即可，但是上面的做法还会请求个人描述信息。

###### 方法三:为router-view组件添加属性key
这种做法非常取巧，非常“暴力”，但非常有效。它本质上是利用虚拟DOM在渲染时通过key来对比两个节点是否相同的原理。通过给router-view组件设置key，可以使每次切换路由时的key都不一样，让虚拟DOM认为router-view组件是一个新节点，从而先销毁组件，然后再重新创建新组件。即使是相同的组件，但是如果url变了，key就变了，Vue.js就会重新创建这个组件。

因为组件是新创建的，所以组件内的生命周期会重复触发。示例如下：
```Vue
01  <router-view :key="$route.fullPath"></router-view>
```
这种方式的坏处很明显，**每次切换路由组件时都会被销毁并且重新创建，非常浪费性能**。其优点更明显，简单粗暴，改动小。为router-view组件设置了key之后，立刻就可以看到问题被解决了。

##### 4、区分Vuex与props的使用边界
>通常，在项目开发中，业务组件会使用Vuex维护状态，使用不同组件统一操作Vuex中的状态。这样不论是父子组件间的通信还是兄弟组件间的通信，都很容易。**对于通用组件**，我会使用props以及事件进行父子组件间的通信（通用组件不需要兄弟组件间的通信）。这样做是因为通用组件会拿到各个业务组件中使用，它要与业务解耦，所以需要使用props获取状态。

通用组件要定义细致的prop，并且尽可能详细，至少需要指定其类型。这样做的好处是：

* 写明了组件的API，所以很容易看懂组件的用法；
* 在开发环境下，如果向一个组件提供格式不正确的prop，Vue.js将会在控制台发出警告，帮助我们捕获潜在的错误来源。

##### 5、避免v-if和v-for一起使用
>Vue.js官方强烈建议不要把v-if和v-for同时用在同一个元素上。

通常，我们在下面两种常见的情况下，会倾向于不同的做法。
* 为了过滤一个列表中的项目（比如v-for="user in users" v-if="user.isActive"），请将users替换为一个计算属性（比如activeUsers），让它返回过滤后的列表。
* 为了避免渲染本应该被隐藏的列表（比如v-for="user in users" v-if="shouldShow-Users"），请将v-if移动至容器元素上（比如ul和ol）。
  
对于第一种情况，Vue.js官方给出的解释是：当Vue.js处理指令时，v-for比v-if具有更高的优先级，所以即使我们只渲染出列表中的一小部分元素，也得在每次重渲染的时候遍历整个列表，而不考虑活跃用户是否发生了变化。通过将列表更换为在一个计算属性上遍历并过滤掉不需要渲染的数据，我们将会获得如下好处。
* 过滤后的列表只会在数组发生相关变化时才被重新运算，过滤更高效。
* 使用v-for="user in activeUsers"之后，我们在渲染时只遍历活跃用户，渲染更高效。
* 解耦渲染层的逻辑，可维护性（对逻辑的更改和扩展）更强。
* 
混合一起写法:
```Vue
01  <ul>
02    <li
03      v-for="user in users"
04      v-if="user.isActive"
05      :key="user.id"
06    >
07      {{ user.name }}
08    </li>
09  </ul>
```
可以更换为在如下的一个计算属性上遍历并过滤列表：
```js
01  computed: {
02    activeUsers: function () {
03      return this.users.filter(function (user) {
04        return user.isActive
05      })
06    }
07  }
```
模板更改为：
```Vue
01  <ul>
02    <li
03      v-for="user in activeUsers"
04      :key="user.id"
05    >
06      {{ user.name }}
07    </li>
08  </ul>
```
对于第二种情况，官方解释是为了获得同样的好处，可以把：
```Vue
01  <ul>
02    <li
03      v-for="user in users"
04      v-if="shouldShowUsers"
05      :key="user.id"
06    >
07      {{ user.name }}
08    </li>
09  </ul>
```
更新为：
```Vue
01  <ul v-if="shouldShowUsers">
02    <li
03      v-for="user in users"
04      :key="user.id"
05    >
06      {{ user.name }}
07    </li>
08  </ul>
```
通过将v-if移动到容器元素，我们不会再检查每个用户的shouldShowUsers，取而代之的是，我们只检查它一次，且不会在shouldShowUsers为false的时候运算v-for。

##### 6、为组件样式设置作用域
>CSS的规则都是全局的，任何一个组件的样式规则都对整个页面有效。因此，我们很容易在一个组件中写了某个样式，而不小心影响了另一个组件的样式，或者自己的组件被第三方库的CSS影响了。**对于应用来说，最佳实践是只有顶级App组件和布局组件中的样式可以是全局的，其他所有组件都应该是有作用域的。**

在Vue.js中，可以通过scoped特性或CSS Modules（一个基于class的类似BEM的策略）来设置组件样式作用域。

对于组件库，我们应该更倾向于选用基于class的策略而不是scoped特性。因为基于class的策略使覆写内部样式更容易，它使用容易理解的class名称且没有太高的选择器优先级，不容易导致冲突。
**使用scoped特性**
```Vue
01  <template>
02    <button class="button button-close">X</button>
03  </template>
04  
05  <!-- 使用scoped特性 -->
06  <style scoped>
07  .button {
08    border: none;
09    border-radius: 2px;
10  }
11  
12  .button-close {
13    background-color: red;
14  }
15  </style>
```
**使用CSS Modules**
```Vue
 <template>
02    <button :class="[$style.button, $style.buttonClose]">X</button>
03  </template>
04  
05  <!-- 使用CSS Modules-->
06  <style module>
07  .button {
08    border: none;
09    border-radius: 2px;
10  }
11  
12  .buttonClose {
13    background-color: red;
14  }
15  </style>
```

###### 避免在scoped中使用元素选择器
>在scoped样式中，类选择器比元素选择器更好，因为大量使用元素选择器是很慢的。

为了给样式设置作用域，Vue.js会为元素添加一个独一无二的特性，例如data-v-f3f3eg9。然后修改选择器，使得在匹配选择器的元素中，只有带这个特性的才会真正生效（比如button[data-v-f3f3eg9]）。

问题在于，大量的元素和特性组合的选择器（比如button[data-v-f3f3eg9]）会比类和特性组合的选择器慢，所以应该尽可能选用类选择器。

不好的例子：
```Vue
01  <template>
02    <button>X</button>
03  </template>
04  
05  <style scoped>
06  button {
07    background-color: red;
08  }
09  </style>
```
好的例子：
```Vue
01  <template>
02    <button class="btn btn-close">X</button>
03  </template>
04  
05  <style scoped>
06  .btn-close {
07    background-color: red;
08  }
09  </style>
```
##### 7、避免隐性的父子组件通信
>我们应该优先通过prop和事件进行父子组件之间的通信，而不是使用this.$parent或改变prop。

**一个理想的Vue.js应用是“prop向下传递，事件向上传递”。**遵循这一约定会让你的组件更容易理解。然而，在一些边界情况下，prop的变更或this.$parent能够简化两个深度耦合的组件。
问题在于，这种做法在很多简单的场景下可能会更方便。但要注意，不要为了一时方便（少写代码）而牺牲数据流向的简洁性（易于理解）。

##### 8、单文件组件如何命名
###### 单文件组件的文件名的大小写
单文件组件的文件名应该始终是单词首字母大写（PascalCase），或者始终是横线连接的（kebab-case）。

单词首字母大写对于代码编辑器的自动补全最为友好，因为这会使JS(X)和模板中引用组件的方式尽可能一致。然而，混用文件的命名方式有时候会导致文件系统对大小写不敏感的问题，这也是横线连接命名可取的原因。

不好的例子：
```
01  components/
02  |- mycomponent.vue
03  components/
04  |- myComponent.vue
```
好的例子：
```
01  components/
02  |- MyComponent.vue
03  components/
04  |- my-component.vue
```
###### 基础组件名
应用特定样式和约定的基础组件（也就是**展示类的、无逻辑的或无状态的组件**）应该全部以一个特定的前缀开头，比如Base、App或V。这些组件可以为你的应用奠定一致的基础样式和行为。它们可能只包括：

* HTML元素
* 其他基础组件
* 第三方UI组件库
**它们绝不会包括全局状态（比如来自Vuex store）**

它们的名字通常包含所包裹元素的名字（比如BaseButton、BaseTable），除非没有现成的对应功能的元素（比如 BaseIcon）。如果你为特定的上下文构建类似的组件，那么它们几乎总会消费这些组件（比如BaseButton可能会用在ButtonSubmit上）。

这样做的几个好处如下。

当你在编辑器中以字母顺序排序时，应用的基础组件会全部列在一起，这样更容易识别。
因为组件名应该始终是多个单词，所以这样做可以避免你在包裹简单组件时随意选择前缀（比如MyButton和VueButton）。
因为这些组件会被频繁使用，所以你可能想把它们放到全局而不是在各处分别导入它们。使用相同的前缀可以让webpack这样工作：
```
01  var requireComponent = require.context("./src", true, /^Base[A-Z]/)
02  requireComponent.keys().forEach(function (fileName) {
03    var baseComponentConfig = requireComponent(fileName)
04    baseComponentConfig = baseComponentConfig.default || baseComponentConfig
05    var baseComponentName = baseComponentConfig.name || (
06      fileName
07        .replace(/^.+\//, '')
08        .replace(/\.\w+$/, '')
09    )
10    Vue.component(baseComponentName, baseComponentConfig)
11  })
```
不好的例子：
```
01  components/
02  |- MyButton.vue
03  |- VueTable.vue
04  |- Icon.vue
```
好的例子：
```
01  components/
02  |- BaseButton.vue
03  |- BaseTable.vue
04  |- BaseIcon.vue
05  components/
06  |- AppButton.vue
07  |- AppTable.vue
08  |- AppIcon.vue
09  components/
10  |- VButton.vue
11  |- VTable.vue
12  |- VIcon.vue
```

###### 单例组件名
>只拥有单个活跃实例的组件以The前缀命名，以示其唯一性。但这并不意味着组件只可用于一个单页面，而是每个页面只使用一次。这些组件永远不接受任何prop，因为它们是为你的应用定制的，而不是应用中的上下文。如果你发现有必要添加prop，就表明这实际上是一个可复用的组件，只是目前在每个页面里只使用了一次。

不好的例子：
```
01  components/
02  |- Heading.vue
03  |- MySidebar.vue
```
好的例子：
```
01  components/
02  |- TheHeading.vue
03  |- TheSidebar.vue
```
