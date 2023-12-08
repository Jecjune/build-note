采用pve下lxc容器运行dns服务器，本文转载自：[Docker搭建DNS服务器 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/615495383)

先[[远程连接#^bb4dff|下载lxc]]并安装docker

创建配置文件存储文件：
```bash
mkdir -p /opt/docker/dns-server
```

### **创建容器运行dns服务器**

```bash
docker run --name dns-server -d --restart=always --publish 53:53/tcp --publish 53:53/udp --publish 10000:10000/tcp --volume /opt/docker/dns-server:/data sameersbn/bind:latest
```

### **参数说明**

- `-p 53:53/udp` 绑定容器53端口到宿主机的`53`端口，`DNS`默认端口;
- `-p 10000:10000` 图形化界面管理器端口;
- `-volume /dns-server:/data` 挂载本地目录作为`dns`配置存储


如果建立不成功,查看日志和输出,如果是53端口占用,请看[[解决53端口占用问题]]，然后重新建立即可。

如果报错说已经docker编号已经被占用，使用
```
docker start dns-server
```

### **访问DNS服务器**

- 访问地址 `https://ip:10000`
- 默认账户和密码`root/password`

![[局域网DNS1.png]]

更改语言,记得刷新网页:
![[局域网DNS2.png]]

设定默认值
![[局域网DNS3.png]]

创建新的主区域-正向映射
![[局域网DNS4.png]]

![[局域网DNS5.png]]
域名写要解析的域名，如：dawalker.top
email随便写写就好
主服务器指向自己就可以了，因为只做本地解析
然后为新建的主区域添加ip映射
![[局域网DNS6.png]]
![[局域网DNS7.png]]
名称就是子级域名，地址就是ip地址

### **重启容器**
```bash
docker restart dns-server
```

# 客户端

修改路由器DNS，指向DNS服务器地址，让DHCP用DNS的解析

对于静态ip的电脑，手动设定DNS地址，重启systemd-resolved服务
```bash
systemctl restart systemd-resolved

systemctl enable systemd-resolved
```

接下来可以继续完成[[局域网反向代理]]，实现网址自动转入局域网服务