首先[[局域网下使用putty远程连接群晖（旧）|ssh连接群晖]]

## 安装镜像
```php
sudo docker search alist
```
```php
sudo docker pull xhofe/alist
```

# 新建容器:
```php
sudo docker run -d --name alist -p 5244:5244 -e PUID=99 -e PGID=999 -v /volume2/docker/alist:/opt/alist/data -v /volume1/行者机器人队:/mnt/nas_out  xhofe/alist:latest
```

开放端口并进行访问，初始密码需要从log中看，如果没看到可以进入容器中使用指令重置密码：
```php
./alist admin set <密码>
```

# 登录并设置文件挂载
登录alist后在管理界面进行文件挂载即可。
挂载路径指在alist页面的路径，根文件路径指的是容器中的文件路径（挂载云盘时可以用-11参数指令指示为云盘根目录）。
挂载百度云盘时需要登录令牌，通过alist的文档，在里面找到 存储-百度云盘 页面，即可找到获取令牌的方法。


# 反向代理配置

^d5845a

- 本程序上传全部使用服务器中转
- 若你使用了反向代理，可能需要在配置中指定上传文件最大大小以及超时时间
- 否则可能出现以及上传成功但是前端超时无响应的情况

如果没有设置，在上传大文件的时候，会出现如下情况：
![[alist安装1.png]]

解决方法如下：
应用商店搜索使用的代理工具nginx：
![[alist安装2.png]]
然后在设置下找到最大上传文件大小，设置为想要设置的大小，我设置为10240MB（10G）：
![[alist安装3.png]]



