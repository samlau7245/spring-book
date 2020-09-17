
构建一个具体的聚合项目简单结构：

<img src="/assets/images/classOne/02.png">

# 通过IDE构建聚合项目

## 创建顶层的pom项目

<img src="/assets/images/classOne/03.png">

* `GroupId` : 公司唯一ID。
* `ArtifcatId` : 项目名。
* `Version` : 版本号。

## 创建子模块工程

<img src="/assets/images/classOne/06.png">

* 聚合工程里可以分为顶级项目（顶级工程、父工程）与子工程，这两者的关系其实就是父子继承的关系。
* 子工程在`maven`里称之为模块（`module`），模块之间是平级，是可以相互依赖的。
* 子模块可以使用顶级工程里所有的资源（依赖），子模块之间如果要使用资源，必须构建依赖（构建关系）
* 一个顶级工程是可以由多个不同的子工程共同组合而成。
* 所有的子模块都是以一个 `jar`包存在于顶级工程里面的。

## 设置子模块之间的依赖

<img src="/assets/images/classOne/07.png">

现在子模块`foodie-dev-common` 、`foodie-dev-pojo`都继承自顶级依赖`foodie-dev`，现在`foodie-dev-pojo`子模块需要依赖`foodie-dev-common`子模块：

<img src="/assets/images/classOne/08.png">

结果：

```xml
<dependencies>
    <dependency>
        <groupId>com.imooc</groupId>
        <artifactId>foodie-dev-common</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

## 创建完整的聚合工程

现在我们需要创建完成的聚合工程，下面是对应的顶级工程和其子模块的模块名：

* `foodie-dev`: 顶级pom工程
	* `foodie-dev-common`  : 子模块,里面存放的是公共的功能。
	* `foodie-dev-pojo` : 子模块，里面放的是实体类(BO、VO、Entity)。
	* `foodie-dev-mapper` : 子模块，里面是数据层或DAO层。
	* `foodie-dev-service` : 子模块，里面存放着业务层。
	* `foodie-dev-api` : 子模块，接口实现层。

子模块之间的继承关系为：`common` -> `pojo` -> `mapper` -> `service` -> `api`

<img src="/assets/images/classOne/09.png">

## 安装聚合项目

<img src="/assets/images/classOne/10.png">

## 为pom工程添加依赖

* parent 依赖：依赖这个就表示这个项目就是个SpringBoot项目。

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath />
</parent>
```

* 设置资源属性：

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
</properties>
```

* 添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <!-- exclusions 表示要排除的jar包 -->
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!-- spring-boot-starter-web：表示SpringBoot的web模块 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

## 创建配置文件、启动类

<img src="/assets/images/classOne/18.png">

```java
package com.imooc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```

### 创建一个测试类

<img src="/assets/images/classOne/19.png">

最后可以直接访问：`http://localhost:8080/hello`

# HikariCP连接数据源

> `HikariCP`是官方的工具。[Github](https://github.com/brettwooldridge/HikariCP)

## pom添加依赖

```xml
<!-- mysql驱动 -->
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.20</version>
</dependency>

<!-- mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

## 添加配置文件

```yml
############################################################
# 配置数据源信息
############################################################
spring:
  datasource:                                           # 数据源的相关配置
      type: com.zaxxer.hikari.HikariDataSource          # 数据源类型：HikariCP
      driver-class-name: com.mysql.cj.jdbc.Driver       # mysql驱动
      url: jdbc:mysql://localhost:3306/foodie-shop-dev?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true
      username: root
      password: root
    hikari:
      connection-timeout: 30000       # 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 默认:30秒
      minimum-idle: 5                 # 最小连接数
      maximum-pool-size: 20           # 最大连接数
      auto-commit: true               # 自动提交
      idle-timeout: 600000            # 连接超时的最大时长（毫秒），超时则被释放（retired），默认:10分钟
      pool-name: DateSourceHikariCP     # 连接池名字
      max-lifetime: 1800000           # 连接的生命时长（毫秒），超时而且没被使用则被释放（retired），默认:30分钟 1800000ms
      connection-test-query: SELECT 1
          
############################################################
# mybatis 配置
############################################################
mybatis:
  type-aliases-package: com.imooc.pojo          # 所有POJO类所在包路径,com.imooc.pojo是本项目pojo的路径。
  mapper-locations: classpath:mapper/*.xml      # mapper映射文件

############################################################
# web访问端口号  约定：8088
############################################################
server:
  port: 8088
  tomcat:
    uri-encoding: UTF-8
  max-http-header-size: 80KB  
```

<img src="/assets/images/classOne/20.png">

# IDE快捷键

## 展开项目目录

* 点击项目根目录 + 按`*`键：展开所有目录结构

<img src="/assets/images/classOne/04.png">

## 文件分屏展示

<img src="/assets/images/classOne/05.png">
