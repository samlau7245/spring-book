
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
	* `foodie-dev-common`  : 子模块
	* `foodie-dev-pojo` : 子模块
	* `foodie-dev-mapper` : 子模块
	* `foodie-dev-service` : 子模块
	* `foodie-dev-api` : 子模块

子模块之间的继承关系为：`common` -> `pojo` -> `mapper` -> `service` -> `api`

<img src="/assets/images/classOne/09.png">

## 安装聚合项目

<img src="/assets/images/classOne/10.png">

# IDE快捷键

## 展开项目目录

* 点击项目根目录 + 按`*`键：展开所有目录结构

<img src="/assets/images/classOne/04.png">

## 文件分屏展示

<img src="/assets/images/classOne/05.png">
