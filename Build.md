# 镜像构建

镜像构建是整个 workflow 中很重要的一个环节，通常我们将镜像构建分类两种类型：

* 直接构建
* 预处理构建

## 直接构建

通常 node/php/python 类型的项目无需预处理，可利用 dockerfile 直接构建。由于开发人员并不是都对 dockerfile 的书写比较熟悉，我们将常见的运行环境以及一些组件进行了容器化，并将之放在 github 上供有需要的同事使用。

* [https://github.com/ImplementsIO/docker-labs](https://github.com/ImplementsIO/docker-labs)

## 预处理构建

Java 中通常需要将项目进行打包成 war 包后（模块包托管于 Maven 仓库）再使用 dockerfile 进行构建；前端项目如 React 也需要一些预处理的工作；我们将此类项目称为 **预处理构建**，此类项目可以使用 Jenkins/CircleCI/TravisCI 进行预处理后构建。

* [基于Jenkins & Docker的CI/CD优化](https://zhuanlan.zhihu.com/p/27147870)
* [使用阿里云容器服务Jenkins 2.0实现持续集成之the tag you want篇](https://yq.aliyun.com/articles/72703)