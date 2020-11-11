# 接入

## SpringBoot工程初始化配置

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
    <relativePath/>
</parent>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
</properties>

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
    
    <!-- 数据库连接依赖 -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>3.4.5</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.20</version>
    </dependency>

</dependencies>
```

## MyBatis-Plus 依赖接入

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.0</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```


# SQL 注入

## SQL写在注解中

### Mapper 

```java
public interface UsersMapper extends BaseMapper<Users> {
    @Select("select * from users ${ew.customSqlSegment}") // ew 是固定的，它和下面 Constants.WRAPPER 常量一样。 customSqlSegment 就是值。
    List<Users> selectAll(@Param(Constants.WRAPPER)Wrapper<Users> wrapper);
}
```

### Test

```java
@Test
public void sqlInjector() {
    LambdaQueryWrapper<Users> lambdaQueryWrapper = Wrappers.<Users>lambdaQuery();
    lambdaQueryWrapper.like(Users::getNickname,"雨");
    List<Users> usersList = usersMapper.selectAll(lambdaQueryWrapper);
    System.out.println(usersList);
}
```

## SQL写在XML中

配置资源路径: application.yml

```yml
############################################################
# MyBatis-Plus 配置信息
############################################################
mybatis-plus:
  mapper-locations:
    # - com/gost/mapper/*.xml
    - classpath:mapper/*.xml
```

### Mapper

```java
public interface UsersMapper extends BaseMapper<Users> {
    List<Users> selectAll(@Param(Constants.WRAPPER)Wrapper<Users> wrapper);
}
```

### XML

```xml
<mapper namespace="com.gost.mapper.UsersMapper">
    <select id="selectAll" resultType="com.gost.pojo.Users">
        select * from users ${ew.customSqlSegment}
    </select>
</mapper>
```

整体的结构树：

```
├── auto-generate-api
│   └── src
│       └── main
│           ├── java
│           │   └── com
│           │       └── gost
│           │           ├── Application.java
│           │           ├── configuration
│           │           │   └── MybatisPlusConfig.java
│           │           └── controller
│           │               ├── HelloController.java
│           │               └── UsersController.java
│           └── resources
│               └── application.yml
├── auto-generate-mapper
│   └── src
│       └── main
│           ├── java
│           │   └── com
│           │       └── gost
│           │           └── mapper
│           │               └── UsersMapper.java
│           └── resources
│               └── mapper
│                   └── UsersMapper.xml
└── auto-generate-pojo
    └── src
        └── main
            └── java
                └── com
                    └── gost
                        └── pojo
                            └── Users.java
```

# 查询

## 通过ID更新

```java
T selectById(Serializable id);
List<T> selectBatchIds(@Param("coll") Collection<? extends Serializable> idList);
List<T> selectByMap(@Param("cm") Map<String, Object> columnMap);
```

```java
@Test
public void selectById(){
     Users users = usersMapper.selectById("1908017YR51G1XWH");

    List<String> batchIds = Arrays.asList("1908017YR51G1XWH","190815GTKCBSS7MW");
    List<Users> usersList = usersMapper.selectBatchIds(batchIds);

    Map<String,Object> map = new HashMap<>();
    map.put("username","imooc");// key:数据表的列名
    List<Users> usersList = usersMapper.selectByMap(map);
}
```

## 通过条件构造器更新

创建条件构造器的两种形式：

```java
QueryWrapper<Users> queryWrapper = new QueryWrapper<Users>();
QueryWrapper<Object> queryWrapper = Wrappers<Users>.query();
```

