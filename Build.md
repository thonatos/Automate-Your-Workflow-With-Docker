# 镜像构建

镜像构建是整个 Workflow 中很重要的一个环节，构建后推送到镜像仓库供后续部署与测试使用。

## 直接构建

Node/PHP/Python 类型的项目通常无需预处理，可利用 Dockerfile 直接构建。

在项目中提供 Dockerfile，通过 Webhook 触发构建流程，构建后的项目会被推送到镜像仓库如：

* Docker Hub
* Aliyun Container Registry

> 在控制台能够看到自动构建的设置以及构建规则

由于开发人员并不是都对 Dockerfile 的书写比较熟悉，我们将常见的运行环境以及一些组件进行了容器化，并将之放在 github 上供有需要的同学使用。

* [https://github.com/ImplementsIO/docker-labs](https://github.com/ImplementsIO/docker-labs)

## 预处理构建

Java 项目通常需要打包成 War 包后（模块包托管于 Maven 仓库）再使用 Dockerfile 进行构建；前端项目如 React 也需要一些预处理的工作；我们将此类项目称为 **预处理构建**，此类项目可以使用 Jenkins/CircleCI/TravisCI 进行预处理后构建。

* [基于Jenkins & Docker的CI/CD优化](https://zhuanlan.zhihu.com/p/27147870)
* [使用阿里云容器服务Jenkins 2.0实现持续集成之the tag you want篇](https://yq.aliyun.com/articles/72703)

**其他链接**

* [Authenticate proxy with nginx](https://docs.docker.com/registry/recipes/nginx/#setting-things-up)
* [Docker Private Registry Notifications](https://zhuanlan.zhihu.com/p/30080610)
* [十分钟配置Docker镜像自动构建](https://zhuanlan.zhihu.com/p/24896056)
* [基于OSS搭建私有（跨区域）Docker镜像仓库](https://zhuanlan.zhihu.com/p/26953037)
* [基于OSS搭建跨区域部署的分布式Docker镜像仓库](https://yq.aliyun.com/articles/7585?spm=5176.100240.searchblog.29.KgPk6U)