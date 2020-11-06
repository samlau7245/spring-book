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