```java
@Test
public void selectByMapper(){
    QueryWrapper<Users> queryWrapper = new QueryWrapper<Users>();

    // 1. 名字中包含雨并且年纪小于40：nickname like '%雨%' and age<40
    queryWrapper.like("nickname","雨").lt("age",40); // lt ：小于
    List<Users> usersList = usersMapper.selectList(queryWrapper);
    System.out.println(usersList);

    // 2.  名字中包含雨并且年纪大于等于20且小于等于40并且邮箱不为空 : nickname like '%雨%' and age between 20 and 40 and email is not null
    queryWrapper.like("nickname","雨").between("age",20,40).isNotNull("email");
    List<Users> usersList = usersMapper.selectList(queryWrapper);
    System.out.println(usersList);

    // 3. 名字为王姓或者年龄大于等于25，按照年龄降序排列，年龄相同则按照ID升序排列： nickname like '王%' or age >= 25 order by age desc,id asc
    queryWrapper.likeRight("nickname","王").or().ge("age",25).orderByDesc("age").orderByAsc("id");
    List<Users> usersList = usersMapper.selectList(queryWrapper);
    System.out.println(usersList);

    // 4. 创建日期为2020年10月10号并且直属上级为名字为王姓 ： date_formate(create_time,'%Y-%m-%d') and manager_id in(select id from user where nickname like '王%')
    queryWrapper.apply("date_formate(create_time,'%Y-%m-%d')={0}","2020-10-10").inSql("manager_id","select id from user where nickname like '王%'");
    List<Users> usersList = usersMapper.selectList(queryWrapper);
    System.out.println(usersList);
}
```

## 分页查询

### 设置分页配置类

```java
package com.gost.configuration;

@Configuration
@MapperScan("com.gost.mapper") // 这么配置是从 Application 中剪切过来的
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MARIADB));
        return mybatisPlusInterceptor;
    }
}
```

测试：

```java
package com.gost;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MPTest {
    @Resource
    private UsersMapper usersMapper;

    @Test
    public void selectByPage(){
         QueryWrapper<Users> queryWrapper = new QueryWrapper<Users>();
         queryWrapper.ge("age",26);
         Page<Users> page = new Page<Users>(1,2);
         IPage<Users> usersIPage = usersMapper.selectPage(page,queryWrapper);
         System.out.println("总页数：" + usersIPage.getPages());
         System.out.println("总纪录数：" + usersIPage.getTotal());
         List<Users> usersList = usersIPage.getRecords();
         System.out.println(usersList);
    }

    @Test
    public void selectByMapsPage(){
         QueryWrapper<Users> queryWrapper = new QueryWrapper<Users>();
         queryWrapper.ge("age",26);
         Page page = new Page<>(1,2);
         // Page page = new Page<>(1,2,false); // false： 不查询总纪录数，默认 true
         IPage<Map<String,Object>> usersIPage = usersMapper.selectMapsPage(page,queryWrapper);
         System.out.println("总页数：" + usersIPage.getPages());
         System.out.println("总纪录数：" + usersIPage.getTotal());
         List<Map<String,Object>> usersList = usersIPage.getRecords();
         System.out.println(usersList);
    }
}
```

### SQL注入进行查询

```java
public interface UsersMapper extends BaseMapper<Users> {
    IPage<Users> selectUsersPage(Page<Users> page,@Param(Constants.WRAPPER)Wrapper<Users> wrapper);
}
```

```xml
<mapper namespace="com.gost.mapper.UsersMapper">
    <select id="selectUsersPage" resultType="com.gost.pojo.Users">
        select * from users ${ew.customSqlSegment}
    </select>
</mapper>
```

```java
@Test
    public void sqlInjector() {
        QueryWrapper<Users> queryWrapper = new QueryWrapper<Users>();
        queryWrapper.ge("age",26);
        Page page = new Page<>(1,2);
        IPage<Users> usersIPage = usersMapper.selectUsersPage(page,queryWrapper);
        System.out.println("总页数：" + usersIPage.getPages());
        System.out.println("总纪录数：" + usersIPage.getTotal());
        List<Users> usersList = usersIPage.getRecords();
        System.out.println(usersList);
    }
```

# 更新(Update)

```java
/* BaseMapper */
int updateById(@Param("et") T entity);
int update(@Param("et") T entity, @Param("ew") Wrapper<T> updateWrapper);
```

## 通过ID更新

```java
@Test
public void updateById() {
	// 用 Entity 去更新
    Users users = new Users();
    users.setId("1908017YR51G1XWH");
    users.setAge(99);
    users.setEmail("aa@qq.com");
    int rows = usersMapper.updateById(users); // 更新的条数
    System.out.println("影响的纪录数：" + rows);
}

```

## 通过条件构造器更新

