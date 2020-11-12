* 手机号短信登录、邮箱登录、用户名密码登录。

* SpringBoot工具依赖：

```xml
<!-- apache 工具类 -->
<dependency>
	<groupId>commons-codec</groupId>
	<artifactId>commons-codec</artifactId>
	<version>1.11</version>
</dependency>
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-lang3</artifactId>
	<version>3.4</version>
</dependency>
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-io</artifactId>
	<version>1.3.2</version>
</dependency>
```

* 业务对象`BO`。
* 枚举创建。
* 保证每个用户的主键唯一化：`org.n3r.idworker` 这个是个ID生成器，在MyBatis Plus 中ID是自动生成的。
	* 把`org.n3r.idworker`导入到`foodie-dev-common`中。
	* 在启动类中把`org.n3r.idworker` 设置扫描:`@ComponentScan(basePackages = {"com.imooc", "org.n3r.idworker"})` 


# 解决跨域问题

<img src="/assets/images/classOne/cp1/102.png">

```java
package com.imooc.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {
    public CorsConfig() {
    }
    @Bean
    public CorsFilter corsFilter(){
        // 1. 添加cors配置信息
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedOrigin("http://localhost:8080"); // 添加允许跨域的域名

        // 设置是否发送cookie信息
        config.setAllowCredentials(true);

        // 设置允许请求的方式
        config.addAllowedMethod("*"); // GET POST ...

        // 设置允许的header
        config.addAllowedHeader("*");

        // 2. 为url添加映射路径
        UrlBasedCorsConfigurationSource corsSource = new UrlBasedCorsConfigurationSource();
        corsSource.registerCorsConfiguration("/**", config);

        // 3. 返回重新定义好的corsSource
        return new CorsFilter(corsSource);
    }
}
```

# cookie & session

* cookie
	* 以键值对的形式存储信息在浏览器。
	* 只能在当前域名或父级域名取到cookie的键值对，不可以跨域。
	* 可以设置有效期。
* session
	* 相当于服务器端的缓存，基于服务器端内存的缓存(非持久化-如果服务器宕机，内存就没了)，可保存请求会话。
	* 每个session通过`JSESSIONID`来区分不同的请求。
	* 可以设置有效期。
	* 以键值对的形式。

给个小Demo:

```java
@GetMapping("/setSession")
public Object setSession(HttpServletRequest request) {
    HttpSession session = request.getSession();
    session.setAttribute("userInfo", "new user");
    session.setMaxInactiveInterval(3600);
    session.getAttribute("userInfo");
    return "ok";
}
```

<img src="/assets/images/classOne/cp1/103.png">

# Spring AOP

通过AOP实现检测服务

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```java
@Aspect
@Component
public class ServiceLogAspect {
    public static final Logger log = LoggerFactory.getLogger(ServiceLogAspect.class);

    /**
     * AOP通知类型：
     * 1. 前置通知：在方法调用之前执行
     * 2. 后置通知：在方法正常调用之后执行
     * 3. 环绕通知(@Around)：在方法调用之前和之后，都分别可以执行的通知
     * 4. 异常通知：如果在方法调用过程中发生异常，则通知
     * 5. 最终通知：在方法调用之后执行
     */

    /**
     * 切面表达式：
     * execution 代表所要执行的表达式主体
     * 第一处 * 代表方法返回类型 *代表所有类型
     * 第二处 包名代表aop监控的类所在的包
     * 第三处 .. 代表该包以及其子包下的所有类方法
     * 第四处 * 代表类名，*代表所有类
     * 第五处 *(..) *代表类中的方法名，(..)表示方法中的任何参数
     *
     * @param joinPoint
     * @return
     * @throws Throwable
     */
    @Around("execution(* com.imooc.service.impl..*.*(..))")
    public Object recordTimeLog(ProceedingJoinPoint joinPoint) throws Throwable {

        log.info("====== 开始执行 {}.{} ======",
                joinPoint.getTarget().getClass(),
                joinPoint.getSignature().getName());

        // 记录开始时间
        long begin = System.currentTimeMillis();

        // 执行目标 service
        Object result = joinPoint.proceed();

        // 记录结束时间
        long end = System.currentTimeMillis();
        long takeTime = end - begin;

        if (takeTime > 3000) {
            log.error("====== 执行结束，耗时：{} 毫秒 ======", takeTime);
        } else if (takeTime > 2000) {
            log.warn("====== 执行结束，耗时：{} 毫秒 ======", takeTime);
        } else {
            log.info("====== 执行结束，耗时：{} 毫秒 ======", takeTime);
        }

        return result;
    }
}
```

# 分类(TO LEARN GOOD)

##  自连接(多表查询)

```sql
SELECT
    * 
FROM
    category f
    LEFT JOIN category c ON f.id = c.father_id 
WHERE
    f.father_id = 1
```

# 商品推荐(TO LEARN GOOD)

```sql
SELECT
    f.id AS rootCatId,
    f.name AS rootCatName,
    f.slogan AS slogan,
    f.cat_image AS catImage,
    f.bg_color AS bgColor,
    i.id AS itemId,
    i.item_name AS itemName,
    ii.url AS itemUrl,
    i.created_time AS createdTime 
FROM
    category f
    LEFT JOIN items i ON f.id = i.root_cat_id
    LEFT JOIN items_img ii ON i.id = ii.item_id 
WHERE
    f.type = 1 
    AND i.root_cat_id = 7 
    AND ii.is_main = 1 
ORDER BY
    i.created_time DESC 
    LIMIT 0,6
```

