# 从 0 开始学微服务

## 从入门到放弃

- 团队能力可能会制约你们部门发展微服务架构
- 团队人数多起来之后，单体应用发布就会变得非常麻烦，忘记合并代码啦、更新依赖啦、打包啦
- 步骤为 `单体应用 --> 微服务架构 --> 容器化应用 --> DevOps`

## 从单体应用走向服务化

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