```java
@Test
public void updateByWrapper() {
	// 用 Entity + UpdateWrapper 去更新
    UpdateWrapper<Users> usersUpdateWrapper = new UpdateWrapper<Users>();
    usersUpdateWrapper.eq("nickname","test").eq("age","1");
    Users users = new Users();
    users.setEmail("aa@qq.com");
    int rows = usersMapper.update(users,usersUpdateWrapper);
    System.out.println("影响的纪录数：" + rows);
}

@Test
public void updateByWrapper2() {
	// 更新单个字段，不想创建Entity可以直接用set去更新
    UpdateWrapper<Users> usersUpdateWrapper = new UpdateWrapper<Users>();
    usersUpdateWrapper.eq("nickname","test").eq("age","1").set("sex","1");
    int rows = usersMapper.update(null,usersUpdateWrapper);
    System.out.println("影响的纪录数：" + rows);
}

@Test
public void updateByWrapperLambda() {
	// Lambda更新表达式
    LambdaUpdateWrapper<Users> lambdaUpdateWrapper = Wrappers.<Users>lambdaUpdate();
    lambdaUpdateWrapper.eq(Users::getNickname,"test").eq(Users::getAge,"1").set(Users::getAge,"30").set(Users::getEmail,"bb@qq.com");
    int rows = usersMapper.update(null,lambdaUpdateWrapper);
    System.out.println("影响的纪录数：" + rows);
}

@Test
public void updateByWrapperLamdaChain() {
	// 链式的Lambda更新表达式
    boolean isUpdate = new LambdaUpdateChainWrapper<Users>(usersMapper).eq(Users::getNickname,"test").eq(Users::getAge,"30").set(Users::getAge,"31").set(Users::getEmail,"cc@qq.com").update();
}
```

# 删除(Delete)

## 通过ID删除

```java
int deleteById(Serializable id);
int deleteByMap(@Param("cm") Map<String, Object> columnMap);
int deleteBatchIds(@Param("coll") Collection<? extends Serializable> idList);
```

```java
@Test
public void delById(){
    usersMapper.deleteById("190815GTKCBSS7MW");

    Map<String,Object> map = new HashMap<>();
    map.put("nickname","abc雨");
    usersMapper.deleteByMap(map);

    List<String> ids = Arrays.asList("1908017YR51G1XWH","1908189H7TNWDTXP");
    int rows = usersMapper.deleteBatchIds(ids);
}
```

## 通过条件构造器删除

```java
int delete(@Param("ew") Wrapper<T> wrapper);
```

```java
@Test
public void delByWrapper(){
    QueryWrapper<Users> queryWrapper = new QueryWrapper<Users>();
    queryWrapper.like("nickname","雨").lt("age",40); // lt ：小于
    int rows = usersMapper.delete(queryWrapper);
}

@Test
public void delByWrapperLambda(){
    LambdaQueryWrapper<Users> lambdaQueryWrapper = Wrappers.<Users>lambdaQuery();
    lambdaQueryWrapper.like(Users::getNickname,"雨");
    int rows = usersMapper.delete(lambdaQueryWrapper);
}
```

# AR模式(Active Record)

# 主键策略

# 基本配置

