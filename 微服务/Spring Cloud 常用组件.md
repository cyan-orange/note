# Spring Cloud Alibaba

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

引入依赖，在 `dependencyManagement` 中添加如下配置。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

# Nacos 服务注册与发现

Nacos 是阿里巴巴开源的一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

1. 
首先需要获取 Nacos Server，[Nacos Server 下载页](https://github.com/alibaba/nacos/releases)

2. 
修改 pom.xml 文件，引入 Nacos Discovery Starter。
```xml
 <dependency>
     <groupId>com.alibaba.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
 </dependency>
```


3. 
在配置文件中配置 Nacos Server 地址
```yaml
server:
  port: 8082

spring:
  application:
    name: service-provider
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
```


4. 
使用 @EnableDiscoveryClient 注解开启服务注册与发现功能
```java
 @SpringBootApplication
 @EnableDiscoveryClient
 public class ProviderApplication {

 	public static void main(String[] args) {
 		SpringApplication.run(ProviderApplication.class, args);
 	}
 }
```



# Nacos 分布式配置管理

1. 
首先，修改 pom.xml 文件，引入 Nacos Config Starter。
```xml
 <dependency>
     <groupId>com.alibaba.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
 </dependency>
```


2. 
在应用的 `/src/main/resources/bootstrap.yml` 配置文件中配置 Nacos Config 元数据
```yaml
spring:
  application:
    name: service-provider

  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        namespace: d1c63875-be0c-46db-81d5-bd037567f9c1
        file-extension: yaml
```


3. 
完成上述两步后，应用会从 Nacos Config 中获取相应的配置，并添加在 Spring Environment 的 PropertySources 中。这里我们使用 [@Value ](/Value ) 注解来将对应的配置注入到 SampleController 的 userName 和 age 字段，并添加 [@RefreshScope ](/RefreshScope ) 打开动态刷新功能 
```java
 @RefreshScope
 class SampleController {

 	@Value("${user.name}")
 	String userName;

 	@Value("${user.age}")
 	int age;
 }
```


4. 
在Nacos控制台的配置列表中添加配置，Data ID: service-provider.yaml，Group: DEFAULT_GROUP，内容如下
```yaml
user:
  name: orange
  age: 23
```



# OpenFeign 服务调用

OpenFeign 是Spring Cloud 的组件，所以要加入spring cloud 的依赖管理

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR12</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Nacos Discovery Starter 默认集成了 Ribbon ，所以对于使用了 Ribbon 做负载均衡的组件，可以直接使用 Nacos 的服务发现。

1. 
首先，修改 pom.xml 文件，引入 OpenFeign Starter。
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```


2. 
使用注解 `@EnableFeignClients` 开启服务调用
```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```


3. 
FeignClient 已经默认集成了 Ribbon ，此处演示如何配置一个 FeignClient。
```java
 @FeignClient(name = "service-provider")
 public interface EchoService {
     @GetMapping(value = "/echo/{str}")
     String echo(@PathVariable("str") String str);
 }
```



# Sentinel 服务熔断

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。 [Sentinel](https://github.com/alibaba/Sentinel) 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

这里只是简单的使用服务熔断

1. 
首先，修改 pom.xml 文件，引入 Sentinel Starter
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```


2. 
Sentinel 适配了 Feign 组件。如果想使用，除了引入 `spring-cloud-starter-alibaba-sentinel`，只需要在配置文件打开 Sentinel 对 Feign 的支持
```yaml
feign:
  sentinel:
    enabled: true
```



这是一个 `FeignClient` 的简单使用示例：

```java
@FeignClient(name = "service-provider", fallback = EchoServiceFallback.class, configuration = FeignConfiguration.class)
public interface EchoService {
    @RequestMapping(value = "/echo/{str}", method = RequestMethod.GET)
    String echo(@PathVariable("str") String str);
}

class FeignConfiguration {
    @Bean
    public EchoServiceFallback echoServiceFallback() {
        return new EchoServiceFallback();
    }
}

class EchoServiceFallback implements EchoService {
    @Override
    public String echo(@PathVariable("str") String str) {
        return "echo fallback";
    }
}
```

# Gateway 网关

简单使用

修改 pom.xml 文件，引入 Sentinel Starter

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

网关配置

```yaml
server:
  port: 8080

spring:
  application:
    name: service-gateway

  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848

    gateway:
      routes:
        - id: service-provider
          uri: lb://service-provider
          predicates:
            - Path=/provider/**
        - id: service-consumer
          uri: lb://service-consumer
          predicates:
            - Path=/consumer/**
```
