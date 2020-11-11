* 在`pom`中添加依赖

```xml
<!-- swagger2 配置 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.4.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.4.0</version>
</dependency>
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>swagger-bootstrap-ui</artifactId>
    <version>1.6</version>
</dependency>
```

* UI: `host:port/swagger-ui.html`
* doc: `host:port/doc.html`

* `@ApiIgnore` 不显示API。

```java
@ApiIgnore
@RestController
public class HelloController {}
```

* `@Api`

```java
@Api(value = "注册登录", tags = {"用于注册登录的相关接口"})
@RestController
@RequestMapping("passport")
public class PassportController {
```

* `@ApiOperation`

```java
@ApiOperation(value = "用户名是否存在", notes = "用户名是否存在", httpMethod = "GET")
@GetMapping("/usernameIsExist")
public IMOOCJSONResult usernameIsExist(@RequestParam String username) {}
```

* `@ApiModel` 模型说明、`@ApiModelProperty` 模型参数说明

```java
@ApiModel(value = "用户对象BO", description = "从客户端，由用户传入的数据封装在此entity中")
public class UserBO {

    @ApiModelProperty(value = "用户名", name = "username", example = "imooc", required = true)
    private String username;
    @ApiModelProperty(value = "密码", name = "password", example = "123456", required = true)
    private String password;
    @ApiModelProperty(value = "确认密码", name = "confirmPassword", example = "123456", required = false)
    private String confirmPassword;
}
```

<img src="/assets/images/useage/35.png"/>

* `ApiParam`

```java
@ApiOperation(value = "获取商品子分类", notes = "获取商品子分类", httpMethod = "GET")
@GetMapping("/subCat/{rootCatId}")
public IMOOCJSONResult subCat(
        @ApiParam(name = "rootCatId", value = "一级分类id", required = true)
        @PathVariable Integer rootCatId) {

    if (rootCatId == null) {
        return IMOOCJSONResult.errorMsg("分类不存在");
    }

    List<CategoryVO> list = categoryService.getSubCatList(rootCatId);
    return IMOOCJSONResult.ok(list);
}
```























































