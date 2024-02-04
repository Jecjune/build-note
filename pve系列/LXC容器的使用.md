
安装流程：[[pve系列/远程连接#^bb4dff|LXC的下载安装]]


# LXC容器时间不正确

LXC容器内部默认使用UTC时间（世界时间/美国时间），和我们日常使用的CST（上海时间）不一样，不修改的话可能会出现日志时间不对，部分网站无法访问的情况。
解决方法：

1. 回到PVE后台，修改LXC容器的配置文件
```php
nano /etc/pve/lxc
```
2. 在配置文件中增加以下内容：
```php
lxc.hook.autodev = sh -c "ln -sf /usr/share/zoneinfo/Asia/Shanghai ${LXC_ROOTFS_MOUNT}/etc/localtime"
```
3. 重启容器即可完成时间同步。
