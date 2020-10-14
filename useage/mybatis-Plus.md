
# 基础使用

项目整体结构：要把生成出来的`*.xml`文件移到`resources/mapper`中，因为这是`mybatis-plus`默认的露肩，也可以自定义路径。

<img src="/assets/images/useage/01.png"/>

## 创建可运行空项目

### 设置依赖:pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mybatis.example</groupId>
    <artifactId>mybatis_use_generater</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 表示这个项目就是个SpringBoot项目 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath />
    </parent>

    <!--  设置资源属性  -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <!-- 设置依赖 -->
    <dependencies>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
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

        <!-- 数据库连接 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>
    </dependencies>
</project>
```

### 设置资源文件:application.yml

```yml
spring:
  datasource:                                           # 数据源的相关配置
    driver-class-name: com.mysql.cj.jdbc.Driver         # mysql驱动
    url: jdbc:mysql://localhost:3306/foodie_shop_dev?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true
    username: root
    password: 12345678
```

### 创建启动项目

```java
package com.mybatis.example;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```

## 接入自动化生成器代码

### 接入依赖

```xml
<!--    自动生成代码相关的依赖    -->
<!-- lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<!-- MyBatis Plus 自动生成器 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.0</version>
</dependency>

<!-- mybatis-plus 代码生成器 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.0</version>
</dependency>

<!-- 默认生成器模版：velocity -->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.21</version>
    <scope>compile</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```

### 设置扫描路径

```java
@SpringBootApplication
@MapperScan("com.mybatis.example.mapper") // 新增这句代码
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```

### 新建自动化生成器

```java
package com.mybatis.example;

import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;

import java.util.Scanner;

public class MysqlGenerator {
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("sam");
        gc.setOpen(false);
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/foodie_shop_dev?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("12345678");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setParent("com.mybatis.example");
        mpg.setPackageInfo(pc);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true);
        strategy.setControllerMappingHyphenStyle(true); // 驼峰转连字符
        mpg.setStrategy(strategy);
        mpg.execute();
    }
}
```

# 参数讲解

```java
StrategyConfig strategy = new StrategyConfig();
strategy.setSuperEntityClass("com.mybatis.example.entity"); // 设置所有生成Entity需要继承的父类
```

像上面的配置生成的代码为：

```java
package com.mybatis.example.entity;
import com.mybatis.example.entity;
@Data
@EqualsAndHashCode(callSuper = true)
public class User extends entity {
	//...
}
```

# 报错

```
Field userService in com.mybatis.example.controller.UserController required a single bean, but 2 were found:
    - userServiceImpl: defined in file [.../service/impl/UserServiceImpl.class]
    - IUserService: defined in file [.../service/IUserService.class]
```

看下配置:


```java
@SpringBootApplication
@MapperScan("com.mybatis.example") // 把这句改成： @MapperScan("com.mybatis.example.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```

# 功能使用

## 排除非表字段

```java
@TableField(exist = false) // false 表明数据库表中没有这字段，默认 true。
private String email; // 例如：Entity中 email 字段在对应的数据表中没有，这这是本地使用的字段。
```

### 

# 资料

* [MybatisX 快速开发插件](https://baomidou.com/guide/mybatisx-idea-plugin.html)
* [MyBatis Plus](https://baomidou.com/)
* [代码生成器配置](https://baomidou.com/config/generator-config.html)
