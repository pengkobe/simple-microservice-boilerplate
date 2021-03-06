# 从 0 开始学微服务

## 从入门到放弃

- 团队能力可能会制约你们部门发展微服务架构
- 团队人数多起来之后，单体应用发布就会变得非常麻烦，忘记合并代码啦、更新依赖啦、打包啦
- 步骤为 `单体应用 --> 微服务架构 --> 容器化应用 --> DevOps`

## 1 到底什么是微服务

- 有些人盲目的引进了微服务，结果团队 hold 不住，造成服务不稳定
- 单体架构变大后，后期会有如下痛点
  - 部署效率低下
  - 团队协作开发成本高
  - 系统高可用性差。服务相互影响
  - 线上发布慢
- 啥是微服务
  - 拆分力度更细
  - 服务独立部署
  - 服务独立维护
  - 服务治理要求高
  
## 2 从单体应用走向服务化

- 单体应用内部的功能有可能会相互影响
- 服务化简单的讲就是将功能模块拆分单独部署单独运维
- 拆分方法是按照业务进行拆分或者按照公共且独立功能维度进行拆分

### 拆分的前置条件

- 服务如何定义，http 还是 rpc？
- 服务如何发布和订阅，注册中心？
- 服务如何监控，需要覆盖业务埋点、数据收集、数据处理、最后到数据展示的全链路
- 服务如何治理，可以设定性能阀值，超过过久就直接返回（熔断）
- 故障如何进行定位
- 建议每个人不要负责超过 3 个大服务

## 初探微服务

### 服务描述

- 描述方式，主要有 RESTful API( Swagger 进行管理)、XML 配置（RPC）以及 IDL 文件（ Thrift 和 gRPC ）三种

### 注册中心

- 服务者根据配置提供自己的信息
- 消费者根据配置获取服务
- 注册中心在服务新增或者销毁时，需要通知消费者
- 监控调用情况

### 服务框架

- 确定采用的协议
- 数据传输方式，多路复用还是单链接，是同步还是异步
- 数据压缩采用什么格式[JSON/JAVA 对象/Protobuf]

### 服务监控

- 指标收集
- 数据处理
- 数据展示

### 服务追踪

- 服务消费者发起请求前，必须提供 requestid
- 若服务提供者有再发起请求，那么在拼接上自己的 requestid，从而实现追踪

### 服务治理

保证在异常情况下，服务人仍然能够运行

- 单机故障，自动去除故障节点
- 单 IDC 故障，自动切换
- 依赖服务不可用，熔断

## 如何发布和引用微服务

还是选用上述的三种方式，如果服务大部分是基于 java 那么实用 xml 最好，跨语言就实用 IDL，需要对外就实用 RESTful api,但是 IDL 虽然跨语言，但是不适合字段多又多变的场景

