
# 入门

## 安装

* [mybatis maven 依赖](https://mvnrepository.com/artifact/org.mybatis/mybatis)
* [mapper-spring-boot-starter](https://mvnrepository.com/artifact/tk.mybatis/mapper-spring-boot-starter/2.1.5)

## 整合 mybatis-pagehelper

* 依赖：

```xml
<!--pagehelper -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.12</version>
</dependency>
```

* 参数配置

```
# 分页插件配置
pagehelper:
  helperDialect: mysql
  supportMethodsArguments: true
```

* 使用：

```java
/**
 * page: 第几页
 * pageSize: 每页显示条数
 */
PageHelper.startPage(page, pageSize);

private PagedGridResult setterPagedGrid(List<?> list, Integer page) {
    PageInfo<?> pageList = new PageInfo<>(list);
    PagedGridResult grid = new PagedGridResult();
    grid.setPage(page);
    grid.setRows(list);
    grid.setTotal(pageList.getPages());
    grid.setRecords(pageList.getTotal());
    return grid;
}

public class PagedGridResult {
    private int page;			// 当前页数
    private int total;			// 总页数
    private long records;		// 总记录数
    private List<?> rows;		// 每行显示的内容
}
```
