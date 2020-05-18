# 微服务架构迁移

## 背景

- 微服务架构是一种架构模式或者说是一种架构风格，它提倡将单一应用程序划分成一组小的服务，每个服务运行在其独立的自己的进程中，服务之间互相协调、互相配合，为用户提供最终价值。服务之间采用轻量级的通信机制互相沟通（通常是基于HTTP的RESTful API）。
- Spring Cloud为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智能路由，微代理，控制总线）。

## 设计原则
  - 兼容门户现有的各系统现状，不影响未集成进微服务的生态的外部应用
  - 兼容后续监管云、容器云和devops

## 工作内容

- 微服务核心组件的研究及搭建

  - Zuul路由网关
    - Zuul集群部署，并和nginx配合完成zuul的集群负载均衡
    - 2个Zuul节点

  - Eureka：
    - 目标：实现注册中心与应用的集群部署，完成服务的注册和发现
    - 3个Eureka Server节点
    - 注册服务的汇总

  - Spring Cloud LB实现负载均衡：
    - Ribbon结合Eureka实现负载均衡的解决方案
    - IRule-LB算法的学习，自定义策略的使用
    - 典型场景的汇总，及不同场景的策略应用
    - 会话保持（redis）

  - LB RestTemplate实现动态的请求发送
    - 场景的汇总
    - RestTemplate多实例的问题

- 资源申请配置

- 门户相关项目的迁移与重构

## 任务

- 完成Eureka Server、Zuul Server集成环境的集群部署
- 完成portal、sso、egov、iam、message等服务集成Eureka、Zuul的改造和联调
  - 需在集成新建一个portal_dev_replicas，在此进行联调测试。因为涉及到appUri的变动
- 完成portal、sso、egov、iam、message等服务集成Spring Cloud LB、LB RestTemplate的改造和联调
  - 在portal_dev_replicas数据库上进行调试
  - 梳理所有业务场景，进行分类：无状态/有状态、可以实现LB的接口/必须通过访问url交互的接口
  - 注意LB RestTemplate、RestTemplate双实例同时存在的问题
  - 优先完成restfulless的交互改造
  - 应充分考虑到重试、熔栽等情况下上层业务的兼容问题
- 实现基于Redis的会话保持、共享存储
  - 解决Spring Session在Redis存储介质中无法正常读取attributes的问题
  - 完成portal、sso、iam等服务的redis改造
- 微服务身份认证问题的两种方案
  - 方案一：Zuul+Spring Cloud Security，在网关上实现身份认证
  - 方案二：各应用保持自己的身份认证，但必须基于redis实现会话保持等
- 测试与部署相关问题：
  - 资源的申请
  - docker部署
  - 提前沟通仿真/生产可能存在的运维工作
    - nginx配置的更改：nginx转发至各app -> nginx转发至zuul，zuul实现app的路由分发
  - 压力、功能测试

## 时间安排

- 微服务核心组件的搭建
  - 上述核心组件的搭建方案确定 4月24日
  - 集成环境部署 4月30日
  - 仿真生产部署（待定）

- 资源申请配置
  - 搭建方案确定后，着手环境资源的申请 4月24日前
  - 集成环境的资源就位 4月28日前
  - 仿真生产（待定）

- 门户及通用服务相关项目的迁移与重构
  - 方案确定后进行迁移和适配，5月14日前完成开发联调
  - 集成初版部署 5月15日
  - 仿真部署 5月25日
  - 生产部署 5月27日

## 参考文档

- [baeldung-spring cloud series](https://www.baeldung.com/spring-cloud-series)
- [spring cloud reference](https://cloud.spring.io/spring-cloud-static/Hoxton.SR3/reference/html/documentation-overview.html#contract-documentation)
- [中文版Spring Cloud Hoxton版本入门教程](https://blog.csdn.net/ThinkWon/article/details/103738851/)
- [Eureka Client Properties](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java)
- [Eureka Instance Properties](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)
- github 各组件的项目

## 重点关注及问题

- portal、sso、egov、iam等应用是否由单机部署（主从热备）升级为集群部署？
  - 集群部署的问题：
    1. 基于Redis的会话共享
    2. 定时任务等业务模块的重构
    3. 数据库连接及事务
    4. 应用间交互（Rest接口）适配
  - 应用单机的问题：
    1. 应用重启或主备切换时，Eureka服务注册及发现
    2. 小Nginx和Api Gateway的关系
- (*) 标注的组件可留给后续版本
- 压力测试 Jmeter
- 集成可用性测试
- 与运维组提前沟通，以免影响上线计划
- Spring Cloud follows the Latest Release
- 参考文档
- 补充大思路/设计原则
  - 兼容门户现有的各系统现状
  - 兼容后续监管云、容器云和devops
- 业务架构设计 -> 微服务的切分方案
- 可实施的方案（包括过渡方案）

## 本地调试中遇到的问题

- Eureka单机：
  - Server的context-path默认是/eureka，只override server.servlet.context-path没用，还需仔细研究一遍server的配置属性
  - Server端需要有security