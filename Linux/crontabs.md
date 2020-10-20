
`crontabs`是定时任务插件。

# 安装、使用

```sh
yum install crontabs
```

查看当前定时任务列表：`crontab -l`

<img src="/assets/images/useage/34.png"/>

编辑定时任务： `crontab -e`

删除所有定时任务： `crontab -r`

```sh
service crond start //启动定时任务
service crond stop //关闭定时任务
service crond restart //重启定时任务
service crond reload //重新载入配置
```

报错处理: `Redirecting to /bin/systemctl restart crond.service`

```sh
# 检测cron定时服务是否自启用。 enable表示已启用自启动 disable标识未启用自启动
systemctl is-enabled crond.service
# 如果未启用，则开启cron自启用
systemctl enable crond.service
# 如果已经启用，想要cron关闭自启动
systemctl disable crond.service
# 查看cron服务的启动状态
systemctl status crond.service
# 启动cron服务
systemctl start crond.service
# 停止cron服务
systemctl stop crond.service
# 重启cron服务
systemctl restart crond.service
# 重新加载cron服务
systemctl reload crond.service
```

# 使用

```bash
#每分钟执行 
* * * * *
#每分钟执行 
*/1 * * * *
# 24:00 执行
0 0 * * *
# 23:59 执行
59 23 * * * 
# 1:00 执行
0 1 * * *
```

|分|时|日|月|周|年(可选)|
| --- | --- | --- | --- | --- | --- |
| 0~59 | 0~23 | 1~31 | 1~12 | 1~7 | 2019/2020/2021/... |

```
 * * *  * * 运行的命令
 │ │ │ │  │
 │ │ │ │  └─── 星期几 (0 - 6) (0到6 0代表周日 1周一)
 │ │ │ └──────── 月份 (1 - 12)
 │ │ └───────────── 每月几号 (1 - 31)
 │ └────────────────── 小时 (0 - 23)
 └─────────────────────── 分钟 (0 - 59)
```
