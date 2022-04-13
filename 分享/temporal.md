## temporal

### 前言

我们需要开发一个响应多个异步事件、跟踪非常复杂的任务的状态的应用程序，如果自己开发，那么保持每个单独组件的健康可能非常困难，通常须要在基础设施上进行大量投资，可视化整个系统的健康状况、定义超时和协调重试，但是扩展和维护这样的系统是一项具有挑战性且成本高昂的工作

Temporal 提供了一种无故障的有状态编程模型，它隐藏或抽象了上述大多数复杂性。

> ##### TEMPORAL AS A DISTRIBUTED SYSTEM
>
> In terms of the CAP theorem, each Temporal cluster is eventually available and highly consistent.
>
> - Because Temporal makes it easy to retry Activities and horizontally scale resources, availability loss doesn't result in a fault, but **in increased latency.**
> - Network failures are prevented from reaching the application level. If persistence nodes are lost or unreachable, your Workflows will not progress, but the data will still be highly consistent.
> - The optional [Multi-cluster Replication](https://docs.temporal.io/docs/server/multi-cluster) feature greatly increases system availability.

### 概念

- Activitys： 一段可运行代码（function），处理业务逻辑的方法，可以包含任何逻辑。由于Temporal提供了各种语言的SDK（go、java、python、php等等）所以Activity是不限制语言的。
- Workflows: Activity的集合，多个Activity可以构成一个Workflow。Temporal 的核心抽象就是故障无感知的有状态的工作流，但是由于确定性的执行要求，它们不允许直接调用任何外部 API, 外部调用均交给Activity调用. 更多的[限制](https://docs.temporal.io/docs/go/how-to-develop-a-workflow-definition-in-go)

- Workers: 真正执行Workflow和Activity代码的服务
- Signals: 信号是一种完全异步和持久的机制，用于将数据发送到正在运行的工作流中（而不是在启动工作流或在活动中轮询外部数据时将数据作为参数传递）。
- Queries: 查询工作流的信息,temporal内置了查询,比如堆栈消息查询,也可以自定义查询方法,返回当前任务的状态之类的.
- Task Queue: temporal服务会将Workerflow分发到不同的任务队列中,worker去取对应的任务执行.
- Temporal Server： 管理worker， 下发任务，监听任务状态
- Temporal SDK：任务发起者，基于sdk开发workflow和activity具体的实现
- web UI： 负责任务的监控、查询等。

- session: 可以绑定多个activity在同一个节点运行，满足有依赖关系的方法，比如文件处理。

### Temporal Server 

Temporal Server 由四个独立可扩展的服务组成：

- Frontend gateway：用于限速、路由、授权
- History subsystem：维护数据（可变状态、队列和定时器），跟踪workflow的执行状态
- Matching subsystem：负责托管任务队列以进行任务调度，将 Worker 与 Task 匹配并将新任务路由到适当的队列
- Worker service：用于执行后台workflows

History、Matching 和 Worker 服务可以在集群内水平扩展。Frontend服务的扩展方式与其他服务不同，因为它没有分片/分区，它只是无状态的。服务发现使用的是[Ringpop](https://github.com/temporalio/ringpop-go)的成员协议

**数据库：**支持 Cassandra、MySQL 和 PostgreSQL 模式。

 **命名空间：** 命名空间是 Temporal 中的基本隔离单元，默认会启动一个“default”命名空间和一个内部的命名空间，同一个命名空间下工作流ID唯一

**安全性：** 支持TLS进行传输加密，Authentication支持服务端，客户端和用户三种模式，用户的身份认证是通过实现`ClaimMapper`，`Authorizer`接口，默认使用JWT ClaimMapper实现。

### 资料：

github： https://github.com/temporalio/temporal

官方文档：https://docs.temporal.io/docs/concepts/introduction

go sdk开发文档：https://docs.temporal.io/docs/go/how-to-develop-a-workflow-definition-in-go

官方示例：https://github.com/temporalio/samples-go

server 部署脚本： https://github.com/temporalio/docker-compose