> **[info] 深入学习**
>
>* 多表联查，联查结果与Mybatis多结构体结合(节点、子节点)。


# 商品详情

# 商品评价

* 字段脱敏。
* 多表联查(users items_comments)

#  商品搜索

**后端金额都是以金额为准**

## 商品搜索

* 数据库SQL

```sql
SELECT
    i.id AS itemId,
    i.item_name AS itemName,
    i.sell_counts AS sellCounts,
    ii.url AS imgUrl,
    tempSpec.priceDiscount AS price 
FROM
    items i
    LEFT JOIN items_img ii ON i.id = ii.item_id
    LEFT JOIN (
        SELECT
            item_id,
            MIN( price_discount ) AS priceDiscount -- 拿到商品的最低价格
        FROM
            items_spec 
        GROUP BY
            item_id 
            ) tempSpec ON i.id = tempSpec.item_id -- tempSpec 临时表
WHERE
    is_main = 1 
```

* Mapper

```xml
<select id="searchItems" resultType="com.imooc.pojo.vo.SearchItemsVO">
    SELECT
        i.id AS itemId,
        i.item_name AS itemName,
        i.sell_counts AS sellCounts,
        ii.url AS imgUrl,
        tempSpec.priceDiscount AS price
    FROM
        items i
        LEFT JOIN items_img ii ON i.id = ii.item_id
        LEFT JOIN (
            SELECT
                item_id,
                MIN( price_discount ) AS priceDiscount
            FROM
                items_spec
            GROUP BY
                item_id
                ) tempSpec ON i.id = tempSpec.item_id -- tempSpec 临时表
    WHERE
        is_main = 1
        <if test="paramsMap.keywords != null and paramsMap.keywords != null">
            AND i.item_name LIKE '%${paramsMap.keywords}%'
        </if>
    ORDER BY
    <choose>
        <!--<when test="paramsMap.sort == 'c' "> --> <!-- c：销量优先，因为MyBais中对与 ' 符号识别不了，所以可以用 &quot; 进行转义 -->
        <when test="paramsMap.sort == &quot;c&quot; ">
            i.sell_counts DESC
        </when>
        <!-- <when test="paramsMap.sort == 'p' "> --> <!-- p：价格优先 -->
        <when test="paramsMap.sort == &quot;p&quot; ">
            tempSpec.priceDiscount DESC
        </when>
        <otherwise> <!-- k：默认排序 -->
            i.item_name ASC
        </otherwise>
    </choose>
</select>
```

## 分类搜索


* SQL

```sql
SELECT
    i.id AS itemId,
    i.item_name AS itemName,
    i.sell_counts AS sellCounts,
    ii.url AS imgUrl,
    tempSpec.priceDiscount AS price 
FROM
    items i
    LEFT JOIN items_img ii ON i.id = ii.item_id
    LEFT JOIN (
SELECT
    item_id,
    MIN( price_discount ) AS priceDiscount 
FROM
    items_spec 
GROUP BY
    item_id 
    ) tempSpec ON i.id = tempSpec.item_id -- tempSpec 临时表
    
WHERE
    is_main = 1 
--  AND i.item_name LIKE '%蛋糕%' 
    AND i.cat_id = 51
-- ORDER BY
    -- i.item_name ASC
    -- i.sell_counts DESC
--  tempSpec.priceDiscount DESC
```

* Mapper

```xml
<select id="searchItemsByThirdCat" resultType="com.imooc.pojo.vo.SearchItemsVO">
    SELECT
    i.id AS itemId,
    i.item_name AS itemName,
    i.sell_counts AS sellCounts,
    ii.url AS imgUrl,
    tempSpec.priceDiscount AS price
    FROM
    items i
    LEFT JOIN items_img ii ON i.id = ii.item_id
    LEFT JOIN (
    SELECT
    item_id,
    MIN( price_discount ) AS priceDiscount
    FROM
    items_spec
    GROUP BY
    item_id
    ) tempSpec ON i.id = tempSpec.item_id -- tempSpec 临时表
    WHERE
    is_main = 1
    <if test="paramsMap.keywords != null and paramsMap.keywords != null">
        AND i.cat_id = #{paramsMap.catId}
    </if>
    ORDER BY
    <choose>
        <!--<when test="paramsMap.sort == 'c' "> --> <!-- c：销量优先，因为MyBais中对与 ' 符号识别不了，所以可以用 &quot; 进行转义 -->
        <when test="paramsMap.sort == &quot;c&quot; ">
            i.sell_counts DESC
        </when>
        <!-- <when test="paramsMap.sort == 'p' "> --> <!-- p：价格优先 -->
        <when test="paramsMap.sort == &quot;p&quot; ">
            tempSpec.priceDiscount DESC
        </when>
        <otherwise> <!-- k：默认排序 -->
            i.item_name ASC
        </otherwise>
    </choose>
</select>
```

# 购物车

登录情况、未登录情况下保存购物车。

* cookie，无需登录。
* session，用户会话。优点：因为session是基于服务器内存，当用户量较少的时候比较好。缺点：当用户比较多时，服务器内存负荷比较大；session只能存在单体服务器中。
* 数据库，创建购物车表来存储购物车里数据。缺点：频繁读取数据库。
* redis，分布式缓存中间件。




























































































