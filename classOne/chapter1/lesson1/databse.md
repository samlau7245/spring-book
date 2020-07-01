
# 数据库建模工具

## PDMan for Mac

[官方网站](http://www.pdman.cn/#/)

### 设置

选择设置所展示的界面中的字段是`默认的字段`。

<img src="/assets/images/classOne/11.png">

### 数据库连接

* url : `jdbc:mysql://IP地址:端口号/数据库名?characterEncoding=UTF-8&useSSL=false&useUnicode=true&serverTimezone=UTC`
	* `IP地址`: 本地测试写`localhost` 
	* `端口号`: `3306` 
	* `数据库名`: `test` ,数据库字符集推荐选择`utf8mb4`,可支持emoji表情。

<img src="/assets/images/classOne/15.png">

### 关系图

<img src="/assets/images/classOne/12.png">

### 模型

<img src="/assets/images/classOne/13.png">

### 版本模型

> 作用：可把表导出到数据库中，并且产生对应的版本。

<img src="/assets/images/classOne/14.png">

创建基线把设计好的数据表导入到数据库中：

<img src="/assets/images/classOne/16.png">

当字段更新了再次同步到数据库中，有两种升级方式：`重建数据表`、`字段增量`，项目开发中推荐使用后者。

<img src="/assets/images/classOne/17.png">

