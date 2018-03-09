# 项目开发

在项目开发中，效率是永恒不变的话题，项目会有如下几个阶段：

* 技术选型
* 项目开发
  * 环境搭建
  * 代码编写
  * 本地测试
* 部署上线

先来明确一下，环境搭建通常在搭建什么？

* 语言：node/php/java/python
* 框架：egg/express/koa/jfinal/django
* 数据库：redis/mongodb/mysql/postgresql

开发框架可以选用各种脚手架完成，例如 YO 或者是团队自有的 git 项目作为基础，在其上继续开发；语言根据当前项目的编程语言确定，但是这里涉及到多版本并存问题：Java OpenJDK8/Java JDK8/Node 7/Node 8/Node 9 并存，可考虑使用 docker 辅助；同理，数据库等配置以及搭建，依托 docker 可以实现数据隔离，亦是可选项。

## 开发实践

此处我们以国内关注量比较高的 CNode 社区项目 egg-cnode 为例，简单介绍开发环境中的 docker 使用。

**项目地址**：[https://github.com/cnodejs/egg-cnode](https://github.com/cnodejs/egg-cnode)

**目录结构**：

```
.
├── app
├── app.js
├── config
├── Dockerfile
├── docker-compose.dev.yml
├── docker-compose.yml
├── init.d
│   └── mongo
│       └── init.js
├── logs
├── LICENSE
├── node_modules
├── package.json
├── run
├── README.md
├── test
└── yarn.lock

7 directories, 8 files
```

**Docker 配置**：

* Dockerfile                             用于本地构建与测试
* docker-compose.yml          用于正式环境部署与提供与线上基本一致的本地发布环境
* docker-compose.dev.yml   用于启动开发环境
* init.d                                     用于初始化脚本（数据库/配置等）

该项目需要的数据库：

* redis
* mongodb

### 本地开发

我们使用 \`docker-compose.dev.yml\` 配置数据库：

```
version: '3'
services:
  redis:
    image: redis:3.2-alpine
    command: redis-server --appendonly yes --requirepass egg_cnode
    volumes:
      - egg-redis:/data
    networks:
      - docker_cnode
    ports:
      - 6379:6379

  mongodb:
    image: mongo:3.2
    restart: always
    environment:      
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=mongodb
      - MONGO_INITDB_DATABASE=egg_cnode
    volumes:
      - egg-mongo:/data/db
      - ./init.d/mongo/:/docker-entrypoint-initdb.d/
    networks:
      - docker_cnode
    ports:
      - 27017:27017

volumes:
  egg-mongo:
  egg-redis:

networks:
  docker_cnode:
    driver: bridge
```

此处我们引入了一个init.d目录，作用是初始化 \`mongodb\` 数据库并添加认证：

```
/* eslint-disable */

/**
 * 1. create custom user
 * 2. create collection (Before MongoDB can save your new database, a collection name must also be specified at the time of creation.)
 */
db.createUser({
  user: 'egg_cnode',
  pwd: 'egg_cnode',
  roles: [
    {
      role: 'readWrite',
      db: 'egg_cnode'
    }
  ]
})

db.egg_cnode.insert({
  egg_cnode: 'egg-cnode'
})
```

> Mysql 也可以使用同样的方式初始化数据库，该脚本只执行一次。

故而如果我们只需要使用数据库，可以在项目根目录执行：

```
# 启动
docker-compose -f docker-compose.dev.yml up -d

# 停止
docker-compose -f docker-compose.dev.yml down -v

# 停止（同时移除持久化数据）
docker-compose -f docker-compose.dev.yml down -v
```

接着就可以按照日常开发一样进行了，不使用时候停止（或者同时清除数据）。

> 此处暴露了端口，方便开发的同学使用对应的客户端连接。

### 本地测试/预发布环境/生产环境

此处我们将三种环境放在一起讲的原因是，使用 docker 带来的巨大优势是将环境一起打包，故而这几种环境其实是完全一致的，差别在于：

* 数据库
* 访问量
* 负载均衡（生产环境如使用 nginx/haproxy，此处亦可以模拟）

那么我们来看下 \`docker-compose.yml\` 配置：

```
version: '3'
services: 
  cnode:
    build:
      context: .
      dockerfile: Dockerfile
      # args:
      #   - env=value
    environment:
      - NODE_ENV=production
      - EGG_SERVER_ENV=prod
      - EGG_REDIS_DB=0
      - EGG_REDIS_HOST=redis
      - EGG_REDIS_PORT=6379
      - EGG_REDIS_PASSWORD=egg_cnode
      - EGG_MONGODB_URL=mongodb://egg_cnode:egg_cnode@mongodb:27017/egg_cnode
      - EGG_PASSPORT_GITHUB_CLIENT_ID=test
      - EGG_PASSPORT_GITHUB_CLIENT_SECRET=test
    depends_on:
      - redis
      - mongodb
    networks:
      - docker_cnode
    ports:
      - 7001:7001

  redis:
    image: redis:3.2-alpine
    command: redis-server --appendonly yes --requirepass egg_cnode
    volumes:
      - egg-redis:/data
    networks:
      - docker_cnode
    # ports:
    #   - 6379:6379

  mongodb:
    image: mongo:3.2
    restart: always
    environment:      
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=mongodb
      - MONGO_INITDB_DATABASE=egg_cnode
    volumes:
      - egg-mongo:/data/db
      - ./init.d/mongo:/docker-entrypoint-initdb.d
    networks:
      - docker_cnode
    ports:
      - 27017:27017

volumes:
  egg-mongo:
  egg-redis:

networks:
  docker_cnode:
    driver: bridge
```

细心的同学也许会发现，这里有点变化：

* redis 端口不再暴露，在生产环境端口异常宝贵，无特殊需求的端口，完全不需要指定
* mongodb 通常会使用外部数据库，这里是为了测试，暴露端口方便同学们连接

启动和上面一样的方法：

```
# 启动
docker-compose -f docker-compose.yml up -d

# 停止
docker-compose -f docker-compose.yml down -v

# 停止（同时移除持久化数据）
docker-compose -f docker-compose.yml down -v
```

> docker-compose 可以不使用 -f；
> 正式环境 cnode 镜像会自动构建，不用本地构建；

这样我们即可获得一个沙箱环境，数据隔离，环境隔离，同时也可以通过 dockerfile 指定 node 版本。

