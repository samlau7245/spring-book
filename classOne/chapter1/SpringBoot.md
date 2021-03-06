# 基础知识整合

## 事务(Transaction)

> **事务(Transaction)** : 是数据库操作的最小工作单元，是作为单个逻辑工作单元执行的一系列操作；这些操作作为一个整体一起向系统提交，要么都执行、要么都不执行；事务是一组不可再分割的操作集合（工作逻辑单元）。

* `REQUIRED` ：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 
* `SUPPORTS` ：支持当前事务，如果当前没有事务，就以非事务方式执行。 
* `MANDATORY` ：支持当前事务，如果当前没有事务，就抛出异常。 
* `REQUIRES_NEW` ：新建事务，如果当前存在事务，把当前事务挂起。 
* `NOT_SUPPORTED` ：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 
* `NEVER` ：以非事务方式执行，如果当前存在事务，则抛出异常。 
* `NESTED` ：支持当前事务，如果当前事务存在，则执行一个嵌套事务，如果当前没有事务，就新建一个事务。

## 日期

* `Date ` : 
* `LocalDate` : 计算日期
* `LocalTime` : 只有时刻
* `LocalDateTime` : 计算日期加时刻，`LocalDateTime`相比`Date`更像是一个工具类，就是为了时间操作使用。

* 获取当前时间对象的方式:

```java
LocalDateTime localDateTime = LocalDateTime.now();
Date date = new Date();
```

## Restful获取接口请求中的参数

<img src="/assets/images/classOne/cp1/105.png">

### @PathVariable

```java
@GetMapping("/subCat/{rootCatId}")
public IMOOCJSONResult subCat(@PathVariable Integer rootCatId) {}
```

### @RequestParam

```java
@PostMapping("/logout")
public IMOOCJSONResult logout(@RequestParam String userId) {}
```

### @RequestBody

```java
@PostMapping("/login")
public IMOOCJSONResult login(@RequestBody UserBO userBO) throws Exception {}

public class UserBO {
    private String username;
    private String password;
    private String confirmPassword;
}
```

# AOP

依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

接入：

```java
@Aspect
@Component
public class ServiceLogAspect {}
```












































