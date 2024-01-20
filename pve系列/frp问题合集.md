# 1.自动断连问题
当前在docker环境运行下的`0.52.3`版本的frp在经过两星期左右的运行后，会出现频繁断连的情况。

查询版本号方法：
```php
docker exec frpc frpc -v
```

客户端日志如下：
![[39d62400a0a0bdff278c4d7a3ce182c.jpg]]

服务端日志会出现` control writer is closing` 、 `Elegantly exit`等语句。

主要表现就是写服务关闭，服务端退出 -> 客户端写超时反复重连 ->一段时间后，frps服务重启，重新完成连接 -> 写服务关闭。。。

### 解决方法：
实际上我并不清楚什么原因，但是经过一下操作后，能重新稳定：

1. 同时移除客户端和服务端的容器并重新新建：
```php
docker stop frps && docker rm frps
```
```php
docker run --restart=always --network host -d -v /etc/frp/frps.toml:/etc/frp/frps.toml --name frps snowdreamtech/frps
```

2. 在frps中追加配置：
打开配置文件:
```php
nano /etc/frp/frps.toml
```
在末尾加上:
```php
webServer.protocol = "websocket"
```

经过上述操作后,frp映射能重新稳定下来.

然后我选择添加自动脚本，让它自动完成移除重置功能。
服务端：
新建脚本:
```
#!/bin/bash 
docker stop frps 
sleep 1 
docker rm frps 
sleep 1 
docker run --restart=always --network host -d -v /etc/frp/frps.toml:/etc/frp/frps.toml --name frps snowdreamtech/frps
```
设置定时任务:
```php
crontab -e
```
添加:
```
0 0 * * 0 /root/frps_reinstall.sh >/dev/null 2>&1
```

客户端:
新建脚本:
```
#!/bin/bash 
docker stop frpc 
sleep 1 
docker rm frpc
sleep 1 
sudo docker run --restart=always --network host -d -v /etc/frp/frpc.toml:/etc/frp/frpc.toml --name frpc snowdreamtech/frpc
```
然后像frps的流程一样设置定时任务即可.