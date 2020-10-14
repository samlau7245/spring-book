# 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```

# 代码测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MPTest {
    @Test
    public void test(){
    }
}
```