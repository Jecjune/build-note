
## 安装镜像
```php
sudo docker search alist
```
```php
sudo docker pull xhofe/alist
```

# 新建容器:
```php
docker run -d --name alist -p 5244:5244 -e PUID=99 -e PGID=999 -v /etc/alist:/opt/alist/data -v /home:/mnt/data  xhofe/alist:latest
```

开放端口并进行访问，初始密码需要从log中看，如果没看到可以进入容器中使用指令重置密码：
```php
./alist admin set <密码>
``` 


# 登录并进行文件挂载

登录alist后在管理界面进行文件挂载即可。
挂载路径指在alist页面的路径，根文件路径指的是容器中的文件路径（挂载云盘时可以用-11参数指令指示为云盘根目录）。
挂载百度云盘时需要登录令牌，通过alist的文档，在里面找到 存储-百度云盘 页面，即可找到获取令牌的方法。