- [Motan](https://www.cnblogs.com/hjcenry/p/5856933.html)

## 5 如何注册和发现微服务

### 注册中心原理

需要提供以下 API

- 服务注册/反注册接口
- 服务订阅/变更接口
- 心跳汇报接口

集群部署

- 选取 leader
- leader 负责数据更新操作

目录存储

- zookeeper 把他们叫作 znode
- znode 可以保存数据和子 znode

服务状态检测

- 其实就是通过唯一 sessionid 去进行判断

服务状态变更

- 通过 Watcher 机制，监听改变获取最新数据

白名单机制

- 用于权限控制

## 6 如何实现 RPC 远程服务调用

弄清楚四个问题

- 如何建立网络连接 http/socket
- 如何处理请求 BIO（同步阻塞）/NIO（同步非阻塞）/AIO（异步非阻塞）
- 采用什么传输协议
- 如何序列化和反序列化

## 7 如何监控微服务调用

### 监控内容

- 用户端监控
- 接口监控
- 资源监控
- 基础监控

### 指标

- 请求量
- 响应时间
- 错误率

### 维度

- 全局维度
- 分机房维度
- 单机维度
- 时间维度
- 核心维度(业务层面)

### 环节

1. 数据采集，包括主动上报和代理收集。注意这块也会影响系统本身的性能
2. 数据传输，UDP、kafka、
3. 数据处理，接口维度聚合、机器维度聚合（索引数据库和时序数据库进行存储）
4. 数据展示，曲线、饼状、格子等

## 8 如何追踪微服务调用

简单地讲就是如何定位错误

### 作用

- 优化性能瓶颈
- 优化链路调用
- 生成网络拓扑
- 透明数据传输

### 服务追踪系统原理

- Dapper 是鼻祖，其它都是衍生出来的，包括 Twitter 的 Zipkin，阿里的鹰眼，美团的 MTrace，首先，需要了解几个概念，traceId（请求）、spanId（调用顺序编号）、annotation（埋点）
- 数据采集，![https://tech.meituan.com/img/mt-mtrace/mtrace9.png](https://tech.meituan.com/img/mt-mtrace/mtrace9.png)
- 数据处理
  - 实时[OLTP(HBase)+Storm/Spark Streaming]
  - 离线[Hive+MapReduceSpark 批处理程序]
- 数据展示
  - 链路图
  - 拓扑图

## 9 微服务治理的手段有哪些

主要解决的几种情况

- 注册中心宕机
- 三者中任何两者之间的网络不通
- 服务提供者性能问题或者短时间不通（很常见）

两种管理手段

- 注册中心摘除不可访问节点（可能出现因网络问题全摘除的情况）
- 消费者摘除不可用节点（貌似更合理）

### 负载均衡

- 随机算法
- 最少活跃调用算法
- 一致性 Hash 算法（出现问题平摊到其它节点）

### 路由规则

主要用于灰度发布与多机房就近访问，配置方法

- 静态配置，存在消费者配置文件里
- 动态配置，即注册中心配置，通过同步实现

### 服务容错

- FailOver 失败自动切换，适合只读场景
- FailBack，根据失败详情觉得下一步操作
- FailFast，记录下日志就返回了，适用于非核心业务

## 10 Dubbo 框架里微服务组件

包括以下几方面内容，建议从 Dubbo 官网去去安装并搭建一个微服务框架

- 发布与引用，见代码
- 服务注册与发现
- 服务调用，四个步骤
- 服务监控
- 服务治理

## 11 服务发布与应用实践

这篇文章里建立将服务发布端的详细服务配置信息转移到服务引用端

## 12 如何将注册中心落地

通过在本地保存 Cluster Sign 值，然后订阅服务变化，可以实现服务更新。

### 几个问题

- 注册中心与消费者是多对多的关系
- 使用并行订阅服务
- 批量反注册服务
- 配置信息最好增量更新，否则会因为网络抖动引发网络风暴

## 13 开源服务注册中心如何选型

两种方案

- 应用内注册于发现，基于 SDK
- 应用外注册于发现(Consul:Registrator:Consul Tmplate)

需要满足

- 高可用性( 集群、多 IDC 部署)
- 数据一致性。CAP 理论实际上是无法同时满足的
  - CP 型注册中心，ZooKeeper 使用的方式，通过选用 leader 实现
  - AP，Eureka 服务器单独保存服务注册地址，可能出现数据不一致

## 14 开源 RPC 框架如何选型

主要分为跨语言平台和特定语言平台两类

- Spring Cloud 全家桶，自带熔断机制，日志分析等等
- Dubbo（Netty 作为通讯框架，支持 RMI/Hession/HTTP/Thrift 等协议）
- Motan（微博）
- TARS（腾讯）
- gRPC（效率高）
- Thrift（支持语言的 z 种类较多）

## 15 如何搭建一个可靠的监控系统

主流有两种方案

- ELK（各类 beats - logstash - elasticsearch - kibana） 为代表的集中式日志解决方案
- Graphite（Carbon:接收被监控节点连接 - Whisper:按时序采集 - Graphite-web），便于接入其它图形化监控系统
- TICK，SQL 查询功能强大
- Prometheus，适合云原生应用

## 16 如何搭建一套服务追踪系统

- OpenZipkin（Twitter），使用范围广、社区活跃
- PinPoint（Naver），精确得多，还能监控到数据库链路，且不用改动业务代码，其为注入式

## 17 如何识别服务节点是否存活

- Zookerper 采用的是注册中心摘除机制，服务消费者从注册中心拉取，但是网络抖动时会有问题
- 心跳开关保护机制，比如只给 10% 的节点回复变更
- 服务摘除机制，不能超过 20% 的节点

### 静态注册中心

实际上相当于配置中心，不过基本上已经够用了，只有在业务上线或者人工操作时才进行更新

## 18 如何使用负载均衡算法

- 随机算法
- 轮询算法
- 加权轮询算法
- 最小活跃连接算法
- 一致性 Hash 算法

考虑场景：节点多且性能差异大，列表经常变化，且有跨数据中心访问，又出现网络抖动，考虑使用**自适应最优选择算法**，即客户端获取服务端性能快照进行排序后再访问

## 19 如何使用服务路由

场景

- 分组调用，含有私有部署和公有云部署
- 灰度发布，即小规模发布
- 流量切换
- 读写分离

规则

- 条件路由
- 脚本路由

获取方式

- 本地配置
- 配置中心管理（最好）
- 动态下发，通过服务治理平台修改

## 20 服务端出现故障时如何应对

故障类型

- 集群故障，由代码问题或者流量高峰导致
  - 限流，给系统设置一个总的最大工作线程数和单个服务最大工作线程数
  - 降级，停止系统中的某些功能，分为三级进行降级
- 单 IDC 故障
  - 基于 DNS 解析流量切换
  - 基于 RPC 分组的流量切换
- 单机故障

  - 自动重启，注意比例不要超过 10%

## 21 服务调用失败时有哪些处理手段

- 超时，设置超时时间
- 重试
- 双发，可以使用备份请求，p999 为超时，p99 可以发起备份请求，最大重试比例设置为 15%
- 熔断，根据断路器状态进行请求( Closed/Open/Half Open),参考 netflix 的 Hystrix

## 22 如何管理服务配置

方式

- 本地配置，修改需要发布，显然比较麻烦
  - 当作代码
  - 抽离到单独的配置文件
- 配置中心（**只有在业务比较复杂的时候采用**），满足
  - 注册
  - 反注册
  - 查看
  - 变更订阅，根据 sign 值判断

应用场景

- 资源服务化，当 Memcached 缓存这些资源膨胀后就不能再本地配置了，而需要统一管理
- 业务动态降级
- 分组流量切换

选型

- Spring Cloud Config，配置存储在 git 中，需要手动刷新
- Disconf，百度采用，通过 Zookeeper 实现推送，也有统一的配置界面来管理
- Apollo，携程采用，支持 Java 和 .NET,http 长连接推送

## 23 如何搭建微服务治理平台？

### 基本功能

- 服务管理
  - 上下线
  - 节点添加/删除
  - 服务查询
  - 服务节点查询
- 服务治理
  - 限流
  - 降级
  - 切流量
- 服务监控
- 问题定位
- 日志查询
- 服务运维( 发布部署/扩缩容 )

### 如何搭建

- Web Portal， 服务管理/治理/监控/运维界面
- API
- DB， 用户权限/操作记录/元数据(通过标识把数据串联起来)

## 24 微服务架构该如何落地？

注册中心、配置中心、RPC 框架、监控系统、追踪系统、服务治理，每个组件都需要专门的专家才能 hold 住...只有架构师才适合做微服务架构的开发。

- 有经验的架构师 + 业务开发人员
- 从一个案例入手，从众多业务中找到一个小案例
- 做好技术取舍， ZooKeeper(HBase 依赖于 Zookeeper) 还是 redis？
- 采用 DevOps
- 统一微服务治理平台
