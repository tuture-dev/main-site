---
title: '从零到部署：用 Vue 和 Express 实现迷你全栈电商应用（七）'
description: '在之前的六篇教程中我们已经基本实现了迷你全栈电商应用，相信大家对于一个全栈应用的开发已经有了一个全面的认知。但是一个追求完美的工程师是不会吝啬他的艺术创造，仅仅实现应用的功能还不能满足用户的高需求，应用的界面效果也是提高用户体验的关键因素。因此本篇教程将基于element-ui组件库重构项目的前端代码，改善迷你电商应用的界面效果，提高用户的体验感。虽然我们可以轻松地引入现成的组件库，但是与之对应的数据处理也值得我们注意，那我会在引入组件库的同时带大家一起踩一踩element-ui给我们挖的坑，毕竟踩坑才能成长嘛。'
tags: ['Vue', 'Element UI']
categories: ['前端', 'Vue', '入门']
date: 2020-03-13T02:00:37.658Z
photos:
  - https://static.powerformer.com/c/092faf4/vue-cover-7-squashed.jpg
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

## 代码重构

这一部分我们主要利用`element-ui`组件库重构之前的项目代码，实现极具美感的迷你电商应用。

这里我们简单介绍一下`element-ui`组件库(如果您了解，您可以跳过这部分)：

> Element UI 是一套采用 Vue 2.0 作为基础框架实现的组件库，一套为开发者、设计师和产品经理准备的基于 Vue 2.0 的组件库，提供了配套设计资源，帮助网站快速成型。

Element UI 文档提供了很多实例代码，一般情况下我们直接拷下示例代码稍微看看改改数据之类的就 OK 了。但是在某些场景下，我们可能又需要使用到一些特殊的功能和属性，而这些功能属性一般在官方提供的组件中都已经内置了，所以我们可以直接先从文档中寻找查看是否有属性或者方法等能够满足我们的需求，从而避免重复造轮子。

### 安装 element-ui 依赖

- npm 安装推荐使用 npm 的方式安装，它能更好地和 webpack 打包工具配合使用。

```
npm i element-ui -S
```

