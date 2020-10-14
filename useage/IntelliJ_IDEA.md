# 快捷键

* `Ctrl+Enter`

<img src="/assets/images/useage/11.png">

* `Ctrl+Alt+O`: 删除代码中无用的 `import`。
* `Shift+CMD+O` : 查找文件。

## 展开项目目录

* 点击项目根目录 + 按`*`键：展开所有目录结构

<img src="/assets/images/classOne/04.png">

## 文件分屏展示

<img src="/assets/images/classOne/05.png">

# 插件 

## Lombok

<img src="/assets/images/useage/12.png">

<img src="/assets/images/useage/13.png">

使用

```java
@Data
public class Users {
    private String nickname;
}

// 使用1
{
	Users::getNickname;
}
//使用2
{
	Users users = new Users();
    users.getNickname();
    users.setNickname("A");
}
```