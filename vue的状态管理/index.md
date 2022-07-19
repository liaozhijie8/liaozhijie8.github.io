# Vue的状态管理

<!--more-->
> **[State Management Pattern](https://vuex.vuejs.org/)**
### 为什么需要状态管理？
* 在回答这个问题之前我们先看一下不用Vuex,是如何进行组件之间的数据传递的
  * 以Vue2为例子，组件之间的数据共享大致分为三部分  
  
| 类型 | 方式 |
| :------: | ------|
| 父组件->子组件   | **子组件中**,通过 props 来自定义属性 **父组件中**,负责把数据，通过 v-bind: 绑定给子组件|
| 子组件->父组件 | **子组件中**调用 this.$emit() 来触发自定义事件. **父组件**中，通过 v-on: 来绑定自定义事件，并提供一个事件处理函数。通过事件处理函数的形参，接收到子组件传递过来的数据。. |
| 兄弟组件共享数据    |EventBus 其实原理与子组件传父组件相似 |

  显然这种方式对于单个数据或者少量数据处理是没有问题的，但是官方提出了两个问题
  * Multiple views may depend on the same piece of state.
  * Actions from different views may need to mutate the same piece of state.
  + 当我们需要多组件共享数据或者处理数据时，会变得繁琐，尤其是当嵌套组件过多时，单向数据的传递会变得尤为复杂且低效
  + 为解决这两个问题,需要一个全局管理数据的工具-----Vuex
  
  ![](../../assets/image/vuex.png)

### 如何使用Vuex
#### 组成部分
| 组成 | 作用 |
| :---- | :---- |
|State | 存放全局数据 |
|Getters | 简单计算处理State中的数据,再传递给组件 |
|Mutations| 通过commit方法改变State中的数据,执行的是同步操作 |
|Actions| 通过dispatch方法改变State中的数据,执行的是异步操作 |
|Modules| 模块化|

### 使用步骤
* 利用createStore方法新建实例
  ```
  import { createStore } from 'vuex'
  import index from './cart/index'
  const store = createStore({
  modules: {
    cart: index //子模块
   }
  })
  export default store
  ```
* 在main函数中注册
  ```
  import { createApp } from 'vue'
  import App from './App.vue'
  import store from './store'  //引入注册
  const app = createApp(App)
  app.use(store)
  app.mount('#app')
  ```  
* 子模块定义数据方法
```
import { Goods } from './interface'
export default {
  // 开启空间命名
  namespaced: true,
  state: {
    goods: [
      { id: 0, name: '牙刷', price: 20, count: 22 },
      { id: 1, name: '平板', price: 2, count: 322 },
      { id: 2, name: '背包', price: 210, count: 266 },
      { id: 3, name: '电脑', price: 203, count: 255 },
      { id: 4, name: '手机', price: 204, count: 23 }
    ],
    totalPrice: 0
  },
  getters: {
    getPrice: (state: Goods) => (price: number) => {
      return state.goods.filter(d => d.price > price)
    }
  },
  mutations: {
    addPrice(state: Goods, id: number) {
      state.totalPrice += state.goods[id].price
    }
  },
  actions: {
    fetchTotalPrice({ commit }, payload: number) {
      commit('addPrice', payload)
    }
  }
}
```
* 在组件中使用 **需要注意引用子模块的方式**
```
import { useStore } from 'vuex'
setup() {
    const store = useStore()
    // 引用方法store.属性.模块名称.数据
    const goods = computed(() => store.state.cart.goods)
    const totalPrice = computed(() => store.state.cart.totalPrice)
    const chooseGoods = (id: number) => {
      // 使用action
      store.dispatch('cart/fetchTotalPrice', id)
    }
    // // // 使用getter
    const priceGoods = computed(() => store.getters['cart/getPrice'](100))
    return {
      goods,
      priceGoods,
      totalPrice,
      chooseGoods
    }
  }
})
```

### 全新的版本**pinia**
> [**Why should I use Pinia?**](https://pinia.vuejs.org/introduction.html)