- CDN 引入目前可以通过 [unpkg.com/element-ui](https://unpkg.com/browse/element-ui@2.13.0/) 获取到最新版本的资源，在页面上引入 js 和 css 文件即可开始使用。

```html
<!-- 引入样式 -->
<link
  rel="stylesheet"
  href="https://unpkg.com/element-ui/lib/theme-chalk/index.css"
/>
<!-- 引入组件库 -->
<script src="https://unpkg.com/element-ui/lib/index.js"></script>
```

> 我们建议使用 CDN 引入 Element 的用户在链接地址上锁定版本，以免将来 Element 升级时受到非兼容性更新的影响。锁定版本的方法请查看 [unpkg.com](https://unpkg.com/)。

### 导入依赖

依赖安装完成之后，我们需要在项目的`main.js`文件中导入并注册依赖。

> 你可以引入整个 Element，或是根据需要仅引入部分组件，这里我们引入了完整的 Element。

```js src/main.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/main.js 查看完整代码
// ...
import App from './App';
import router from './router';
import store from './store';
[tuture-add]import ElementUI from 'element-ui'
[tuture-add]import 'element-ui/lib/theme-chalk/index.css'

Vue.config.productionTip = false;
Vue.component('ValidationProvider', ValidationProvider);
[tuture-add]Vue.use(ElementUI);

// ...
```

在`main.js`文件中我们首先导入了`element-ui`组件库，需要注意的是我们要单独引入样式文件；除此之外还要使用`Vue.use()`注册组件库。

至此，一个基于 Vue 和 Element 的开发环境已经搭建完毕，现在就可以愉快地使用组件库进行代码重构了。

### 重构导航栏

我们首先来到`App`组件，这里之前是采用普通的`nav`标签展示首页导航，显得甚是简陋，现在我们可以使用`element-ui`组件库提供的`el-menu`导航菜单组件重构导航栏，绝对酷炫。

```html src/App.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/App.vue 查看完整代码
<template>
  <div id="app">
    [tuture-del]
    <nav>
      [tuture-del]
      <div class="container">
        [tuture-del]
        <ul class="nav__left">
          [tuture-del]
          <li>
            [tuture-del] <router-link to="/">Home</router-link> [tuture-del]
          </li>
          [tuture-del]
          <li>
            [tuture-del]
            <router-link to="/admin">Admin</router-link> [tuture-del]
          </li>
          [tuture-del]
          <li>
            [tuture-del] <router-link to="/cart">Cart</router-link> [tuture-del]
          </li>
          [tuture-del]
        </ul>
        [tuture-del]
      </div>
      [tuture-del]
    </nav>
    [tuture-del] [tuture-add]
    <el-menu
      [tuture-add]
      class="menu"
      [tuture-add]
      :default-active="activeIndex2"
      [tuture-add]
      mode="horizontal"
      [tuture-add]
      @select="handleSelect"
      [tuture-add]
      background-color="#545c64"
      [tuture-add]
      text-color="#fff"
      [tuture-add]
      active-text-color="#ffd04b"
    >
      [tuture-add]
      <el-menu-item index="1"
        ><router-link to="/" tag="div">Home</router-link></el-menu-item
      >
      [tuture-add]
      <el-submenu index="2">
        [tuture-add] <template slot="title">Admin</template> [tuture-add]
        <el-menu-item index="2-1"
          ><router-link to="/admin" tag="div"
            >查看商品</router-link
          ></el-menu-item
        >
        [tuture-add]
        <el-menu-item index="2-2"
          ><router-link to="/admin/new" tag="div"
            >添加商品</router-link
          ></el-menu-item
        >
        [tuture-add]
        <el-menu-item index="2-3"
          ><router-link to="/admin/manufacturers" tag="div"
            >查看生产商</router-link
          ></el-menu-item
        >
        [tuture-add]
        <el-menu-item index="2-4"
          ><router-link to="/admin/manufacturers/new" tag="div"
            >添加生产商</router-link
          ></el-menu-item
        >
        [tuture-add]
      </el-submenu>
      [tuture-add]
      <el-menu-item index="3"
        ><router-link to="/cart" tag="div">Cart</router-link></el-menu-item
      >
      [tuture-add]
    </el-menu>
    <router-view />
  </div>
</template>
// ...
<script>
  export default {
    name: 'App',
  [tuture-add]  data() {
  [tuture-add]    return {
  [tuture-add]      activeIndex: '1',
  [tuture-add]      activeIndex2: '1'
  [tuture-add]    };
  [tuture-add]  },
  [tuture-add]  methods: {
  [tuture-add]    handleSelect(key, keyPath) {
  [tuture-add]        console.log(key, keyPath);
  [tuture-add]    }
  [tuture-add]  }
  [tuture-add]
  };
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
  [tuture-add].el-submenu__title {
  [tuture-add]  margin-left: 40px;
  [tuture-add]}
</style>
```

这里导航栏组件的使用相信大家都能看懂，这里我们只讲一下比较特殊的地方。我们不需要在意 `data` 属性以及 `handleSelect `方法，我们暂时用不到。这里一个特殊的地方就是 `el-menu-item `标签中的 `tag` 属性，我们将其值设置为 "div" 表示将该标签渲染为 "div" 盒子，如果不设置该属性则该标签默认渲染为 "a" 标签，导致标签包裹的内容带有下划线，因此这里 `tag` 属性的设置是为了去除下划线。

### 重构商品列表

重新修改 `ProductList` 组件，由于该组件中的子组件 `ProductItem` 进行了重构，因此这里也需要做一定的修改，看到后面 `ProductItem` 组件的重构您就会明白我们这里修改的用意。

```html src/components/products/ProductList.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/components/products/ProductList.vue 查看完整代码
<template>
  <div>
    <div class="products">
      <div class="container">This is ProductList</div>
      [tuture-del]
      <template v-for="product in products">
        [tuture-del]
        <product-item :product="product" :key="product._id"></product-item>
        [tuture-del]
      </template>
      [tuture-add]
      <!-- <template v-for="product in products"> -->
      [tuture-add]
      <product-item :products="products"></product-item> [tuture-add]
      <!-- </template> -->
    </div>
  </div>
</template>
// ...
```

这里之前是将从本地获取的 `products` 数组利用 `v-for` 将 `product` 对象遍历到每个 `ProductItem` 组件中分别进行展示，但是我们这里取消了 `v-for` 遍历 `products` 数组，选择直接将 `products` 数组传入 `ProductItem` 组件中。请允许我先在这里卖个关子，继续往下看。

重新进入 `ProductItem` 组件进行修改，这里我们使用了 `element-ui` 组件库提供的 `el-table` 表格组件取代了原始标签来展示商品信息列表。

```html src/components/products/ProductItem.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/components/products/ProductItem.vue 查看完整代码
<template>
[tuture-del]  <div>
[tuture-del]    <div class="product">
[tuture-del]      <router-link :to="'/detail/' + product._id" class="product-link">
[tuture-del]        <p class="product__name">产品名称：{{product.name}}</p>
[tuture-del]        <p class="product__description">介绍：{{product.description}}</p>
[tuture-del]        <p class="product__price">价格：{{product.price}}</p>
[tuture-del]        <p class="product.manufacturer">生产厂商：{{product.manufacturer.name}}</p>
[tuture-del]        <img :src="product.image" alt="" class="product__image">
[tuture-del]      </router-link>
[tuture-del]      <product-button :product="product"></product-button>
[tuture-del]    </div>
[tuture-add]  <div class="products">
[tuture-add]    <el-table
[tuture-add]    class="table"
[tuture-add]    :data="products"
[tuture-add]    max-height="250">
[tuture-add]      <el-table-column
[tuture-add]        prop="name"
[tuture-add]        label="产品名称"
[tuture-add]        width="180">
[tuture-add]      </el-table-column>
[tuture-add]      <el-table-column
[tuture-add]        prop="description"
[tuture-add]        label="介绍"
[tuture-add]        width="180">
[tuture-add]      </el-table-column>
[tuture-add]      <el-table-column
[tuture-add]        prop="price"
[tuture-add]        label="价格"
[tuture-add]        width="180">
[tuture-add]      </el-table-column>
[tuture-add]      <el-table-column
[tuture-add]        prop="manufacturer.name"
[tuture-add]        label="生产厂商"
[tuture-add]        width="180">
[tuture-add]      </el-table-column>
[tuture-add]      <!-- <el-table-column
[tuture-add]        label="图片"
[tuture-add]        width="200">
[tuture-add]        <img :src="image" alt="" class="product__image">
[tuture-add]      </el-table-column> -->
[tuture-add]      <el-table-column
[tuture-add]        label="操作"
[tuture-add]        width="180">
[tuture-add]        <template slot-scope="scope">
[tuture-add]          <product-button :id="scope.row._id"></product-button>
[tuture-add]        </template>
[tuture-add]      </el-table-column>
[tuture-add]    </el-table>
  </div>
[tuture-add]  <!-- <div class="product">
[tuture-add]    <router-link :to="'/detail/' + product._id" class="product-link">
[tuture-add]      <p class="product__name">产品名称：{{product.name}}</p>
[tuture-add]      <p class="product__description">介绍：{{product.description}}</p>
[tuture-add]      <p class="product__price">价格：{{product.price}}</p>
[tuture-add]      <p class="product.manufacturer">生产厂商：{{product.manufacturer.name}}</p>
[tuture-add]      <img :src="product.image" alt="" class="product__image">
[tuture-add]    </router-link>
[tuture-add]    <product-button :product="product"></product-button>
[tuture-add]  </div> -->
</template>

<style>
[tuture-del].product {
[tuture-del]  border-bottom: 1px solid black;
[tuture-add].products {
[tuture-add]  padding-top: 10px;
[tuture-add]  text-align: center;
}
[tuture-del]
[tuture-del].product__image {
[tuture-del]  width: 100px;
[tuture-del]  height: 100px;
[tuture-add].table {
[tuture-add]  margin:0 auto;
[tuture-add]  width: 900px;
[tuture-add]}
[tuture-add].el-table .cell {
[tuture-add]  text-align: center;
[tuture-add]  padding-top: 10px;
[tuture-add]  padding-bottom: 10px;
}
[tuture-add]/* .product__image {
[tuture-add]  width: 50%;
[tuture-add]} */
</style>

<script>
import ProductButton from './ProductButton';
export default {
  name: 'product-item',
[tuture-del]  props: ['product'],
[tuture-add]  props: ['products'],
  components: {
    'product-button': ProductButton,
  }
}
</script>
```

其实这里的修改相信大家都能看懂，我们就简单的做一下介绍。您可能还记得我们在上面卖的一个关子，为什么我们直接向该组件中传入了 `products` 数组而不是遍历的 `product` 对象？相信大家看了该组件的重构也能豁然开朗，那就是因为我们使用的 `el-table` 表格组件需要传入一个数组作为 `data` 属性，并将每个元素对象作为 `prop` 传入表格，按照对应的列名展示出来。

除此之外，相信大家也发现了最后一个 `el-table-column` 标签中并没有定义 `prop` 属性，这是因为最后一列单元格中放置的是按钮而不是商品信息，该按钮是用于对指定行对象进行指定操作，这里我们使用 `scope.row` 获取指定行对象并将其 id 传递给了 `ProductButton` 按钮组件。

{% note info %}
**提示**

通过 slot-scope 可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据。
{% endnote %}

再次进入 `ProductButton` 组件进行修改，这里我们使用了 `element-ui` 组件库提供的 `el-button` 按钮组件替代之前普通的 `button` 标签并修改了对应的数据处理。

```html src/components/products/ProductButton.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/components/products/ProductButton.vue 查看完整代码
<template>
  <div>
    [tuture-del]
    <button v-if="isAdding" class="button" @click="addToCart">
      加入购物车
    </button>
    [tuture-del]
    <button v-else class="button" @click="removeFromCart(product._id)">
      从购物车移除
    </button>
    [tuture-add]
    <el-button
      [tuture-add]
      v-if="isAdding"
      [tuture-add]
      @click="addToCart"
      [tuture-add]
      type="text"
      [tuture-add]
      size="small"
    >
      [tuture-add] 加入购物车 [tuture-add]
    </el-button>
    [tuture-add]
    <el-button
      [tuture-add]
      v-else
      [tuture-add]
      @click="removeFromCart(id)"
      [tuture-add]
      type="text"
      [tuture-add]
      size="small"
    >
      [tuture-add] 从购物车移除 [tuture-add]
    </el-button>
  </div>
</template>

<script>
  export default {
  [tuture-del]  props: ['product'],
  [tuture-add]  props: ['id'],
    computed: {
  [tuture-add]    product() {
  [tuture-add]      let product = this.$store.getters.allProducts.find(product => product._id === this.id)
  [tuture-add]      return product;
  [tuture-add]    },
      // ...
    },
    methods: {
      // ...
    }
  }
</script>
```

这里我们首先简单地使用了 `el-button` 按钮组件，然后将从父组件获取的 `product` 对象修改为了 `id`，因为我们在 `ProductItem` 组件中传入的是指定对象的 `id`，因此我们在按钮组件中定义了计算属性 `product`，从本地获取指定 `id` 的 `product` 对象。

我们已经迫不及待把项目跑起来了，看看我们的首页导航以及商品信息列表发生了怎样不可思议的改变：

![](https://static.powerformer.com/c/092faf4/170bd7f1d39bb647.png)

### 重构商品信息功能

这部分内容主要是有关商品信息功能的重构，包括商品信息列表的展示、修改指定商品信息以及添加新商品，我们都使用了 `element-ui` 组件库提供的组件进行重构，提高用户操作商品信息时的交互体验。

首先我们进入 `Products` 组件，同样使用了 `element-ui` 组件库提供的 `el-table` 组件替换了之前普通表格来展示商品信息列表。

```html src/pages/admin/Products.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/pages/admin/Products.vue 查看完整代码
<template>
[tuture-del]  <div>
[tuture-del]    <table class="table">
[tuture-add]  <div class="products">
[tuture-add]    <el-table
[tuture-add]    class="table"
[tuture-add]    :data="products">
[tuture-add]      <el-table-column
[tuture-add]        prop="name"
[tuture-add]        label="名称"
[tuture-add]        width="180">
[tuture-add]      </el-table-column>
[tuture-add]      <el-table-column
[tuture-add]        prop="price"
[tuture-add]        label="价格"
[tuture-add]        width="180">
[tuture-add]      </el-table-column>
[tuture-add]      <el-table-column
[tuture-add]        prop="manufacturer.name"
[tuture-add]        label="制造商"
[tuture-add]        width="180">
[tuture-add]      </el-table-column>
[tuture-add]      <el-table-column
[tuture-add]        label="操作"
[tuture-add]        width="200">
[tuture-add]        <template slot-scope="scope">
[tuture-add]          <el-button class="modify" type="text" size="small"><router-link :to="'/admin/edit/' + scope.row._id">修改</router-link></el-button>
[tuture-add]          <el-button class="remove" @click="removeProduct(scope.row._id), deleteRow(scope.$index, products)" type="text" size="small">删除</el-button>
[tuture-add]        </template>
[tuture-add]      </el-table-column>
[tuture-add]    </el-table>
[tuture-add]    <!-- <table class="table">
      <thead>
        <tr>
          <th>名称</th>
          // ...
          <td class="remove"><a @click="removeProduct(product._id)" href="#">删除</a></td>
        </tr>
      </tbody>
[tuture-del]    </table>
[tuture-add]    </table> -->
  </div>
</template>

<style>
[tuture-del]table {
[tuture-del]  margin: 0 auto;
[tuture-add].products {
[tuture-add]  padding-top: 10px;
[tuture-add]  text-align: center;
[tuture-add]}
[tuture-add].table {
[tuture-add]  margin:0 auto;
[tuture-add]  width: 740px;
[tuture-add]}
[tuture-add].el-table .cell {
[tuture-add]  text-align: center;
[tuture-add]  padding-top: 10px;
[tuture-add]  padding-bottom: 10px;
}
[tuture-del]
.modify {
  color: blue;
}

[tuture-del].remove a {
[tuture-add].remove {
  color: red;
}
</style>
 // ...
```

细心的大家肯定已经发现了这里的表格有点似曾相识，没错，这里的表格与 `ProductItem` 组件中的表格非常相似，都是用来展示本地商品信息，但是两者的区别是对商品对象的操作，`ProductItem` 组件中的按钮组件是用于将商品添加或移出购物车，而该组件中的按钮组件是用于修改或删除商品对象。

这是我们重构之后的商品信息列表：

![](https://static.powerformer.com/c/092faf4/170bd7fc2f9b7080.png)

然后我们先对修改功能进行重构，再次进入 `Edit` 组件，我们在这里做了数据处理修改，目的是尝试解决商品信息表单无法编辑问题。

```html src/pages/admin/Edit.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/pages/admin/Edit.vue 查看完整代码
// ...
<script>
import ProductForm from '@/components/products/ProductForm.vue';
export default {
[tuture-add]  data: {
[tuture-add]    model() {
[tuture-add]      const product = this.$store.getters.productById(this.$route.params['id']);
[tuture-add]      // 这里返回 product 的拷贝，是为了在修改 product 的拷贝之后，在保存之前不修改本地 Vuex stire 的 product 属性
[tuture-add]      return { ...product, manufacturer: { ...product.manufacturer } };
[tuture-add]    }
[tuture-add]  },
  created() {
    const { name } = this.model;
    if (!name) {
      // ...
  computed: {
    manufacturers() {
      return this.$store.getters.allManufacturers;
[tuture-del]    },
[tuture-del]    model() {
[tuture-del]      const product = this.$store.getters.productById(this.$route.params['id']);
[tuture-del]
[tuture-del]      // 这里返回 product 的拷贝，是为了在修改 product 的拷贝之后，在保存之前不修改本地 Vuex stire 的 product 属性
[tuture-del]      return { ...product, manufacturer: { ...product.manufacturer } };
    }
  },
  methods: {
    // ...
```

这里我们把定义的计算属性 `model` 修改为 `data` 属性，因为我们发现如果商品对象 `model` 作为计算属性传给子组件 `ProductForm` 进行信息展示时，无法进行表单编辑，大家可以运行起来尝试一下是否可以进行编辑。我们初始猜想是 `el-form` 表单组件中的表单数据对象 `model` 不能来自计算属性，否则无法进行编辑，因此我们首度尝试将该组件中的计算属性 `model` 放到 `data` 属性中。

再次进入 `ProductForm` 组件进行重构，这里我们使用了 `element-ui` 组件库提供的 `el-form` 表单组件替换之前的普通表单展示商品信息。

```html src/components/products/ProductForm.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/components/products/ProductForm.vue 查看完整代码
<template>
  [tuture-del]
  <form @submit.prevent="saveProduct">
    [tuture-add]
    <div class="productInfo">
      [tuture-add]
      <el-form class="form" ref="form" :model="model" label-width="180px">
        [tuture-add]
        <el-form-item label="Name">
          [tuture-add] <el-input v-model="model.name"></el-input> [tuture-add]
        </el-form-item>
        [tuture-add]
        <el-form-item label="Price">
          [tuture-add] <el-input v-model="model.price"></el-input> [tuture-add]
        </el-form-item>
        [tuture-add]
        <el-form-item label="Manufacturer ">
          [tuture-add]
          <el-select
            v-model="model.manufacturer.name"
            clearable
            placeholder="请选择制造商"
          >
            [tuture-add]
            <el-option
              [tuture-add]
              v-for="manufacturer in manufacturers"
              [tuture-add]
              :key="manufacturer._id"
              [tuture-add]
              :label="manufacturer.name"
              [tuture-add]
              :value="manufacturer.name"
            >
              [tuture-add]
            </el-option>
            [tuture-add]
          </el-select>
          [tuture-add]
        </el-form-item>
        [tuture-add]
        <el-form-item label="Image ">
          [tuture-add] <el-input v-model="model.image"></el-input> [tuture-add]
        </el-form-item>
        [tuture-add]
        <el-form-item label="Description ">
          [tuture-add]
          <el-input type="textarea" v-model="model.description"></el-input>
          [tuture-add]
        </el-form-item>
        [tuture-add]
        <el-form-item>
          [tuture-add]
          <el-button v-if="isEditing" type="primary" @click="onSubmit"
            >Update Product</el-button
          >
          [tuture-add]
          <el-button v-else @click="onSubmit">Add Product</el-button>
          [tuture-add]
        </el-form-item>
        [tuture-add]
      </el-form>
      [tuture-add]
    </div>
    // ... [tuture-del]
  </form>
  // ...
</template>

<script>
  export default {
    props: ['model', 'manufacturers', 'isEditing'],
  [tuture-add]  created() {
  [tuture-add]    console.log(this.model)
  [tuture-add]  },
    methods: {
  [tuture-del]    saveProduct() {
  [tuture-add]    onSubmit() {
        this.$emit('save-product', this.model)
      }
    }
  }
</script>
[tuture-add]<style>
  [tuture-add].productInfo {
  [tuture-add]  padding-top: 10px;
  [tuture-add]}
  [tuture-add].form {
  [tuture-add]  margin: 0 auto;
  [tuture-add]  width: 500px;
  [tuture-add]}
  [tuture-add].el-input__inner {
  [tuture-add]  height: 60px;
  [tuture-add]}
  [tuture-add]
</style>
```

相信大家也能轻松的看懂 `el-form` 表单组件的使用，这里的 `model` 属性表示表单数据对象，我们可以使用 `v-model` 将表单数据对象中的信息双向绑定到相应的表单内组件上。特别提醒一下商品对象 `model` 中的 `manufacturer` 是一个制造商对象，包含制造商 `id` 和 `name` 属性。

现在我们再进入 `New` 组件进行重构，当我们发现 `Edit` 组件中的问题之后，我们同样尝试将该组件中的计算属性 `model` 定义到 `data` 属性中。

```html src/pages/admin/New.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/pages/admin/New.vue 查看完整代码
// ...
<script>
  import ProductForm from '@/components/products/ProductForm.vue';
  export default {
  [tuture-add]  data() {
  [tuture-add]    return {
  [tuture-add]      model: {manufacturer:{name: ''}}
  [tuture-add]    }
  [tuture-add]  },
    // ...
    computed: {
      manufacturers() {
        return this.$store.getters.allManufacturers;
  [tuture-del]    },
  [tuture-del]    model() {
  [tuture-del]      return {};
      }
    },
    // ...
  }
</script>
```

因为该组件是新建商品组件，因此我们定义的是一个空对象 `model`，但是我们需要给其一个默认初始形式 `{% raw %}model: {manufacturer: {name: ' '}}{% endraw %}`，防止在子组件表单中无法访问 `name` 属性导致报错。

现在我们添加或者修改商品信息的表单界面变成了这样：

![](https://static.powerformer.com/c/092faf4/170bd807ce1da156.png)

### 重构制造商信息功能

制造商信息功能包括制造商信息展示，添加制造商以及修改制造商信息，同重构商品信息功能一样，我们也使用了 `element-ui` 组件库提供的组件进行重构，提高用户操作制造商信息时的交互体验。

首先我们进入 `Manufacturers` 组件进行重构，同 `Products` 组件类似，我们使用了 `element-ui` 组件库提供的 `el-table` 表格组件替换了之前普通的表格展示制造商信息列表。

```html src/pages/admin/Manufacturers.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/pages/admin/Manufacturers.vue 查看完整代码
<template>
[tuture-del]  <div>
[tuture-del]    <table class="table">
[tuture-add]  <div class="manufacturers">
[tuture-add]    <el-table
[tuture-add]    class="table"
[tuture-add]    :data="manufacturers">
[tuture-add]      <el-table-column
[tuture-add]        prop="name"
[tuture-add]        label="制造商"
[tuture-add]        width="180">
[tuture-add]      </el-table-column>
[tuture-add]      <el-table-column
[tuture-add]        label="操作"
[tuture-add]        width="200">
[tuture-add]        <template slot-scope="scope">
[tuture-add]          <el-button class="modify" type="text" size="small"><router-link :to="'/admin/manufacturers/edit/' + scope.row._id">修改</router-link></el-button>
[tuture-add]          <el-button class="remove" @click="removeManufacturer(scope.row._id), deleteRow(scope.$index, products)" type="text" size="small">删除</el-button>
[tuture-add]        </template>
[tuture-add]      </el-table-column>
[tuture-add]    </el-table>
    // ...
[tuture-del]    </table>
    // ...
  </div>
</template>

<style>
[tuture-del]table {
[tuture-del]  margin: 0 auto;
[tuture-add].manufacturers {
[tuture-add]  padding-top: 20px;
[tuture-add]  text-align: center;
[tuture-add]}
[tuture-add].table {
[tuture-add]  margin:0 auto;
[tuture-add]  width: 380px;
[tuture-add]}
[tuture-add].el-table .cell {
[tuture-add]  text-align: center;
[tuture-add]  padding-top: 10px;
[tuture-add]  padding-bottom: 10px;
}
[tuture-del]
// ...
</style>
 // ...
```

这是我们重构后的制造商信息列表：

![](https://static.powerformer.com/c/092faf4/170bd8133a39a38f.png)

再次进入 `NewManufacturers` 组件进行重构，同样的将定义的计算属性 `model` 放到 `data` 属性中：

```html src/pages/admin/NewManufacturers.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/pages/admin/NewManufacturers.vue 查看完整代码
// ...
<script>
import ManufacturerForm from '@/components/ManufacturerForm.vue';
export default {
[tuture-del]  computed: {
[tuture-del]    model() {
[tuture-del]      return {};
[tuture-add]  data() {
[tuture-add]    return {
[tuture-add]      model: {}
    }
  },
  methods: {
    // ...
```

然后进入子组件 `ManufacturerForm` 中进行重构，同 `ProductForm` 组件类似，使用 `element-ui` 组件库提供的 `el-form` 表单组件替换了之前普通的表单展示制造商信息：

```html src/components/ManufacturerForm.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/components/ManufacturerForm.vue 查看完整代码
<template>
[tuture-del]  <form @submit.prevent="saveManufacturer">
[tuture-add]  <div class="manufacturerInfo">
[tuture-add]    <el-form class="form" ref="form" :model="model" label-width="180px">
[tuture-add]      <el-form-item label="Name">
[tuture-add]        <el-input v-model="model.name"></el-input>
[tuture-add]      </el-form-item>
[tuture-add]      <el-form-item>
[tuture-add]        <el-button v-if="isEditing" type="primary" @click="onSubmit">Update Manufacturer</el-button>
[tuture-add]        <el-button v-else @click="onSubmit">Add Manufacturer</el-button>
[tuture-add]      </el-form-item>
[tuture-add]    </el-form>
[tuture-add]  </div>
[tuture-add]  <!-- <form @submit.prevent="saveManufacturer">
    <div class="form-group">
      <label>Name</label>
      <input
        // ...

    <div class="form-group new-button">
      <button class="button">
[tuture-del]        <i class="fa fa-pencil"></i>
[tuture-add]        <i class="fa fa-pencil"></i> -->
        <!-- Conditional rendering for input text -->
[tuture-del]        <span v-if="isEditing">Update Manufacturer</span>
[tuture-add]        <!-- <span v-if="isEditing">Update Manufacturer</span>
        <span v-else>Add Manufacturer</span>
      </button>
    </div>
[tuture-del]  </form>
[tuture-add]  </form> -->
</template>

<script>
export default {
  props: ['model', 'isEditing'],
  methods: {
[tuture-del]    saveManufacturer() {
[tuture-add]    onSubmit() {
      this.$emit('save-manufacturer', this.model)
    }
  }
}
</script>
[tuture-add]<style>
[tuture-add].manufacturerInfo {
[tuture-add]  padding-top: 10px;
[tuture-add]}
[tuture-add].form {
[tuture-add]  margin: 0 auto;
[tuture-add]  width: 500px;
[tuture-add]}
[tuture-add].el-input__inner {
[tuture-add]  height: 50px;
[tuture-add]}
[tuture-add]</style>
```

这是我们重构后用户添加或者修改制造商信息时的表单界面：

![](https://static.powerformer.com/c/092faf4/170bd81d39c3eb45.png)

最后我们进入 `Cart` 组件进行重构，我们会发现该组件与 `ProductList` 组件极其相似，因为两者都复用了子组件 `ProductItem`，该组件是为了展示购物车商品信息列表。

```html src/pages/Cart.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/b6d9a87f1af8f2a068cfb5a2d392fd1dbaec84c3/src/pages/Cart.vue 查看完整代码
// ...
    <div class="title">
      <h1>{{msg}}</h1>
    </div>
[tuture-del]    <template v-for="product in cart">
[tuture-del]      <product-item :product="product" :key="product._id"></product-item>
[tuture-del]    </template>
[tuture-add]    <product-item :products="cart"></product-item>
  </div>
</template>

// ...
```

这是重构后的购物车界面：

![](https://static.powerformer.com/c/092faf4/170bd82508c14642.png)

### 小结

这一节我们主要就是使用 `element-ui` 组件库进行项目代码的重构，实现了首页导航栏、商品信息功能、制造商信息功能以及购物车的页面升级，提高了用户的交互体验。但是这也造成了部分功能逻辑的瘫痪，我们在下一节会带大家一起去解决问题。

## 修复 ElementUI 表单的双向绑定问题

上一节我们使用了 `element-ui` 组件库完成项目代码重构，可是当我们把项目跑起来之后发现表单信息仍然无法编辑，说明我们之前的尝试失败。不过我们并没有灰心，而是选择继续尝试，这一节我们又尝试新方法来修复`element-ui` 表单双向绑定问题。

大家遇到的问题应该是这样子：

![](https://static.powerformer.com/c/092faf4/170bd829d4626c61.png)

### 重构 Edit 组件

我们首先进入 `Edit` 组件进行修复，这里我们主要恢复了原先的数据定义。

```html src/pages/admin/Edit.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/02f5761fdc540a3cf3e0549b2a4e3f616b4f1a48/src/pages/admin/Edit.vue 查看完整代码
<template>
  <div>
    <div class="title">
      <h1>This is Admin/Edit</h1>
    </div>
    <product-form
      @save-product="updateProduct"
      :model="model"
      :manufacturers="manufacturers"
      :isEditing="true"
[tuture-del]      ></product-form>
[tuture-add]    ></product-form>
  </div>
</template>

<script>
[tuture-del]import ProductForm from '@/components/products/ProductForm.vue';
[tuture-add]import ProductForm from "@/components/products/ProductForm.vue";
export default {
[tuture-del]  data: {
[tuture-del]    model() {
[tuture-del]      const product = this.$store.getters.productById(this.$route.params['id']);
[tuture-del]      // 这里返回 product 的拷贝，是为了在修改 product 的拷贝之后，在保存之前不修改本地 Vuex stire 的 product 属性
[tuture-del]      return { ...product, manufacturer: { ...product.manufacturer } };
[tuture-del]    }
[tuture-del]  },
  created() {
[tuture-del]    const { name } = this.model;
[tuture-add]    const { name = "" } = this.modelData || {};
[tuture-add]
    if (!name) {
[tuture-del]      this.$store.dispatch('productById', {
[tuture-del]        productId: this.$route.params['id']
[tuture-add]      this.$store.dispatch("productById", {
[tuture-add]        productId: this.$route.params["id"]
      });
    }

    if (this.manufacturers.length === 0) {
[tuture-del]      this.$store.dispatch('allManufacturers');
[tuture-add]      this.$store.dispatch("allManufacturers");
    }
  },
  computed: {
    manufacturers() {
      return this.$store.getters.allManufacturers;
[tuture-add]    },
[tuture-add]    model() {
[tuture-add]      const product = this.$store.getters.productById(this.$route.params["id"]);
[tuture-add]      const res = { ...product, manufacturer: { ...product.manufacturer } };
[tuture-add]
[tuture-add]      return res;
    }
  },
  methods: {
    updateProduct(product) {
[tuture-del]      this.$store.dispatch('updateProduct', {
[tuture-del]        product,
[tuture-del]      })
[tuture-add]      this.$store.dispatch("updateProduct", {
[tuture-add]        product
[tuture-add]      });
    }
  },
  components: {
[tuture-del]    'product-form': ProductForm
[tuture-add]    "product-form": ProductForm
  }
[tuture-del]}
[tuture-add]};
</script>
```

我们又将替换到 `data` 属性中的 `model` 对象恢复到了计算属性中，用于缓存 `model` 对象信息，提高性能。我们打算在下面的 `ProductForm` 组件中进行修复表单无法编辑的问题。

### 重构 ProductForm 组件

再次进入 `ProductForm` 组件中，我们尝试另一种方法来修复表单无法编辑的问题。

```html src/components/products/ProductForm.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/02f5761fdc540a3cf3e0549b2a4e3f616b4f1a48/src/components/products/ProductForm.vue 查看完整代码
<template>
  <div class="productInfo">
[tuture-del]    <el-form class="form" ref="form" :model="model" label-width="180px">
[tuture-add]    <el-form class="form" ref="form" label-width="180px">
      <el-form-item label="Name">
        <el-input v-model="model.name"></el-input>
      </el-form-item>
      // ...
        <el-input v-model="model.price"></el-input>
      </el-form-item>
      <el-form-item label="Manufacturer ">
[tuture-del]        <el-select v-model="model.manufacturer.name" clearable placeholder="请选择制造商">
[tuture-add]        <el-select
[tuture-add]          v-model="modelData.manufacturer.name"
[tuture-add]          clearable
[tuture-add]          placeholder="请选择制造商"
[tuture-add]        >
          <el-option
            v-for="manufacturer in manufacturers"
            :key="manufacturer._id"
            :label="manufacturer.name"
[tuture-del]            :value="manufacturer.name">
[tuture-add]            :value="manufacturer.name"
[tuture-add]          >
          </el-option>
        </el-select>
      </el-form-item>
      // ...
        <el-input type="textarea" v-model="model.description"></el-input>
      </el-form-item>
      <el-form-item>
[tuture-del]        <el-button v-if="isEditing" type="primary" @click="onSubmit">Update Product</el-button>
[tuture-add]        <el-button v-if="isEditing" type="primary" @click="onSubmit"
[tuture-add]          >Update Product</el-button
[tuture-add]        >
        <el-button v-else @click="onSubmit">Add Product</el-button>
      </el-form-item>
    </el-form>
  // ...
      </div>
    </div> -->

[tuture-del]    <!-- <div class="col-lg-4 col-md-4 col-sm-12 col-xs-12">
[tuture-add]  <!-- <div class="col-lg-4 col-md-4 col-sm-12 col-xs-12">
      <div class="form-group">
        <label>Image</label>
        <input
          // ...
      <div class="form-group new-button">
        <button class="button">
          <i class="fa fa-pencil"></i> -->
[tuture-del]          <!-- Conditional rendering for input text -->
[tuture-del]          <!-- <span v-if="isEditing">Update Product</span>
[tuture-add]  <!-- Conditional rendering for input text -->
[tuture-add]  <!-- <span v-if="isEditing">Update Product</span>
          <span v-else>Add Product</span>
        </button>
      </div>
    // ...

<script>
export default {
[tuture-del]  props: ['model', 'manufacturers', 'isEditing'],
[tuture-add]  data() {
[tuture-add]    return {
[tuture-add]      modelData: { manufacturer: { name: "" } }
[tuture-add]    };
[tuture-add]  },
[tuture-add]  props: ["model", "manufacturers", "isEditing"],
  created() {
[tuture-del]    console.log(this.model)
[tuture-add]    const product = this.model;
[tuture-add]
[tuture-add]    this.modelData = { ...product, manufacturer: { ...product.manufacturer } };
[tuture-add]  },
[tuture-add]  watch: {
[tuture-add]    model(val, oldVal) {
[tuture-add]      this.modelData = val;
[tuture-add]    }
  },
  methods: {
    onSubmit() {
[tuture-del]      this.$emit('save-product', this.model)
[tuture-add]      this.$emit("save-product", this.modelData);
    }
  }
[tuture-del]}
[tuture-add]};
</script>
<style>
.productInfo {
  // ...
```

这里我们没有直接使用从父组件获取的 `model` 对象作为表单数据对象，而是在该组件中自定义一个 `modelData` 对象，并使用默认初始形式。然后在组件刚被创建时，先将从父组件获取的 `model` 对象赋值给一个临时变量 `product`，然后将 `product` 浅拷贝到 `modelData` 对象中，这样就避免了表单数据对象使用计算属性。但是这仅仅完成了一半的工作，因为我们需要实现双向绑定的效果，因此我们需要监测表单组件的变化，通过使用 `watch` 方法监测用户的输入，然后将新数据储存到 `modelData` 对象中，这样就成功实现了双向绑定，而且表单也能随意进行编辑。

但是这里我们仅仅在下拉菜单中使用了 `modelData` 对象进行尝试，因此后面我们会在整个表单内组件使用该对象。

### 小结

这一节我们主要带大家修复了 `element-ui` 表单双向绑定问题，通过自定义 `modelData` 对象以及 `watch` 方法监测表单数据的改变实现了表单数据的双向绑定，并且解决了表单无法编辑的问题。但是仅仅在下拉菜单中进行尝试，后面我们会重构整个商品信息表单组件。

## 完善表单双向绑定问题

### 重构 ProductForm 组件

再次进入 `ProductForm` 组件，我们需要完善上一节遗留的问题，也就是仅仅对商品信息表单中的下拉菜单进行了尝试，并且尝试成功，因此这一节我们需要将 `modelData` 对象导入所有表单内组件中，解决其他表单内组件无法编辑的问题。

```html src/components/products/ProductForm.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/5c7666c1f66b288f2c02b04c964c53791ba4ecb5/src/components/products/ProductForm.vue 查看完整代码
<template>
  <div class="productInfo">
    <el-form class="form" ref="form" label-width="180px">
      <el-form-item label="Name">
[tuture-del]        <el-input v-model="model.name"></el-input>
[tuture-add]        <el-input v-model="modelData.name"></el-input>
      </el-form-item>
      <el-form-item label="Price">
[tuture-del]        <el-input v-model="model.price"></el-input>
[tuture-add]        <el-input v-model="modelData.price"></el-input>
      </el-form-item>
      <el-form-item label="Manufacturer ">
        <el-select
          // ...
        </el-select>
      </el-form-item>
      <el-form-item label="Image ">
[tuture-del]        <el-input v-model="model.image"></el-input>
[tuture-add]        <el-input v-model="modelData.image"></el-input>
      </el-form-item>
      <el-form-item label="Description ">
[tuture-del]        <el-input type="textarea" v-model="model.description"></el-input>
[tuture-add]        <el-input type="textarea" v-model="modelData.description"></el-input>
      </el-form-item>
      // ...
    </el-form>
  </div>
  // ...
</template>
 // ...
```

### 小结

这一节我们带大家补充了上一节遗留的问题，也就是复制下拉菜单中的尝试到其他表单内组件中，保证整个表单组件都能够顺利地实现编辑功能。

## 解决操作商品信息表单报错问题

### 重构 ProductForm 组件

相信大家在对商品信息表单进行添加或者修改操作时，控制台会出现 `id` 属性未定义的错误，我们首先应该进入报错的组件中进行调试，大家应该都看到了报错信息出现在 `ProductForm` 组件中，因此我们需要进入 `ProductForm` 组件进行调试。

```html src/components/products/ProductForm.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/11118c364dd8c3bbe0380075f980767417ec1f33/src/components/products/ProductForm.vue 查看完整代码
// ...
<script>
  export default {
    // ...
    props: ["model", "manufacturers", "isEditing"],
    created() {
      const product = this.model;
  [tuture-del]
      this.modelData = { ...product, manufacturer: { ...product.manufacturer } };
    },
    watch: {
      // ...
    },
    methods: {
      onSubmit() {
  [tuture-add]      const manufacturer = this.manufacturers.find(item => item.name === this.modelData.manufacturer.name);
  [tuture-add]      this.modelData.manufacturer = manufacturer;
        this.$emit("save-product", this.modelData);
      }
    }
  };
</script>
// ...
```

首先大家应该清楚商品对象中还包含了相应的制造商对象，并且制造商对象中包含了 `id` 属性和 `name` 属性。但是我们应该可以发现商品信息表单中的下拉菜单双向绑定的是商品对象中的制造商对象的 `name` 属性，因此在 `watch` 方法中存储到 `modelData` 对象中的制造商对象也只有 `name` 属性，但是后端数据库要求制造商对象必须也要有 `id` 属性，这就是我们点击更新商品信息出现报错的原因。

这里我们使用了本地制造商数组的 `find` 方法，检索到了对应 `name` 的制造商对象并将其覆盖掉 `modelData` 对象中的制造商对象，这样我们的 `modelData` 对象中的制造商对象就是一个符合后端数据库要求的对象了。

### 小结

这一节我们带大家分析并尝试解决了操作商品信息表单出现 `id` 属性未定义的问题。

## 添加动态效果及消息提示

我们注意到了当用户进行添加或修改商品或者制造商信息时，难免会遇到更新延迟的问题，这个时候如果页面毫无反馈会显得些许尴尬，因此我们认为只要用户进行添加或者修改操作，在后端数据同步完成之前我们为页面添加一个动态加载的效果，给用户一个反馈表示数据正在处理中，请耐心等待；并且在后端同步完成之后为页面添加一个消息提示，给用户一个反馈表示数据处理成功，这样就避免了尴尬的场面，提高了用户的交互体验。

### 实现 loading 动态加载效果

再次进入 `ManufactureForm` 组件，实现用户在添加或者修改制造商信息时且当后端数据同步完成之前，页面出现 `loading`动态加载效果。

```html src/components/ManufacturerForm.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/38caf6d47d40776d073f517ea7ff25c9e21b0a8b/src/components/ManufacturerForm.vue 查看完整代码
<template>
  <div class="manufacturerInfo">
[tuture-del]    <el-form class="form" ref="form" :model="model" label-width="180px">
[tuture-add]    <el-form
[tuture-add]    class="form"
[tuture-add]    ref="form"
[tuture-add]    label-width="180px"
[tuture-add]    v-loading="loading"
[tuture-add]    element-loading-text="拼命加载中"
[tuture-add]    element-loading-spinner="el-icon-loading"
[tuture-add]    element-loading-background="rgba(0, 0, 0, 0.8)">
      <el-form-item label="Name">
[tuture-del]        <el-input v-model="model.name"></el-input>
[tuture-add]        <el-input v-model="manufacturerData.name"></el-input>
      </el-form-item>
      <el-form-item>
[tuture-del]        <el-button v-if="isEditing" type="primary" @click="onSubmit">Update Manufacturer</el-button>
[tuture-add]        <el-button v-if="isEditing" type="primary" native-type="submit" @click="onSubmit">Update Manufacturer</el-button>
        <el-button v-else @click="onSubmit">Add Manufacturer</el-button>
      </el-form-item>
    </el-form>
  </div>
  // ...
</template>

<script>
export default {
  props: ['model', 'isEditing'],
[tuture-add]  data() {
[tuture-add]    return {
[tuture-add]      manufacturerData: {name: ''}
[tuture-add]    }
[tuture-add]  },
[tuture-add]  created() {
[tuture-add]    this.manufacturerData = this.model
[tuture-add]  },
[tuture-add]  watch: {
[tuture-add]    model(val, oldVal) {
[tuture-add]      this.manufacturerData = val;
[tuture-add]    }
[tuture-add]  },
[tuture-add]  computed: {
[tuture-add]    loading() {
[tuture-add]      return this.$store.state.showLoader
[tuture-add]    }
[tuture-add]  },
  methods: {
    onSubmit() {
[tuture-del]      this.$emit('save-manufacturer', this.model)
[tuture-add]      this.$emit('save-manufacturer', this.manufacturerData);
    }
  }
}
</script>
// ...
```

首先我们在该组件中使用了 `element-ui` 组件库提供的自定义指令 `v-loading`，通过判断 `loading` 为 true 还是 false 来决定是否实现动态加载效果。这里我们通过获取本地状态中的 `showLoader` 属性作为 `loading` 属性值，因为在用户刚进行添加或修改操作时，向后端发起数据请求，此时本地状态中的 `showLoader` 属性值为 true，当成功获取到了数据响应之后，也就是后端数据同步完成，此时 `showLoader` 属性值为 false，这样就实现了在指定时间显示动态加载效果；除此之外，我们还按照 `ProductForm` 组件补充修改了数据处理，解决制造商表单组件编辑问题。

同样进入 `ProductForm` 组件进行修改，实现用户在添加或修改商品信息时，且当后端数据同步完成之前，页面出现 `loading` 动态加载效果。

```html src/components/products/ProductForm.vue https://github.com/tuture-dev/vue-online-shop-frontend/blob/38caf6d47d40776d073f517ea7ff25c9e21b0a8b/src/components/products/ProductForm.vue 查看完整代码
<template>
  <div class="productInfo">
[tuture-del]    <el-form class="form" ref="form" label-width="180px">
[tuture-add]    <el-form
[tuture-add]    class="form"
[tuture-add]    ref="form"
[tuture-add]    label-width="180px"
[tuture-add]    v-loading="loading"
[tuture-add]    element-loading-text="拼命加载中"
[tuture-add]    element-loading-spinner="el-icon-loading"
[tuture-add]    element-loading-background="rgba(0, 0, 0, 0.8)">
      <el-form-item label="Name">
        <el-input v-model="modelData.name"></el-input>
      </el-form-item>
      // ...
        <el-input type="textarea" v-model="modelData.description"></el-input>
      </el-form-item>
      <el-form-item>
[tuture-del]        <el-button v-if="isEditing" type="primary" @click="onSubmit"
[tuture-add]        <el-button v-if="isEditing" type="primary" native-type="submit" @click="onSubmit"
          >Update Product</el-button
        >
        <el-button v-else @click="onSubmit">Add Product</el-button>
      // ...
      this.modelData = val;
    }
  },
[tuture-add]  computed: {
[tuture-add]    loading() {
[tuture-add]      return this.$store.state.showLoader
[tuture-add]    }
[tuture-add]  },
[tuture-add]  // updated() {
[tuture-add]  //   // 等待后端完成商品信息的同步更新，设置loading为false
[tuture-add]  //   const newProductName = this.$store.getters.productById(this.model._id);
[tuture-add]  //   const isLoaded = this.modelData.name === newProductName;
[tuture-add]  //   if(isLoaded) {
[tuture-add]  //     this.loading = false;
[tuture-add]  //   }
[tuture-add]  // },
  methods: {
    onSubmit() {
[tuture-add]      // 由于表单中只绑定了modelData.manufacturer.name，
[tuture-add]      // 缺少manufacturer._id,但是后端需要manufacturer整个对象,
[tuture-add]      // 所以需要将manufacturers中对应的manufacturer找出并覆盖到modelData中
      const manufacturer = this.manufacturers.find(item => item.name === this.modelData.manufacturer.name);
      this.modelData.manufacturer = manufacturer;
[tuture-add]
      this.$emit("save-product", this.modelData);
    }
  }
// ...
```

### 实现消息提示功能

首先进入 `actions.js` 文件进行修改，由于发送网络请求数据的操作在该文件中执行，因此我们可以将消息提示功能添加到这里。

```js src/store/actions.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/38caf6d47d40776d073f517ea7ff25c9e21b0a8b/src/store/actions.js 查看完整代码
// ...
[tuture-add]import { Message } from 'element-ui';

// ...
export const productActions = {
  allProducts({ commit }) {
    // ...
  },
  productById({ commit }, payload) {
    // ...
  },
  removeProduct({ commit }, payload) {
    commit(REMOVE_PRODUCT);

    const { productId } = payload;
[tuture-del]    axios.delete(`${API_BASE}/products/${productId}`).then(() => {
[tuture-add]    axios.delete(`${API_BASE}/products/${productId}`)
[tuture-add]    .then(() => {
      // 返回 productId，用于删除本地对应的商品
      commit(REMOVE_PRODUCT_SUCCESS, {
        productId,
      });
[tuture-add]      Message({
[tuture-add]        message: '恭喜你，商品删除成功！',
[tuture-add]        type: 'success'
[tuture-add]      })
[tuture-add]    })
[tuture-add]    .catch(() => {
[tuture-add]      Message.error('不好意思，商品删除失败！');
    })
  },
  updateProduct({ commit }, payload) {
    commit(UPDATE_PRODUCT);

    const { product } = payload;
[tuture-del]    axios.put(`${API_BASE}/products/${product._id}`, product).then(() => {
[tuture-add]    axios.put(`${API_BASE}/products/${product._id}`, product)
[tuture-add]    .then(response => {
      commit(UPDATE_PRODUCT_SUCCESS, {
[tuture-del]        product,
[tuture-add]        product: response.data,
      });
[tuture-add]      Message({
[tuture-add]        message: '恭喜你，商品更新成功！',
[tuture-add]        type: 'success'
[tuture-add]      })
[tuture-add]    })
[tuture-add]    .catch(() => {
[tuture-add]      Message.error('不好意思，商品更新失败！');
    })
  },
  addProduct({ commit }, payload) {
    commit(ADD_PRODUCT);

    const { product } = payload;
[tuture-del]    axios.post(`${API_BASE}/products`, product).then(response => {
[tuture-add]    axios.post(`${API_BASE}/products`, product)
[tuture-add]    .then(response => {
      commit(ADD_PRODUCT_SUCCESS, {
        product: response.data,
      })
[tuture-del]      console.log(response.data);
[tuture-add]      Message({
[tuture-add]        message: '恭喜你，商品添加成功！',
[tuture-add]        type: 'success'
[tuture-add]      })
[tuture-add]    })
[tuture-add]    .catch(() => {
[tuture-add]      Message.error('不好意思，商品添加失败！');
    })
  }
};
 // ...
export const manufacturerActions = {
  allManufacturers({ commit }) {
    // ...
  },
  manufacturerById({ commit }, payload) {
    // ...
  },
  removeManufacturer({ commit }, payload) {
    commit(REMOVE_MANUFACTURER);

    const { manufacturerId } = payload;
[tuture-del]    axios.delete(`${API_BASE}/manufacturers/${manufacturerId}`).then(() => {
[tuture-add]    axios.delete(`${API_BASE}/manufacturers/${manufacturerId}`)
[tuture-add]    .then(() => {
      // 返回 manufacturerId，用于删除本地对应的制造商
      commit(REMOVE_MANUFACTURER_SUCCESS, {
        manufacturerId,
      });
[tuture-add]      Message({
[tuture-add]        message: '恭喜你，制造商删除成功！',
[tuture-add]        type: 'success'
[tuture-add]      })
[tuture-add]    })
[tuture-add]    .catch(() => {
[tuture-add]      Message.error('不好意思，制造商删除失败！');
    })
  },
  updateManufacturer({ commit }, payload) {
    commit(UPDATE_MANUFACTURER);

    const { manufacturer } = payload;
[tuture-del]    axios.put(`${API_BASE}/manufacturers/${manufacturer._id}`, manufacturer).then(() => {
[tuture-add]    axios.put(`${API_BASE}/manufacturers/${manufacturer._id}`, manufacturer)
[tuture-add]    .then(response => {
      commit(UPDATE_MANUFACTURER_SUCCESS, {
[tuture-del]        manufacturer,
[tuture-add]        manufacturer: response.data,
      });
[tuture-add]      Message({
[tuture-add]        message: '恭喜你，制造商更新成功！',
[tuture-add]        type: 'success'
[tuture-add]      })
[tuture-add]    })
[tuture-add]    .catch(() => {
[tuture-add]      Message.error('不好意思，制造商更新失败！');
    })
  },
  addManufacturer({ commit }, payload) {
    commit(ADD_MANUFACTURER);
[tuture-del]
    const { manufacturer } = payload;
[tuture-del]    axios.post(`${API_BASE}/manufacturers`, manufacturer).then(response => {
[tuture-add]    axios.post(`${API_BASE}/manufacturers`, manufacturer)
[tuture-add]    .then(response => {
      commit(ADD_MANUFACTURER_SUCCESS, {
        manufacturer: response.data,
[tuture-add]      });
[tuture-add]      Message({
[tuture-add]        message: '恭喜你，制造商添加成功！',
[tuture-add]        type: 'success'
      })
    })
[tuture-add]    .catch(() => {
[tuture-add]      Message.error('不好意思，制造商添加失败！');
[tuture-add]    })
  }
}
```

这里我们首先导入了 `element-ui` 组件库提供的 `Message` 消息提示组件，并在网络请求成功之后添加成功消息提醒，在请求失败之后添加失败消息提醒。

然后进入 `mutations.js` 文件进行修改，这里的修改是为本地购物车数据处理添加消息提示。

```js src/store/mutations.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/38caf6d47d40776d073f517ea7ff25c9e21b0a8b/src/store/mutations.js 查看完整代码
// ...
[tuture-add]import { Message } from 'element-ui';

// ...
export const cartMutations = {
  [ADD_TO_CART](state, payload) {
    const { product } = payload;
[tuture-del]    state.cart.push(product)
[tuture-add]    state.cart.push(product);
[tuture-add]    Message({
[tuture-add]      message: '恭喜你，成功加入购物车！',
[tuture-add]      type: 'success'
[tuture-add]    })
  },
  [REMOVE_FROM_CART](state, payload) {
    const { productId } = payload
    state.cart = state.cart.filter(product => product._id !== productId)
[tuture-add]    Message({
[tuture-add]      message: '恭喜你，成功移除购物车！',
[tuture-add]      type: 'success'
[tuture-add]    })
  },
};

// ...
```

同样的我们首先需要导入 `element-ui` 组件库提供的 `Message` 消息提示组件，当用户进行添加或者移除购物车操作时，执行操作成功消息提醒。

我们在进行添加、删除、修改以及加入或移除购物车操作时都会得到这样的反馈：

![](https://static.powerformer.com/c/092faf4/170bd8414598aa64.png)

### 小结

这一节我们主要做的几点工作：

- 为表单组件添加 `element-ui` 组件库提供的 `v-loading` 指令，实现动态加载效果；
- 添加了 `element-ui` 组件库提供的 `Message` 消息提示组件，实现用户操作表单信息后得到的反馈消息提示。

## 解决表单信息修改后无法显示最新

重构到这里相信有些朋友已经迫不及待地将项目跑起来了，但是总是事与愿违，但是大家丝毫不用方，只要您跟着我们一步一步脚踏实地地去分析问题，那么什么问题都会迎刃而解了。现在的问题就是当用户对商品或者制造商进行信息修改时，点击更新之后表单却又显示了旧信息。

大家遇到的状况应该是这样：

![](https://static.powerformer.com/c/092faf4/170bd845ae228eb9.png)

![](https://static.powerformer.com/c/092faf4/170bd848f59fad3b.png)

数据出现问题我们应该根据

`vue`

的单向数据流原则进行调试，当用户对表单信息进行更新时，应该首先向后端发起网络请求，然后将最新数据同步到本地状态中进行展示，因此我们来到

`actions.js`

文件中进行调试。

### 提交最新数据

再次进入 `actions.js` 文件进行调试，我们可以大胆的猜测网络请求成功之后提交到 `mutations.js` 文件中的数据对象不是用户修改的最新数据。

```js src/store/actions.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/aa23775d642373413aa51b97bc3e7915ac9e2095/src/store/actions.js 查看完整代码
// ...
export const productActions = {
  // ...
  updateProduct({ commit }, payload) {
    commit(UPDATE_PRODUCT);

    const { product } = payload;
    axios.put(`${API_BASE}/products/${product._id}`, product)
    .then(response => {
      commit(UPDATE_PRODUCT_SUCCESS, {
[tuture-del]        product: response.data,
[tuture-add]        product: product,
      });
      Message({
        message: '恭喜你，商品更新成功！',
        type: 'success'
      })
    })
    .catch(() => {
      Message.error('不好意思，商品更新失败！');
    })
  },
  // ...
};
 // ...
export const manufacturerActions = {
  // ...
  updateManufacturer({ commit }, payload) {
    commit(UPDATE_MANUFACTURER);

    const { manufacturer } = payload;
    axios.put(`${API_BASE}/manufacturers/${manufacturer._id}`, manufacturer)
    .then(response => {
      commit(UPDATE_MANUFACTURER_SUCCESS, {
[tuture-del]        manufacturer: response.data,
[tuture-add]        manufacturer: manufacturer,
      });
      Message({
        message: '恭喜你，制造商更新成功！',
        type: 'success'
      })
    })
    .catch(() => {
      Message.error('不好意思，制造商更新失败！');
    })
  },
  // ...
}
```

我们在这里将网络请求成功时提交的载荷修改为了最新数据对象，然后提交到对应类型的 `mutation` 中进行本地数据的更新。

### 将最新数据同步到本地

紧接着我们需要进入 `mutations.js` 文件，将其获取到的最新数据同步到本地状态中。

```js src/store/mutations.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/aa23775d642373413aa51b97bc3e7915ac9e2095/src/store/mutations.js 查看完整代码
// ...
export const productMutations = {
  // ...
  [UPDATE_PRODUCT_SUCCESS](state, payload) {
    state.showLoader = false;

    const { product: newProduct } = payload;
    state.product = newProduct;
[tuture-del]    state.products = state.products.map(product => {
[tuture-add]    state.products = state.products.map( product => {
      if (product._id === newProduct._id) {
        return newProduct;
      }
[tuture-del]
      return product;
[tuture-del]    })
[tuture-add]    });
[tuture-add]
[tuture-add]    state.product = newProduct;
  },
  [ADD_PRODUCT](state) {
    state.showLoader = true;
  },
  // ...
};
 // ...
export const manufacturerMutations = {
  // ...
  [UPDATE_MANUFACTURER_SUCCESS](state, payload) {
    state.showLoader = false;

    const { manufacturer: newManufacturer } = payload;
    state.manufacturers = state.manufacturers.map(manufacturer => {
      if (manufacturer._id === newManufacturer._id) {
        return newManufacturer;
      }
[tuture-del]
      return manufacturer;
    });
[tuture-add]
[tuture-add]    state.manufacturer = newManufacturer;
  },
  [ADD_MANUFACTURER](state) {
    state.showLoader = true;
  },
  // ...
}
```

### 小结

这一节我们主要带大家分析并尝试解决了表单信息修改后无法显示最新信息的问题。

本篇教程为大家呈现了在实际开发过程中，使用 element-ui 组件库对电商应用前端代码进行重构所遇到的一些问题，并且我们一步一步地带大家去分析及尝试解决问题。希望这篇教程让大家对 element-ui 组件库的使用需要注意的问题有一个大致的了解，重要的是分析和尝试解决问题的能力。好了，到这里我们的项目基本上可以愉快地跑起来了，用户的交互体验感明显得到了改善。
