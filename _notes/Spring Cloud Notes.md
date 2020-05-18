# Spring Cloud Notes

## Eureka

### Overview

- 服务注册中心，Eureka Server(记录服务的注册表，监控服务的健康)、Eureka Instance(service provider)、Eureka Client(service consumer)
- *？？？几个关键的概念：Env、Region、Zone、Data Center等*

### Service Registry

- 几个关键属性的默认值
  - appName（serviceId）= ${spring.application.name}
  - virtual host = ${spring.application.name}
  - non-secure port = ${server.port}

- heath check
  - 注册成功后的Service默认status一直是UP
  - eureka.client.healthcheck.enabled=true必须在applcation.properties中配置才能正常生效
- 应用集群部署的时候，每个instance有独特的intanceId，默认的取值是${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${s
erver.port}}}

### Service Discovery

- 在程序里引用EurekaClient

```java
@Autowired
private EurekaClient discoveryClient;
public String serviceUrl() {
InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
return instance.getHomePageUrl();
}
```

- client总是优先选择same zone的eureka server的服务注册表，instance在注册服务的时候通过metadata指定自己的service在哪个zone是可用的

```java
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

- 通过集成Spring Cloud LoadBalancer做到多zone服务的负载均衡
  - 这块讲的比较笼统，目前还没想到应用场景

### Eureka Server

- Http API endpoints: "/eureka/*"

## Zuul

### Zuul Proxy

- zuul.ignored-services = "*": 不允许通过访问服务id来访问服务，取代的方式是必须通过context-path进行访问
- 集群部署时，默认使用RibbonClient来进行负载均衡。支持static/dynamic两种方式获取severlist，具体参见下面的Ribbon。当集成Eureka时默认使用dynamic方式

### Zuul Routes

- “/actuator/routes”: 通过actuator查看zuul代理的路由有哪些，默认是disable
- "/actuator/routes/details": 查看详细路由信息

  ```java
  management.endpoints.web.exposure.include=health,info,routes
  management.endpoint.routes.enabled=true
  ```

- PatternServiceRouteMapper：使用正则表达式来动态建立route pattern和serviceId的映射，省去了每次新增服务都要在zuul上配置一遍的hard code。当请求pattern没有match到任何一个serviceId时，会使用配置文件中默认的静态配置

```java
@Bean
public PatternServiceRouteMapper serviceRouteMapper() {
  //将“/v1/myusers/**”的请求路由到serviceId matched的服务上，如myusers-v1
return new PatternServiceRouteMapper(
"(?<name>^.+)-(?<version>v.+$)",  //serverId的正则表达式
"${version}/${name}"); //route pattern
}
```

- 管理请求的prefix
  - zuul默认会在转发请求前把proxy prefix从请求路径中去掉。```java zuul.routes.users.stripPrefix=false```
  - 给请求设置通用的前缀 ```java zuul.prefix=/api```
  - **【巧用prefix的例子】如何实现actuator endpoints的对外/内不同的访问控制？** <br> 在zuul上对/actuctor/**请求加上前缀/admin，从而使得外部的请求必须具有admin权限的用户才能访问，而内部的请求不经过zuul，从而避免了登录认证，如springboot-admin-server和各个client间的通信

## Ribbon

### Overview

- Ribbon组成部分
  - Rule：轮询策略
  - Ping：a component running in background to ensure liveness of servers
  - ServerList：服务提供方的sever list. this can be static or dynamic.

### Ribbon Client Config

- override default beans via external configurations
  
- .properties file属性:
  - prefix: ```<clientName>.robbin.*```

  ```java
  NFLoadBalancerClassName
  NFLoadBalancerRuleClassName
  NFLoadBalancerPingClassName
  NIWSServerListClassName
  NIWSServerListFilterClassName
  ```

### Rules

- Common Rules
  - RoundRobinRule:轮询算法（默认/fallback）
  - AvailabilityFilteringRule:避开阻断或高并发的server，阻断次数越高等待时间越长
  - WeightedResponseTimeRule：根据server的响应时间设定sever的权重，根据权重随机挑选server
- Customized Rules
  - 可以通过实现IRule接口来自定义自己的负载均衡规则
  - IRule.choose(Object key)中入参Object key对应的是RequestContext中key为FilterConstants.LOAD_BALANCER_KEY的header。可以通过在zuul的pre filter中加入FilterConstants.LOAD_BALANCER_KEY的header，然后在Ribbon的IRule.choose()方法中根据header来决定选择哪个server

### ServerList

- ConfigurationBasedServerList(static)

```java
sample-client.ribbon.listOfServers=www.microsoft.com:80,www.yahoo.com:80,www.google.com:80
```

- DiscoveryEnabledNIWSServerList(dynamic)
  - integrate with EurekaClient.
  - pre-requirements：
    - **启用DiscoveryEnabledNIWSServerList**
    - **sevice provider必须在eureka server注册vipAddress(logical eureka service identifier)**
  - server cluster must be identified via VipAddress in a property:

  ```java
  #enable DiscoveryEnabledNIWSServerList for Ribbon Client
  myClient.ribbon.NIWSServerListClassName=com.netflix.niws.loadbalancer.DiscoveryEnabledNIWSServerList
  # the server must register itself with Eureka server with VipAddress "myservice"
  myClient.ribbon.DeploymentContextBasedVipAddresses=myservice
  ```

- ServerListFilter
  - 对ServerList进行过滤，ServerListFilter is a component used by DynamicServerListLoadBalancer
  - ZoneAffinityServerListFilter：优先使用同zone的server

  ```java
  myclient.ribbon.EnableZoneAffinity=true
  ```

  - ServerListSubsetFilter: 同一个ribbon client总是使用同一个subset server list，但是会替换掉不可用的server

  ```java
  myClient.ribbon.NIWSServerListFilterClassName=com.netflix.loadbalancer.ServerListSubsetFilter
  # only show client 5 servers. default is 20.
  myClient.ribbon.ServerListSubsetFilter.size=5
  ```

### Others

- Ribbon API: 在程序里使用Ribbon的api进行配置等，可以@Autowired LoadBalancerClient类
- Ribbon Client的配置默认使用lazy loading（即第一个请求发生的时候），可以更换为eagerly load

```java
ribbon.eager-load.enabled=true
ribbon.eager-load.clients=client1, client2, client3
```

## Spring Cloud LoadBalancer

- 可以使用Sring RestTemplate

### Spring Cloud LoadBalancer integrations

- client-side LB, zuul等组件集成的时候默认使用的ribbon，可以设置使用Spring Cloud LoadBalancer

### Caching

- 支持caffeine/default
- spring.cloud.loadbalancer.cache.enabled = true

