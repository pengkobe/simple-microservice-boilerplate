# part2

## 25 微服务为什么要容器化

DevOps 事实上就是开发与运维的结合，运维遇到的问题很多情况下是系统环境和软件环境的不一致。

- 容器就像运行在宿主机上的另一个操作系统
- Docker 不仅可以打包应用程序本身，还可以打包应用程序的所有依赖，甚至操作系统

### 容器化实践

- 利用 Docker 镜像分层机制，共用相关环境，逐层打包
  - 基础环境层 (OS)
  - 运行环境层 (java)
  - Web 容器层 (tomcat)
  - 业务代码层

### 两个问题

- 测试和发布工作量提升
- 由于环境差异导致的复杂度提升

## 26 微服务容器化运维：镜像仓库和资源调度

容器化天生就是为微服务而生，

- Puppet，在使用容器用的是这个工具进行发布
- 私有化镜像仓库对大团队来说很有必要
  - 权限控制
    - 登录
    - 按项目划分（开发者、管理员、Guest）
  - 镜像同步
    - 主从复制
    - P2P
  - 高可用性
    - 实现多 IDC 部署
  - 资源调度，使用 Docker Machine 可以解决，但是不适合跑了一段时间业务的。往往需要开发一层 DCP
    - 物理机集群(12 核 16G --> 32 核 32 G)
    - 虚拟机集群（ 如基于 Open Stack ）
    - 公有云集群
    - 开始时可以借助 Ansible 等软件对机器进行配置分发进行基本软件安装与配置

## 27 微服务容器化运维：容器调度和服务编排

适用于成百上千机器的部署，开源的主要有：

- Docker Swarm，门槛低
- MesosPhere Mesos
- 谷歌 Kubernetes，门槛较高

需要解决的问题

- 主机过滤
  - 存活过滤
  - 硬件过滤，根据机器特性
- 调度策略 spread/binpack

### 服务编排

- 服务依赖，参考 docker-compose
- 服务发现
  - 基于 Ngnix reload，可惜延迟较大（10%），但是有在研究可行的解决方案，Nginx 与 Consul 一起研发( 见下篇 )
  - 基于注册中心
- 自动扩缩容

## 28 微服务容器化运维: 微博容器运维平台 DCP

- Harbor 为基础搭建了私有的镜像仓库，由于有用到阿里云，可以使用 Harbor 的主从镜像复制机制进行同步
- 要考虑白天黑夜的流量差，要考虑阿里云并发创建的限制
- Ansible 向所有主机下发配置软件，并通过 callback queue 写入 DB( 而非并发 )
- swarm --> Roam，基于 Swarm Manageer 以及 Swarm Client 以及 Consul 进行调研，具体参见图示
- 服务发现
  - HTTP 服务(Nginx)
  - Motan RPC，向注册中心注册服务
  - 自动扩缩容，使用 Config Watcher 监控，然后 CronTrigger 实现调度策略变更

## 29 微服务如何实现 DevOps

DevOps 是一种新型的业务开发模型，业务开发不仅需要负责业务的测试和线上发布等全生命周期。

- CI，持续集成，能自动进行代码检查、单元测试和打包部署到测试环境，进行集成测试和跑自动化测试用例
- CD，持续部署，能自动部署到生成环境进行集成测试，灰度发布，达到要求后自动部署到线上
  - 其实也叫做持续交付，只要达到能发布就行了，不需要自动发布，一般由人工发布

### 工具

- Jenkins
- GitLab
  - 持续集成，合并到 Develop 分支的 Merge Rquest 全部能测试通过
  - 持续交付 Develop 分支代码能够在生成环境测试通过， 并进行小流量灰度测试
  - 持续部署，合并 Develop 到 Master 分支
  - 使用 .gitlab-ci.yml

### 关键点

- 持续集成
  - 代码检查
  - 单元测试
  - 集成测试，使用 kubernetes 对集群进行管理，测试完成之后再回收
- 持续交付
  - 如何从线上生产环境中摘除节点
  - 如何观察服务是否正常，日志是个好办法
- 持续部署
  - 要考虑核心业务与非核心业务，有时候手动发布反而更好，风险更低

## 30 如何做好微服务容量规划

难点

- 容器服务众多
- 服务接口表现差异巨大
- 服务部署集群规模大小不同
- 服务依赖

### 容量评估

1. 选择适合压测指标，比如接口返回大于 1s 的接口比例不能高于 1%
2. 压测获取单机的最大容量

- 单机压测
  - 日志回放
  - TCP-Copy，流量拷贝
- 集群
  - 节点摘除增加单机流量
  - 按照请求返回速率机型分级加权统计
  - 实时获取集群运行负荷，单机获取提交，按集群统计分析

### 调度决策

使用水位线进行决策，分为安全线和致命线

- 扩容，先按 30% 进行
- 缩容，每隔 5 分钟，按 5、10、50、100 （%）进行缩容，每分钟采集，5个埋点3个满足才缩容

## 31 微服务多机房部署实践

### 解决几个问题

- 一切正常请求哪个？
- 数据如何同步
  - 通过消息队列或者 RPC 方式更新缓存，数据库还是使用 binlog 方式
- 数据如何保持一致性
  - 通过对账机制实现

## 32 微服务混合云部署实践

### 跨云服务数据同步

- 私有云公有云网络隔离，搭建双专线处理
- 数据库能否上云，取决于隐私性，一般来说，缓存上云，数据库在本地

### 跨云服务容器运维
