
<img src="/assets/images/classOne/cp1/04.png">

# 环境搭建

* 云服务系统搭建：`CentOS 7`。(有教程)
* 搭建JDK。(有教程)
* 搭建Tomcat。(有教程)
* 搭建数据库：`MariaDB`。(有教程)

<img src="/assets/images/classOne/cp1/25.png">

* `tomcat`: 端口`8080`，用于放前端代码。
* `tomcat-api`: 端口`8088`，用于放后端接口代码。

# 项目多环境部署

* 开发环境：`dev`
* 测试环境：`test`
* 生产环境：`prod`

多环境会涉及到URL地址的更改，`SpringBoot`提供了`profiles`，可以支持多环境。

* `application.yml` : 主配置文件。
* `application-{name}.yml` : 环境配置文件，是挂载到主配置文件中的。

<img src="/assets/images/classOne/cp1/18.png">

```yml
# application.yml
spring:
  profiles:
    active: dev # 激活 dev 环境的配置；这样开发环境和线上环境的部署就方便很多。
```

<img src="/assets/images/classOne/cp1/19.png">


```yml
spring:
  datasource:                                           
    url: jdbc:mysql://localhost:3306/foodie_shop_dev?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true

# 这里 localhost 
	1. 如果此项目和数据库是放在同一个服务器的可以写：localhost 或者 127.0.0.1。
	2. 如果是集群或者分布式的部署，localhost 要改成云服务器的内网IP。
```

<img src="/assets/images/classOne/cp1/20.png">

# 打包(war)

打包的方式：

* `jar`: jar包是一种服务化的概念。在 `SpringCloud` 中所有的服务打包都是以jar的形式存在。
* `war`: war包是一种应用程序的概念。可以对外提供接口和服务，单体服务基本war包满足。

**项目中已经有内置的Tomcat服务，但是如果项目要打成war包，就得依赖外部的Tomcat，就得把内置Tomcat给去掉。**

4. 项目启动
使用jar包运行程序可以直接使用`Application`，如果使用war包执行程序需要依托war包的启动类。

## 步骤

```
foodie-dev // 顶层pom项目
├── foodie-dev-api
│   └── pom.xml 最底层打包项目
├── foodie-dev-common
│   └── pom.xml
├── foodie-dev-mapper
│   └── pom.xml
├── foodie-dev-pojo
│   └── pom.xml
├── foodie-dev-service
│   └── pom.xml
└── pom.xml

`common` -> `pojo` -> `mapper` -> `service` -> `api`
```

### 修改打包方式

```xml
<!-- foodie-dev 的 pom.xml -->
<project>
    <artifactId>foodie-dev-api</artifactId>
    <!-- war打包[1] 改成war打包 -->
    <packaging>war</packaging>
</project>
```

### Maven删除Tomcat依赖

<img src="/assets/images/classOne/cp1/22.png">

### 添加 Servelet依赖

```xml
<!-- foodie-dev pom.xml -->
<project>
    <dependencies>
        <!-- spring-boot-starter-web：表示SpringBoot的web模块 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- war打包[2] 去掉内置Tomcat -->
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- war打包[3] 移除内置Tomcat以后，它包含的一些Servelet也就没了，需要添加额外的Servelet依赖。 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

### 添加war启动项

```
├── foodie-dev-api.iml
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── imooc
        │           ├── Application.java
        │           ├── WarStartApplication.java // 新建 WarStartApplication 类作为war启动项。
        │           └── controller
        │               └── HelloController.java
        └── resources
            ├── application-dev.yml
            ├── application-prod.yml
            └── application.yml
```

```java
package com.imooc;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

// war打包[4] 添加war包启动类
public class WarStartApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        // 指向Application启动类
        return builder.sources(Application.class);
    }
}
```

### 安装依赖

<img src="/assets/images/classOne/cp1/23.png">


```
foodie-dev
├── foodie-dev-api
│   ├── src
│   └── target // 执行完 install 后的产物，
├── foodie-dev-common
│   ├── src
│   └── target // 执行完 install 后的产物
├── foodie-dev-mapper
│   ├── src
│   └── target // 执行完 install 后的产物
├── foodie-dev-pojo
│   ├── src
│   └── target // 执行完 install 后的产物
├── foodie-dev-service
│   ├── src
│   └── target // 执行完 install 后的产物
└── src
    └── main
```

<img src="/assets/images/classOne/cp1/24.png">

把`foodie-dev-api-1.0-SNAPSHOT.war`后版本号删除`foodie-dev-api.war`。

### 上传包到服务器

把`foodie-dev-api.war`上传到服务器`Tomcat(8088)`路径：`/usr/local/tomcat-api/webapps`中，war包会被自动解压。

<img src="/assets/images/classOne/cp1/26.png">

测试：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public Object hello(){
        return "Hello World!";
    }
}
```

<img src="/assets/images/classOne/cp1/27.png">


