[MyBatis-Plus 基本配置](https://baomidou.com/config/#%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE)

# 通用Service

```java
@Autowired
private IUsersService usersService;

@Test
public void serviceDemo(){
    // 查
    usersService.getOne(Wrappers.<Users>lambdaQuery().gt(Users::getNickname,"A"),false);
    usersService.lambdaQuery().gt(Users::getAge,25).list();

    // 改
    usersService.lambdaUpdate().gt(Users::getAge,25).set(Users::getAge,30).update();

    // 删
    usersService.lambdaUpdate().gt(Users::getAge,25).remove();

    // 增
    Users users1 = new Users();
    users1.setId("123");
    users1.setUsername("AA");
    users1.setPassword("AA-AA");
    users1.setFace("face");
    users1.setCreatedTime(LocalDateTime.now());
    users1.setUpdatedTime(LocalDateTime.now());


    List<Users> usersList = Arrays.asList(users1);
    usersService.saveBatch(usersList);
}
```


# 逻辑删除

> 逻辑删除 : 就是假删除，纪录没被删除只是通过标示表中字段状态来判断是否是已经删除的记录。

```yml
############################################################
# MyBatis-Plus 配置信息
############################################################
mybatis-plus:
  global-config:
    db-config:
      logic-not-delete-value: 0 # 逻辑未删除字段值
      logic-delete-value: 1 # 逻辑已删除字段值
```

数据库设计：

```sql
CREATE TABLE `users` (
  `id` varchar(64) NOT NULL COMMENT '主键id 用户id',
  `deleted` int DEFAULT '0' COMMENT '逻辑删除标识(0 未删除 1 已删除)',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表 ';
```

Entity模型：

```java
@Data
@EqualsAndHashCode(callSuper = false)
public class Users implements Serializable {
    private static final long serialVersionUID = 1L;
    // 逻辑删除标识(0 未删除 1 已删除)
    @TableLogic
    @TableField(select = false) // 这个字段只是标识纪录删除状态，可以在查询时不查找该字段
    private Integer deleted;
}
```

测试：

```java
@Test
public void tableLogic(){
    int rows = usersMapper.deleteById("123");
    System.out.println("影响的纪录数：" + rows);

    // List<Users> usersList = usersMapper.selectList(null);
    // System.out.println(usersList);
}
```

<img src="/assets/images/useage/14.png"/>

> 如果是用Wrapper来自定义查询，默认也会把`deleted=1`纪录也查询出来的，这个得自己去设置条件。

# 自动填充

> 新增时间、修改时间、新增人、修改人对应的字段可以设置自动填充。

数据库：

```sql
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `id` varchar(64) NOT NULL COMMENT '主键id 用户id',
  `created_time` datetime COMMENT '创建时间 创建时间',
  `updated_time` datetime COMMENT '更新时间 更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表 ';
```

Entity配置：

```java
@Data
@EqualsAndHashCode(callSuper = false)
public class Users implements Serializable {
    private static final long serialVersionUID = 1L;
    private String id;
    /**
     * 创建时间 创建时间
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdTime;

    /**
     * 更新时间 更新时间
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedTime;
}
```

创建填充处理器：

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        if (metaObject.hasSetter("createdTime1")) { // 通过判断 setter 方法，来判断是否存在 createdTime 属性
            System.out.println("createdTime insertFill");
            setFieldValByName("createdTime", LocalDateTime.now(),metaObject);
        }
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        Object object = getFieldValByName("updatedTime",metaObject);
        if (object == null) { // 如果外部没有填充 boject == null， 那就会走默认填充逻辑，如果外部给字段赋值那 object != null 直接使用外部值。
            System.out.println("updatedTime updateFill");
            setFieldValByName("updatedTime",LocalDateTime.now(),metaObject);
        }
    }
}
```

# 乐观锁插件

> 场景：当要更新一条记录的时候，希望这条记录没有被别人更新。

<img src="/assets/images/useage/15.png"/>

> * 乐观锁： 多读场景。
> * 悲观锁： 多写场景。

```sql
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `id` varchar(64) NOT NULL COMMENT '主键id 用户id',
  `username` varchar(32) NOT NULL COMMENT '用户名 用户名',
  `version` int DEFAULT '1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表 ';
```

Entity:

```java
@Data
@EqualsAndHashCode(callSuper = false)
public class Users implements Serializable {
    private static final long serialVersionUID = 1L;
    private String id;
    private String username;
    @Version
    private Integer version;
}
```

乐观锁配置插件：

```java
@Configuration
@MapperScan("com.gost.mapper")
public class MybatisPlusConfig {
    // 乐观锁配置插件
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor(){
        return new OptimisticLockerInterceptor();
    }
}
```

测试:

```java
@Test
public void optTest(){
    int version = 1; // 假设这个版本号是从数据库中查出来的

    Users users = new Users();
    users.setId("12311");
    users.setUsername("BBB");
    users.setVersion(version);

    boolean isUpdated = usersService.updateById(users);
    System.out.println("是否更新成功：" + isUpdated);
}
```

<img src="/assets/images/useage/16.png"/>

# 性能分析插件

**有性能损耗，不建议在生产环境使用。**

> 用于输出SQL输出和执行时间。

[执行 SQL 分析打印](https://baomidou.com/guide/p6spy.html)

## 接入

添加依赖

```xml
<!-- https://mvnrepository.com/artifact/p6spy/p6spy 执行 SQL 分析打印 -->
<dependency>
    <groupId>p6spy</groupId>
    <artifactId>p6spy</artifactId>
    <version>3.8.7</version>
