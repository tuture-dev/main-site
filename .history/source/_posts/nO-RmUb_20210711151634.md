---
title: '从零到部署：用 Vue 和 Express 实现迷你全栈电商应用（九）：使用 Authing 集成完整的用户系统'
description: '在之前的迷你电商应用中，我们的网站缺少了一个关键组成部分：用户鉴权系统，包括登录、注册、以及权限管理等相应的配置。徒手实现这些功能固然可行，但是对于一支崇尚精益的团队来说，选择可靠的身份认证服务（IDaaS）是更加明智的选择，不仅能够提供完善且丰富的身份认证和用户鉴权功能，还确保遵循最佳安全实践和优秀的可扩展性。在这篇教程中，我们将手把手带你在本系列之前完成的迷你电商应用中快速集成 Authing 用户认证模块，提供一致、流畅、安全的身份认证体验。'
tags: ['Vue', 'Express', 'Authing', 'OAuth', '第三方登录']
categories: ['前端', 'Vue', '入门']
date: 2020-05-17T02:25:27.336Z
photos:
  - https://static.powerformer.com/c/092faf4/vue-cover-9.png
---

<div class="profileBox">
  <div class="avatarBox">
    <a href="https://github.com/tuture-dev"><img src="/images/avatars/tuture-dev.jpg" alt="" class="avatar"></a>
  </div>
  <div class="rightBox">
    <div class="infoBox">
    <a href="https://github.com/tuture-dev"><p class="nickName">@tuture-dev</p></a>
  </div>
  <div class="codeBox">
    <a href="https://github.com/tuture-dev/vue-online-shop-frontend"><span class="codeText">查看代码</span></a>
  </div>
  </div>
</div>

## 添加用户界面及路由

```js client/src/main.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/0bc64d13046e72ce7fd9529cb7efbb69c011d1c7/client/src/main.js 查看完整代码
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
[tuture-del]import Vue from 'vue';
[tuture-del]import { ValidationProvider } from 'vee-validate';
[tuture-add]import Vue from "vue";
[tuture-add]import { ValidationProvider } from "vee-validate";

[tuture-del]import App from './App';
[tuture-del]import router from './router';
[tuture-del]import store from './store';
[tuture-del]import ElementUI from 'element-ui'
[tuture-del]import 'element-ui/lib/theme-chalk/index.css'
[tuture-add]import App from "./App";
[tuture-add]import router from "./router";
[tuture-add]import store from "./store";
[tuture-add]import ElementUI from "element-ui";
[tuture-add]import "element-ui/lib/theme-chalk/index.css";

Vue.config.productionTip = false;
[tuture-del]Vue.component('ValidationProvider', ValidationProvider);
[tuture-add]Vue.component("ValidationProvider", ValidationProvider);
Vue.use(ElementUI);

/* eslint-disable no-new */
new Vue({
[tuture-del]  el: '#app',
[tuture-add]  el: "#app",
  router,
  store,
  components: { App },
[tuture-del]  template: '<App/>',
[tuture-add]  template: "<App/>"
});
```

```html client/src/pages/user/Index.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/0bc64d13046e72ce7fd9529cb7efbb69c011d1c7/client/src/pages/user/Index.vue 查看完整代码
<template>
  <div>
    <div class="container">
      <div class="container">
        <h1 class="user-title">
          <router-link to="/" tag="div">登录/注册</router-link>
        </h1>
        <router-view />
      </div>
    </div>
  </div>
</template>

<style>
  .user-title:hover {
    cursor: pointer;
  }

  .container {
    margin-top: 40px;
  }
</style>
```

```html client/src/pages/user/Login.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/0bc64d13046e72ce7fd9529cb7efbb69c011d1c7/client/src/pages/user/Login.vue 查看完整代码
<template>
  <div id="login-form">用户登录</div>
</template>

<script>
  export default {};
</script>
```

```html client/src/pages/user/Setting.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/0bc64d13046e72ce7fd9529cb7efbb69c011d1c7/client/src/pages/user/Setting.vue 查看完整代码
<template>
  <div>settings</div>
</template>

<script>
  export default {
    data() {
      return {
        model: { manufacturer: { name: '', _id: '' } },
      };
    },
    mounted() {},
  };
</script>
```

```js client/src/router/index.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/0bc64d13046e72ce7fd9529cb7efbb69c011d1c7/client/src/router/index.js 查看完整代码
[tuture-del]import Vue from 'vue';
[tuture-del]import Router from 'vue-router';
[tuture-add]import Vue from "vue";
[tuture-add]import Router from "vue-router";

[tuture-del]import Home from '@/pages/Home';
[tuture-del]import Cart from '@/pages/Cart';
[tuture-del]import Detail from '@/pages/Detail';
[tuture-add]import Home from "@/pages/Home";
[tuture-add]import Cart from "@/pages/Cart";
[tuture-add]import Detail from "@/pages/Detail";

// Admin Components
[tuture-del]import Index from '@/pages/admin/Index';
[tuture-del]import New from '@/pages/admin/New';
[tuture-del]import Products from '@/pages/admin/Products';
[tuture-del]import Edit from '@/pages/admin/Edit';
[tuture-del]import Manufacturers from '@/pages/admin/Manufacturers';
[tuture-del]import NewManufacturers from '@/pages/admin/NewManufacturers';
[tuture-del]import EditManufacturers from '@/pages/admin/EditManufacturers';
[tuture-add]import Index from "@/pages/admin/Index";
[tuture-add]import New from "@/pages/admin/New";
[tuture-add]import Products from "@/pages/admin/Products";
[tuture-add]import Edit from "@/pages/admin/Edit";
[tuture-add]import Manufacturers from "@/pages/admin/Manufacturers";
[tuture-add]import NewManufacturers from "@/pages/admin/NewManufacturers";
[tuture-add]import EditManufacturers from "@/pages/admin/EditManufacturers";
[tuture-add]import UserIndex from "@/pages/user/Index";
[tuture-add]import Login from "@/pages/user/Login";
[tuture-add]import Setting from "@/pages/user/Setting";

Vue.use(Router);

[tuture-del]export default new Router({
[tuture-add]const router = new Router({
  routes: [
    {
[tuture-del]      path: '/',
[tuture-del]      name: 'Home',
[tuture-del]      component: Home,
[tuture-add]      path: "/",
[tuture-add]      name: "Home",
[tuture-add]      component: Home
    },
    {
[tuture-del]      path: '/admin',
[tuture-del]      name: 'Admin',
[tuture-add]      path: "/admin",
[tuture-add]      name: "Admin",
      component: Index,
      children: [
        {
[tuture-del]          path: 'new',
[tuture-del]          name: 'New',
[tuture-del]          component: New,
[tuture-add]          path: "new",
[tuture-add]          name: "New",
[tuture-add]          component: New
        },
        {
[tuture-del]          path: '',
[tuture-del]          name: 'Products',
[tuture-del]          component: Products,
[tuture-add]          path: "",
[tuture-add]          name: "Products",
[tuture-add]          component: Products
        },
        {
[tuture-del]          path: 'edit/:id',
[tuture-del]          name: 'Edit',
[tuture-del]          component: Edit,
[tuture-add]          path: "edit/:id",
[tuture-add]          name: "Edit",
[tuture-add]          component: Edit
        },
        {
[tuture-del]          path: 'manufacturers',
[tuture-del]          name: 'Manufacturers',
[tuture-del]          component: Manufacturers,
[tuture-add]          path: "manufacturers",
[tuture-add]          name: "Manufacturers",
[tuture-add]          component: Manufacturers
        },
        {
[tuture-del]          path: 'manufacturers/new',
[tuture-del]          name: 'NewManufacturers',
[tuture-del]          component: NewManufacturers,
[tuture-add]          path: "manufacturers/new",
[tuture-add]          name: "NewManufacturers",
[tuture-add]          component: NewManufacturers
        },
        {
[tuture-del]          path: 'manufacturers/edit/:id',
[tuture-del]          name: 'EditManufacturers',
[tuture-del]          component: EditManufacturers,
[tuture-del]        },
[tuture-add]          path: "manufacturers/edit/:id",
[tuture-add]          name: "EditManufacturers",
[tuture-add]          component: EditManufacturers
[tuture-add]        }
      ]
    },
    {
[tuture-del]      path: '/cart',
[tuture-del]      name: 'Cart',
[tuture-del]      component: Cart,
[tuture-add]      path: "/cart",
[tuture-add]      name: "Cart",
[tuture-add]      component: Cart
[tuture-add]    },
[tuture-add]    {
[tuture-add]      path: "/detail/:id",
[tuture-add]      name: "Detail",
[tuture-add]      component: Detail
    },
    {
[tuture-del]      path: '/detail/:id',
[tuture-del]      name: 'Detail',
[tuture-del]      component: Detail,
[tuture-add]      path: "/user",
[tuture-add]      name: "User",
[tuture-add]      component: UserIndex,
[tuture-add]      children: [
[tuture-add]        {
[tuture-add]          path: "login",
[tuture-add]          name: "Login",
[tuture-add]          component: Login
[tuture-add]        },
[tuture-add]        {
[tuture-add]          path: "settings",
[tuture-add]          name: "Settings",
[tuture-add]          component: Setting
[tuture-add]        }
[tuture-add]      ]
    }
[tuture-del]  ],
[tuture-add]  ]
});
[tuture-add]
[tuture-add]export default router;
```

