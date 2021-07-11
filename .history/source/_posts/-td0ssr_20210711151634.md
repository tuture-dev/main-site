---
title: '从零到部署：用 Vue 和 Express 实现迷你全栈电商应用（八）'
description: '本篇是系列文章终篇，我们将实现应用的部署，这篇教程将首先 Docker 来容器化你的应用，接着教你配置 MongoDB 的身份验证机制，给你的数据库添加一份安全守护，最后我们会带你使用阿里云的容器镜像服务将整个全栈应用部署到云端，使得你互联网上的用户可以访问你的网站，希望这篇教程能解决长期困扰你的部署上云的问题！'
tags: ['Docker', 'Docker Compose']
categories: ['前端', 'Vue', '入门']
date: 2020-03-14T03:10:55.150Z
photos:
  - https://static.powerformer.com/c/092faf4/vue-cover-8-squashed.jpg
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

## 应用容器化和 Docker Compose 配置

首先，如果你是一路跟着前面七篇教程一路敲过来的，那么将整个 Vue 前端项目放到新创建的 `client` 目录中，把整个 Express 后端项目放到新创建的 `server` 目录。如果你打算直接从这一篇开始学习部署，可以通过直接下载我们提供的代码：

```bash
git clone -b deploy-start https://github.com/tuture-dev/vue-online-shop-frontend.git
```

在正式开始整个全栈应用的容器化之前，让我们通过一张图来梳理一下思路：

