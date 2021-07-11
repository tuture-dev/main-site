---
title: '从零到部署：用 Vue 和 Express 实现迷你全栈电商应用（二）'
description: '这是本教程的第二个部分，使用 Node.js 的 Express 路由框架来编写我们迷你淘宝的 API 后台部分，在这个教程中，你将学到一个基础的 Web 后台 API 应用如何搭建，包括中间件、路由等，如何通过 Node.js 数据库接口包操作 MongoDB 数据库，并且通过 Postman 来测试我们写好的 API。'
tags: ['Express', 'MongoDB']
categories: ['前端', 'Vue', '入门']
date: 2019-10-21T00:00:00.509Z
photos:
  - https://static.powerformer.com/c/cb08dc8/express2.png
---

<div class="profileBox">
  <div class="avatarBox">
    <a href="https://github.com/pftom"><img src="/images/avatars/pftom.jpg" alt="" class="avatar"></a>
  </div>
  <div class="rightBox">
    <div class="infoBox">
    <a href="https://github.com/pftom"><p class="nickName">@pftom</p></a>
  </div>
  <div class="codeBox">
    <a href="https://github.com/pftom/vue-online-shop-backend"><span class="codeText">查看代码</span></a>
  </div>
  </div>
</div>

## 快速开始

我们在平时所用到的一些网站、App，它们会将我们的数据进行保存，当我们关闭这些网站或者 App 后，下次打开还能看到我们之前的一些文字、视频记录。在迷你全栈电商应用实战系列的第二篇教程中，我们将通过基于 Node.js 平台的 [Express](http://expressjs.com) 框架实现后端 API 数据接口，并且将数据存储在 [MongoDB](https://mongodb.com/) 中。这样我们的网站就能够记录用户添加的商品，并且无论以后什么时候打开，都能获取我们之前的记录。

迷你全栈电商应用实战系列的全部教程如下：

1. [**第一部分**](https://tuture.co/2019/10/17/0b662ce/)：用 Vue 搭建前端项目的骨架，实现基于嵌套、动态路由的多页面跳转。
2. **第二部分**（也就是这篇）：用 Express 实现后端 REST API，并使用 MongoDB 进行数据存储。
3. **第三部分**（✍ 写作中）：通过 Vue 的双向数据绑定和模板语法实现数据获取与修改，并用 Vuex 实现前端的状态管理。
4. **第四部分**（✍ 写作中）：用 Element UI 组件库让界面更加专业美观，并且用 Docker 和 Docker Compose 部署我们的全栈项目。

{% note info %}
**提示**

阅读这篇文章需要你对 Express 框架有一定的了解。如果不熟悉的话，可以阅读[这篇教程](https://tuture.co/2019/11/26/cd5b993/)快速上手哦。
{% endnote %}

### 初始化项目

首先，让我们使用 [express-generator](https://github.com/expressjs/generator) 脚手架来初始化我们的 Express 项目。在终端运行如下命令来安装：

```bash
npm install -g express-generator
```

打开终端，输入如下命令测试是否安装成功：

```Bash
express --version # 4.15.5
```

然后输入如下命令初始化我们的 Express 项目：

```bash
express --view=ejs vue-online-shop-backend
```

当项目初始化成功之后，接下来通过如下命令开启项目：

```Bash
cd vue-online-shop-backend
npm install
npm start
```

接着打开浏览器，输入 `http://localhost:3000/` 查看我们初始好的项目效果。

![](https://static.powerformer.com/c/cb08dc8/f6399092aa34edf6f8e6140f404edcab.png)

### 初探脚手架代码

通过  express-generator 初始化的项目代码中，我们在整个教程中只需要了解下面四个文件：

- `app.js`：Express 应用主文件
- `bin/www`：用来开启服务器的脚本
- `routes/index.js`：路由主文件
- `views/index.ejs`：主页的模板文件，这里由于我们只打算实现 API 数据接口，所以不用关心

与之前的 [Express 教程](https://tuture.co/2019/11/26/cd5b993/)不同的是，脚手架代码并没有把所有的路由都放在 `app.js` 中，而是根据不同的子应用（users、index）进行了拆分，这也与该系列[第一篇教程](https://tuture.co/2019/10/17/0b662ce/)中 vue-router 的嵌套路由不谋而合。

我们大致地看一下 `app.js` 和 `routes/index.js` 的内容，快速过一遍 Express 的基础知识。首先来看一下 `app.js` 的代码：

```js app.js https://github.com/pftom/vue-online-shop-backend/blob/7c749c5/app.js 查看完整代码
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var index = require('./routes/index');
var users = require('./routes/users');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use('/', index);
app.use('/users', users);

// catch 404 and forward to error handler
app.use(function (req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handler
app.use(function (err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```

开头是导入相关依赖，然后通过调用 `express()` 初始化 express 实例，接着我们设置了模板引擎为 `ejs`，以及模板引擎的存放目录，然后就是一系列中间件的加载使用，最后导出 express 实例，丢给  `bin/www` 脚本进行调用并启动服务器。

{% note info %}
**提示**

这里我们可以看到，我们导入的两个路由 `index` 和 `users`，也和其他中间件一样被处理，所以在 Express 中 “一切皆中间件”。
{% endnote %}

让我们再来看一看我们的路由部分 `routes/index.js`，路由是我们 API 服务器的核心，我们对数据进行增删改查都需要访问特定的路由接口，我们在整个教程中几乎都是围绕路由的操作。

```js routes/index.js https://github.com/pftom/vue-online-shop-backend/blob/7c749c5/routes/index.js 查看完整代码
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function (req, res, next) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

上面的代码，首先导入 `express`，然后使用其属性方法生成了一个  `router` 实例，接着定义了 `get` 这一 HTTP 方法来处理以 `GET` 方法访问我们服务器地址为 `/` 时如何进行处理，最后导出我们的 `index` 路由。

我们的 API 服务器实际上就是通过 HTTP 的各种方法（POST、DELETE、PUT、GET 等）访问我们定义的路由，进而对数据库进行相应的增删改查操作以获取我们期望的数据。

### 小结

通过简单的讲解 `express-generator` 脚手架为我们生成的上面四个文件，我们学到了如下知识：

- 在 Express 中，一切皆中间件（Middlewares），我们通过组合中间件来处理复制的后端逻辑。
- 我们的 API 服务器实际上就是通过定义一系列路由，当以不同的 HTTP 方法访问这些路由接口时，对数据进行对应的增删改查操作。
- 虽然 Express 也可以通过模板引擎展示用户界面，但是由于我们的迷你电商应用的前端已经用 Vue 来实现了，所以不需要模板引擎。

## 接入 MongoDB 数据库

解决数据持久化存储最流行的方案无疑是数据库，而 MongoDB 凭借其优异的性能、可扩展性和灵活的数据模式，从众多数据库产品中脱颖而出。并且，MongoDB 的核心功能是基于 BSON（Binary JSON）实现的，甚至提供了 JavaScript Shell，因此在 Node 社区更是深受欢迎。MongoDB 可以从其[官网](https://www.mongodb.com/download-center/community)上下载。下载并安装好之后，新打开一个终端（命令控制台），运行以下命令打开数据库（Windows 用户可以搜索 mongo.exe 并打开）：

```Bash
$ mongod
2019-12-22T18:10:25.285+0800 I CONTROL  [initandlisten] MongoDB starting : pid=14475 port=27017 dbpath=/data/db 64-bit host=mRc
2019-12-22T18:10:25.285+0800 I CONTROL  [initandlisten] db version v3.6.0
2019-12-22T18:10:25.285+0800 I CONTROL  [initandlisten] git version: a57d8e71e6998a2d0afde7edc11bd23e5661c915
...
```

开启 `mongod` 后会输出很多日志信息，并且可以通过 `localhost:27017` 进行访问。然后回到我们之前开启的终端，安装 Mongoose 这个 npm 包：

```Bash
$ npm install mongoose
```

Mongoose 是 MongoDB 最流行的 ODM（Object Document Mapping，对象文档映射），使用起来要比底层的 MongoDB Node 驱动更方便。

接着我们在我们的 `app.js` 文件中导入 `mongoose` ，并且通过 `mongoose` 提供的接口连接我们的 MongoDB 数据库：

```js app.js https://github.com/pftom/vue-online-shop-backend/blob/e5edda0/app.js 查看完整代码
var express = require('express');
var path = require('path');
[tuture-del]var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
[tuture-add]const mongoose = require('mongoose');

var index = require('./routes/index');
var users = require('./routes/users');
 // ...
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

[tuture-add]// Datbase connection here
[tuture-add]
[tuture-add]mongoose.connect(`mongodb://localhost:27017/test`);
[tuture-add]
[tuture-add]
// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
// ...
```

接着我们通过 `npm start` 运行服务器，我们就在 Express 中连接上了我们的 MongoDB 数据库，虽然现在还看不到任何效果，我们马上会编写路由来操作数据库来测试连接的有效性。

## 允许资源跨域访问

接着我们要做一点额外的操作，尽管它看起来和我们的项目没什么关联性，但是确是一个必要的一环，那就是开启资源跨域访问 CORS （Cross-Origin Resources Sharing）。CORS 是用来限制此域名下的资源访问解决方案，当它关闭时，另外一个域名访问此域名的资源时会被拒绝。如果想详细了解什么是 CORS，这里推荐一篇[阮一峰](http://www.ruanyifeng.com/blog/2016/04/cors.html)的文章，里面很细致的讲解了 CORS 的原理。

我们打开 `app.js` 文件，添加如下代码：

```js app.js https://github.com/pftom/vue-online-shop-backend/blob/2014dde/app.js 查看完整代码
// ...
app.set('view engine', 'ejs');

// Datbase connection here
[tuture-del]
mongoose.connect(`mongodb://localhost:27017/test`);

[tuture-add]// CORS config here
[tuture-add]app.all('/*', function(req, res, next) {
[tuture-add]  // CORS headers
[tuture-add]  res.header("Access-Control-Allow-Origin", "*"); // restrict it to the required domain
[tuture-add]  res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS');
[tuture-add]  // Set custom headers for CORS
[tuture-add]  res.header('Access-Control-Allow-Headers', 'Content-type,Accept,X-Access-Token,X-Key');
[tuture-add]  if (req.method == 'OPTIONS') {
[tuture-add]    res.status(200).end();
[tuture-add]  } else {
[tuture-add]    next();
[tuture-add]  }
[tuture-add]});
[tuture-add]

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
// ...
```

通常意义上，上面的代码存在很多问题，我们一般意义上会使用 [NPM](https://www.npmjs.com/package/cors) 包 `cors` 来解决，当然我们这里使用了比较简单粗暴的方式。

## 设计数据库的 Schemas 和 Models

我们要在服务器中通过 mongoose 与 MongoDB 数据库进行交互，需要定义 `Schema` 和 `Model`。通过定义它们来告诉 mongoose 你需要的数据结构和对应的数据类型是什么。

我们来创建 `model/index.js` 文件编写我们的 `Schema` 。

```js model/index.js https://github.com/pftom/vue-online-shop-backend/blob/02de67f/model/index.js 查看完整代码
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const model = mongoose.model.bind(mongoose);
const ObjectId = mongoose.Schema.Types.ObjectId;

const productSchema = Schema({
  id: ObjectId,
  name: String,
  image: String,
  price: Number,
  description: String,
  manufacturer: { type: ObjectId, ref: 'Manufacturer' },
});

const manufacturerSchema = Schema({
  id: ObjectId,
  name: String,
});

const Product = model('Product', productSchema);
const Manufacturer = model('Manufacturer', manufacturerSchema);

module.exports = { Product, Manufacturer };
```

`Schema` 接收一个 JavaScript 对象来描述我们需要的数据结构和对应的数据类型，除了我们熟知的像 `String`、`Number` 等数据类型外，`ObjectId` 是一个特殊的数据类型，我们用它来定义我们的单个 MongoDB 文档的主键，用于标志存储数据的唯一性。

我们还可以看到，在我们的 `productSchema` 中，`manufacturer` 数据结构我们定义了一个 `ref` 属性，这是 MongoDB 为我们提供的类似关系数据库的外键功能，允许我们创建一对多的数据文档，所以 `productSchema` 的 `manufacturer` 属性对应着的数据类型为一条 `Manufacturer` 记录。

接着我们通过 `model` 来创建对于的数据模型，然后导出我们创建好的数据模型。这里的 `model` 就是经典的 MVC 设计模式中的 **Model**。

## 完成 API 路由

路由是 Express 的关键组成部分，也是客户端与服务器进行交互的入口，在 Express 路由中接受两个参数：Request 和 Response，一个用来获取客户端的请求，一个用来发送给客户端服务器的响应。

打开 `app.js` 文件，加入如下代码：

```js app.js https://github.com/pftom/vue-online-shop-backend/blob/95e9a4b/app.js 查看完整代码
// ...

var index = require('./routes/index');
var users = require('./routes/users');
[tuture-add]const api = require('./routes/api');

var app = express();

// ...

app.use('/', index);
app.use('/users', users);
[tuture-add]app.use('/api/v1', api);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  // ...
```

可以看到，我们导入了 `api` 路由，并定义了访问路径 `/api/v1`。所有访问 `/api/v1` 及其子路径如 `/api/v1/xxx` 都会激活 `api` 处理函数，在经典的 MVC 设计模式中，`api` 也被成为 **Controllers** 。

接着我们编写 `api` Controllers，在这里面定义操作商品和制造商的路由接口，这里我们将采用经典的 [RESTful API ](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)来编写我们的路由接口：

```js routes/api/index.js https://github.com/pftom/vue-online-shop-backend/blob/95e9a4b/routes/api/index.js 查看完整代码
const express = require('express');
const router = express.Router();
const productController = require('../../controllers/product');
const manufacturerController = require('../../controllers/manufacturer');

router.get('/manufacturers', manufacturerController.all);
router.get('/manufacturers/:id', manufacturerController.byId);
router.post('/manufacturers', manufacturerController.create);
router.put('/manufacturers/:id', manufacturerController.update);
router.delete('/manufacturers/:id', manufacturerController.remove);

router.get('/products', productController.all);
router.get('/products/:id', productController.byId);
router.post('/products', productController.create);
router.put('/products/:id', productController.update);
router.delete('/products/:id', productController.remove);

module.exports = router;
```

可以看到，我们将 `index` Controller 里面导入了我们的 `productController` 和 `manufacturerController` 。然后定义了一系列路由。

这里操作 `manufacturer` 的前五个路由的功能如下：

- `GET /manufacturers` 获取所以的制造商（manufacturers）
- `GET /manufacturers/:id` 获取单个制造商，这里 `:id` 代表动态路由，用于匹配任意字符串：`/manufacturers/<any-string>`。
- `POST /manufacturers` 用户创建单个制造商
- `PUT /manufacturers/:id` 用于修改单个制造商
- `DELETE /manufacturers/:id` 用于删除单个制造商

对应的 `product` 的五个路由功能如下：

- `GET /products` 获取所以的产商品（products）
- `GET /products/:id` 获取单个商品，这里 `:id` 代表动态路由，用于匹配任意字符串：`/products/<any-string>`。
- `POST /products` 用户创建单个商品
- `PUT /products/:id` 用于修改单个商品
- `DELETE /products/:id` 用于删除单个商品

最后我们导出我们的路由。

接下来我们来看一看具体的 `manufacturer` Controller。

```js controllers/manufacturer.js https://github.com/pftom/vue-online-shop-backend/blob/95e9a4b/controllers/manufacturer.js 查看完整代码
const Model = require('../model');
const { Manufacturer } = Model;

const manufacturerController = {
  all(req, res) {
    Manufacturer.find({}).exec((err, manfacturers) => res.json(manfacturers));
  },
  byId(req, res) {
    const idParams = req.params.id;

    Manufacturer.findOne({ _id: idParams }).exec((err, manufacturer) =>
      res.json(manufacturer),
    );
  },
  create(req, res) {
    const requestBody = req.body;
    const newManufacturer = new Manufacturer(requestBody);

    newManufacturer.save((err, saved) => {
      Manufacturer.findOne({ _id: newManufacturer._id }).exec(
        (err, manfacturer) => res.json(manfacturer),
      );
    });
  },
  update(req, res) {
    const idParams = req.params.id;
    let manufacturer = req.body;

    Manufacturer.updateOne(
      { _id: idParams },
      { ...manufacturer },
      (err, updated) => {
        res.json(updated);
      },
    );
  },
  remove(req, res) {
    const idParams = req.params.id;

    Manufacturer.findOne({ _id: idParams }).remove((err, removed) =>
      res.json(idParams),
    );
  },
};

module.exports = manufacturerController;
```

可以看到我们定义了一个 `manufacturerController` 对象，用来组织一系列对 `manufacturer` 进行增删改查的操作。

我们在开头导入了我们之前定义的 `ManufacturerModel`，这是 Mongoose 为我们提供的操作数据库的接口，我们通过定义在 Model 上的一系列如 find、findOne、updateOne、deleteOne 执行我们对数据的增删改成操作。

最后是我们的 `product` Controller ，它内部的操作和我们上面讲到的 `manufacturer` Controller 基本一致。

```js controllers/product.js https://github.com/pftom/vue-online-shop-backend/blob/95e9a4b/controllers/product.js 查看完整代码
const Model = require('../model');
const { Product } = Model;

const productController = {
  all(req, res) {
    Product.find({})
      .populate('manufacturer')
      .exec((err, products) => res.json(products));
  },
  byId(req, res) {
    const idParams = req.params.id;

    Product.findOne({ _id: idParams })
      .populate('manufacturer')
      .exec((err, product) => res.json(product));
  },
  create(req, res) {
    const requestBody = req.body;
    const newProduct = new Product(requestBody);

    newProduct.save((err, saved) => {
      Product.findOne({ _id: newProduct._id })
        .populate('manufacturer')
        .exec((err, product) => res.json(product));
    });
  },
  update(req, res) {
    const idParams = req.params.id;
    const product = req.body;

    console.log('idParams', idParams);
    console.log('product', product);

    Product.updateOne({ _id: idParams }, { ...product }, (err, updated) => {
      res.json(updated);
    });
  },
  remove(req, res) {
    const idParams = req.params.id;

    Product.findOne({ _id: idParams }).remove((err, removed) =>
      res.json(idParams),
    );
  },
};

module.exports = productController;
```

编写完上面的代码并保存，打开终端输入 `npm start` 来开启我们的服务器。

因为我们的服务器在开启时要连接 MongoDB 数据库，所以要确保本地的 MongoDB 数据库已经开启，我们可以通过如下命令来开启：

```Bash
$ mongod
```

好了，现在我们的 API 服务器就搭建完成了，现在我们通过 API 测试工具 [POSTman](https://www.getpostman.com/) 来测试一下我们 API 是否成功。

测试 `GET /api/v1/manufacturers`：

![](https://static.powerformer.com/c/cb08dc8/89b9a7511314e56444db9f04e1fc8aa6.png)

测试 `POST /api/v1/manufacturers`：我们添加手机制造商 `"一加"`

![](https://static.powerformer.com/c/cb08dc8/b739bcedeac40735ce6cb7e96437d7be.png)

测试 `PUT /api/v1/manufacturers/:id`：这里我们把 `"一加"` 改成 `"One Plus"`

![](https://static.powerformer.com/c/cb08dc8/1b0f5b9135b8a0b95381157cef9df74d.png)

![](https://static.powerformer.com/c/cb08dc8/9ece79248ed36c94cfaa73d7fcac0fb1.png)

测试 `DELETE /api/v1/manufacturers/:id`：我们把刚刚添加的 `"一加"` 删掉

![](https://static.powerformer.com/c/cb08dc8/14335c49c2896d7b6c9ed4e218aebfff.png)

![](https://static.powerformer.com/c/cb08dc8/3c3a095b9887842a59c29507e366ce53.png)

最后测试添加商品 `product`，`POST /api/v1/products`：这里我们在定义 `product` 的数据属性时，加入了 `Manufacturer` 作为外键，所以创建的时候对应的 `manufacturer` 属性要为某个 `Manufacturer` 的 `ObjectId`，比如我们这里添加小米的新产品 `Mix Alpha` ：

![](https://static.powerformer.com/c/cb08dc8/c27e38b5de01eb2cc113d718fd41608c.png)

可以看到我们添加了 `manufacturer` 为 `5da72eaac6ccea32f823c247` 的小米制造商的新手机 `"Mix Alpha"`。

### 小结

自此，我们的 API 服务器就搭建完成了，在这篇教程里面我们学到了如下知识：

- 了解 Express 的路由以及如何用 mongoose 连接 MongoDB 数据库
- 编写路由、Model 和 Controllers
- 使用 POSTman 来测试我们编写的 API

相信通过本篇教程的学习，你对使用 Node 和 Express 编写 API 后端服务器有了一个基本的了解，现在我们了解了 Vue 基础知识，了解了如何搭建后端服务器，接下来我们将考虑如何使用 Vue 构建大型应用，下一篇教程我们再见！
