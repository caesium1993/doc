# IAM （用户身份与访问控制） 客户端模块

## 使用方法（适用于 Spring Boot 项目）

- 使用本模块的项目需继承公共pom

```xml
<parent>
  <groupId>org.neris.support</groupId>
  <artifactId>spring-boot-app-parent</artifactId>
  <version>1.0.0</version>
  <relativePath/>
</parent>
```

- 使用本模块的项目需引入依赖

```xml
<dependencies>
  ...
  <dependency>
    <groupId>org.neris.support</groupId>
    <artifactId>support</artifactId>
    <version>1.5.0</version>
  </dependency>
  <dependency>
    <groupId>org.neris.support</groupId>
    <artifactId>iam</artifactId>
    <version>0.1.0</version>
  </dependency>
  ...
</dependencies>
```

- 在项目中创建配置类，并引入iam的默认配置

  - 集成0.1.0版本iam的项目，引入org.neris.support.iam.modules.client.config.TemporaryIamClientConfig.class 临时配置，该配置默认通过接口与管理端进行交互

  ```java
  @Configuration
  @Import(TemporaryIamClientConfig.class)
  public class IamConfig {
    ...
  }
  ```

  - 后续版本iam，引入org.neris.support.iam.modules.client.config.DefaultIamClientConfig.class 默认配置，该配置默认通过HTTP请求和管理端进行交互

  ```java
  @Configuration
  @Import(DefaultIamClientConfig.class)
  public class IamConfig {
    ...
  }
  ```

- 在项目的配置文件（application-*.properties）中，启动iam，并指定交互方式

  - 集成0.1.0版本的iam项目，采用api形式进行交互

  ```java
  iam.client.enabled=true
  iam.client.enabled-method=api
  ```

  - 集成后续版本的iam项目，采用rest形式进行交互

  ```java
  iam.client.enabled=true
  iam.client.enabled-method=rest
  ```

  - 为避免同名bean扫描冲突，在项目中开启如下配置：
  
  ```java
  spring.main.allow-bean-definition-overriding=true
  ```