## 使用 Authing 接入用户系统

在这一步骤中，我们将正式使用 [Authing](https://authing.cn/?utm_source=tuture) 接入用户系统。Authing 是国内出色的身份认证云，能让我们轻松集成身份认证相关的逻辑，对于个人开发者来说，其[免费使用额度](https://authing.cn/pricing?utm_source=tuture)也是相当充足的。

首先，让我们访问 Authing 官方网站，点击右上角的登录按钮，如下图所示：

![](https://static.powerformer.com/c/092faf4/172281d3055eca8c.png)

进入到登录页面后，我们输入帐户名和密码，会直接为我们创建帐号：

![](https://static.powerformer.com/c/092faf4/172281d3095f926b.png)

进入到控制台后，让我们创建一个新的[用户池](https://docs.authing.cn/authing/quickstart/basic#yong-hu-chi?utm_source=tuture)（顾名思义，就是用来管理和存储一系列用户的数据和信息），如下图所示：

![](https://static.powerformer.com/c/092faf4/172281d309214195.jpeg)

在创建用户池的时候，输入我们想要的用户池名称和专属域名后，选择类型为 Web，最后点击，我们的第一个用户池边创建好了。点击“基础配置”书签，可以查看到刚才创建用户池的一些关键信息，特别是用户池 ID，如下图所示：

![](https://static.powerformer.com/c/092faf4/172281d32d134626.jpeg)

{% note warning %}
**注意**

后续应用开发时，所有的用户池 ID（`userPoolId`）请替换成自己帐户的真实 ID。
{% endnote %}

### 配置 Vuex Mutations 和 Store

由于我们的应用使用了 [Vuex](https://vuex.vuejs.org/) 来解决状态管理问题，因此我们首先需要定义身份验证相关的 Mutation。这里我们定义两个新的 Mutation：

- `SET_USER` ：设置用户身份数据
- `LOGOUT` ：退出登录

在 `client/src/store/mutation-types.js` 中添加上面三个 Mutation 常量，代码如下：

```js client/src/store/mutation-types.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/f8e83ff89ed688548582d08d4c81a57981187a17/client/src/store/mutation-types.js 查看完整代码
[tuture-del]export const ALL_PRODUCTS = 'ALL_PRODUCTS';
[tuture-del]export const ALL_PRODUCTS_SUCCESS = 'ALL_PRODUCTS_SUCCESS';
[tuture-add]export const ALL_PRODUCTS = "ALL_PRODUCTS";
[tuture-add]export const ALL_PRODUCTS_SUCCESS = "ALL_PRODUCTS_SUCCESS";

[tuture-del]export const PRODUCT_BY_ID = 'PRODUCT_BY_ID';
[tuture-del]export const PRODUCT_BY_ID_SUCCESS = 'PRODUCT_BY_ID_SUCCESS';
[tuture-add]export const PRODUCT_BY_ID = "PRODUCT_BY_ID";
[tuture-add]export const PRODUCT_BY_ID_SUCCESS = "PRODUCT_BY_ID_SUCCESS";

[tuture-del]export const ADD_PRODUCT = 'ADD_PRODUCT';
[tuture-del]export const ADD_PRODUCT_SUCCESS = 'ADD_PRODUCT_SUCCESS';
[tuture-add]export const ADD_PRODUCT = "ADD_PRODUCT";
[tuture-add]export const ADD_PRODUCT_SUCCESS = "ADD_PRODUCT_SUCCESS";

[tuture-del]export const UPDATE_PRODUCT = 'UPDATE_PRODUCT';
[tuture-del]export const UPDATE_PRODUCT_SUCCESS = 'UPDATE_PRODUCT_SUCCESS';
[tuture-add]export const UPDATE_PRODUCT = "UPDATE_PRODUCT";
[tuture-add]export const UPDATE_PRODUCT_SUCCESS = "UPDATE_PRODUCT_SUCCESS";

[tuture-del]export const REMOVE_PRODUCT = 'REMOVE_PRODUCT';
[tuture-del]export const REMOVE_PRODUCT_SUCCESS = 'REMOVE_PRODUCT_SUCCESS';
[tuture-add]export const REMOVE_PRODUCT = "REMOVE_PRODUCT";
[tuture-add]export const REMOVE_PRODUCT_SUCCESS = "REMOVE_PRODUCT_SUCCESS";

[tuture-del]export const ADD_TO_CART = 'ADD_TO_CART';
[tuture-del]export const REMOVE_FROM_CART = 'REMOVE_FROM_CART';
[tuture-add]export const ADD_TO_CART = "ADD_TO_CART";
[tuture-add]export const REMOVE_FROM_CART = "REMOVE_FROM_CART";

[tuture-del]export const ALL_MANUFACTURERS = 'ALL_MANUFACTURER';
[tuture-del]export const ALL_MANUFACTURERS_SUCCESS = 'ALL_MANUFACTURER_S';
[tuture-add]export const ALL_MANUFACTURERS = "ALL_MANUFACTURER";
[tuture-add]export const ALL_MANUFACTURERS_SUCCESS = "ALL_MANUFACTURER_S";

[tuture-del]export const MANUFACTURER_BY_ID = 'MANUFACTURER_BY_ID';
[tuture-del]export const MANUFACTURER_BY_ID_SUCCESS = 'MANUFACTURER_BY_ID_SUCCESS';
[tuture-add]export const MANUFACTURER_BY_ID = "MANUFACTURER_BY_ID";
[tuture-add]export const MANUFACTURER_BY_ID_SUCCESS = "MANUFACTURER_BY_ID_SUCCESS";

[tuture-del]export const ADD_MANUFACTURER = 'ADD_MANUFACTURER';
[tuture-del]export const ADD_MANUFACTURER_SUCCESS = 'ADD_MANUFACTURER_SUCCESS';
[tuture-add]export const ADD_MANUFACTURER = "ADD_MANUFACTURER";
[tuture-add]export const ADD_MANUFACTURER_SUCCESS = "ADD_MANUFACTURER_SUCCESS";

[tuture-del]export const UPDATE_MANUFACTURER = 'UPDATE_MANUFACTURER';
[tuture-del]export const UPDATE_MANUFACTURER_SUCCESS = 'UPDATE_MANUFACTURER_SUCCESS';
[tuture-add]export const UPDATE_MANUFACTURER = "UPDATE_MANUFACTURER";
[tuture-add]export const UPDATE_MANUFACTURER_SUCCESS = "UPDATE_MANUFACTURER_SUCCESS";

[tuture-del]export const REMOVE_MANUFACTURER = 'REMOVE_MANUFACTURER';
[tuture-del]export const REMOVE_MANUFACTURER_SUCCESS = 'REMOVE_MANUFACTURER_SUCCESS';
[tuture-add]export const REMOVE_MANUFACTURER = "REMOVE_MANUFACTURER";
[tuture-add]export const REMOVE_MANUFACTURER_SUCCESS = "REMOVE_MANUFACTURER_SUCCESS";
[tuture-add]
[tuture-add]export const SET_USER = "SET_USER";
[tuture-add]export const UPDATE_USER = "UPDATE_USER";
[tuture-add]export const LOGOUT = "LOGOUT";
```

然后我们在 `client/src/store/mutations.js` 中实现上面定义用户相关 Mutation，代码如下：

```js client/src/store/mutations.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/f8e83ff89ed688548582d08d4c81a57981187a17/client/src/store/mutations.js 查看完整代码
// ...
  UPDATE_MANUFACTURER_SUCCESS,
  REMOVE_MANUFACTURER,
  REMOVE_MANUFACTURER_SUCCESS,
[tuture-del]} from './mutation-types';
[tuture-del]import { Message } from 'element-ui';
[tuture-add]  SET_USER,
[tuture-add]  UPDATE_USER,
[tuture-add]  LOGOUT
[tuture-add]} from "./mutation-types";
[tuture-add]import { Message } from "element-ui";
[tuture-add]
[tuture-add]export const userMutations = {
[tuture-add]  [SET_USER](state, payload) {
[tuture-add]    state.user = payload;
[tuture-add]  },
[tuture-add]  [LOGOUT](state) {
[tuture-add]    state.user = {};
[tuture-add]  }
[tuture-add]};

export const productMutations = {
  [ALL_PRODUCTS](state) {
    // ...
    state.showLoader = false;

    const { productId } = payload;
[tuture-del]    state.products = state.products.filter(product => product._id !== productId);
[tuture-add]    state.products = state.products.filter(
[tuture-add]      product => product._id !== productId
[tuture-add]    );
  },
  [UPDATE_PRODUCT](state) {
    state.showLoader = true;
  // ...

    const { product: newProduct } = payload;
    state.product = newProduct;
[tuture-del]    state.products = state.products.map( product => {
[tuture-add]    state.products = state.products.map(product => {
      if (product._id === newProduct._id) {
        return newProduct;
      }
      // ...
    const { product } = payload;
    state.cart.push(product);
    Message({
[tuture-del]      message: '恭喜你，成功加入购物车！',
[tuture-del]      type: 'success'
[tuture-del]    })
[tuture-add]      message: "恭喜你，成功加入购物车！",
[tuture-add]      type: "success"
[tuture-add]    });
  },
  [REMOVE_FROM_CART](state, payload) {
[tuture-del]    const { productId } = payload
[tuture-del]    state.cart = state.cart.filter(product => product._id !== productId)
[tuture-add]    const { productId } = payload;
[tuture-add]    state.cart = state.cart.filter(product => product._id !== productId);
    Message({
[tuture-del]      message: '恭喜你，成功移除购物车！',
[tuture-del]      type: 'success'
[tuture-del]    })
[tuture-del]  },
[tuture-add]      message: "恭喜你，成功移除购物车！",
[tuture-add]      type: "success"
[tuture-add]    });
[tuture-add]  }
};

export const manufacturerMutations = {
  // ...
    state.showLoader = false;

    const { manufacturerId } = payload;
[tuture-del]    state.manufacturers = state.manufacturers.filter(manufacturer => manufacturer._id !== manufacturerId);
[tuture-add]    state.manufacturers = state.manufacturers.filter(
[tuture-add]      manufacturer => manufacturer._id !== manufacturerId
[tuture-add]    );
  },
  [UPDATE_MANUFACTURER](state) {
    state.showLoader = true;
  // ...
    const { manufacturer } = payload;
    state.manufacturers = state.manufacturers.concat(manufacturer);
  }
[tuture-del]}
[tuture-add]};
```

最后我们在 Vuex Store 中集成相应的状态与 Mutation，修改 `client/src/store/index.js` ，代码如下：

```js client/src/store/index.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/f8e83ff89ed688548582d08d4c81a57981187a17/client/src/store/index.js 查看完整代码
// ...
[tuture-add]import { productGetters, manufacturerGetters } from "./getters";
[tuture-add]import {
[tuture-add]  productMutations,
[tuture-add]  cartMutations,
[tuture-add]  manufacturerMutations,
[tuture-add]  userMutations
[tuture-add]} from "./mutations";
[tuture-add]import { productActions, manufacturerActions } from "./actions";

Vue.use(Vuex);

export default new Vuex.Store({
  strict: true,
  state: {
    // ...
[tuture-add]    // userInfo
[tuture-add]    user: {}
  },
  mutations: {
    ...productMutations,
    ...cartMutations,
    ...manufacturerMutations,
[tuture-add]    ...userMutations
  },
  // ...
});
```

### 在根组件 App 中集成用户逻辑

让我们打开根组件 `client/src/App.vue`，在其中添加一个 `mounted` 方法，使得在整个应用刚启动时获取并检查用户身份数据。修改代码如下：

```html client/src/App.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/f8e83ff89ed688548582d08d4c81a57981187a17/client/src/App.vue 查看完整代码
// ...

<script>
  export default {
  [tuture-del]  name: "App"
  [tuture-add]  name: "App",
  [tuture-add]  mounted() {
  [tuture-add]    const userInfo = localStorage.getItem("userInfo");
  [tuture-add]
  [tuture-add]    if (userInfo) {
  [tuture-add]      this.$store.commit("SET_USER", JSON.parse(userInfo));
  [tuture-add]    }
  [tuture-add]  }
  };
</script>

// ...
```

可以看到，我们从 `localStorage` 中检查是否有 `userInfo` 数据，如果有的话通过 `SET_USER` Mutation 将用户身份数据存入状态中。

### 在头部组件 Header 中集成用户逻辑

打开头部组件 `client/src/components/Header.vue` ，我们在其中添加用户系统相关的逻辑，修改代码如下：

```html client/src/components/Header.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/f8e83ff89ed688548582d08d4c81a57981187a17/client/src/components/Header.vue 查看完整代码
// ...

<script>
  export default {
    props: ["activeIndex"],
    data() {
      return {
  [tuture-del]      isLogged: false
  [tuture-add]      model: { manufacturer: { name: "", _id: "" } }
      };
    },
  [tuture-add]  computed: {
  [tuture-add]    isLogged() {
  [tuture-add]      let token = this.$store.state.user.token;
  [tuture-add]
  [tuture-add]      return !!token;
  [tuture-add]    },
  [tuture-add]    avatar() {
  [tuture-add]      let photo = this.$store.state.user.photo;
  [tuture-add]
  [tuture-add]      return photo;
  [tuture-add]    }
  [tuture-add]  },
    methods: {
      handleLogout() {
  [tuture-del]      // 处理登出逻辑
  [tuture-add]      localStorage.removeItem("token");
  [tuture-add]      localStorage.removeItem("userInfo");
  [tuture-add]      this.$store.commit("LOGOUT");
      }
    }
  };
</script>
```

可以看到，我们主要做了以下改变：

- 把 `isLogged` 从原先的 `data` 变成了一个计算属性，通过从 Vuex Store 中获取 `token` 是否存在来判断是否登录
- 添加了 `avatar` 计算属性，用于从 Store 中获取用户头像
- 实现了 `handleLogout` 方法，用于处理登出逻辑，包括从 `localStorage` 中去除 `token` 和 `userInfo` 数据，并发起一个 `LOGOUT` Mutation 用于更新 Store 的状态

### 通过 Authing Guard 实现登录页面

[Guard ](https://docs.authing.cn/authing/sdk/guard?utm_source=tuture)是 Authing 推出的可嵌入登录表单，能够让我们用几行代码为整个应用集成登录和注册功能，集成后的效果如下：

![](https://static.powerformer.com/c/092faf4/172281d333d68545.png)

整体效果还是很 OK 的，而且我们可以通过一些配置项轻松实现定制，下面我们就来看看怎么实现吧。

首先，我们通过引入 Authing UMD 构建文件来集成 [Authing Guard](https://docs.authing.cn/authing/sdk/guard?utm_source=tuture)。在 `client/index.html` 文件中通过 `script` 标签引入：

```html client/index.html https://github.com/tuture-dev/vue-online-shop-frontend/blob/f8e83ff89ed688548582d08d4c81a57981187a17/client/index.html 查看完整代码
<!DOCTYPE html>
<html>
  <head>
    [tuture-del]
    <meta charset="utf-8" />
    [tuture-del]
    <meta name="viewport" content="width=device-width,initial-scale=1.0" />
    [tuture-add]
    <meta charset="utf-8" />
    [tuture-add]
    <meta name="viewport" content="width=device-width,initial-scale=1.0" />
    <title>vue-online-shop</title>
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
    [tuture-add]
    <script src="https://cdn.jsdelivr.net/npm/@authing/guard/dist/Guard.umd.min.js"></script>
  </body>
</html>
```

接着打开登录页面组件 `client/src/pages/user/Login.vue`，修改代码如下：

```html client/src/pages/user/Login.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/f8e83ff89ed688548582d08d4c81a57981187a17/client/src/pages/user/Login.vue 查看完整代码
<template>
  [tuture-del]
  <div id="login-form">用户登录</div>
  [tuture-add]
  <div id="login-form"></div>
</template>

<script>
  [tuture-del]export default {};
  [tuture-add]export default {
  [tuture-add]  data() {
  [tuture-add]    return {
  [tuture-add]      model: { manufacturer: { name: "", _id: "" } }
  [tuture-add]    };
  [tuture-add]  },
  [tuture-add]  mounted() {
  [tuture-add]    const appId = "";
  [tuture-add]    const userPoolId = "";
  [tuture-add]    const domain = "https://tuture-first.authing.co";
  [tuture-add]
  [tuture-add]    const form = new Guard(userPoolId, {
  [tuture-add]      logo: "https://tuture.co/images/avatar.png",
  [tuture-add]      title: "图雀全栈迷你电商",
  [tuture-add]      mountId: "login-form",
  [tuture-add]      hideClose: true
  [tuture-add]    });
  [tuture-add]
  [tuture-add]    const that = this;
  [tuture-add]
  [tuture-add]    form.on("authenticated", userInfo => {
  [tuture-add]      that.$store.commit("SET_USER", userInfo);
  [tuture-add]      localStorage.setItem("token", JSON.stringify(userInfo.token));
  [tuture-add]      localStorage.setItem("userInfo", JSON.stringify(userInfo));
  [tuture-add]
  [tuture-add]      that.$router.push("/");
  [tuture-add]    });
  [tuture-add]  }
  [tuture-add]};
</script>
```

我们在 `mounted` 生命周期方法中初始化 `Guard` 实例。在初始化 `Guard` 实例时，第一个参数是用户池 ID（**_记得换成自己的用户池 ID！_**），可以通过 Authing 控制台获取，第二个参数则是 Guard 组件的一些选项参数：

- `logo` 是我们整个网站的 Logo 图片链接
- `title` 是整个登录表单的标题
- `mountId` 是用于挂载登录表单的 DOM ID，这里就是模板中唯一的 `div` 元素 `login-form`
- `hideClose` 用于隐藏关闭按钮，因为我们把登录做成了一个独立的页面，不希望用户把登录表单关掉（这样整个页面就一片空白啦）

{% note info %}
**提示**

关于 `Guard` 完整的构造函数 API，请参考[官方文档](https://docs.authing.cn/authing/sdk/guard#gou-zao-han-shu-new-guarduserpoolid-options?utm_source=tuture)。
{% endnote %}

在初始化 `Guard` 组件后，我们还需要添加身份验证成功后的监听事件函数，即 `form.on("authenticated", handler)` 。可以看到，在回调函数中，我们做了三件事：

1. 发出 `SET_USER` Mutation，修改 Store 状态
2. 在 `localStorage` 中存储登录后获取的用户信息
3. 通过 `$router` 路由重定向到首页

{% note info %}
**提示**

更多回调事件，可参考[完整事件列表](https://docs.authing.cn/authing/sdk/guard#hui-tiao-shi-jian-onevent-callback?utm_source=tuture)。
{% endnote %}

配置完成后，开启应用，点击登录按钮，就可以看到我们炫酷的登录页面了：

![](https://static.powerformer.com/c/092faf4/172281d336625305.jpeg)

看上去很不错！

## 添加权限管理和路由守卫

在这一步中，我们将配置权限管理和路由守卫。权限管理很容易理解，就是当用户进行某些需要登录的操作（例如添加到购物车）时判断是否已经登录，如果未登录则重定向到登录页面。所谓[路由守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html?utm_source=tuture)（或称**导航守卫**），就是在进入一个具体的路由（页面）**之前**，判断用户是否具备足够的权限，如果权限不够，则直接重定向到登录页面，否则允许进入该页面。

在我们的应用中，主要有三个地方需要配置权限：

- 商品添加按钮（ProductionButton）
- 购物车（Cart）
- 后台管理（Admin）

让我们逐个击破吧。

### 添加商品添加按钮的权限管理

首先，我们需要为商品添加按钮配置权限管理。打开 `client/src/components/products/ProductButton.vue` 组件，修改 `methods` 中的 `addToCart` 和 `removeFromCart` 方法，代码如下：

```html client/src/components/products/ProductButton.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/10225335e6195fd3cb4d98c84b64f3d2807048af/client/src/components/products/ProductButton.vue 查看完整代码
// ...
<script>
  export default {
    // ...
    methods: {
      addToCart() {
  [tuture-del]      this.$store.commit("ADD_TO_CART", {
  [tuture-del]        product: this.product
  [tuture-del]      });
  [tuture-add]      const token = localStorage.getItem("token");
  [tuture-add]      const that = this;
  [tuture-add]
  [tuture-add]      if (token) {
  [tuture-add]        this.$store.commit("ADD_TO_CART", {
  [tuture-add]          product: this.product
  [tuture-add]        });
  [tuture-add]      } else {
  [tuture-add]        this.$confirm(
  [tuture-add]          "你还未登录，点击去登录跳转登录页面，点击取消回到主界面",
  [tuture-add]          "提示",
  [tuture-add]          {
  [tuture-add]            confirmButtonText: "去登录",
  [tuture-add]            cancelButtonText: "取消",
  [tuture-add]            type: "warning"
  [tuture-add]          }
  [tuture-add]        )
  [tuture-add]          .then(() => {
  [tuture-add]            that.$router.push("/user/login");
  [tuture-add]          })
  [tuture-add]          .catch(() => {
  [tuture-add]            this.$message({
  [tuture-add]              type: "info",
  [tuture-add]              message: "你已取消"
  [tuture-add]            });
  [tuture-add]          });
  [tuture-add]      }
      },
      removeFromCart(productId) {
  [tuture-del]      this.$store.commit("REMOVE_FROM_CART", {
  [tuture-del]        productId
  [tuture-del]      });
  [tuture-add]      const token = localStorage.getItem("token");
  [tuture-add]      const that = this;
  [tuture-add]
  [tuture-add]      if (token) {
  [tuture-add]        this.$store.commit("REMOVE_FROM_CART", {
  [tuture-add]          productId
  [tuture-add]        });
  [tuture-add]      } else {
  [tuture-add]        this.$alert(
  [tuture-add]          "点击去登录跳转登录页面，点击取消回到主界面",
  [tuture-add]          "你还未登录",
  [tuture-add]          {
  [tuture-add]            confirmButtonText: "去登录",
  [tuture-add]            cancelButtonText: "取消"
  [tuture-add]          }
  [tuture-add]        )
  [tuture-add]          .then(() => {
  [tuture-add]            that.$router.push("/user/login");
  [tuture-add]          })
  [tuture-add]          .catch(() => {
  [tuture-add]            this.$message({
  [tuture-add]              type: "info",
  [tuture-add]              message: "你已取消"
  [tuture-add]            });
  [tuture-add]          });
  [tuture-add]      }
      }
    }
  };
</script>
```

可以看到，实现权限管理的思路很简单：先从 `localStorage` 中判断用于鉴权的 `token` 是否存在，如果存在则表明已登录，执行相应的 Mutation；如果不存在 `token`，则弹出 Alert 提示框询问用户是否需要跳转到登录页面进行登录。

### 实现购物车的路由守卫

然后我们来实现购物车的路由守卫。幸运的是，Vue Router 已经为我们提供了[组件级别的路由守卫的方法](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E7%BB%84%E4%BB%B6%E5%86%85%E7%9A%84%E5%AE%88%E5%8D%AB?utm_source=tuture) `beforeRouteEnter` 。打开 `client/src/pages/Cart.vue`，修改代码如下：

```html client/src/pages/Cart.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/10225335e6195fd3cb4d98c84b64f3d2807048af/client/src/pages/Cart.vue 查看完整代码
// ...
<script>
  // ...

  export default {
    name: "home",
    // ...
  [tuture-add]  beforeRouteEnter(to, from, next) {
  [tuture-add]    const token = localStorage.getItem("token");
  [tuture-add]
  [tuture-add]    if (!token) {
  [tuture-add]      next("/user/login");
  [tuture-add]    } else {
  [tuture-add]      next();
  [tuture-add]    }
    }
  };
</script>
```

依然是通过 `localStorage` 中尝试获取 `token` 来判断登录状态，然后通过 `next` 函数进入合适的路由。

### 实现后台管理的路由守卫

类似地，我们实现后台管理页面的路由守卫。打开 `client/src/pages/admin.Index.vue`，添加路由守卫方法，代码如下：

```html client/src/pages/admin/Index.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/10225335e6195fd3cb4d98c84b64f3d2807048af/client/src/pages/admin/Index.vue 查看完整代码
// ...
<script>
  // ...

  export default {
    // ...
  [tuture-add]  beforeRouteEnter(to, from, next) {
  [tuture-add]    const token = localStorage.getItem("token");
  [tuture-add]
  [tuture-add]    if (!token) {
  [tuture-add]      next("/user/login");
  [tuture-add]    } else {
  [tuture-add]      next();
  [tuture-add]    }
    }
  };
</script>
```

完成这一步后，打开应用，我们来看一下添加了权限管理和路由守卫的之后的应用会是怎么样的：

![](https://static.powerformer.com/c/092faf4/172281d34166fbdd.gif)

## 将用户系统与现有的数据库集成

仅仅实现登录和注册功能是远远不够的，我们还需要将用户系统集成到现有的数据库中。例如我们在添加商品时，希望能够和具体的用户绑定。

所幸我们使用的是 MongoDB 数据库，因此不像传统的关系型数据库那样需要繁杂的表结构更新，只需修改数据模型定义即可。

### 更新 Mongoose 数据定义

首先让我们来更新一波 Mongoose 数据定义。打开 `server/model/index.js` ，修改代码如下：

```js server/model/index.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/d1c5dad8ef4db3d706cb489dad23996846b9d6b7/server/model/index.js 查看完整代码
// ...

const productSchema = Schema({
  id: ObjectId,
  name: String,
  image: String,
[tuture-del]  price: Number,
[tuture-add]  price: String,
  description: String,
[tuture-del]  manufacturer: { type: ObjectId, ref: 'Manufacturer' }
[tuture-add]  user: String,
[tuture-add]  manufacturer: { type: ObjectId, ref: "Manufacturer" },
});

const manufacturerSchema = Schema({
  id: ObjectId,
  name: String,
[tuture-add]  user: String,
});

// ...
```

可以看到，我们主要是在两个数据模型 `productSchema` 和 `manufacturerSchema` 中加入了 `user` 字段，其他均无需改变。

### 配置 Vuex Action

接着我们修改项目的 Action，主要是在两个新增数据的 Action（`addProduct` 和 `addManufacturer`）创建模型时记录用户数据。打开 `client/src/store/actions.js`，修改代码如下：

```js client/src/store/actions.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/d1c5dad8ef4db3d706cb489dad23996846b9d6b7/client/src/store/actions.js 查看完整代码
// ...
export const productActions = {
  // ...
[tuture-del]  addProduct({ commit }, payload) {
[tuture-add]  addProduct({ commit, state }, payload) {
    commit(ADD_PRODUCT);

    const { product } = payload;
[tuture-add]    const _id = state.user._id;
    axios
[tuture-del]      .post(`${API_BASE}/products`, product)
[tuture-add]      .post(`${API_BASE}/products`, {
[tuture-add]        ...product,
[tuture-add]        user: _id,
[tuture-add]        manufacturer: product.manufacturer._id
[tuture-add]      })
      .then(response => {
        // ...
      })
      .catch(() => {
        // ...
      });
  }
};

export const manufacturerActions = {
  // ...
[tuture-del]  addManufacturer({ commit }, payload) {
[tuture-add]  addManufacturer({ commit, state }, payload) {
    commit(ADD_MANUFACTURER);
    const { manufacturer } = payload;
[tuture-add]    const _id = state.user._id;
[tuture-add]
    axios
[tuture-del]      .post(`${API_BASE}/manufacturers`, manufacturer)
[tuture-add]      .post(`${API_BASE}/manufacturers`, { ...manufacturer, user: _id })
      .then(response => {
        // ...
      })
      .catch(() => {
        // ...
      });
  }
};
```

这里我们在前端发起请求创建新数据时，把 `user` 的 `_id` 也传了进去，这样数据库里面对应的商品和制造商就会记录相应的用户 ID 啦。

## 添加账户设置并修改信息

在最后一步中，我们将借助 [Authing SDK](https://docs.authing.cn/authing/sdk/sdk-for-javascript?utm_source=tuture) 实现更细粒度的用户身份管理，以及个人信息设置页面。首先用 npm 安装 Authing 的 JavaScript SDK：

```bash
npm install authing-js-sdk
```

### 修改 Header 的帐户设置链接

首先，让我们修改 Header 头部中的帐户设置链接。打开 `client/src/components/Header.vue`，修改代码如下：

```html client/src/components/Header.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/f25eb7cbe23d905594334ddeede9a1e13d2d8462/client/src/components/Header.vue 查看完整代码
<template>
  <div class="header">
    // ...
    <div class="header-right">
      <el-dropdown v-if="isLogged">
        <el-avatar class="el-dropdown-link" :src="avatar"></el-avatar>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item>
            [tuture-del]
            <a href="https://tuture-first.authing.co/login/profile">账户设置</a>
            [tuture-add]
            <router-link to="/user/settings" tag="div">账户设置</router-link>
          </el-dropdown-item>
          <el-dropdown-item>
            <div @click="handleLogout">退出登录</div>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      // ...
    </div>
  </div>
</template>
// ...

<script>
  [tuture-add]import Authing from "authing-js-sdk";
  [tuture-add]
  export default {
    // ...
    methods: {
  [tuture-del]    handleLogout() {
  [tuture-add]    async handleLogout() {
  [tuture-add]      const userPoolId = "";
  [tuture-add]
  [tuture-add]      const token = JSON.parse(localStorage.getItem("token"));
  [tuture-add]      const userId = JSON.parse(localStorage.getItem("userInfo"))._id;
  [tuture-add]      const authing = new Authing({
  [tuture-add]        userPoolId
  [tuture-add]      });
  [tuture-add]
  [tuture-add]      try {
  [tuture-add]        const res = await authing.checkLoginStatus(token);
  [tuture-add]        console.log("res", res);
  [tuture-add]
  [tuture-add]        await authing.logout(userId);
  [tuture-add]
  [tuture-add]        this.$message({
  [tuture-add]          message: "成功登出",
  [tuture-add]          type: "success"
  [tuture-add]        });
  [tuture-add]      } catch (err) {
  [tuture-add]        console.log("err", err);
  [tuture-add]      }
  [tuture-add]
        localStorage.removeItem("token");
        localStorage.removeItem("userInfo");
        this.$store.commit("LOGOUT");
      }
    }
  };
</script>
```

可以看到，我们主要做了两点变动：

1. 在模板中调整“账户设置”的链接，从原先 Authing 的链接替换成了本应用的 `/user/settings` 路由，这个我们后面马上会实现
2. 在 `handleLogout` 方法中，我们在 `localStorage` 抹去用户信息之前，通过 `authing.checkLoginStatus` 检查登录状态，然后通过 `authing.logout` 执行登出操作

### 实现 Setting 帐户设置页面

打开之前已经创建好的设置页面 `client/src/pages/user/Setting.vue`，实现用户个人信息设置页面，代码如下：

```html client/src/pages/user/Setting.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/f25eb7cbe23d905594334ddeede9a1e13d2d8462/client/src/pages/user/Setting.vue 查看完整代码
<template>
[tuture-del]  <div>settings</div>
[tuture-add]  <div>
[tuture-add]    <app-header></app-header>
[tuture-add]    <div class="user-container">
[tuture-add]      <div class="user-form">
[tuture-add]        <el-upload
[tuture-add]          class="avatar-uploader"
[tuture-add]          action="https://imgkr.com/api/files/upload"
[tuture-add]          :show-file-list="false"
[tuture-add]          :on-success="handleAvatarSuccess"
[tuture-add]        >
[tuture-add]          <img v-if="imageUrl" :src="imageUrl" class="avatar" />
[tuture-add]          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
[tuture-add]        </el-upload>
[tuture-add]
[tuture-add]        <el-form
[tuture-add]          :model="user"
[tuture-add]          :rules="rules"
[tuture-add]          ref="ruleForm"
[tuture-add]          label-width="100px"
[tuture-add]          class="demo-ruleForm"
[tuture-add]        >
[tuture-add]          <el-form-item label="昵称" prop="nickName">
[tuture-add]            <el-input v-model="user.nickname"></el-input>
[tuture-add]          </el-form-item>
[tuture-add]          <el-form-item>
[tuture-add]            <el-button type="primary" @click="submitForm('ruleForm')"
[tuture-add]              >更新</el-button
[tuture-add]            >
[tuture-add]          </el-form-item>
[tuture-add]        </el-form>
[tuture-add]      </div>
[tuture-add]    </div>
[tuture-add]  </div>
</template>

[tuture-add]<style>
[tuture-add].avatar-uploader .el-upload {
[tuture-add]  border: 1px dashed #d9d9d9;
[tuture-add]  border-radius: 6px;
[tuture-add]  cursor: pointer;
[tuture-add]  position: relative;
[tuture-add]  overflow: hidden;
[tuture-add]}
[tuture-add].avatar-uploader .el-upload:hover {
[tuture-add]  border-color: #409eff;
[tuture-add]}
[tuture-add].avatar-uploader-icon {
[tuture-add]  font-size: 28px;
[tuture-add]  color: #8c939d;
[tuture-add]  width: 178px;
[tuture-add]  height: 178px;
[tuture-add]  line-height: 178px;
[tuture-add]  text-align: center;
[tuture-add]}
[tuture-add].avatar {
[tuture-add]  width: 178px;
[tuture-add]  height: 178px;
[tuture-add]  display: block;
[tuture-add]}
[tuture-add]
[tuture-add].user-form {
[tuture-add]  width: 500px;
[tuture-add]}
[tuture-add]
[tuture-add].user-container {
[tuture-add]  display: flex;
[tuture-add]  flex-direction: row;
[tuture-add]  justify-content: center;
[tuture-add]}
[tuture-add]</style>
[tuture-add]
<script>
[tuture-add]import Header from "@/components/Header.vue";
[tuture-add]import Authing from "authing-js-sdk";
[tuture-add]
export default {
  data() {
    return {
[tuture-del]      model: { manufacturer: { name: "", _id: "" } }
[tuture-add]      user: {},
[tuture-add]      imageUrl: "",
[tuture-add]      rules: {
[tuture-add]        nickname: [
[tuture-add]          { required: true, message: "请输入你的昵称", trigger: "blur" },
[tuture-add]          { min: 3, max: 25, message: "长度在 3 到 25 个字符", trigger: "blur" }
[tuture-add]        ]
[tuture-add]      }
    };
  },
[tuture-del]  mounted() {}
[tuture-add]  created: function() {
[tuture-add]    const user = this.$store.state.user;
[tuture-add]    const userInfo = localStorage.getItem("userInfo");
[tuture-add]
[tuture-add]    if (user && Object.keys(user).length === 0 && userInfo) {
[tuture-add]      this.user = JSON.parse(userInfo);
[tuture-add]      this.imageUrl = this.user.photo;
[tuture-add]    } else {
[tuture-add]      this.user = { ...user };
[tuture-add]      this.imageUrl = user.photo;
[tuture-add]    }
[tuture-add]  },
[tuture-add]  components: {
[tuture-add]    "app-header": Header
[tuture-add]  },
[tuture-add]  methods: {
[tuture-add]    async handleAvatarSuccess(res, file) {
[tuture-add]      if (res.code === 200) {
[tuture-add]        this.imageUrl = res.data;
[tuture-add]      } else {
[tuture-add]        this.$message.error("图片上传失败");
[tuture-add]      }
[tuture-add]    },
[tuture-add]    async submitForm(formName) {
[tuture-add]      const nickname = this.user.nickname;
[tuture-add]      const photo = this.imageUrl;
[tuture-add]      const userId = this.user._id;
[tuture-add]      const user = this.user;
[tuture-add]      const that = this;
[tuture-add]
[tuture-add]      this.$refs[formName].validate(async valid => {
[tuture-add]        if (valid) {
[tuture-add]          const token = localStorage.getItem("token");
[tuture-add]          const userPoolId = "";
[tuture-add]
[tuture-add]          const authing = new Authing({
[tuture-add]            userPoolId
[tuture-add]          });
[tuture-add]
[tuture-add]          const login = await authing.login({
[tuture-add]            email: "",
[tuture-add]            password: ""
[tuture-add]          });
[tuture-add]
[tuture-add]          console.log("nickName", nickname);
[tuture-add]          try {
[tuture-add]            await authing.update({
[tuture-add]              _id: login._id,
[tuture-add]              photo,
[tuture-add]              nickname
[tuture-add]            });
[tuture-add]
[tuture-add]            const newUser = { ...user, nickname, photo };
[tuture-add]            localStorage.setItem("userInfo", JSON.stringify(newUser));
[tuture-add]            that.$store.commit("SET_USER", newUser);
[tuture-add]
[tuture-add]            this.$message({
[tuture-add]              message: "修改信息成功",
[tuture-add]              type: "success"
[tuture-add]            });
[tuture-add]          } catch (err) {
[tuture-add]            console.log("err", err);
[tuture-add]
[tuture-add]            this.$message.error("修改信息失败");
[tuture-add]          }
[tuture-add]        } else {
[tuture-add]          console.log("error submit!!");
[tuture-add]          return false;
[tuture-add]        }
[tuture-add]      });
[tuture-add]    }
[tuture-add]  }
};
</script>
```

我们主要看一下 `script` 和 `template` 部分。首先在 `script` 部分中，我们的组件包括：

- `data` 字段定义了模板中所需要的数据，包括 `user` 、`imageUrl`（头像链接）以及 `rules`（表单校验规则）
- `created` 生命周期方法用于从 Vuex Store 以及 `localStorage` 中获取用户数据（`localStorage` 的优先级更高一些），然后初始化上面的 `data` 字段
- `components` 用于指定 `app-header` 组件为我们刚才修改好的 `Header` 组件
- `methods` 中定义了 `handleAvatarSuccess` 和 `submitForm` 两个 Handler，分别用于处理头像上传成功以及提交表单的逻辑。在 `submitForm` 方法中，我们先从表单中获取到相应的数据，然后通过 `authing.update` 更新用户数据，成功后再修改 Vuex Store 中的状态

### 调整 App 根组件

让我们调整一下 App 根组件。打开 `client/src/App.vue` ，修改代码如下：

```html client/src/App.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/f25eb7cbe23d905594334ddeede9a1e13d2d8462/client/src/App.vue 查看完整代码
// ...

<script>
  [tuture-add]import Authing from "authing-js-sdk";
  [tuture-add]
  export default {
    name: "App",
    mounted() {
  [tuture-del]    const userInfo = localStorage.getItem("userInfo");
  [tuture-add]    this.checkLogin();
  [tuture-add]  },
  [tuture-add]  methods: {
  [tuture-add]    async checkLogin() {
  [tuture-add]      const token = localStorage.getItem("token");
  [tuture-add]
  [tuture-add]      if (token) {
  [tuture-add]        const userPoolId = "";
  [tuture-add]
  [tuture-add]        const authing = new Authing({
  [tuture-add]          userPoolId
  [tuture-add]        });
  [tuture-add]
  [tuture-add]        const result = await authing.checkLoginStatus(JSON.parse(token));
  [tuture-add]
  [tuture-add]        if (result.status) {
  [tuture-add]          const userInfo = localStorage.getItem("userInfo");

  [tuture-del]    if (userInfo) {
  [tuture-del]      this.$store.commit("SET_USER", JSON.parse(userInfo));
  [tuture-add]          if (userInfo) {
  [tuture-add]            this.$store.commit("SET_USER", JSON.parse(userInfo));
  [tuture-add]          }
  [tuture-add]        } else {
  [tuture-add]          localStorage.removeItem("token");
  [tuture-add]          localStorage.removeItem("userInfo");
  [tuture-add]        }
  [tuture-add]      }
      }
    }
  };
</script>

// ...
```

可以看到，我们主要实现了一个 `checkLogin` 方法，用于在整个应用刚挂载时检查登录状态，如果登录成功，则从 `storage` 里面取出数据并设置进 Redux Store ，如果登录失效，则清空本地的 `storage` 信息。

### 调整其他页面

最后我们调整一下其他页面的一些细节。修改 `client/src/pages/user/Index.vue`，代码如下：

```html client/src/pages/user/Index.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/f25eb7cbe23d905594334ddeede9a1e13d2d8462/client/src/pages/user/Index.vue 查看完整代码
<template>
  <div>
    <div class="container">
      [tuture-del]
      <div class="container">
        [tuture-del]
        <h1 class="user-title">
          [tuture-del]
          <router-link to="/" tag="div">登录/注册</router-link> [tuture-del]
        </h1>
        [tuture-del] <router-view /> [tuture-del]
      </div>
      [tuture-add] <router-view />
    </div>
  </div>
</template>

[tuture-del]<style>
  [tuture-del].user-title:hover {
  [tuture-del]  cursor: pointer;
  [tuture-del]}
  [tuture-del]
  [tuture-del].container {
  [tuture-del]  margin-top: 40px;
  [tuture-del]}
  [tuture-del]
</style>
[tuture-add]<style></style>
```

继续修改 `client/src/pages/user/Login.vue`，代码如下：

```html client/src/pages/user/Login.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/f25eb7cbe23d905594334ddeede9a1e13d2d8462/client/src/pages/user/Login.vue 查看完整代码
<template>
  [tuture-del]
  <div id="login-form"></div>
  [tuture-add]
  <div>
    [tuture-add]
    <h1 class="user-title">
      [tuture-add]
      <router-link to="/" tag="div">用户界面</router-link> [tuture-add]
    </h1>
    [tuture-add]
    <div id="login-form"></div>
    [tuture-add]
  </div>
</template>

[tuture-add]<style>
  [tuture-add].user-title:hover {
  [tuture-add]  cursor: pointer;
  [tuture-add]}
  [tuture-add]
</style>
[tuture-add] // ...
```

当保存上面的修改的代码，我们可以看到如下的效果：

![](https://static.powerformer.com/c/092faf4/172281d34c19bd30.gif)

## 集成微信、QQ 登录或 Github 登录

通过上述流程，我们就完成了一个完整的用户系统及其与现有系统的整合，但是有同学发现了，我们在平时生活或工作中，除了常规的手机号+验证码、邮箱密码等，还会有一些更方便的登录方式，如微信登录、QQ 登录等，那么我们如何集成这些方便的登录呢？实际上可能看起来很复杂，但是在我们现有的基础上，用 Authing 可以很方便的集成微信、QQ 登录等。

{% note warning %}
**注意**

只有企业才能集成微信或 QQ 登录，如果你是个人开发者，那么这一节你可以跳过哦 🤓
{% endnote %}

### 集成微信扫码登录

首先去微信官方文档完成注册，然后申请一个[微信网页应用](https://open.weixin.qq.com/cgi-bin/frame?t=home/web_tmpl&lang=zh_CN)，然后获取到微信网页应用的 AppID 和 AppSecret：

![](https://static.powerformer.com/c/092faf4/172281d3566d0e0d.png)

接着滑动到底部，将授权回调域改为 `oauth.authing.cn`

![](https://static.powerformer.com/c/092faf4/172281d360ce9fa8.png)

然后我们开始去 Authing 控制台，在相应微信登录里面，填入刚刚获取的 `AppID` 和 `AppSecret` ：

![](https://static.powerformer.com/c/092faf4/172281d366bb0ba1.png)

![](https://static.powerformer.com/c/092faf4/172281d36d70102e.png)

注意到上面我们第三个参数 “重定向地址” 填写了我们现在 Vue 全栈电商应用的开发服务器地址，读者应该根据自己当前的需要地址进行对应的填写。

大功告成，通过上面的步骤我们就配置好了微信网页登录，现在你应该可以看到如下的效果：

![](https://static.powerformer.com/c/092faf4/172281d37286a52f.gif)

我的天！好神奇！就是上面几下手工点按配置，我们就集成好了微信登录！😅

### 集成 QQ 登录

按照和接入微信网页登录类似的方式，我们前往[ QQ 互联中心](https://connect.qq.com/manage.html#/)，注册一个账号，并创建一个网页应用。

然后进入网页应用，将授权回调地址填写为：`https://oauth.authing.cn/oauth/qq/redirect ` ，接着回到 Authing 控制台，我们配置 QQ 登录：

![](https://static.powerformer.com/c/092faf4/172281d37ca48ed6.png)

![](https://static.powerformer.com/c/092faf4/172281d3865db313.png)

保存之后，大功告成！我们的应用里面就有了 QQ 登录，和微信登录同样简单！

### 集成 Github 登录

最后我们再来尝试集成一下开发者比较喜爱的 Github 登录，看看 Authing 是如何简化这一劳动的呢？

首先根据 Github 指引，创建一个 [OAuth 应用](https://developer.github.com/apps/building-oauth-apps/creating-an-oauth-app/)。

![](https://static.powerformer.com/c/092faf4/172281d38eaec73b.png)

然后填入如下内容：

![](https://static.powerformer.com/c/092faf4/172281d3990fc36f.png)

其中红框框出来的内容需要填入 Authing 相关的 `https://oauth.authing.cn/oauth/github/redirect`，然后创建好应用之后，取出 `Client ID` 和 `Client Secret` ：

![](https://static.powerformer.com/c/092faf4/172281d3a81ef29d.png)

之后就是类似之前的操作，进入 Authing 控制台，配置 Github 相关的内容：

![](https://static.powerformer.com/c/092faf4/172281d3a8503117.png)

最后可以看到如下效果：

![](https://static.powerformer.com/c/092faf4/172281d3b53de278.gif)

## 小结

至此，本篇教程也就结束了，相信你已经感受到了 Authing 身份认证机制的强大与便捷。在当今 Serverless 时代，越来越多标准化的流程（例如身份验证、人工智能应用等等）正在逐渐迈向云端，成为一种可直接消费的资源，而我们作为应用终端的开发者则可以将更多的时间和精力放在打磨和完善自身的产品上，在一定程度上解放了生产力。

图雀社区秉承“加速技术传播”的理念，致力于推广能够真正让开发者和用户的生活变得更美好的技术。
