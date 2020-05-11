# Spring Cloud Notes

## Zuul

### Zuul Proxy

- zuul.ignored-services = "*": 不允许通过访问服务id来访问服务，取代的方式是必须通过context-path进行访问

### Zuul Routes

- “/actuator/routes”: 通过actuator查看zuul代理的路由有哪些，默认是disable

  ```java
  management.endpoints.web.exposure.include=health,info,routes
  management.endpoint.routes.enabled=true
  ```