![](https://static.powerformer.com/c/092faf4/170e3df21624b4e9.png)

可以看到，我们将使用三个容器：

- `nginx` 容器包括了 Nginx 服务器（存放了 Vue 框架实现的前端静态页面）
- `api` 容器则包括了我们用 Express 框架实现的 API 服务器
- `db` 容器则是 MongoDB 数据库

我们将整个应用通过 Nginx 实现反向代理。也就是说，用户想要访问我们的应用，必须得先经过 Nginx。并且，所有获取前端资源的请求（例如 HTML、CSS、JS 等静态文件资源），Nginx 可以直接返回；所有获取 API 端点的请求（例如 `/api/v1/products` ），则将请求转交给给 API 服务器，然后再将 API 服务器返回的 JSON 数据返回给用户。

这种经典的架构有以下优势：

- 通过 Nginx 可以实现访问控制，过滤掉不合法的请求
- 解决了前后端跨域的问题，因为前端页面和后端 API 都通过同一个端点访问
- 整个应用架构对用户透明，可以轻松进行配置扩容，并且 Nginx 搭配了负载均衡

### 前端应用的容器化

首先，让我们来容器化之前用 Vue 完成的前端应用。进入 `client` 目录，然后把 Vue 前端项目构建成静态页面：

```bash
npm run build
# 或者 yarn build
```

然后添加 `client/config/nginx.conf` 配置文件，代码如下：

```conf client/config/nginx.conf https://github.com/tuture-dev/vue-online-shop-frontend/blob/40e98722ac395db0546ac896d7497313e46081a1/client/config/nginx.conf 查看完整代码
server {
    listen 80;
    root /www;
    index index.html;
    sendfile on;
    sendfile_max_chunk 1M;
    tcp_nopush on;
    gzip_static on;

    location /api/v1 {
      proxy_pass http://api:3000;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

其中需要关注的就是两条 `location` 规则：

- 如果访问 `/api/v1` ，那么一律把请求传递给 `api` 容器
- 如果访问 `/` ，则直接返回前端静态页面（index.html）

然后在前端访问后端的代码中，我们需要做一点改变。打开 `client/src/store/actions.js` 文件，修改 `API_BASE` 常量如下：

```js client/src/store/actions.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/40e98722ac395db0546ac896d7497313e46081a1/client/src/store/actions.js 查看完整代码
// ...
import { Message } from 'element-ui';

[tuture-del]const API_BASE = 'http://localhost:3000/api/v1';
[tuture-add]const API_BASE = '/api/v1';

export const productActions = {
  // ...
};

export const manufacturerActions = {
  // ...
}
```

这样修改后，前端实际访问的 API 就取决于当前该页面的 URL，而不是硬编码的 `localhost:3000` 。

在做好准备工作后，我们就要正式开始容器化了。

{% note info %}
**提示**

如果你对 Docker 的核心概念不太熟悉，推荐学习一波我们图雀社区的[《一杯茶的时间，上手 Docker》](https://tuture.co/2020/01/01/442cc8d/)，帮助你快速掌握**镜像**和**容器**这两个重要概念，并手把手带你容器化第一个应用。
{% endnote %}

在 `client` 目录下创建 `Dockerfile` 文件，代码如下：

```Dockerfile client/Dockerfile https://github.com/tuture-dev/vue-online-shop-frontend/blob/40e98722ac395db0546ac896d7497313e46081a1/client/Dockerfile 查看完整代码
FROM nginx:1.13

# 删除 Nginx 的默认配置
RUN rm /etc/nginx/conf.d/default.conf

# 添加自定义 Nginx 配置
COPY config/nginx.conf /etc/nginx/conf.d/

# 将前端静态文件拷贝到容器的 /www 目录下
COPY dist /www
```

创建 `client/.dockerignore` 文件，确保在构建镜像时忽略掉 `node_modules` ：

```dockerignore client/.dockerignore https://github.com/tuture-dev/vue-online-shop-frontend/blob/40e98722ac395db0546ac896d7497313e46081a1/client/.dockerignore 查看完整代码
node_modules
```

### 后端应用的容器化

容器化前端应用之后，接下来就开始准备后端应用的容器化。首先我们要把硬编码的 MongoDB 连接字符串改成通过环境变量注入。修改 `server/app.js` 文件，代码如下：

```js server/app.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/40e98722ac395db0546ac896d7497313e46081a1/server/app.js 查看完整代码
// ...
// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

// Datbase connection here
[tuture-del]mongoose.connect(`mongodb://localhost:27017/test`);
[tuture-add]mongoose.connect(process.env.MONGO_URI || `mongodb://localhost:27017/test`);

// ...
```

接着创建 `server/Dockerfile` ，代码如下：

```Dockerfile server/Dockerfile https://github.com/tuture-dev/vue-online-shop-frontend/blob/40e98722ac395db0546ac896d7497313e46081a1/server/Dockerfile 查看完整代码
FROM node:10

# 指定工作目录为 /usr/src/app，接下来的命令全部在这个目录下操作
WORKDIR /usr/src/app

# 将 package.json 拷贝到工作目录
COPY package.json .

# 安装 npm 依赖
RUN npm config set registry https://registry.npm.taobao.org && npm install

# 拷贝源代码
COPY . .

# 设置环境变量
ENV NODE_ENV=production
ENV MONGO_URI=mongodb://db:27017/test
ENV HOST=0.0.0.0
ENV PORT=3000

# 开放 3000 端口
EXPOSE 3000

# 设置镜像运行命令
CMD [ "node", "./bin/www" ]
```

和前端一样，创建 `server/.dockerignore` 文件，确保 `node_modules` 不会被打包到镜像中去：

```dockerignore server/.dockerignore https://github.com/tuture-dev/vue-online-shop-frontend/blob/40e98722ac395db0546ac896d7497313e46081a1/server/.dockerignore 查看完整代码
node_modules
```

### Docker Compose 配置

[Docker Compose](https://docs.docker.com/compose/) 是一个强大的多容器管理工具，通过一个 YAML 文件配置完成后，只需要一个命令就可以启动全部容器（服务）。在项目根目录创建 `docker-compose.yml` ，代码如下：

```yml docker-compose.yml https://github.com/tuture-dev/vue-online-shop-frontend/blob/40e98722ac395db0546ac896d7497313e46081a1/docker-compose.yml 查看完整代码
version: '3'

services:
  db:
    image: mongo
    restart: always
  api:
    build: server
    restart: always
  nginx:
    build: client
    restart: always
    ports:
      - 8080:80
```

可以看到，我们创建了三个 `service` ，对应我们的三个容器（`db` ，`api` 和 `nginx` ）：

- `db` 服务指定镜像为 `mongo` ，然后设置 `restart: always` ，确保因某种原因停止后自动重启
- `api` 服务指定镜像通过 `server` 目录构建，端口映射规则为 `3000:3000`
- `nginx` 服务指定镜像通过 `client` 目录构建，端口映射规则为 `8080:80`

{% note warning %}
**注意**

在指定每个 service 时，如果使用 `image` 字段指定镜像，那么就会直接从镜像仓库拉取该镜像，这里我们的 `db` 服务就是如此。如果使用 `build` 字段指定镜像，则会根据指定的目录下的 `Dockerfile` 文件构建镜像，例如这里我们指定 `server` 和 `client` 目录分别构建 `api` 和 `nginx`。
{% endnote %}

{% note info %}
**提示**

Docker Compose 默认为所有服务创建了一个 Docker 网络，使得容器之间可以通过服务发现的机制进行相互通信（而不是通过固定 IP），这也就是为什么在 Nginx 配置中可以直接指定 `http://api:3000` ，以及将 MongoDB 连接字符串设置为 `mongodb://db:27017/test` 。如果想要透彻理解 Docker 网络，可浏览之前发布的[《Docker 筑梦师系列（一）：实现容器互联》](https://tuture.co/2020/01/12/cd44c84/)。
{% endnote %}

一切就绪，我们在电商根目录下通过一个命令实现整个应用的构建 + 运行：

```bash
docker-compose up --build
```

初次构建可能需要相当久的时间（拉取基础镜像），这时候不妨给自己点一杯咖啡 ☕️，静静等待结果的到来~ 如果你在控制台看到了如下输出，就代表镜像已经构建完毕了：

![](https://static.powerformer.com/c/092faf4/170e3df21661b0dd.png)

接着每个镜像会输出各自的日志信息。我们通过 `docker ps` 命令进一步确认三个容器的状态：

![](https://static.powerformer.com/c/092faf4/170e3df219c9c305.png)

OK，我们可以通过 `localhost:8080` 访问我们的站点了！

并且，我们还通过内网（例如同一 WiFi 下的其他设备）访问我们的站点，通过查询本机的内网 IP（例如 `192.168.1.1` ），然后在手机的浏览器里面输入这个 IP 地址，就可以通过 `192.168.1.1:8080` 访问。查询本机内网 IP 可以自行去搜索引擎找答案哦。

### 小结

在这一小节中，我们学习了：

- 通过 Nginx 容器提供前端静态页面，并将后端请求转发给 API 容器
- 容器化后端应用，建立与数据库的连接
- 通过 Docker Compose 一键构建和启动应用

## 配置 MongoDB 的身份验证

在之前的部署配置中，有一个重大的安全隐患：我们的 MongoDB 数据库没有配置任何的身份验证措施，这意味着所有能够访问数据库的请求都可以对数据库作出任何修改！接下来，我们就来搞定 MongoDB 的身份验证，为我们的数据安全保驾护航。

### 修改 MongoDB 连接设置

首先，我们修改 `server/app.js` 中的 MongoDB 连接设置，代码如下：

```js server/app.js https://github.com/tuture-dev/vue-online-shop-frontend/blob/cc4d59c976050064882d1e08f448c4525d35b2dc/server/app.js 查看完整代码
// ...

// Datbase connection here
[tuture-del]mongoose.connect(process.env.MONGO_URI || `mongodb://localhost:27017/test`);
[tuture-add]mongoose.connect(process.env.MONGO_URI || `mongodb://localhost:27017/test`, {
[tuture-add]  useNewUrlParser: true,
[tuture-add]  useUnifiedTopology: true,
[tuture-add]  user: process.env.MONGO_USER,
[tuture-add]  pass: process.env.MONGO_PASSWORD,
[tuture-add]});

// ...
```

四个选项的含义分别如下：

- `useNewUrlParser` ：使用新的 MongoDB 驱动 URL 解析器
- `useUnifiedTopology` ：使用新的连接管理引擎，能够大大提高连接的稳定性，支持重连
- `user` ：连接用户名，通过环境变量注入
- `pass` ：连接密码，通过环境变量注入

### Dockerfile 中注入环境变量

然后在 `server/Dockerfile` 中加入这些环境变量：

```Dockerfile server/Dockerfile https://github.com/tuture-dev/vue-online-shop-frontend/blob/cc4d59c976050064882d1e08f448c4525d35b2dc/server/Dockerfile 查看完整代码
// ...

# 设置环境变量
ENV NODE_ENV=production
[tuture-del]ENV MONGO_URI=mongodb://db:27017/test
[tuture-add]ENV MONGO_URI=mongodb://db:27017/admin
[tuture-add]ENV MONGO_USER=mongoadmin
[tuture-add]ENV MONGO_PASSWORD=secret
ENV HOST=0.0.0.0
ENV PORT=3000

// ...
```

注意到我们调整了 `MONGO_URI` ，把数据库从 `test` 设置为默认生成的 `admin` ，这是为了使用 `admin` 作为[鉴权数据库](https://docs.mongodb.com/manual/core/security-users/#user-authentication-database)（Authentication Database）。

### Docker Compose 中配置初始密码

最后在 `docker-compose.yml` 里面为 `db` 服务添加初始密码环境变量：

```yml docker-compose.yml https://github.com/tuture-dev/vue-online-shop-frontend/blob/cc4d59c976050064882d1e08f448c4525d35b2dc/docker-compose.yml 查看完整代码
// ...
  db:
    image: mongo
    restart: always
[tuture-add]    environment:
[tuture-add]      MONGO_INITDB_ROOT_USERNAME: mongoadmin
[tuture-add]      MONGO_INITDB_ROOT_PASSWORD: secret
  api:
    build: server
    restart: always
  // ...
```

OK，一切就绪，首先把之前创建的容器群彻底删除：

```bash
docker-compose down --volumes
```

`down` 命令的含义就和之前的 `up` 刚好相反，把 `up` 创建的所有镜像、容器、网络、数据卷全部删除，并且我们通过 `--volumes` 参数删除 MongoDB 容器创建的数据卷。

{% note warning %}
**注意**

如果不把之前 MongoDB 容器的数据卷删干净，接下来创建带有身份验证的 MongoDB 容器就会复用之前的数据卷，直接跳过初始化用户的过程（笔者在这个地方踩了接近两个小时的坑）。如果你担心数据卷还没删干净，可以运行 `docker volume prune` 。
{% endnote %}

然后重新构建并开启容器群：

```bash
docker-compose up --build
```

这时候再检查我们的应用（访问 `localhost:8080` ），应该看到一切正常。不过一颗悬着的心终于放下了——这次我们的数据库不再处于“裸奔”状态了！

### 小结

这一节中，我们完整地实践了一波如何为 MongoDB 容器配备身份验证。不过平心而论，我们采用的方法还是相当原始的，把机密信息明文写在代码文件中。在大型的容器编排系统中（例如 Kubernetes 和 Docker Swarm），都集成了完善的、企业级的机密信息管理方案。由于这一系列教程的入门性质，我们就点到为止啦。

此外，我们也没有讲 MongoDB 数据库备份和恢复的细节，如果想要了解和学习，可以阅读我们之前的[《Docker 筑梦师系列（二）：上手容器数据管理》](https://tuture.co/2020/03/06/0X8ssR3/)。

## 使用阿里云镜像仓库服务

到了这一步，实际上我们已经可以轻松地进行应用部署了。通过 SSH（或其他方式）连接到远程主机后，然后运行以下命令：

```bash
# 把仓库抓下来
git clone https://github.com/tuture-dev/vue-online-shop-frontend.git
cd vue-online-shop-frontend

# 构建前端代码
cd client && npm install && npm run build && cd ..

# 通过 Docker Compose 启动所有容器，并进入守护态运行
docker-compose up -d --build
```

这个时候，通过远程主机的 IP（或域名）加上端口号（这里是 `8080` ，当然你可以在 `docker-compose.yml` 中自行修改 `nginx` 服务的端口配置）。例如我们远程主机的 IP 是 `1.2.3.4` ，那么就可以通过 `1.2.3.4:8080` 访问我们的网站啦！

实际上，我们还可以通过一种更高效的方式进行镜像的分发与部署——**云端的镜像仓库服务**。

### Docker Hub 和镜像命名规则

实际上，Docker 公司已经做了一个叫 [Docker Hub](https://hub.docker.com/) 的镜像仓库，提供了丰富的官方维护镜像，以及自定义镜像的存储和分发。我们在平时用的镜像（例如 `mongo` 、`nginx` 、`node` 等）都是 Docker Hub 上的官方镜像（或者是其他代理加速器）。

镜像的命名规则如下：

```
<registry_name>/<username>/<image_name>
```

其中：

- `registry_name` 代表镜像仓库的名称，如果省略的话就是 Docker Hub
- `username` 代表镜像仓库的用户名，如果和 `registry_name` 一起省略的话就是 Docker 官方镜像
- `image_name` 就是镜像名称

Docker Hub 虽说是官方出品，但实际上存在以下问题：

1. 免费用户支持 1 个私有镜像
2. 上传和拉取的速度在国内不稳定
3. 没有镜像安全扫描功能

而我们接下来要体验的阿里云镜像仓库服务则能很好地解决以上问题。

### 体验阿里云镜像仓库服务

首先让我们访问[阿里云](https://www.aliyun.com/)的官方网站，然后在产品列表中找到“镜像仓库服务”，点击开通。开通后进入控制台，创建镜像命名空间，如下图所示：

![](https://static.powerformer.com/c/092faf4/170e3df21636b67e.png)

名称随意填写，这里我们填的是 `vue-online-shop` 。创建后如下图所示：

![](https://static.powerformer.com/c/092faf4/170e3df21a7ca3b2.png)

创建好命名空间后，就可以为我们应用的每个镜像（除了 MongoDB 数据库镜像）创建相应的镜像仓库。点击“创建镜像仓库”按钮，如下图所示：

![](https://static.powerformer.com/c/092faf4/170e3df21cb0520a.jpeg)

第一步，填写镜像仓库相关信息：

![](https://static.powerformer.com/c/092faf4/170e3df248bd0cc9.jpeg)

第二步，选择代码源，这里我们选择“本地仓库”：

![](https://static.powerformer.com/c/092faf4/170e3df248d09a1e.jpeg)

创建好两个镜像仓库（`api` 和 `nginx` ）后，可以看到镜像列表如下：

![](https://static.powerformer.com/c/092faf4/170e3df24a00c61c.jpeg)

OK，然后点击单个仓库的“管理”按钮，按照指示进行镜像的上传。在这里我们贴一下示例代码（**实际操作时按自己控制台的指示说明为准**）：

```bash
# 登录阿里云镜像仓库，aliyunUser 改成自己的账户名
docker login --username=aliyunUser registry.cn-shanghai.aliyuncs.com

# 构建和推送 api 镜像
docker build -t registry.cn-shanghai.aliyuncs.com/vue-online-shop/api server
docker push registry.cn-shanghai.aliyuncs.com/vue-online-shop/api

# 构建和推送 nginx 镜像
docker build -t registry.cn-shanghai.aliyuncs.com/vue-online-shop/nginx client
docker push registry.cn-shanghai.aliyuncs.com/vue-online-shop/nginx
```

{% note info %}
**提示**

实际部署时，建议给每个镜像加上一个标签，推荐是当前 Git 提交的 Hash，例如：

```bash
docker build -t registry.cn-shanghai.aliyuncs.com/vue-online-shop/api:9ca500a server
```

{% endnote %}

在镜像推送完成后，我们把 `docker-compose.yml` 中的 `api` 和 `nginx` 服务改成使用云端镜像（_下面是我的镜像仓库地址，记得改成你自己的喔_）：

```yml docker-compose.yml https://github.com/tuture-dev/vue-online-shop-frontend/blob/b8afe6837849e25e1386f78c89786cce23159272/docker-compose.yml 查看完整代码
// ...
      MONGO_INITDB_ROOT_USERNAME: mongoadmin
      MONGO_INITDB_ROOT_PASSWORD: secret
  api:
[tuture-del]    build: server
[tuture-add]    image: registry.cn-shanghai.aliyuncs.com/vue-online-shop/api
    restart: always
  nginx:
[tuture-del]    build: client
[tuture-add]    image: registry.cn-shanghai.aliyuncs.com/vue-online-shop/nginx
    restart: always
    ports:
      - 8080:80
```

搞定之后，我们只需把这个 `docker-compose.yml` 文件放到远程主机上，然后在所在的目录开启 Docker Compose 容器群即可：

```bash
# 拉取所有镜像的最新版本
docker-compose pull

# 启动所有容器
docker-compose up -d
```

### 小结

在这一步中，我们：

- 首先了解了如何通过 Git 抓取代码的方式在远程主机上进行部署
- 然后了解了 Docker Hub 及镜像命名的规则，并分析了一波 Docker Hub 的缺陷
- 接着一步步带你体验和使用阿里云镜像仓库服务，轻松实现镜像的分发与部署

终于，我们的迷你全栈电商系列实战就到此为止啦 🎉🎉，感谢一路看下来始终不离不弃、热爱学习的你 😘！让我们在接下来更精彩的文章中再见 👋~
