---
title: SpringCloud整合Nacos（一）
date: 2024-04-07 18:21:08
tags: ["SpringCloud","Nacos"]
categories: ["SpringCloud","Nacos"]

---
---
title: SpringCloud整合Nacos（一）
date: 2024-04-07 18:21:08
tags: ["SpringCloud","Nacos"]
categories: ["SpringCloud","Nacos"]


---

# SpringCloud整合nacos

github地址: https://github.com/zpnvliba/springcloud-alibaba.git

整体项目结构:

![image-20240409171513212](https://raw.githubusercontent.com/zpnvliba/images/main/202404091715551.png)

## 一、创建基础maven项目

创建一个基础的maven项目只保留pom文件即可，pom文件中引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zp</groupId>
    <artifactId>springcloud-alibaba</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.9.RELEASE</version>
        <relativePath/>
    </parent>
    <modules>
        <module>user-service</module>
        <module>order-service</module>
    </modules>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR10</spring-cloud.version>
        <mysql.version>5.1.47</mysql.version>
        <mybatis.version>2.1.1</mybatis.version>
    </properties>


    <dependencyManagement>
        <dependencies>
            <!-- springCloud -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--nacos的配置依赖-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
                <version>2.2.5.RELEASE</version>
            </dependency>
            <!--nacos的注册依赖-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
                <version>2.2.5.RELEASE</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

</project>
```

## 二、创建生产者、消费者

​	在maven项目下创建两个子SpringBoot项目，并在pom中引入nacos的配置依赖和注册发现依赖。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.zp</groupId>
    <artifactId>user-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>user-service</name>
    <description>user-service</description>
    <parent>
        <groupId>com.zp</groupId>
        <artifactId>springcloud-alibaba</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>

</project>
```

## 三、填写配置文件

userservice、orderservice项目中的bootstrap.yml和application.yml都需要添加上这两个配置

**bootstrap.yml：**比application.yml文件更早读取。<font color='red'>**注意：nacos的这个配置一定要配置在bootstrap.yml中，如果配置在application.yml中，项目启动时会报错。**</font>

```yml
spring:
  application:
    name: userservice
  cloud:
    nacos:
      config:
        server-addr: 192.168.200.129:8848
```

**application.yml**

```yml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.200.129:8848 
```

注意：在启动类上可以添加@EnableDiscoveryClient注解也可以不添加

## 四、启动Nacox和项目

启动后就可以在Nacox中看到活动的实例

![image-20240407180835826](https://raw.githubusercontent.com/zpnvliba/images/main/202404071820033.png)

## 五、负载均衡

在调用别的服务的服务中配置它调用别的服务时的负载均衡策略，比如：userservice调用orderservice时，希望用randomRule策略调用orderservice。那么就在userservice服务中配置如下配置：

```java
package com.zp.userservice.config;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @author zp
 */
@Configuration
public class MyConfig {
    @Bean
    public IRule randomRule(){
        return new RandomRule();
    }
    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

配置完毕后，userservice调用orderservice时就用随机策略了，当然可以配置别的策略。默认是轮询的方式。

## 六、配置中心

nacos当配置中心时，可以在不动服务的情况下改动一些yml中的配置数据。

1. ### 在bootstrap.yml中添加如下配置

   ![image-20240407184352145](https://raw.githubusercontent.com/zpnvliba/images/main/202404071846249.png)
   这三个配置和nacos配置中的这几个属性相匹配。这样就能读取nacos中配置的属性了
   ![image-20240407184453530](https://raw.githubusercontent.com/zpnvliba/images/main/202404071846250.png)

   ![image-20240407184442872](https://raw.githubusercontent.com/zpnvliba/images/main/202404071846252.png)

## 七、伪RPC-Feign

#### 1、引入依赖

```yml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### 2、添加注解

![image-20210714175102524](https://raw.githubusercontent.com/zpnvliba/images/main/202404072213677.png)

#### 3、在要调用的服务中创建一个接口

这个接口中的@FeignClient("userservice") 填写的是要调用服务的名称

@RequestMapping("/hello") 这个注解要个被调用的服务中的路径一致

![image-20240407214133992](https://raw.githubusercontent.com/zpnvliba/images/main/202404072213678.png)

总结：

​	① 引入依赖

​	② 添加@EnableFeignClients注解

​	③ 编写FeignClient接口

​	④ 使用FeignClient中定义的方法代替RestTemplate

## 七.一 优化feign

### 1、创建一个新model,feign-api

![image-20210714204557771](https://raw.githubusercontent.com/zpnvliba/images/main/202404091715552.png)

### 2、添加依赖

在feign-api中然后引入feign的starter依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 3、配置

将user-service(调用方)中的config、clients的feign的配置移动到feign-api模块中

### 4、userservice（调用方）导入feign-api模块

如果出现
![image-20210714205623048](https://raw.githubusercontent.com/zpnvliba/images/main/202404091715553.png)

在userservice(调用方)启动类中的@EnableFeignClients()注解中加入basePackages = "com.zp.service.orderservice"

或者 加入clients = {UserClient.class}



#  



## 八、网管-Gateway

架构图：

![image-20210714210131152](F:/typoraimages/202404061421483.png)

1、 创建gateway服务，引入依赖

![image-20210714210919458](F:/typoraimages/202404061421484.png)

2、在gateway服务中添加依赖

```
<!--网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--nacos服务发现依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

3、在gateway服务中添yml配置

```
server:
  port: 10010 # 网关端口
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos地址
    gateway:
      routes: # 网关路由配置
        - id: user-service # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
            filters: # 过滤器
        	- AddRequestHeader=Truth, Itcast is freaking awesome! # 添加请求头
```