</dependency>
```

修改数据库连接：

```yaml
spring:
  datasource:                                           # 数据源的相关配置
    type: com.zaxxer.hikari.HikariDataSource          # 数据源类型：HikariCP
    # driver-class-name: com.mysql.cj.jdbc.Driver       # mysql驱动
    #url: jdbc:mysql://localhost:3306/foodie-shop?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    url: jdbc:p6spy:mysql://localhost:3306/foodie-shop?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true
```

添加`spy.properties`配置 ：

```properties
#3.2.1以上使用
modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
#3.2.1以下使用或者不配置
#modulelist=com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2
```

<img src="/assets/images/useage/17.png"/>

测试结果:

<img src="/assets/images/useage/18.png"/>


# 多租户SQL解析器

多租户技术是一种软件架构技术。 即面向企业的用户，共用一套系统程序的情况下，保证不用类型用户之间的数据隔离。一般有三种数据隔离方案：

* `独立数据库` : 一个租户一个数据库。隔离级别最高。
* `共享数据库独立Scheme` : 所有租户共享数据库，每个租户都有独立的scheme。
* `共享数据库共享数据表共享Scheme` : 隔离级别最低、安全性最低。

## 共享数据库共享数据表共享Scheme

需要依赖分页插件

# 动态表名SQL解析器
# SQL注入器

# CURD接口

## Service CURD

### 增

```java
// 插入一条记录（选择字段，策略插入）
boolean save(T entity);
// 插入（批量）
boolean saveBatch(Collection<T> entityList);
// 插入（批量）
// batchSize 插入批次数量
boolean saveBatch(Collection<T> entityList, int batchSize);

// TableId 注解存在更新记录，否插入一条记录
boolean saveOrUpdate(T entity);
// 根据updateWrapper尝试更新，否继续执行saveOrUpdate(T)方法
boolean saveOrUpdate(T entity, Wrapper<T> updateWrapper);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
```

### 删

```java
// 根据 entity 条件，删除记录
boolean remove(Wrapper<T> queryWrapper);
// 根据 ID 删除
boolean removeById(Serializable id);
// 根据 columnMap 条件，删除记录
boolean removeByMap(Map<String, Object> columnMap);
// 删除（根据ID 批量删除）
boolean removeByIds(Collection<? extends Serializable> idList);
```

### 改

```java
// 根据 UpdateWrapper 条件，更新记录 需要设置sqlset
boolean update(Wrapper<T> updateWrapper);
// 根据 whereEntity 条件，更新记录
boolean update(T entity, Wrapper<T> updateWrapper);
// 根据 ID 选择修改
boolean updateById(T entity);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList, int batchSize);
```

### 查-单条

```java
// 根据 ID 查询
T getById(Serializable id);
// 根据 Wrapper，查询一条记录。结果集，如果是多个会抛出异常，随机取一条加上限制条件 wrapper.last("LIMIT 1")
T getOne(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录
T getOne(Wrapper<T> queryWrapper, boolean throwEx);
// 根据 Wrapper，查询一条记录
Map<String, Object> getMap(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录
<V> V getObj(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```

### 查-所有

```java
// 查询所有
List<T> list();
// 查询列表
List<T> list(Wrapper<T> queryWrapper);
// 查询（根据ID 批量查询）
Collection<T> listByIds(Collection<? extends Serializable> idList);
// 查询（根据 columnMap 条件）
Collection<T> listByMap(Map<String, Object> columnMap);
// 查询所有列表
List<Map<String, Object>> listMaps();
// 查询列表
List<Map<String, Object>> listMaps(Wrapper<T> queryWrapper);
// 查询全部记录
List<Object> listObjs();
// 查询全部记录
<V> List<V> listObjs(Function<? super Object, V> mapper);
// 根据 Wrapper 条件，查询全部记录
List<Object> listObjs(Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录
<V> List<V> listObjs(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```

### 查-分页

```java
// 无条件分页查询
IPage<T> page(IPage<T> page);
// 条件分页查询
IPage<T> page(IPage<T> page, Wrapper<T> queryWrapper);
// 无条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page);
// 条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page, Wrapper<T> queryWrapper);
```

### 查-统计

```java
// 查询总记录数
int count();
// 根据 Wrapper 条件，查询总记录数
int count(Wrapper<T> queryWrapper);
```

## Mapper CURD

<!-- ### 增
### 删
### 改
### 查 -->

# 条件构造器

## 使用Wrapper自定义SQL

# 执行 SQL 分析打印






















































