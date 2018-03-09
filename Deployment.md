# 项目部署

基于 docker 的 workflow 中，项目部署是最后一步，目前只介绍基于阿里云容器服务的几种部署方式。


### 阿里云容器服务

* [Maidops](https://github.com/thonatos/maidops)
* [容器服务触发器](https://help.aliyun.com/document_detail/63192.html?spm=5176.11065259.1996646101.searchclickresult.56a625e9cCTsH6)
* [容器服务应用API](https://help.aliyun.com/document_detail/26072.html?spm=a2c4g.11186623.6.986.Uj4wo5)

### 其他

目前容器编排工具比较常见的有：

- Docker Swarm
- Kubernetes
- Other（Marathon/Nomad/ECS）

首当其冲的自然是 k8s/docker swarm，他们都提供了 http 接口，调用即可实现简单的 CURD 操作。